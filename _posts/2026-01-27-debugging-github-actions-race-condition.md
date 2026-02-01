---
title: "Debugging a GitHub Actions Race Condition: A Deep Dive into CI/CD Diagnostics"
date: 2026-01-27
author: peng
categories: [DevOps & Computing]
tags: [github-actions, ci-cd, devops, git, debugging, workflow, gh-pages]
math: false
---

After merging a PR to the main branch of the [crmPack](https://github.com/openpharma/crmPack) R package repository, I noticed the GitHub Pages deployment workflow failed with a cryptic error: <code style="color: red;">atomic push failed for ref refs/heads/gh-pages</code>. The confusing part? The documentation site was actually updated and working perfectly. This apparent contradiction sent me down a diagnostic rabbit hole that taught me valuable lessons about CI/CD debugging and GitHub Actions architecture.

In this post, I'll share my complete diagnostic journey, the root cause analysis, and—most importantly—the systematic methodology for debugging CI/CD issues that you can apply to your own projects.

---

## The Initial Problem

Here's what I saw after merging commit [`af8506d`](https://github.com/openpharma/crmPack/commit/af8506d637707e979c4f643d74f5840647f511b0) to main:

```
error: atomic push failed for ref refs/heads/gh-pages. status: 5
! [rejected] HEAD -> gh-pages (fetch first)
error: failed to push some refs
hint: Updates were rejected because the remote contains work that you do not have locally
```

At first glance, this looked like a classic merge conflict. But something felt off:

1. **The docs site was working**: The pkgdown documentation at `https://openpharma.github.io/crmPack/main/` reflected my latest changes
2. **The author was wrong**: The error mentioned `danielinteractive` as the author, but I didn't make any push—it was an automated workflow
3. **The timing was suspicious**: Multiple deployment commits appeared within minutes of each other

This didn't match the pattern of a typical merge conflict or human error. Time to investigate.

---

## Step 1: Examining the Deployment History

The first step in any CI/CD diagnostic is to understand what actually happened. I started by examining the `gh-pages` branch history:

```bash
git fetch origin gh-pages
git log origin/gh-pages --oneline -10 --format="%h %s (Author: %an <%ae>) [%ai]"
```

**Output:**
```
9f5ec90c8 deploy: af8506d... (danielinteractive) [2026-01-26 03:31:06 +0000]
223a63ec4 deploy: af8506d... (danielinteractive) [2026-01-26 03:27:44 +0000]
2b7984141 Update pkgdown documentation af8506d... (github-actions[bot]) [2026-01-26 03:27:20 +0000]
```

**Key observations:**
- **Three commits** all referencing my source commit `af8506d`
- **Two different authors**: `github-actions[bot]` and `danielinteractive`
- **Tight timing**: 03:27:20 → 03:27:44 (24 seconds) → 03:31:06 (3 minutes 22 seconds later)

**Hypothesis formed**: Multiple automated processes tried to push to `gh-pages` nearly simultaneously, and the third one failed because the branch had moved forward.

---

## Step 2: Understanding Git's "Atomic Push Failed" Error

Git's "atomic push failed" error occurs when you try to push commits but the remote branch has new commits you don't have locally. This is Git's safety mechanism to prevent overwriting someone else's work.

**The sequence that causes this:**

1. Workflow A fetches `gh-pages` (state: commit X)
2. Workflow B fetches `gh-pages` (state: commit X)
3. Workflow A makes changes and pushes (state: commit Y) ✓ **SUCCESS**
4. Workflow B makes changes and tries to push (state: commit Z)
5. Git rejects Workflow B's push because remote is now at Y, not X ✗ **FAILED**

This is exactly what happened with my deployment workflows.

---

## Step 3: Correlating Source and Deployment Timing

To verify this theory, I checked when my source commit hit the main branch:

```bash
git log origin/main --oneline -3 --format="%h %s (Author: %an) [%ai]"
```

**Output:**
```
af8506d63 789 new helper function... (pengguanya) [2026-01-26 04:17:44 +0100]
3091c88e4 fix issue with last two plots (Daniel) [2026-01-20 21:50:59 +0800]
```

**Timeline reconstruction:**
- My commit merged: `04:17:44 +0100` = `03:17:44 UTC`
- First deployment: `03:27:20 UTC` (10 minutes later)
- Second deployment: `03:27:44 UTC` (24 seconds after first)
- Third deployment: `03:31:06 UTC` (3m 22s after second) → **FAILED**

The 10-minute lag from merge to first deployment is normal—workflows need time to start, install dependencies, and build artifacts. But three deployment attempts for one merge? That's abnormal.

---

## Step 4: Investigating the Workflow Configuration

Now I needed to understand *why* three deployments happened. I examined the workflow files:

```bash
ls .github/workflows/
# Output: check.yaml  docs.yaml  licenses.yaml  links.yaml  rhub.yaml  ...
```

Two workflows caught my attention:

### Workflow 1: `docs.yaml`
```yaml
name: Docs 📚

on:
  push:
    branches:
      - main
    paths:
      - "**.md"
      - "**.Rmd"
      - "man/**"
      # ... other documentation files

jobs:
  docs:
    name: Pkgdown Docs 📚
    uses: insightsengineering/r.pkg.template/.github/workflows/pkgdown.yaml@main
```

**Purpose**: Builds and deploys the pkgdown documentation website to `gh-pages`.

### Workflow 2: `check.yaml`
```yaml
name: Check 🛠

on:
  push:
    branches:
      - main

jobs:
  r-cmd:
    name: R CMD Check 🧬
    uses: insightsengineering/r.pkg.template/.github/workflows/build-check-install.yaml@main

  coverage:
    name: Coverage 📔
    uses: insightsengineering/r.pkg.template/.github/workflows/test-coverage.yaml@main

  # ... other jobs
```

**Purpose**: Runs comprehensive package checks including R CMD check, test coverage, linting, and spell checking.

**The critical finding**: Both workflows:
1. Trigger on `push` to `main` (no coordination between them)
2. Call external **reusable workflows** from `insightsengineering/r.pkg.template`
3. These external workflows likely both push to `gh-pages` (docs and coverage reports)

---

## Step 5: Understanding Reusable Workflows

The `uses: org/repo/.github/workflows/file.yaml@main` syntax invokes **reusable workflows**—external workflow definitions maintained by another team or organization. This is powerful for DRY (Don't Repeat Yourself) principles, but it obscures what's actually happening.

To understand what these workflows do, I could:

```bash
# Clone the template repository
git clone https://github.com/insightsengineering/r.pkg.template
cd r.pkg.template/.github/workflows

# Examine what they deploy
grep -n "git push\|gh-pages" pkgdown.yaml test-coverage.yaml
```

**Likely behavior** (based on common patterns):
- `pkgdown.yaml`: Builds documentation → pushes to `gh-pages/main/`
- `test-coverage.yaml`: Generates coverage report → pushes to `gh-pages/coverage/`

Both pushing to the same branch without coordination = race condition.

---

## Step 6: Checking for Concurrency Controls

Modern GitHub Actions support **concurrency controls** to prevent race conditions:

```yaml
concurrency:
  group: pages-deployment-${{ github.ref }}
  cancel-in-progress: false
```

This tells GitHub: "Only one workflow from this concurrency group can run at a time. Queue others."

I checked if either workflow had this:

```bash
grep -A3 "concurrency:" .github/workflows/*.yaml
```

**Result**: No output. Neither workflow has concurrency controls.

**Root cause confirmed**: Two workflows push to `gh-pages` in parallel without coordination, causing the race condition.

---

**Note on the Author Identity**: You might notice the deployment commits show `danielinteractive` as the author. Daniel is the repository maintainer (we work together), and the reusable workflows are configured to use his identity for automated bot commits—a common practice where the maintainer "signs off" on automated deployments.

---

## The Complete Picture: Race Condition Anatomy

Here's what actually happened:

![Race Condition Flow Diagram](/assets/img/2026-01-27-debugging-github-actions-race-condition/race-condition-diagram.svg)
*Figure: GitHub Actions race condition workflow showing parallel deployments and timing*

**What happened at 03:31:06**:
- The third deployment job fetched `gh-pages` based on an outdated reference
- It tried to push its changes
- Git detected that the remote had moved forward (from deployments #1 and #2)
- Git refused the push to prevent overwriting newer work
- Result: Failed workflow (but docs were already deployed successfully)

---

## The Solution: Four Options

### Option 1: Do Nothing (Recommended for This Case)

**When to use**: If the deployment succeeded (check the live site).

**Why it works**: The error is cosmetic—logged in the third workflow's output, but the first two workflows successfully deployed the content.

**Verification**:
```bash
# Check gh-pages has your commit
git log origin/gh-pages --oneline -3 --grep="af8506d"

# Visit the live site
# https://openpharma.github.io/crmPack/main/
```

**Action**: None required.

### Option 2: Re-run the Failed Workflow

**When to use**: To clear the red X in GitHub Actions UI.

**Steps**:
1. Navigate to `https://github.com/OWNER/REPO/actions`
2. Find the failed workflow run
3. Click "Re-run failed jobs"

**Why it works**: Race conditions are transient—the second attempt will likely succeed.

### Option 3: Trigger an Empty Commit

**When to use**: To force fresh workflow runs without re-running old logs.

```bash
git commit --allow-empty -m "chore: retrigger CI after race condition"
git push origin main
```

**Why it works**: New commit triggers new workflows without the timing conflict.

### Option 4: Manually Dispatch the Workflow

**When to use**: To rebuild docs without making a commit.

```bash
gh workflow run docs.yaml --ref main
```

Or via GitHub UI: Actions tab → Select workflow → "Run workflow" button.

---

## Long-Term Fix: Add Concurrency Controls

The proper solution is to prevent the race condition entirely. The pipeline maintainers should add concurrency controls to both workflows:

**In `.github/workflows/docs.yaml`:**
```yaml
name: Docs 📚

concurrency:
  group: gh-pages-deployment-${{ github.ref }}
  cancel-in-progress: false

on:
  push:
    branches:
      - main
# ... rest of workflow
```

**In `.github/workflows/check.yaml`:**
```yaml
name: Check 🛠

concurrency:
  group: gh-pages-deployment-${{ github.ref }}
  cancel-in-progress: false

on:
  push:
    branches:
      - main
# ... rest of workflow
```

**How it works**:
- Both workflows use the same `group` name
- GitHub queues them instead of running in parallel
- The second workflow waits for the first to complete
- No race condition possible

---

## My Diagnostic Methodology: A Reusable Framework

This investigation taught me a systematic approach to debugging CI/CD issues. Here's the framework I developed:

![CI/CD Diagnostic Framework](/assets/img/2026-01-27-debugging-github-actions-race-condition/diagnostic-framework.svg)
*Figure: Systematic framework for diagnosing CI/CD failures*

### 1. Understand the Error Category

Read the error message carefully and categorize it:
- **Git push errors**: "non-fast-forward", "merge conflict", "atomic push failed"
- **Authentication errors**: "permission denied", "bad credentials"
- **Build errors**: Compilation failures, test failures
- **Timeout errors**: Workflow or job timeouts

Each category suggests different diagnostic paths.

### 2. Build a Timeline

Use `git log` with custom formatting to reconstruct what happened:

```bash
# Timeline with timestamps
git log origin/gh-pages --format="%ai | %h | %s | %an" -10

# Visual timeline with graph
git log --all --graph --format="%C(yellow)%h%C(reset) %C(blue)%ai%C(reset) %s %C(green)(%an)%C(reset)"

# Filter by date range
git log --after="2026-01-26 03:00" --before="2026-01-26 04:00" --all
```

**Look for**:
- Clusters of commits at similar times
- Multiple commits referencing the same source commit
- Unusual author patterns (bots, automation)

### 3. Correlate Source and Deployment

Compare when the source changed vs when deployments happened:

```bash
# When did the trigger commit merge?
git log origin/main -1 --format="%H %ai"

# When did deployments happen?
git log origin/gh-pages --grep="SOURCE_COMMIT_HASH" --format="%ai | %s"

# Calculate the lag
# Expected: 5-15 minutes for workflow startup and build
# Unexpected: Multiple deployments, huge delays
```

### 4. Inspect Workflow Configuration

```bash
# List all workflows
ls -la .github/workflows/

# Check what triggers each workflow
grep -A5 "^on:" .github/workflows/*.yaml

# Identify reusable workflows
grep -r "uses:.*\.yaml" .github/workflows/

# Check for concurrency controls
grep -A2 "concurrency:" .github/workflows/*.yaml

# Find workflows that deploy
grep -l "gh-pages\|deploy\|publish" .github/workflows/*.yaml
```

### 5. Use GitHub CLI for Runtime Information

```bash
# List recent workflow runs
gh run list --limit 10

# Get runs for specific commit
gh run list --commit COMMIT_HASH

# View specific run details
gh run view RUN_ID

# Download logs
gh run download RUN_ID

# Watch a running workflow
gh run watch
```

### 6. Examine External Dependencies

For reusable workflows (`uses: org/repo/.github/workflows/file.yaml`):

```bash
# Clone the external repo
git clone https://github.com/org/repo
cd repo/.github/workflows

# Read the workflow definition
cat file.yaml

# Search for deployment steps
grep -n "git push\|git commit\|actions/deploy" file.yaml
```

### 7. Form and Test Hypothesis

Based on evidence:
1. **State your hypothesis**: "Two workflows push to gh-pages simultaneously without coordination"
2. **Predict behavior**: "Should see multiple commits to gh-pages within seconds of each other"
3. **Verify**: Check git log, workflow logs, timing
4. **Refine**: If evidence doesn't match, revise hypothesis

---

## Debugging Commands Cheat Sheet

Here are the commands I used most frequently during this investigation:

<details>
<summary><strong>Click to expand the command reference</strong></summary>

<div markdown="1">

```bash
# Git History Analysis
git log origin/BRANCH --oneline -N --format="%h %s (%an) [%ai]"
git log --all --grep="PATTERN"
git log --after="DATE" --before="DATE"
git log origin/main..origin/branch  # Commits in branch but not main

# Workflow Investigation
ls .github/workflows/
grep -A5 "^on:" .github/workflows/*.yaml
grep "uses:.*\.yaml" .github/workflows/*.yaml
grep -A2 "concurrency:" .github/workflows/*.yaml

# GitHub CLI
gh run list --limit 10 --commit HASH
gh run view RUN_ID
gh workflow list
gh workflow view WORKFLOW_NAME
gh workflow run WORKFLOW_NAME --ref main

# Timeline Reconstruction
git log --all --graph --format="%C(yellow)%h%C(reset) %ai %s"
git reflog  # Local operations history

# Checking Current State
git fetch --all
git branch -r --contains COMMIT_HASH  # Which branches have this commit
git show-ref  # All refs and their commits
```

</div>

</details>

---

## Conclusion

What started as a confusing "atomic push failed" error turned into a deep dive into GitHub Actions race conditions. The root cause was two workflows pushing to `gh-pages` simultaneously without concurrency controls—a classic race condition that's easily prevented with proper workflow configuration.

The most valuable outcome wasn't just solving this specific issue (which turned out to be cosmetic), but developing a systematic diagnostic methodology: build a timeline with `git log`, correlate source and deployments, inspect workflow configurations, and form testable hypotheses. This framework applies to any CI/CD debugging challenge.

If you're working with GitHub Actions and multiple workflows, I hope this investigation helps you debug more effectively and design more robust pipelines.
