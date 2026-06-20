---
title: "Rootless LLM Fine-Tuning on DGX Spark: GPU Containers Without Root"
date: 2026-06-20
author: peng
categories: [DevOps & Computing]
tags: [dgx-spark, podman, rootless, cdi, aarch64, containers, no-sudo, troubleshooting]
math: false
---

In my previous [DGX Spark posts](/posts/dgx-spark-setup-llm-fine-tuning/) I built and [profiled](/posts/gpu-profiling-llm-fine-tuning-dgx-spark/) an LLM fine-tuning pipeline that ran in a Docker container. This time I went rootless: same Grace Blackwell GB10, but no `sudo`, no writes to `/etc`, and no Docker daemon to lean on.

That constraint is the rule, not the exception. On shared DGX appliances, corporate hosts, and client environments, root is the default blocker. Teams get the GPU but not the privileges, and stall before the first training run. So this post is two things: a field diagnosis of the eight problems standing between an unprivileged user and a working GPU container, and [`pgpu`](https://github.com/pengguanya/pgpu), a small toolkit that collapses the whole fight into `pgpu doctor && pgpu setup` so a team can stand up rootless GPU training on locked-down hardware without waiting on IT.

First the eight problems, then the toolkit that automates them.

---

## Why Podman, and Why Rootless Changes Everything

Docker is a client talking to a root daemon. When you run `docker run --gpus all`, the daemon, running as root, invokes NVIDIA's runtime hook to inject the GPU. The privilege is borrowed from the daemon; you never see it.

Podman is daemonless. Each container is a child of *your* process, in *your* user namespace, with *your* privileges. There's no root daemon to borrow from. That's the whole appeal for a locked-down host, and also the source of every problem below. Every bit of GPU plumbing the Docker daemon used to do silently as root, you now arrange yourself, declaratively, as an unprivileged user.

---

## What Broke (and Why)

### 1. `--gpus all` doesn't exist

Podman has no `--gpus` flag. Rootless GPU access goes through the Container Device Interface (CDI): a vendor-neutral spec that *declares* how to inject a device, listing which `/dev/nvidia*` nodes to map, which driver libraries to mount, and which hooks to run.

```bash
# Docker
docker run --gpus all ... 

# Podman (rootless), via CDI
podman run --device nvidia.com/gpu=all ...
```

But `--device nvidia.com/gpu=all` doesn't work by magic. It's a lookup into a CDI spec file that has to exist first. Which is problem 2.

### 2. There is no CDI spec until you generate one

Out of the box, `--device nvidia.com/gpu=all` fails with `unresolvable CDI devices`. Docker never needed a spec because its root daemon ran the injection hook live. Rootless Podman has no root helper, so you describe the injection up front:

```bash
nvidia-ctk cdi generate --output=/etc/cdi/nvidia.yaml
```

Except `/etc/cdi` needs root to write. Generating the spec itself runs fine as a normal user (it only reads driver and device info), so I sent the output to my home directory instead:

```bash
mkdir -p ~/.config/cdi
nvidia-ctk cdi generate --output=$HOME/.config/cdi/nvidia.yaml
```

That produced a valid spec listing the GB10, `/dev/nvidia*`, and every driver library. Good. Now I just had to make Podman look there. (Problem 5. Hold that thought.)

### 3. NFS `$HOME` can't back the image store

The first `podman` command printed:

```
Network file system detected as backing store.
```

Podman keeps image layers in `~/.local/share/containers`, and its `overlay` driver can't use NFS as the backing store: the kernel won't allow an overlay upper directory on NFS. On a managed host, your home is usually NFS-mounted. The fix is to put the store on local disk. The only local filesystem I could write to was `/tmp`, an ext4 NVMe with 3.4 TB free:

```toml
# ~/.config/containers/storage.conf
[storage]
driver = "overlay"
graphroot = "/tmp/$USER-pgpu/storage"
runroot   = "/tmp/$USER-pgpu/run"
```

Keying the path on `$USER` matters. On a shared box, two people both using `/tmp` for `graphroot` will otherwise clobber each other.

### 4. No subuid ranges, so single-UID mapping

Next warning:

```
no subuid ranges found for user "pengg3" in /etc/subuid
```

Rootless Podman normally maps a *range* of ~65k subordinate UIDs (from `/etc/subuid`) into the container, so files owned by various UIDs inside an image extract cleanly. I had no entry, and adding one needs root. Podman fell back to single mapping, where only my one UID exists inside the container. The catch shows up at pull time: the NGC PyTorch image owns files across several UIDs, and extraction fails with `lchown ... invalid argument` because those UIDs don't exist in my namespace.

The no-root fix is to tell Podman to tolerate the chown failures:

```toml
# ~/.config/containers/storage.conf
[storage.options.overlay]
ignore_chown_errors = "true"
```

Files that would be chowned to a missing UID just stay owned by root-in-namespace, which maps back to me on the host. For a container I run as root anyway, that's harmless. The image extracts, and as a bonus the files it writes come back owned by *me*, not `root`, with none of the `sudo chown` cleanup that rootful Docker forces.

### 5. The system Podman was too old to find my spec

With the spec in `~/.config/cdi` and storage on local disk, `--device nvidia.com/gpu=all` *still* failed: `unresolvable CDI devices`. Debug logging showed Podman parsing the device name but loading zero specs.

The system Podman was 4.9.3. The `cdi_spec_dirs` option in `containers.conf`, the setting that tells Podman to scan extra directories for CDI specs, didn't exist until Podman 5.0. On 4.9.3 it's silently ignored, so Podman only scanned its built-in defaults, `/etc/cdi` and `/run/cdi`, both root-owned and both empty. There was no user-writable directory anywhere in its search path.

That's the trap: I couldn't write `/etc/cdi` (no root), and I couldn't make the old Podman look in my home (no feature). The way out was a newer Podman, still without root. There are statically-linked Podman bundles ([mgoltzsche/podman-static](https://github.com/mgoltzsche/podman-static)) that drop `podman`, `crun`, `conmon`, `netavark`, and `fuse-overlayfs` into a tarball you unpack into `$HOME`:

```bash
curl -fL -o /tmp/podman.tar.gz \
  https://github.com/mgoltzsche/podman-static/releases/latest/download/podman-linux-arm64.tar.gz
mkdir -p ~/opt/podman-static
tar -xzf /tmp/podman.tar.gz -C ~/opt/podman-static --strip-components=1
export PATH="$HOME/opt/podman-static/usr/local/bin:$PATH"
```

Podman 5.8.3, entirely in my home directory, no sudo. It honors `cdi_spec_dirs`:

```toml
# ~/.config/containers/containers.conf
[engine]
cdi_spec_dirs = ["/home/USER/.config/cdi", "/etc/cdi", "/run/cdi"]
```

One subtlety cost me a while: my home was a symlink (`/home/USER` → `/pstore/home/USER`), and Podman scans CDI directories by literal path without resolving symlinks. The `cdi_spec_dirs` entry has to be the *real* path.

### 6. A stale lock from the old Podman

First run of the new Podman:

```
Error: failed to open 2048 locks in /libpod_rootless_lock_93297: numerical result out of range
```

and then, on a system command:

```
failed to reexec: Permission denied
```

I went down an AppArmor rabbit hole here. The host had `kernel.apparmor_restrict_unprivileged_userns = 1` set (modern Ubuntu restricts which binaries can create unprivileged user namespaces), which looked like the obvious culprit for a home-dir binary with no AppArmor profile. It wasn't. `unprivileged_userns_clone` was also `1`, and the namespace creation worked fine.

The real cause was mundane. Podman 4.9.3 and 5.8.3 size their shared-memory lock region differently, and the old version had left a stale lock file in `/dev/shm`. The new Podman tripped over it. Clearing the leftover rootless runtime state fixed it at once:

```bash
rm -f /dev/shm/libpod_rootless_lock_$(id -u)
rm -rf "$XDG_RUNTIME_DIR/libpod" "$XDG_RUNTIME_DIR/containers"
podman system renumber
```

Then `podman run --device nvidia.com/gpu=all ... nvidia-smi` finally printed the GB10. CUDA forward-compatibility quietly kicked in too: a CUDA 13.1 user-space driver layered over the 580.142 kernel driver, so the container's newer CUDA ran against the older host driver.

### 7. Trust the matmul, not the banner

Building the image taught a different lesson. A `pip install` pulled in vLLM, whose hard dependency pin *upgraded* the NGC container's pre-built PyTorch (`2.10.0a0`, the build made specifically for Blackwell `sm_121`) to a stock `2.11.0+cu130` from PyPI. The container's banner still advertised the old version (it's static text), so the only way to know whether the GPU still worked was to make it compute:

```python
import torch
x = torch.randn(1024, 1024, device="cuda")
print((x @ x).sum().item())   # a number, not "no kernel image for device"
```

It returned a number, so the swap was harmless this time. But on a Blackwell chip, "`nvidia-smi` sees the GPU" and "PyTorch can launch a kernel on it" are different claims. Check the second one.

### 8. The host-Python trap, again

My pipeline's `prefetch.py`, which pre-downloads the model on the host to avoid stalled downloads inside the container, failed with `No module named 'datasets'`, even though it's written to self-bootstrap a virtualenv. The bug was in the guard: it asked "am I in a virtualenv?" (`sys.prefix != sys.base_prefix`) and returned early if so. Run from inside a project `.venv` that lacked the dependency, it skipped the bootstrap entirely.

The fix is to ask the right question, "are the packages importable?", instead of "am I in a venv?":

```python
def ensure_venv():
    try:
        import datasets, transformers, huggingface_hub  # noqa
        return
    except ImportError:
        pass
    # ... create the temp venv, install deps, os.execv into it
```

It's the same theme as the [first post](/posts/dgx-spark-setup-llm-fine-tuning/)'s PEP 668 section. On aarch64 DGX hosts the host Python is a minefield: no CUDA wheels, `uv`'s TLS rejects NVIDIA's cert chain, Debian's PEP 668 blocks `pip install`. A host-side tool has to probe for what it needs and bootstrap its own isolated environment rather than trust whatever interpreter it happens to run under.

---

## Automating All Eight: `pgpu`

By the time `nvidia-smi` printed the GB10, I'd touched `nvidia-ctk`, two config files, a home-dir Podman install, `/dev/shm`, and a Python bug. Nobody on a collaborator's box should have to rediscover that sequence, so I packaged it into a small toolkit: [`pgpu`](https://github.com/pengguanya/pgpu).

It's deliberately bash-only, with no Python and nothing beyond `bash` and `podman`, for the reason this whole post keeps circling back to: infrastructure that *sets up* the environment can't itself depend on a fragile host Python. It probes the host and resolves a tier:

| Tier | When | What it does |
|---|---|---|
| 0, native | new system Podman, sudo or writable `/etc/cdi`, subuid present | minimal config, CDI spec in `/etc/cdi` |
| 1, user-CDI | Podman ≥ 5, no sudo | CDI spec in `~/.config/cdi`, storage redirect if `$HOME` is NFS |
| 2, static | Podman missing or too old | fetch static Podman 5.x into `$HOME`, then behave like tier 1 |

On top of the tier it layers the cross-cutting fixes: `ignore_chown_errors` when there's no subuid, a user-scoped overlay store on local disk when `$HOME` is NFS, and a `clean` command for the stale-lock problem. The whole bring-up becomes:

```bash
pgpu doctor    # probe the host, print the resolved tier and planned actions
pgpu setup     # apply it: CDI spec, storage.conf, containers.conf, static podman if needed
pgpu build     # build the project image
pgpu run -- nvidia-smi
```

Any training or inference image plugs in through a per-project `pgpu.conf`; the toolkit hard-codes nothing about my model. What took me most of a day to work out, a collaborator now runs in two commands.

---

## Rootless Is a Different Trust Model, Not a Smaller One

The first post in this series ended with "DGX Spark is not a smaller cluster, it's a different environment." Rootless containers are the same kind of shift. This isn't Docker minus a flag; it's a trust model you build up from yourself instead of borrowing down from a daemon.

**The privilege you don't have is the design constraint.** Docker injects the GPU, manages cgroups, and owns the image store as root. Rootless Podman does none of that for you. Every one of the eight problems is the same problem in a different place: something the root daemon used to do silently, now arranged declaratively by an unprivileged user. CDI handles the GPU, `ignore_chown_errors` the filesystem, `cdi_spec_dirs` the discovery.

**The constraints stack, and order matters.** No single fix unblocked me. The NFS store had to move, chown errors had to be tolerated, the spec had to be discoverable, and Podman had to be new enough, all at once. Miss one and the others still look broken. Writing them down (and then automating them) is what turns the maze back into a checklist.

**"No sudo" is a requirement, not a personal handicap.** The reason to package this isn't that my setup was special; it's that *most* enterprise compute looks like this. A toolkit that turns root-restricted GPU access into `pgpu doctor && pgpu setup` is the difference between a team scaling onto shared hardware and a team filing IT tickets. The DGX Spark made me solve it once. The point was to make sure nobody on the next box has to.
