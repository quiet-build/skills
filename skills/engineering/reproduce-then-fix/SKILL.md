---
name: reproduce-then-fix
description: Use when fixing a bug, defect, regression, or "this is broken / behaves wrong" report and you want a verified fix — not a guess. Enforces: write a failing test that reproduces the bug first, confirm it fails, find the root cause, fix it, run the FULL suite after each change, and commit only when the repro passes with zero new regressions. Triggers on "fix this bug", "this is broken", "reproduce the bug", "why does X fail", "fix the regression", "make a failing test for this".
---

# Reproduce Then Fix

## Overview

A bug fix is not done when the symptom disappears. It is done when **a test that failed for the exact reported reason now passes, the root cause is understood, and the full suite proves nothing else broke.**

Core principle: **a fix you cannot reproduce is a fix you cannot trust.** Without a failing test first, you have no proof the bug existed, no proof your change addressed it, and no guard against it returning. Symptom-chasing produces "it works on my machine" fixes that mask the real cause.

## The Iron Rule

```
NO FIX BEFORE A FAILING REPRODUCTION TEST
```

Do **not** touch source code until a test reproduces the bug and you have watched it fail. Reproduce → confirm RED → find cause → fix → confirm GREEN → confirm no regressions → commit.

**No exceptions:**
- Not for "it's an obvious one-liner"
- Not for "I can see the cause already"
- Not for "writing a test takes longer than the fix"
- Not "I'll add the test after I confirm the fix works"

An obvious cause makes the test cheap to write, not unnecessary. A test written after a passing fix proves nothing — it was green the moment you wrote it.

## When to Use

- "There's a bug: [symptom]. Fix it."
- "This used to work and now it's broken" (regression)
- "Why does X return the wrong value / throw / hang?"
- Any defect with an observable, describable symptom

**When NOT to use:**

- Pure refactor with no behavior change (use a refactor workflow; rely on existing tests)
- New feature work (use TDD from the spec, not a bug repro)
- A symptom you genuinely cannot reproduce yet — first **invest in reproduction** (logs, inputs, environment); if it's truly non-deterministic, say so and treat flakiness as the bug
- Trivial typo in a string/comment with no logic path (just fix it, but say why no test)

## Workflow

1. Restate the bug as a concrete, observable failure.
2. Write a test that reproduces it exactly. Run it. **Confirm it fails for the reported reason.**
3. Find the root cause — trace from the failure to the source, don't guess.
4. Make the smallest fix that addresses the cause.
5. Run the **full** test suite after each change.
6. Treat any newly-failing test as a regression **you** introduced — fix it before continuing.
7. Commit only when the repro passes and zero other tests regressed.
8. Show before/after test output.

## 1. Restate the bug as an observable failure

Pin the symptom to something a test can assert. Capture, in one or two lines:

- **Input / trigger:** the exact call, request, or steps
- **Expected:** what should happen
- **Actual:** what happens now (wrong value, exception, hang, wrong state)

If you cannot state expected-vs-actual concretely, **stop and ask** — a vague bug produces a vague test that proves nothing.

## 2. Write the reproduction test and watch it fail (RED)

Write the test in the project's existing framework and location — match neighboring tests' style. Assert the **specific** wrong behavior, not something adjacent.

Run **only that test** first and confirm it fails:

```bash
# Detect the project's real runner — do NOT assume npm.
# Check: package.json, Makefile, justfile, pyproject.toml, Cargo.toml, go.mod, CI config.
<runner> <path-to-new-test>     # e.g. pnpm test path/to/foo.test.ts -t "repro: ..."
```

The failure must be **for the reported reason**. A test that fails with an import error, a typo, or the wrong assertion is not a reproduction.

**Capture this RED output** — it is half of the before/after you owe the user.

Red flags that you have NOT reproduced the bug:
- The test fails with a different error than the symptom describes
- The test passes on the first run (you are testing the wrong path)
- You had to change source code to make it fail

If you cannot make it fail, you do not understand the bug yet. Keep investigating; do not start fixing.

## 3. Find the root cause — don't guess

Trace from the failing assertion back to the source of the wrong value or behavior. Read the actual code path. Use logs, a debugger, `git log`/`git blame` on the suspect lines, and bisection if it's a regression:

```bash
git log -p <suspect-file>
git blame -L <start>,<end> <suspect-file>
git bisect start; git bisect bad; git bisect good <known-good-sha>
```

Distinguish **cause** from **symptom**. If you patch where the wrong value is *used* rather than where it is *produced*, the bug will resurface elsewhere. State the root cause in one sentence before fixing.

## 4. Make the smallest fix that addresses the cause

Change only what the root cause requires. Do not refactor adjacent code, rename things, or "improve" nearby logic in the same change — that pollutes the diff and hides whether the fix worked.

## 5. Run the FULL suite after each change

After **every** source change, run the whole suite — not just the repro test:

```bash
<runner>            # e.g. pnpm test / pytest / go test ./... / cargo test
```

The repro test passing in isolation is necessary but not sufficient. A fix that passes its own test while silently breaking three others is a net negative.

If the suite is genuinely too slow to run after every edit, run the **repro test + the affected module's tests** on each iteration and the **full suite before committing** — and say which you did.

## 6. Treat new failures as your regression

If a test that was green before your change is now red, **you caused it.** It is not "pre-existing," not "unrelated," not "flaky" — until proven otherwise.

- Stop. Fix the regression before continuing with anything else.
- Do **not** delete, skip, `xfail`, or weaken the failing test to get green.
- Do **not** add `@ts-ignore`, `# type: ignore`, or commit with `--no-verify`.
- Only after fixing the cause may you re-run and continue.

To claim a failure is genuinely pre-existing, prove it: stash your change (`git stash`) and show the same test fails without it. Otherwise, assume you broke it.

## 7. Commit only when green and regression-free

Commit when **both** hold:

- The reproduction test **passes**.
- The **full** suite shows zero new failures versus the baseline.

The commit should contain the fix **and** the reproduction test together — the test is the permanent guard against this bug returning. Write a message that names the root cause, not just the symptom.

```bash
git add <fix-files> <repro-test>
git commit -m "fix: <root cause> — add regression test for <symptom>"
```

## 8. Show before/after test output

Report to the user:

- The bug restated (input / expected / actual)
- The reproduction test (path + what it asserts)
- **Before:** the RED output from step 2 (test failing for the reported reason)
- The root cause in one sentence
- The fix (files changed, why)
- **After:** the GREEN output — repro test passing **and** full-suite summary showing no regressions
- The commit SHA

Before/after output is not optional decoration — it is the evidence that the fix is real.

## Red Flags — STOP and restart the loop

- Editing source before a test reproduces the bug
- "It's obvious, I'll skip the test"
- "I'll write the test after I confirm the fix"
- Repro test passes on first run, or fails with the wrong error
- Patching where the bad value is *used*, not where it's *produced*
- Running only the repro test, never the full suite
- A newly-red test dismissed as "unrelated" or "pre-existing" without proof
- Deleting / skipping / weakening a test, or using `--no-verify` / `@ts-ignore` to get green
- Committing the fix without the reproduction test
- Reporting "fixed" with no before/after output

**All of these mean: stop, return to the failing-test-first loop.**

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Fixing before reproducing | Write the failing test first; watch it fail for the reported reason |
| Test that passes immediately | You're testing the wrong path — reproduce the actual symptom |
| Test fails with wrong error | Not a reproduction; fix the test until it fails for the bug's reason |
| Guessing at the cause | Trace from the failing assertion to the source; read the code path |
| Patching the symptom site | Fix where the wrong value is produced, not where it's consumed |
| Running only the new test | Run the full suite after every change |
| "That failure is unrelated" | Prove it with `git stash`; otherwise assume you caused it |
| Skipping/weakening a failing test | Fix the root cause; never silence the suite |
| Writing the test after the fix | A test green-on-arrival proves nothing — write it first |
| Committing fix without the test | Commit fix + repro test together as the regression guard |
| Bundling refactors with the fix | Keep the fix minimal; refactor separately |
| Reporting "done" with no output | Show RED-before and GREEN-after, including full-suite result |

## Rationalizations — and the reality

| Excuse | Reality |
|--------|---------|
| "The cause is obvious, no test needed" | Obvious cause → test is cheap. Write it. It also guards against the bug returning. |
| "Writing the test takes longer than the fix" | The test is the deliverable, not overhead. A fix you can't reproduce isn't trustworthy. |
| "I'll add the test after I verify the fix" | A test written against a passing fix is green on arrival — it proves nothing about the bug. |
| "That other failure was already broken" | Prove it by stashing your change. Until then, you caused it. |
| "Skipping the flaky test gets me green" | The flakiness is a bug too. Silencing the suite hides the next regression. |
| "It works when I run it manually" | Manual checks aren't repeatable or CI-visible. The failing-then-passing test is. |
