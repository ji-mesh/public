# AGENT-EXECUTION-RULES — Binding rules for every agent session

**Status: binding since 2026-11-11.** Applies to EVERY agent session and subagent:
OpenCode sessions, AGY sessions, Jules runs, and spawned subagents. Read together with
`AGENTS.md` (section 1 = mandatory reading order) — this mandate adds execution discipline.

## 1. Worktrees are MANDATORY for code work

- Every session that changes code **starts by creating a worktree** with a feature branch
  (OpenCode: `POST /api/worktree` / worktree tool; CLI: `git worktree add <path> -b <branch>`).
- **Never edit feature code directly in the shared main working tree.** Two agents editing
  the same file in one tree is a guaranteed race (incident: `schemas/src/agy.ts`, 6 duplicate
  subagents 2026-11-11).
- Branch naming: `feature/<DEV-id>-<slug>` or `fix/<slug>` → PR → review → merge
  (AGENTS.md section 3 — never push to main).
- **Declared exception (must be stated in the session reply when used):** files that are
  *live-mounted into running containers* — `provisioning/docker/browser-webrtc-hub/`
  `{server.js, replay-db.js, session-catalog.js, agy-wrapper.js, public/**, data/**}` —
  when the task requires live verification against the running hub. After verification the
  change must be committed on a feature branch (or handed back for PR) in the same session.
- Worktree files are NOT visible to host-mounted containers — copy/apply before testing live,
  and say so in the report.

## 2. Mandates & documentation

- Mandatory reading order stays AGENTS.md section 1 (docs INDEX → PACKAGE-ADOPTION →
  FRAME-IFRAME → sprints → handoffs → NO-MOCK).
- Schema work: `schemas/src/` is SSOT; upstream API docs are archived under
  `source/jimesh-docs/development/reference/` and **must be referenced in the schema file
  header** (see `schemas/src/agy.ts`, `schemas/src/sessions.ts` for the pattern).
- New docs go into `source/jimesh-docs/` (never into top-level `docs/` — see `docs/README.md`).
- EN-only for code, docs, commits (AGENTS.md section 3).

## 3. Subagent handling

- **Background is the default.** Foreground/blocking only when the operator explicitly
  must wait for the result — otherwise the session becomes unreachable.
- **One task = one subagent.** Give each subagent **disjoint file ownership**; never spawn
  two agents that write the same file.
- **Before spawning:** check for an existing session/branch/PR for the same task — send
  follow-ups (`sessionID`) instead of re-spawning (quota discipline, AGENTS.md section 4).
- Every subagent prompt must contain: exact file paths, the reference docs to read first,
  the conventions to follow (typing, validators, EN comments, header doc references),
  an explicit "do NOT touch other files" rule, and a verification step (tsc/syntax check).
- **Verify before reporting done:** run the type/syntax check yourself on the result —
  never trust a subagent report without a check.
- Duplicate spawns that race on one file are a **process incident**: report it, verify the
  winner line-by-line, do not let a loser overwrite it.

## 4. Mention-only messaging (tagging-only subchannels)

- Channels flagged `mentionOnly` deliver an agent's message to other agents **only when it
  contains `@all` or `@<agent-id>`**. Plain posts remain visible in the thread but are not
  injected as prompts into other agents.
- Agents MUST use `@all` / `@<agent>` whenever they intend to call someone — this is part
  of the runtime instructions (opencode-runner `entrypoint.sh`, AGY runtime `AGENTS.md`).
- UI: composer tooltip + channel settings toggle explain the mode (operator-facing copy may
  be German, consistent per section).
- Rationale: prevents accidental broadcast storms — pairs with the throttle rules below.

## 5. Throttling & emergency stops

- Rules SSOT: `source/jimesh-docs/development/reference/agent-governance/throttling-and-emergency-stops.md`.
- Respect `ThrottleRejection` (429 semantics): back off, do not retry-loop.
- **Never self-lift** a stop/block; only the operator (or a higher orchestrator for its own
  subtree) lifts incidents. ESC ESC (UI) = operator emergency stop incl. subagents.
