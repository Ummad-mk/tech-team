---
name: git-ops
description: Git discipline that agents follow universally — conflict-resolution protocol, safe rebase/recovery rules, checkpoint-before-destructive, commit hygiene, and forbidden commands. Use before any git operation (commits, merges, rebase, reset, checkout, push, pull), when resolving merge conflicts, when recovering from a broken working tree or failed operation, when creating branches or tags, or before any history-modifying operation.
---

# Git Ops — the discipline every git command follows

One bad `git reset --hard` can destroy a day of agent work across sessions. History-modifying power is treated like a live weapon: snapshot first, recoverable always.

## Situational awareness — before ANY git action
- Read state first, always: `git status --short`, `git log --oneline -3`, current branch. If uncommitted changes exist, decide their fate BEFORE staging/merging — never let a merge or branch switch silently carry or clobber them.
- `git fetch` before assuming anything about remote state — never reason from a stale memory of the remote.

## Forbidden without explicit, verifiable clearance
- `git reset --hard`, `git clean -f/-fd`, `git checkout -- <file>` — all destroy uncommitted work silently. Before any of them: confirm nothing valuable is lost (diff it, or checkpoint below).
- `git push --force` — on any branch. If force is genuinely required, `--force-with-lease` and a stated reason; on shared branches it's never required, it's a proven mistake.
- Amend/rebase/rewrite of commits already pushed — shared history is a contract; fix forward with a follow-up commit or `git revert` instead.

## Checkpoint discipline (the recovery mindset)
- Before ANY risky operation (rebase, reset, branch surgery), snapshot: commit WIP to the branch or `git stash push -m "<what+when>"`, or create a throwaway branch. Reflog makes everything recoverable — but only work that was committed/exists.
- A rebase gone wrong mid-flight: `git rebase --abort`, not improvisation. Never abandon a repo mid-operation without recording its state in the report.
- Recovery order when things look broken: `git reflog` → identify the last known-good state → restore (branch/reset to it); reflog entries survive ~90 days, so "lost" commits usually aren't.

## Conflicts — semantic, not textual
- Never take all "ours" or all "theirs" bulk. Resolve per-conflict: read BOTH candidates and the merge ancestor (`git merge-base`), understand what each side changed, then synthesize the correct combined behavior.
- A conflict whose two sides touch the same logic = a real decision: re-run the affected behavior (test/build) before committing the merge; if intent is unclear, escalate to the boss/user — a wrong "resolved" conflict is a shipwreck discovered weeks later.
- Repeated conflicts on the same files: resolve once right, then propose fixing the root cause (branch hygiene, file layout, better boundaries) — and log the pattern via self-improving-agent.

## Commits and branches
- Commits are atomic and honest: one logical change per commit; imperative subject ≤ 72 chars that a stranger can recon; bodies carry the *why*, not a transcript of the diff; no "fix fix wip 2" noise in real history (WIP belongs on a temp branch or clearly labeled).
- Never commit secrets, credentials, `.env` values, or generated/artifact directories on impulse — verify `.gitignore` covers `node_modules/`, `dist/`, `build/`, and secret-adjacent files BEFORE the first commit of any repo.
- Feature branches lower-case-hyphen-named; branch from the correct base; commits go to the branch, not to `main/master`, unless the project's conventions explicitly commit there.
- Partial staging (`git add -p`) when a file mixes unrelated concerns — don't make a cherry-pick impossible later.
- Tags: release tags are annotated and created in the release flow (release-manager runbook), never as casual side effects.

## With the team
- Parallel windows/sessions on one repo follow `co-work` claims — a declared merge owner serializes merges; two agents "just fixing the merge" in parallel is contention, not cooperation.
- `git bisect` is the regression-hunter's tool (pairs with `debugger`); on finding the culprit, report the commit + the reason, don't deliver a verdict without the reproduction evidence.
- Every mysterious state, force-push temptation, or bad merge outcome → `self-improving-agent` CAPTURE — the discipline gets sharper each incident.
