---
title: "From Invisible Error to Reusable Diagnostic Skill: Debugging a VPN Failure with AI"
date: 2026-03-19
author: peng
categories: [AI & ML]
tags: [claude-code, openconnect, vpn, debugging, linux, automation, agent-skills]
math: false
---

My VPN stopped connecting. Not with a helpful error message --- the Alacritty terminal window spawned by my `opencon` command opened for a fraction of a second and closed. No output, no log, nothing to diagnose. Just a terminal flash and silence.

This is the story of how I used an AI coding agent to diagnose the failure, trace it to an undocumented protocol behavior in OpenConnect, implement a fix, and then turn the entire investigation into a reusable diagnostic skill that captures what we learned. The debugging itself took a few hours. But the interesting part is what happened afterward: transforming a one-off fix into a system that accumulates knowledge over time.

---

## The Setup

I run Linux on my workstation and connect to the corporate VPN using [OpenConnect](https://www.infradead.org/openconnect/) with GlobalProtect protocol. The connection requires SAML SSO authentication --- the kind where a browser window opens, you log in through your company's identity provider, and the auth token gets passed back to the VPN client.

The connection is managed by two bash scripts:

- **`opencon`** (`open_con_client.sh`): A wrapper that spawns an Alacritty terminal and runs the core script inside it.
- **`opencon_core`** (`open_con.sh`): The actual connection logic --- launches a Python SAML browser GUI ([`gp-saml-gui`](https://github.com/dlenski/gp-saml-gui)), captures the authentication cookie, and pipes it to `openconnect`.

This had been working for months. Then one day, it didn't.

---

## The Invisible Error

The first challenge was not the VPN error itself --- it was that I couldn't see the error. The Alacritty terminal window opened and closed in under a second. Whatever went wrong was printed to a terminal that no longer existed.

This is a common class of problem with wrapper scripts that spawn terminal windows: if the child process exits (whether success or failure), the terminal closes immediately. There's no "press any key to continue" by default.

**Fix for visibility:** I added a `-d`/`--debug` flag to `open_con_client.sh` that does two things: pipes all output through `tee` to a timestamped log file, and keeps the terminal open with a "Press Enter to close" prompt:

```bash
if [[ "$1" == "-d" || "$1" == "--debug" ]]; then
    LOGFILE="/tmp/opencon_$(date +%Y%m%d_%H%M%S).log"
    $terminal -e bash -c "opencon_core 2>&1 | tee '$LOGFILE'; \
        echo '--- Exit code: '\$?' ---' | tee -a '$LOGFILE'; \
        echo 'Press Enter to close...'; read"
else
    $terminal -e opencon_core
fi
```

Now I could actually read what was happening. The error was:

```
SAML authentication successful. Connecting to VPN...
  Server: gwgp_rmu.example.net
  User: <username>
POST https://gwgp_rmu.example.net/ssl-vpn/login.esp
Got HTTP response: 512
auth-failed-password-empty
```

HTTP 512, `auth-failed-password-empty`. The SAML browser login worked --- I got a cookie. But OpenConnect's gateway authentication rejected it.

---

## The Investigation

### Hypothesis 1: Stdin Pipe Exhaustion

The original script piped the cookie via `echo "$COOKIE" | sudo openconnect --passwd-on-stdin`. My first thought was that stdin was being consumed before OpenConnect read it. I tested with `yes "$COOKIE"` to provide the cookie infinitely:

```bash
yes "$COOKIE" | sudo openconnect --passwd-on-stdin ...
```

Same error. Stdin wasn't the issue --- or at least, not the only issue.

### Hypothesis 2: Cookie Format

Maybe the cookie was malformed or truncated. I printed the raw SAML output from `gp_saml_gui.py` and inspected each field:

```
HOST='https://portalgp.example.net/gateway:prelogin-cookie'
USER='<username>'
COOKIE='<long base64 string>'
OS='linux-64'
```

The cookie looked valid. But something caught my attention: the `HOST` URL contained `portalgp.example.net` --- that's the **portal**, not a **gateway**.

### Hypothesis 3: Portal vs. Gateway Authentication

This was the turning point. I traced what OpenConnect does with a portal cookie by reading its verbose output:

1. OpenConnect authenticates against the **portal** using the SAML cookie --- **succeeds**
2. Portal discovers 12 available gateways and selects one (Basel: `gwgp_rmu.example.net`)
3. OpenConnect tries to authenticate against the **gateway** using the same portal cookie --- **fails**

The portal cookie is a proof-of-identity for the portal. The gateway expects its own cookie. They are not interchangeable.

I found [OpenConnect issue #147](https://gitlab.com/openconnect/openconnect/-/issues/147) which confirmed this: since OpenConnect v8.10, the client reads the prelogin cookie from stdin twice during portal-mediated connections --- once for portal auth, once for gateway auth. A single `echo | --passwd-on-stdin` pipe is consumed on the first read, leaving nothing for the second. But even if you provide the cookie twice, the portal's cookie is fundamentally invalid for the gateway.

### Root Cause

The script was authenticating against the **portal** (`portalgp.example.net`) and then relying on OpenConnect to negotiate with the gateway. This two-phase flow breaks because:

1. Portal cookies are not valid for gateway authentication
2. The stdin pipe is consumed during the first authentication phase
3. The gateway rejects the attempt with `auth-failed-password-empty`

---

## The Fix

The solution was to skip the portal entirely and authenticate directly against the **gateway**:

```bash
# Before (broken): authenticate against portal
saml_output=$(/usr/bin/python3 "$GP_SAML_GUI" "$PORTAL" \
    -c "$CERT_PATH" --key "$KEY_PATH" --clientos Linux)

# After (working): authenticate against gateway directly
saml_output=$(/usr/bin/python3 "$GP_SAML_GUI" "$gateway" --gateway \
    -c "$CERT_PATH" --key "$KEY_PATH" --clientos Linux)
```

The `--gateway` flag tells `gp_saml_gui.py` to run the SAML flow against the gateway endpoint instead of the portal. This produces a cookie that the gateway accepts directly. No portal redirect, no second stdin read, no cookie mismatch.

Since each gateway has its own hostname, I added a mapping array to make the script aware of all available gateways:

```bash
declare -A GATEWAYS=(
    ["Basel"]="gwgp_rmu.example.net"
    ["Mannheim"]="gwgp_mah.example.net"
    ["Tokyo"]="gwgp_rt5.example.net"
    ["Santa_Clara"]="gwgp_sc1.example.net"
    # ... 12 gateways total
)

DEFAULT_AUTHGROUP="Basel"
```

The script resolves the gateway hostname from the authgroup name and passes it to `gp_saml_gui.py`. One stdin read, one authentication phase, working VPN.

---

## What AI Actually Did Here

I want to be specific about the AI's contribution, because "I used AI" is vague and unhelpful.

I worked with Claude Code (an AI coding agent running in my terminal) throughout this debugging session. Here is what it did:

**What AI was good at:**
- **Reading and correlating logs**: It parsed the verbose OpenConnect output and immediately identified the portal-to-gateway redirect sequence. I would have found this too, but it took seconds instead of minutes.
- **Finding the upstream issue**: It searched for `auth-failed-password-empty` in the context of OpenConnect and surfaced the GitLab issue about stdin double-reads.
- **Proposing structured experiments**: When the first fix attempt failed (`--cookie=` flag, which changed the protocol flow), it immediately suggested why and proposed the next approach.
- **Editing scripts safely**: It modified `open_con.sh` and `open_con_client.sh` with proper error handling, preserving the existing structure.

**What AI was NOT good at (where I drove the process):**
- **Recognizing the portal vs. gateway distinction**: The AI initially tried several approaches that worked around the stdin issue without questioning whether the cookie itself was valid for the gateway. I noticed the `portalgp.example.net` in the HOST field and asked "is a portal cookie even valid for gateway auth?" That question changed the direction of the investigation.
- **Testing**: The AI couldn't run `opencon` and see if the VPN actually connected. I was the one who tested each attempt and reported results back.
- **Domain knowledge**: Understanding that corporate GlobalProtect deployments have portals and gateways as separate entities, and that authentication flows differ between them --- that came from experience, not from the AI.

The debugging was collaborative. The AI handled the grunt work (parsing logs, writing code, searching for upstream issues) while I drove the high-level reasoning and testing. This is the pattern I've found most productive: **human hypothesis, AI execution, human verification**.

---

## From One-Off Fix to Reusable Knowledge

Here is where the story could have ended: VPN works, commit the fix, move on. But I noticed something. The debugging session had produced a significant amount of structured knowledge:

- A detailed understanding of how GlobalProtect portal vs. gateway authentication works
- Seven distinct error patterns with their causes and fixes
- A diagnostic workflow that could be followed step-by-step
- Specific commands for checking dependencies, certificates, network, and processes

This knowledge would be useful the next time something breaks. But if it lives only in my memory (or buried in chat history), it's effectively lost. The next failure will require re-discovery.

### Building a Diagnostic Skill

I created an [Agent Skill](https://opencode.ai/docs/skills/) --- a structured Markdown file that encodes the diagnostic workflow and accumulated knowledge. The skill lives at `~/.claude/skills/vpn-diagnose/SKILL.md` and is automatically available to the AI agent in future sessions. The `~/.claude/skills/` path is a shared discovery location for both [OpenCode](https://opencode.ai) and [Claude Code](https://docs.anthropic.com/en/docs/claude-code), so the same skill works with either agent.

The skill defines a five-step diagnostic workflow:

1. **Gather debug logs**: Check for recent `/tmp/opencon_*.log` files, or instruct the user to reproduce with `opencon -d`
2. **Check dependencies**: Verify `openconnect`, `python3`, `WebKit2Gtk`, `gp_saml_gui.py`, HIP report wrapper
3. **Check certificates**: Verify existence, expiry, and key/cert match
4. **Check network**: Test gateway reachability, DNS, existing VPN tunnel
5. **Check processes**: Find stale `openconnect` or `gp_saml_gui` instances

More importantly, it encodes seven known error patterns with their causes and fixes:

```markdown
### `auth-failed-password-empty` (HTTP 512)

**Cause**: Authenticating against the **portal** instead of
directly against a **gateway**. Portal cookies are not valid
for gateway authentication.

**Fix**: Ensure `gp_saml_gui.py` is called with `--gateway`
flag and targets a gateway hostname, NOT the portal.
```

Each pattern includes the error string to match, the root cause explanation, the fix, and verification steps. When a new error is encountered, the skill instructs the AI to update both itself and the documentation with the new pattern.

### The Knowledge Accumulation Loop

This creates a feedback loop:

```
VPN fails → AI loads skill → follows diagnostic workflow
                                      ↓
                              matches known pattern? → apply fix
                                      ↓ (no)
                              investigate → find root cause
                                      ↓
                              fix the issue
                                      ↓
                              update skill with new pattern
                                      ↓
                              update documentation
```

Each failure makes the system smarter. The first time an error occurs, it requires investigation. The second time, the skill already knows the answer. This is not machine learning --- it's structured knowledge capture. But the effect is similar: the system improves with experience.

### Why a Skill and Not Just Documentation?

Documentation is passive. A `CLAUDE.md` troubleshooting section is useful for humans reading it, but the AI agent needs to be told to read it, and then needs to figure out which section is relevant.

A skill is active. When the AI recognizes a VPN-related problem in conversation, it can load the skill, which injects the entire diagnostic workflow and known error patterns into context. The AI then follows the workflow autonomously: checking logs, running dependency checks, matching errors against patterns, and suggesting fixes --- all without requiring the user to direct each step.

The difference is between "here's a reference document" and "here's a procedure to follow." Both have value, but the procedure is what enables autonomous diagnosis.

---

## The Broader Pattern

This experience illustrates a workflow I've started using for all infrastructure failures:

1. **Make errors visible**: If output disappears (closed terminals, truncated logs, swallowed exceptions), fix observability first. You can't debug what you can't see.

2. **Use AI for the grunt work**: Log parsing, upstream issue searching, code editing, structured experiments. Let the AI handle the mechanical parts while you focus on hypothesis formation and testing.

3. **Capture the knowledge**: After fixing the issue, ask: "If this breaks again in six months, what would I need to know?" Write that down --- not just the fix, but the diagnostic path, the error patterns, and the verification steps.

4. **Make the knowledge actionable**: Don't stop at documentation. Encode the diagnostic workflow in a format the AI can use autonomously (an Agent Skill, a runbook, a structured checklist). The next time something breaks, the AI should be able to follow the procedure without human guidance for known issues.

5. **Close the loop**: Include instructions for the AI to update the knowledge base when it encounters new patterns. The system should get better over time, not just at the point of initial creation.

This is not about replacing human judgment. The moment where I noticed `portalgp.example.net` in the HOST field and questioned whether a portal cookie was valid for gateway auth --- that was the breakthrough, and it required domain knowledge that no AI had. But everything before and after that insight (reading logs, searching issues, writing code, structuring the knowledge) was work the AI handled more efficiently than I would have.

---

## Conclusion

What started as a VPN that wouldn't connect became an exercise in three things: debugging an undocumented protocol behavior, collaborating with AI on a systems problem, and building infrastructure that learns from failures.

The fix was changing one flag (`--gateway`) and one target (gateway hostname instead of portal). Finding it required understanding that GlobalProtect portals and gateways have separate authentication domains --- a detail that lives in no official documentation I could find, only in an OpenConnect GitLab issue thread.

But the lasting value isn't the fix. It's the diagnostic skill that now encodes everything this investigation taught me. The next VPN failure won't require re-discovery. The AI will load the skill, follow the workflow, match the error against known patterns, and either fix it or investigate methodically. And if it finds something new, it will update the skill for next time.

The gap between "something broke" and "we know how to fix it" is where institutional knowledge lives. Making that knowledge executable --- not just documented, but something an AI agent can follow autonomously --- is how you stop solving the same problems twice.
