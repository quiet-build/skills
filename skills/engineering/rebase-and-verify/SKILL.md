---
name: rebase-and-verify
description: Use when rebasing the current branch onto another branch (main/develop/etc.) and needing conflicts resolved without breaking lint, type checks, unit tests, or e2e tests — then independently reviewed. Pass --simple (alias --fast) for a quick low-risk rebase that runs lint/eslint only and skips type checks, unit/e2e tests, and the independent review. Triggers on "rebase onto main", "rebase to latest", "rebase and resolve conflicts", "catch up with main", "sync my feature branch with main", "my branch is behind main", "main moved, update my PR branch".
---

# Rebase and Verify

## Overview

Rebasing onto a moving target branch is not done when the rebase command exits cleanly. It is done when **conflicts are resolved by intent, every quality gate still passes, and an independent reviewer confirms the branch's own functionality survived.**

Core principle: **a green `git rebase` says the patches applied — it says nothing about whether the code still works.** Conflict markers can vanish while the merged result is logically wrong. Treat the rebase as untrusted until gates pass and a reviewer signs off.

## Modes

| Mode | Flag | Gates | Independent review | Use when |
|------|------|-------|--------------------|----------|
| **Full** (default) | _(none)_ | Lint + types + unit + e2e | Yes | Default. Any non-trivial rebase, or conflicts that touched source logic. |
| **Simple** | `--simple` (alias `--fast`) | **Lint/eslint only** | Skipped | Low-risk rebase you want quickly — few/no conflicts, or conflicts limited to imports/lockfiles/formatting. |

`--simple` keeps every **safety** step (pre-flight, backup branch, shared-branch check, conflict resolution by intent) — those are cheap and never skipped. It only drops the slow verification: type checks, unit tests, e2e tests, and the independent review. The lint/eslint gate stays because it is fast and catches the most common post-rebase breakage.

**Simple mode is a deliberate trade of safety for speed.** It does NOT prove the branch's own features still work. Two rules:

- **Loudly disclose** in the final report that you ran in simple mode and list every gate that was skipped.
- If conflict resolution touched **real source logic** (not just imports, lockfiles, or formatting), **stop and recommend full mode** — that is exactly the case where skipping tests and review is dangerous. Proceed in simple mode only if the user confirms.

## When to Use

- "Rebase this branch onto the latest main / develop / `<target>`"
- "Catch this branch up with main and resolve conflicts"
- "My branch is behind main / main moved — update my PR branch"
- Long-lived feature branch that has drifted behind its base

**When NOT to use:**

- Trivial fast-forward with no divergence (just `git pull --rebase`)
- Merge-commit workflows where the team forbids rebasing shared history
- Branches already pushed/shared where rewriting history may disrupt collaborators, unless the user explicitly confirms

## Default Target

The target branch defaults to **`main`** only when it exists and the base remote is unambiguous. Prefer the actual PR/base remote when available. Otherwise:

1. Use the branch's configured upstream/default remote if clear.
2. Use `origin/<target>` only when `origin` is the obvious base remote.
3. If multiple remotes or possible base branches exist, stop and ask.

Detection commands:

```bash
git remote -v
git remote show origin | sed -n 's/.*HEAD branch: //p'   # remote default branch
git branch -vv                                            # local branches + their upstreams
```

## Workflow

1. Validate repo state and create recovery points.
2. Fetch the selected base remote and target.
3. Check whether rebasing would rewrite a pushed/shared branch.
4. Rebase onto the exact fetched target SHA.
5. Resolve conflicts by intent, one file at a time.
6. Run lint, type checks, unit tests, and e2e tests where available.
7. Fix real regressions and re-run gates.
8. Get an independent review.
9. Report recovery points, conflict resolutions, gates, and review verdict.

## 1. Pre-flight — validate state, then back up before history changes

First, confirm it is safe to operate. These checks do not move history:

```bash
git rev-parse --is-inside-work-tree      # are we in a repo at all?
git rev-parse --abbrev-ref HEAD          # prints "HEAD" if detached
git status --short                        # dirty tree?
```

Check whether Git is already mid-operation (worktree-safe — `.git` may be a file, not a dir):

```bash
test -e "$(git rev-parse --git-path rebase-merge)" \
  || test -e "$(git rev-parse --git-path rebase-apply)" \
  || test -e "$(git rev-parse --git-path MERGE_HEAD)" \
  && echo "A rebase/merge is already in progress — STOP"
```

If already mid-rebase, mid-merge, detached HEAD, or outside a Git repo, **stop and explain** the problem.

Then create recovery points before any fetch/rebase/checkout/edit:

```bash
CURRENT=$(git rev-parse --abbrev-ref HEAD)
PRE_REBASE_SHA=$(git rev-parse HEAD)
BACKUP_BRANCH="backup/${CURRENT}-$(date +%Y%m%d-%H%M%S)"
git branch "$BACKUP_BRANCH" "$PRE_REBASE_SHA"
```

**Report the backup branch and pre-rebase SHA immediately.**

A backup branch protects **committed** work only. If the working tree is dirty, do not rebase yet — commit or stash first, and report the stash ref if you stash:

```bash
git stash push -u -m "pre-rebase-${CURRENT}-$(date +%Y%m%d-%H%M%S)"
```

Never rebase with a dirty tree.

## 2. Fetch and identify the exact target

```bash
git fetch <remote> <target>
TARGET_SHA=$(git rev-parse <remote>/<target>)
```

Use `<remote>/<target>`, not local `<target>` — the local branch may be stale. Keep `TARGET_SHA`; use this exact SHA for the rebase, the report, and the review diff.

Optional `rerere`:

```bash
git config rerere.enabled true
```

This persists in the repo's local config and lets Git remember conflict resolutions across re-runs. **Mention the side effect.** If you do not want a persistent repo-config mutation, skip it.

## 3. Shared-branch check

Rebasing rewrites commits. If the branch has an upstream, inspect divergence before continuing:

```bash
git rev-parse --abbrev-ref --symbolic-full-name @{upstream} 2>/dev/null
git rev-list --left-right --count @{upstream}...HEAD 2>/dev/null
```

Interpretation:

- **No upstream:** usually safe to proceed.
- **Upstream exists, branch is known solo:** proceed.
- **Upstream exists and others may depend on it:** stop and confirm with the user.
- **Both sides have commits:** stop and explain that local and remote have diverged.

Never force-push later unless the user asks. If pushing rewritten history, use only `git push --force-with-lease` — never `git push --force`.

## 4. Rebase onto the exact target SHA

```bash
git rebase "$TARGET_SHA"
```

Rebasing onto the exact SHA avoids silently changing base if the remote moves again during the task.

## 5. Resolve conflicts by intent

Conflict rules:

- Understand **both** sides before editing. Use `git log --merge -p <file>` and nearby history to learn why each side changed.
- Never blindly use whole-file `--ours` or `--theirs`, except for genuinely regenerated artifacts.
- **Preserve both intents.** If the target refactored an API and the feature branch added behavior, wire the feature to the new API.
- If two valid interpretations exist with different behavior, **stop and ask the user.**

After each resolved conflict:

```bash
git add <files>
git -c core.editor=true rebase --continue   # core.editor=true (or GIT_EDITOR=true) avoids hanging on an editor
```

For lockfiles:

- Merge dependency **manifests** by intent first.
- Then **regenerate** the lockfile with the repo's package manager.
- Do not hand-merge lockfiles unless the repo has no practical regeneration path.

```bash
pnpm install        # or: npm install / yarn install
cargo generate-lockfile
poetry lock
```

Use the repo's existing package manager and lockfile format.

## 6. Run quality gates

> **Simple mode (`--simple` / `--fast`):** run the **Lint** row only, then skip the rest of this table and step 7 entirely. Still detect the project's real lint command (do not assume `npm run lint`). If no lint command exists at all, say so — there is no fast gate to run, and you should recommend full mode.

Detect the project's actual commands — **do not assume npm.** Check: `package.json`, `Makefile`, `justfile`, `pyproject.toml`, `Cargo.toml`, `go.mod`, `.github/workflows`, existing CI scripts.

Run whatever exists, in this order:

| Gate | Typical command (verify per repo) |
|------|-----------------------------------|
| Lint | `npm run lint` / `ruff check` / `golangci-lint run` |
| Types | `npm run typecheck` / `tsc --noEmit` / `mypy` / `cargo check` |
| Unit tests | `npm test` / `pytest` / `go test ./...` / `cargo test` |
| E2E tests | `npm run test:e2e` / `playwright test` / `cypress run` |

For monorepos, prefer affected-scoped gates when available, falling back to root scripts only when those are unavailable or inappropriate:

```bash
turbo run lint test --filter=...[<remote>/<target>]
nx affected -t lint test
```

If e2e is too heavy to run locally, say so explicitly and run the smallest meaningful subset.

A gate failure after rebase is a **real signal** — fix the root cause. Do NOT:

- Delete or skip a failing test
- Add `@ts-ignore` or equivalent
- Disable lint rules
- Use `--no-verify`
- Loosen assertions just to pass

If a failure looks unrelated to the rebase, **verify before calling it pre-existing**: check target-branch CI if available, or temporarily test the exact `TARGET_SHA` if practical.

## 7. Independent review

> **Simple mode (`--simple` / `--fast`):** skip this step. Note the skipped review in the final report.

Get a fresh review from a context that did **not** perform the merge. Use whichever mechanism is available:

- A fresh subagent / reviewer agent — **on Claude Code, default to Opus 4.8 with medium reasoning**
- A separate clean review pass
- Human/user review if no independent agent is available

Give the reviewer the diff against the exact base:

```bash
git diff "$TARGET_SHA"...HEAD
```

Also provide: files that had conflicts, how each conflict was resolved, and the gate commands + results.

Reviewer charge:

> Verify this branch's own features still work after rebasing onto `<target>`. Focus on conflict-resolution sites: did the merge drop logic, wire a feature to a stale API, or create a plausible-but-wrong combination of both sides? Confirm gates pass. Report any functional regression with file:line.

Act on findings:

- **Real regression:** fix it, re-run gates, and re-review if non-trivial.
- **Speculative concern:** report it as a question, not as completion-blocking fact.
- **No findings:** include the clean verdict in the final report.

## 8. Report

Tell the user:

- Current branch
- Backup branch and pre-rebase SHA
- Target branch and target SHA
- Whether the branch had an upstream / shared-branch risk
- Files that conflicted, and how each was resolved
- Quality gates run, with exact commands and results
- Independent review verdict
- Whether any gates were skipped, and why
- **If simple mode:** state `Mode: simple` explicitly and list every skipped gate (type checks, unit tests, e2e tests, independent review) so the user knows verification was partial

Do not force-push unless the user explicitly asks. If pushing rewritten history, use only:

```bash
git push --force-with-lease
```

## Red Flags — STOP

- Not inside a Git repo
- Detached HEAD
- Existing rebase/merge/cherry-pick in progress
- Dirty working tree
- Fetch target is ambiguous, or multiple plausible base remotes with no clear PR/base context
- Pushed/shared branch without user confirmation
- Local and upstream have diverged unexpectedly
- Whole-file side-pick on source conflicts
- Hand-merging lockfiles instead of regenerating
- Skipping/deleting tests to get green, or silencing lint/type/test failures
- Self-review only when an independent reviewer is available
- Force-pushing without explicit user request

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Rebasing before recovery points | Validate repo state, then create the backup branch before history changes |
| Assuming `origin/main` is the base | Detect PR/base remote or ask when ambiguous |
| Backing up dirty work | Backup branch protects commits only; stash or commit uncommitted changes |
| Rebasing a pushed branch without checking divergence | Use upstream + `rev-list --left-right --count` |
| Rebasing onto stale local branch | Fetch and rebase onto the exact fetched target SHA |
| `rebase --continue` hanging | Use `git -c core.editor=true rebase --continue` |
| Whole-file side-pick | Resolve by intent and preserve both sides |
| Hand-merging lockfiles | Merge manifests, then regenerate the lockfile |
| Root-level gates in a monorepo | Prefer affected-scoped commands |
| Running only unit tests | Run lint, types, unit, and e2e where available |
| Silencing a failing gate | Fix the root cause |
| Reviewing with merge-biased context only | Use fresh reviewer context where available |
| `git push --force` | Use `git push --force-with-lease`, only when requested |

## Recovery

Mid-rebase, abort:

```bash
git rebase --abort
```

Finished but wrong, not pushed:

```bash
git reset --hard ORIG_HEAD
# or reset to the explicit backup:
git reset --hard backup/<branch>-<timestamp>
```

Reflog is the final recovery path:

```bash
git reflog
git reset --hard <pre-rebase-sha>
```
