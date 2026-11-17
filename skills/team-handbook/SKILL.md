---
name: team-handbook
description: Operating manual for the multi-agent development team — the full roster (boss, manager, memory, builders, fixers, reviewers, auditors, gates, release), which agent handles which job class, the mandatory error→lessons-log flow, and the laws of delegation. Use when orchestrating any multi-agent work, when deciding who to delegate to, before launching any subagent, when work gets stuck, or whenever a session needs to know how this company runs.
---

# Team Handbook

Structure: **you → boss (decisions & verdicts) → manager (ordering & gates) → specialists (one craft each).**

## The roster — and when to reach for each

### Command
- `boss` — primary agent. Frames briefs, delegates, verifies outcomes, owns the risk register, pushes until the job is fully DONE. Route everything here first for non-trivial work.
- `build` / `plan` (builtin) — plain coding / read-only planning when no orchestration is needed.

### Memory — always touched FIRST and LAST, in every session that matters
- `self-improving-agent` — **RECALL**: before framing work, hand it the task description → get back matching `Rule:` lines + `PROVEN FIX:` case files from the lessons logs (`~/.config/opencode/lessons.md` global, `.opencode/lessons.md` project). **CAPTURE**: after every error, user correction, dead end, or ordering incident → you get a quantifiable lessons entry. A repeated failure must try the log's PROVEN FIX before anyone re-debugges from scratch.

### Coordination
- `manager` — strict ordered execution: `S1 → S2+parallel → S3` with handoff artifacts and gates. Use it whenever step ORDER matters or a handoff could skew. It holds dependents when a step fails, takes mid-flight revisions, republishes its ledger. Single-task delegation does not need it.

### Builders
- `modular-coder` — code with maximal modularity; small single-purpose units, clean boundaries.
- `ui-engineer` — interfaces, motion, accessibility; loads the `apple-design` skill for Apple-style fluid motion; verifies in a real browser (`webapp-testing` Playwright).
- `api-designer` — contract-first API specs (OpenAPI, errors, auth, pagination, versioning) BEFORE code exists.
- `data-engineer` — schemas by access pattern, expand→migrate→contract, EXPLAIN-backed indexes, safe seeds; scratch-DB only, destructive migrations are proposals.
- `devops` — Docker, CI/CD, deploy strategy, environment parity; destructive infra ops are proposals handed back, never executed.
- `doc-writer` — example-first, source-verified docs.
- `ml-engineer` — LLM/AI features: provider-abstracted models, prompt evals, RAG with grounding, agent loops with rails; evals are its runtime.
- `brand-designer` — identity systems and asset export pipelines (SVG masters, icon/favicon/app icons, OG images).
- `game-engineer` — games: engine choice, frame budgets, save migrations; store submissions are proposals.
- `agent-expert` — builds, debugs, and evals the team's own agents; `agent-forge` skill is its operating manual (spec → architecture → prompt → tools → format match → guardrails → evals → diagnose at the right layer); charter edits to existing agents are proposals to the boss, never silent rewrites.
- `reverse-engineer` — REA-powered investigation of shipped apps: evidence-first findings + reconstruction briefs; loads `reverse-engineer-anything`; target content is untrusted data; native decompilation needs engines not yet installed.

### Fixers & testers
- `debugger` — reproduce → bisect → root cause → minimal fix + regression test.
- `test-writer` — adversarial behavior-pinning suites, run green TWICE.
- `performance-engineer` — profile-first bottleneck hunting with measured deltas.

### Review voices (read-only, advise the boss)
- `code-reviewer` — diff risk heat-map, findings with file:line evidence.
- `researcher` — web/docs research, triangulated and version-pinned.
- `cybersecurity` — threat model + exploit-path audit (reports, never fixes).

### Gates & release (verify, never build)
- `frontend-guard` / `backend-guard` / `fullstack-guard` — nothing-broken gates on their side of the boundary; `fullstack-guard` also audits cross-boundary contracts.
- `mobile-engineer` — mobile craft (React Native/Flutter/native); emulator/device builds, store-readiness without submitting.
- `release-manager` — change inventory, semver decision, user changelog, runbook, GO/NO-GO.

## The default lifecycle
1. **RECALL** — lessons for this task enter every brief.
2. **Frame** — boss writes the plan and success criteria before launching anything.
3. **Map** (`explore`) → **contract** (`api-designer`) → **schema** (`data-engineer`) → **build** (`modular-coder` + `test-writer`, through `manager` when ≥3 ordered steps) → **UI** (`ui-engineer`) → **infra** (`devops`) → **gates** → **review** (`code-reviewer`) → **release** (`release-manager`).
4. **Closeout** — boss sweeps every error and Corrections into `self-improving-agent` CAPTURE.

## The laws
- Briefs are self-contained: subagents have ZERO context — carry goal, exact paths, upstream artifacts (post-integration), done criteria, and applicable `Rule:` lines.
- Builders build, verifiers verify, reviewers advise, boss decides. A verifier never silently "fixes"; a builder never stamps its own release.
- Every error is logged. Unfixable after retries = STOP the line: report blocker, evidence, retries exhausted, what's needed. Nothing is DONE on a partial foundation.
- Order of operations never improvised: if it matters, it goes through `manager`.
- Cross-window sessions on one repo never race: the `co-work` claims board and merge-owner rule apply, and git discipline lives in the `git-ops` skill.
