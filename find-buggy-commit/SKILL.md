---
name: find-buggy-commit
description: Find the exact commit that introduced a bug, regression, or behavioral change using binary search. Supports autonomous mode (running tests automatically) and interactive mode (guided step-by-step). Use when the user wants to find when something broke, track down a regression, locate a bad commit, or figure out which commit caused a problem.
---

This skill helps the user find the exact commit that introduced a problem using `git bisect`. It operates in two modes: **autonomous** (proactively testing commits) and **interactive** (asking the user to verify each commit).

## Workflow

### Step 1: Understand the Problem

Ask the user to describe:

1. **What broke or changed** — the symptom they are observing
2. **A known bad commit** — where the problem exists (default: `HEAD`)
3. **A known good commit** — where the problem did not exist (a tag, branch, SHA, or relative ref like `HEAD~20`)
4. **Custom terminology** — if this is not a simple bug search, ask whether alternative terms are more appropriate:
   - `old`/`new` for changes that are not strictly bugs (e.g., "when did this behavior start?")
   - Custom terms for specific domains (e.g., `fast`/`slow` for performance regressions)

### Step 2: Pre-flight Checks

Before starting bisect:

1. **Dirty working tree** — run `git status --short`. If there are uncommitted changes, warn the user and suggest stashing (`git stash`) before proceeding. Do not start bisect with a dirty tree.
2. **Validate refs** — confirm both the good and bad commits exist and that the good commit is an ancestor of the bad commit.
3. **Estimate scope** — run `git rev-list --count <good>..<bad>` to show how many commits are in the range and approximately how many steps bisect will need (log2 of the count).

### Step 3: Choose Mode

Ask the user which mode to use:

#### Option A — Autonomous Mode

Ask the user: *"Is there a command I can run to automatically test whether each commit is good or bad?"*

Good candidates for autonomous testing:
- A specific test command: `npm test`, `pytest tests/specific_test.py`, `cargo test test_name`
- A build command: `make`, `npm run build`
- A custom script or one-liner that exits 0 for good and non-zero for bad
- A grep or check: `sh -c "node script.js | grep 'expected output'"`

If the user provides a test command:
1. Validate the command works on the current commit first — run it and check exit code
2. If it fails on the bad commit and succeeds on the good commit, proceed
3. If validation is inconclusive, fall back to interactive mode

If the user cannot provide a test command but wants autonomous mode, try to infer a test strategy:
- Look at the project's test runner and suggest a specific test related to the symptom
- Suggest a build-only check if the issue is a build break
- If no automated strategy is viable, explain why and fall back to interactive mode

#### Option B — Interactive Mode

The user will manually test each commit. Proceed directly to Step 4.

### Step 4: Run Bisect

#### Starting the Session

```bash
# Standard terminology
git bisect start
git bisect bad <bad-ref>
git bisect good <good-ref>

# Custom terminology
git bisect start --term-new=<term-new> --term-old=<term-old>
git bisect <term-new> <bad-ref>
git bisect <term-old> <good-ref>
```

You may also limit bisect to specific paths if the user identified the affected area:
```bash
git bisect start -- <path1> <path2>
```

#### Autonomous Execution

Run `git bisect run <command>` with the validated test command. Monitor the output for:
- **Exit 0**: commit is good
- **Exit 1-124, 126-127**: commit is bad
- **Exit 125**: commit cannot be tested (bisect will skip it)
- **Negative exit codes or signals**: bisect aborts — intervene and diagnose

If `git bisect run` completes successfully, it will output the first bad commit. Jump to Step 5.

If `git bisect run` fails or produces ambiguous results, switch to interactive mode for the remaining steps.

#### Interactive Execution

For each commit that git checks out:

1. Show the user the current commit info:
   ```bash
   git log -1 --format="%H %s (%an, %ar)"
   ```
2. Show the diff summary to help orient them:
   ```bash
   git diff --stat HEAD~1..HEAD
   ```
3. Tell the user what to test and ask them to report the result
4. Based on their answer, mark the commit:
   ```bash
   git bisect good   # or the custom old term
   git bisect bad    # or the custom new term
   git bisect skip   # if the commit cannot be tested
   ```
5. Repeat until bisect identifies the first bad commit

**When to use `git bisect skip`:**
- The commit does not compile or is otherwise untestable
- The commit is unrelated to the area being investigated
- The user is unsure and wants to move on

Warn the user that skipping commits near the culprit may result in a range of suspects instead of a single commit.

### Step 5: Present Results

Once bisect completes:

1. Show the identified commit in detail:
   ```bash
   git log -1 --format="full" <culprit-sha>
   git diff --stat <culprit-sha>~1..<culprit-sha>
   ```
2. Show the actual diff of the commit to help the user understand what changed:
   ```bash
   git show <culprit-sha>
   ```
3. If bisect returned a range (due to skips), explain which commits are suspects and suggest the user investigate each one
4. Provide a brief analysis: what the commit changed, who authored it, and any obvious connection to the reported symptom

### Step 6: Save Log and Reset

Before resetting, save the bisect log for future reference:

```bash
mkdir -p .bisect-logs
git bisect log > .bisect-logs/<YYYY-MM-DD>-<slug>.log
```

Where `<slug>` is a short lowercase-hyphenated description of the issue being investigated.

Then clean up:

```bash
git bisect reset
```

This returns the working tree to the original branch/commit. If the user wants to checkout the bad commit for further investigation, use:

```bash
git bisect reset bisect/bad
```

Inform the user that the log was saved and can be replayed later with `git bisect replay .bisect-logs/<filename>.log`.

## Guardrails

- **Never start bisect with uncommitted changes** — always check for a clean working tree first
- **Always reset at the end** — a forgotten bisect session leaves the repo in a detached HEAD state
- **Always save the log before resetting** — this is the only record of the bisect process
- **Prefer autonomous mode when possible** — it is faster and less error-prone, but only if a reliable test command exists
- **Do not guess good/bad** — if the test result is ambiguous, use `skip` or ask the user
- **Handle `git bisect run` failures gracefully** — if the automated run aborts, capture the state and continue interactively
- **Respect the user's choice of mode** — if they prefer interactive, do not push autonomous mode
