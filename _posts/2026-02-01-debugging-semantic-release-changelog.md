---
title: "Why My CHANGELOG Never Updated: Debugging python-semantic-release"
date: 2026-02-01
author: peng
categories: [DevOps & Computing]
tags: [gitlab-ci, ci-cd, semantic-release, python, changelog, debugging]
math: false
---

My GitLab CI/CD pipeline showed "Job succeeded" for the release stage, but CHANGELOG.md never changed. The version bumped correctly, tags were created, yet the changelog remained empty. No errors and no warings to trace. After hours of investigation, it turned out the issue wasn't what I configured, but what I *didn't*—an undocumented default setting that silently disables file updates unless you know the secret handshake.


---

## The Initial Problem

After setting up `python-semantic-release` for automatic versioning, I expected:
1. Push a `fix:` commit to main
2. Pipeline runs, version bumps (e.g., 2.3.4 → 2.3.5)
3. CHANGELOG.md gets updated with the new release

What actually happened:
- Version bumped ✓
- Tag created ✓
- CHANGELOG.md unchanged ✗

The pipeline logs looked promising:
```
$ semantic-release version
2.3.5
The next version is: 2.3.5! 🚀
No build command specified, skipping
$ semantic-release publish
Job succeeded
```

No errors. No warnings. The changelog simply wasn't touched.

---

## Step 1: Verify the File Was Never Modified

First, I listed all commits that ever touched CHANGELOG.md (this should include automated release commits if the tool is working):

```bash
git log --all --oneline -- CHANGELOG.md
```

**Output:**
```
f4a8e2c ci: implement dev/prod workflow with semantic release
```

Only one commit—my initial manual creation. None of the `chore(release):` commits from semantic-release ever modified it.

---

## Step 2: Check What Release Commits Actually Contain

Next, I picked one of the automated `chore(release):` commits and checked what files it actually modified:

```bash
git show 7b3d91a --stat  # A release commit for v2.3.5
```

**Output:**
```
 pyproject.toml  | 2 +-
 src/__init__.py | 2 +-
 2 files changed, 2 insertions(+), 2 deletions(-)
```

Only `pyproject.toml` and `src/__init__.py`—no CHANGELOG.md. The changelog wasn't being generated or wasn't being committed.

---

## Step 3: Run Verbose Diagnostics Locally

To see what semantic-release was actually doing, I ran it locally with verbose output:

```bash
semantic-release -vv version --no-commit --no-tag --no-push 2>&1 | grep -i changelog
```

**Key output:**
```
INFO  No contents found in PosixPath('.../templates'), using default changelog template
```

It found no custom templates and used the default—but there was no message about *writing* the changelog. The template was loaded but nothing was written to the file.

---

## Step 4: Compare My Config to Defaults

I generated the default configuration to compare (this reveals settings not always documented):

```bash
semantic-release generate-config
```

**Default config showed:**
```toml
[semantic_release.changelog]
changelog_file = ""
mode = "update"
insertion_flag = "<!-- version list -->"
```

> 💡 **Key finding:** The `insertion_flag` setting isn't in the official docs, but appears in the tool's default config output. This was the critical clue—it told me the tool expects a marker in the file.

**My config had:**
```toml
[tool.semantic_release.changelog]
exclude_commit_patterns = [...]

[tool.semantic_release.changelog.default_templates]
changelog_file = "CHANGELOG.md"
```

I had the `changelog_file` set but was missing the `mode` setting. By digging into the `mode` options in the docs and puzzling over the mysterious `insertion_flag`, I finally pieced together what was happening.

---

## The Root Cause

In `python-semantic-release`, there are two changelog modes:

| Mode | Behavior |
|------|----------|
| `update` | Inserts new entries at a specific marker in existing file |
| `init` | Regenerates the entire changelog from git history |

The default is `mode = "update"`, which looks for an **insertion marker** (`<!-- version list -->`) in your CHANGELOG.md. If the marker doesn't exist, **it silently does nothing**.

My CHANGELOG.md looked like this:
```markdown
# Changelog

All notable changes to this project will be documented in this file.
```

No marker. So semantic-release had nowhere to insert new entries and silently skipped the entire operation.

---

## The Solution

Two options:

**Option 1: Add the insertion marker (for `update` mode)**
```markdown
# Changelog

All notable changes to this project will be documented in this file.

<!-- version list -->
```

**Option 2: Use `init` mode (simpler)**
```toml
[tool.semantic_release.changelog]
mode = "init"
```

I chose `mode = "init"` because it regenerates the complete changelog from git history on each release—cleaner and requires no manual setup.

---

## Additional Gotcha: Orphaned Tags

After fixing the config, the pipeline still said "No release will be made, 2.3.5 has already been released!" because previous failed attempts had created tags without changelog updates.

**Fix:** Delete the orphaned tags from remote:
```bash
git push origin --delete v2.3.3 v2.3.4 v2.3.5
```

Then the next release could proceed fresh.

---

## Key Learnings

The fix was one line: `mode = "init"`. Finding it took hours because "Job succeeded" lied—the file never changed. When CI says success but nothing happens, don't trust logs. Check the actual output (`git show COMMIT --stat`), run locally with `-vv`, and most importantly: use local tools like `generate-config` to see the tool's real defaults. The docs didn't mention `insertion_flag`, but the tool's own config output did. That undocumented setting was the key to cracking the mystery.
