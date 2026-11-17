---
name: co-work
description: Concurrency protocol for multiple OpenCode sessions/agents working in the SAME project at the same time — claim-before-edit registry with heartbeats, overlap detection, stale-claim takeover, and git worktree isolation for parallel streams. Use before editing files in any project, before launching parallel agents or streams, whenever concurrent sessions might overlap each other, when creating or merging worktrees, or when symptoms like disappearing changes, reverting files, or "file in use" errors appear.
---

# Co-Work — cross-session concurrency protocol

Agents in different terminal windows share one filesystem and nothing else. Two sessions editing the same files race; one quietly overwrites the other. No cross-window "coordinator agent" can see both sessions — the filesystem is the only shared memory. So shared state lives on disk, and every session follows this protocol mechanically.

## The claims board
- Location: `<project>/.opencode/claims/` — create the directory if missing. If the project is a git repo, add `.opencode/claims/` to `.gitignore` (it's session state, not history).
- One small JSON file per live session — `.opencode/claims/<unique-id>.json` (session id, or `host-user-pid-<hhmm>`). You write ONLY your own file; you READ everyone else's. Never rewrite another session's claim — conflicts are resolved by reading, not by overwriting.

```json
{
  "owner": "boss",
  "session": "<session id or host-user-pid>",
  "task": "one line of what this session is doing",
  "status": "working",
  "started": "<ISO8601>",
  "heartbeat": "<ISO8601>",
  "scope": ["src/auth/**", "package.json"]
}
```

## The protocol — follow exactly, no improvisation
1. **CLAIM BEFORE EDIT** — before your first write (or before delegating writes), scan all claim files, then create yours with your full intended touch-scope: files, globs, directories. Solo sessions register too — cheap, and the board is situational awareness for everyone.
2. **HEARTBEAT** — update `heartbeat` after every work batch, and at minimum every 10 active minutes. Growing the job means growing `scope` in the same write. Never touch files that were never in scope without amending the claim FIRST.
3. **RE-CHECK BEFORE EVERY WRITE BATCH** — rescan the board immediately before each scoped edit batch (this is the load-bearing rule; the initial check alone is not safety):
   - A LIVE claim (heartbeat < 15 min old) intersects your scope and is not yours →
     - Same `task` as yours → the other session is probably duplicating your work: **stop**, report the duplication to the user or boss. Never double-build.
     - Different task → do not touch those files. Pick one: (a) wait and re-check, (b) re-scope to genuinely disjoint files, (c) deep unavoidable overlap with a real stream → worktree mode below. Co-editing one file with a live claimee is a **user decision** — surface it, don't attempt it.
   - Reads are always free. Only write scopes conflict.
4. **STALE TAKEOVER** — heartbeat > 15 min old means the session died (crash, window closed). Archive the file as `<id>.stale.json` (never delete — that's evidence), log the takeover, then proceed only after a full re-check.
5. **RELEASE** — delete your claim file when your part is done. Boss/guard end-of-job reports include "claims released" as a checklist item.

## Worktree mode — long parallel streams with unavoidable overlap
- Git repo required. Use the builtin opencode worktree tooling (Code Mode: `tools.opencode.worktree_create/list/remove`) when available, otherwise plain `git worktree add <path> <base-branch>`. One worktree per concurrent stream; each stream's agents claim the worktree path as their scope so the main checkout stays untouched until merges.
- **One merge owner** — exactly one session holds a claim with `task: "serial merge"` (scope: repo root). It merges streams one at a time, in the order the boss/manager sets. No other session ever merges. Two worktrees conflict → rebase the younger onto the merged result; force-push and history surgery are forbidden.
- A worktree is removed only by its owner session, or after explicit user confirmation — removal can discard uncommitted work; the opencode tool's `force` parameter exists precisely for the confirmed case.

## Rules
- **The board is only as safe as scopes are honest.** Underclaiming is the cardinal violation: if you really will touch `package.json` and `src/api/**`, claim exactly that. A claim that says "src/**" blocks the whole tree — claim what you need, no more, no less.
- Network-of-trust check: if the board already contains a claim whose scope covers files you didn't plan to touch, that 发现 means re-plan (wait/split/worktree), never "it'll be fine".
- Every overlap discovery, stale takeover, and merge conflict goes to `self-improving-agent` (CAPTURE, category `error`) — recurring cross-session contention is a planning failure, so the fix belongs in the plan, and the lessons log is where plans learn.
- No git repo? Claim-board mode still works fully. Worktree mode is unavailable — fall back to serialized edits (wait out live claims) or hard scope splits.
- Symptom alarm: uncommitted changes appear that you didn't make, or files revert — rescan the board immediately. That signature means an unclaimed rival session is working blind alongside you; get it claimed before anyone continues.

## Wiring that makes this real
- Orchestration pieces (boss step 2/parallel streams; manager rules) point here: parallel briefs are born with **disjoint scopes written into the brief**, and a designated merge owner — the board then validates the plan instead of discovering races mid-flight.
- Specialists claim through their briefs: every multi-file brief tells the specialist to register (or verify) the session claim covering its assigned scope before its first edit.
