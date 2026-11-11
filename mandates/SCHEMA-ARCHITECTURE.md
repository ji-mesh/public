# SCHEMA-ARCHITECTURE MANDATE (operator 01.11: "jedes repo ein eigenes schema folder — in jimeshOps nur die regeln — wir müssen alles aufräumen")

**Binding since 2026-11-01. Replaces the "one central schemas package in jimeshOps" reading of DEV-140/142.**

## 1. Problem (audit 2026-11-01)

- `jimeshOps/schemas/src/` currently holds ALL wire-types (files, calendar, crm, media, logs, carousel) — but **zero repos import it yet** (no `@jimesh/schemas` consumer found in jimeshIO/jimesh2).
- At the same time, per-repo schema material ALREADY exists and drifted: `jimeshIO/frontend/animation-schemas/` + `jimeshIO/docs/schemas/`, `jimesh-docs/development/schemas/` + `COMPONENT-SCHEMA.md`, `jimesh2/internal/logschema` (S222-7).
- Central wire-types in the ops repo = wrong ownership: the consuming repo can't evolve its contract without a cross-repo PR — governance and code are entangled.

## 2. Target structure (binding)

**Schemas live where they are consumed. jimeshOps owns the RULES, not the types.**

| Repo | `schemas/` folder owns |
|---|---|
| `ji-mesh/jimeshOps` | **GOVERNANCE ONLY**: `schemas/RULES.md` (the rules below), `schemas/REGISTRY.md` (which schema lives where, version, owner), validation scripts (NFC/trace/strict-typing lint). NO concrete wire-types. |
| `ji-mesh/jimeshIO` | website/widget contracts: `files/editor` wire-types (FileBrowserWidget/EditorFlow payloads), `carousel` (SCarGroup/SCarSlide), animation-slides (from `frontend/animation-schemas`, consolidated), `media playback` types |
| `ji-mesh/jimesh2` | dashboard/backend contracts: `crm` (Deal/Contact/Activity), `calendar` (KCalendarEvent + recurrence), `media delivery` (DVideoAsset, KSubscription), `logs` (canonical mirror of internal/logschema) |
| `ji-mesh/jimesh-docs` | content contracts: article front-matter, component-doc schema (from COMPONENT-SCHEMA.md), docs-navigation schema |
| `ji-mesh/ad1` | calendar API side: follows jimesh2 calendar contract; Pydantic/gRPC stubs generated FROM the TS source (DEV-150 output), never hand-diverged |

## 3. The rules (jimeshOps enforces, every repo obeys)

1. Strict typed — no `any`, no unbounded strings (unions/enums/branded types).
2. Every wire record carries `ts` (ISO-8601 UTC), `traceId` (`X-JiMesh-Trace-Id`), audit-chain ref.
3. Unicode-safe: label/text fields NFC-normalized UTF-8, no control chars (<0x20 except \n \t).
4. One schema file per lane, named after the lane (`files.ts`, `carousel.ts`, …), barrel-exported.
5. Cross-repo imports of schema TYPES are forbidden — shared contracts go through the owning repo's published package or generated stubs (Pydantic/Proto/SQL from DEV-150 pipeline).
6. Every schema change = PR into the OWNING repo + REGISTRY.md version bump in jimeshOps (docs-only PR) + §9 update on the linked issue.

## 4. `@jimesh/schemas` disposition

- The npm/workspaces package becomes a **release-time aggregate**: each owning repo publishes/exports its `schemas/` folder; `@jimesh/schemas` re-exports them on release (thin meta-package, versioned by REGISTRY).
- Until the migration lands, `jimeshOps/schemas/src/*.ts` stay in place UNTOUCHED (they are the reference the per-repo moves copy from) — no edits, no deletions mid-migration.

## 5. Migration plan (execution gated)

**Gate: only AFTER the current rebase wave (jimeshIO PRs #14/#15/#19/#25/#27) and the queued night-run batch have merged.** No schema file moves while 5+ PRs conflict — the move PRs would conflict too.

1. Copy (not move) wire-types from `jimeshOps/schemas/src/` into the owning repos per §2, repo-native naming.
2. Each owning repo: wire the folder into its build (tsconfig paths / package exports).
3. jimeshOps: delete moved types, add `schemas/RULES.md` + `schemas/REGISTRY.md` + validation scripts.
4. Update consumers (FileBrowserWidget/EditorFlow/dashboard mounts) to import from their own repo folder.
5. DEV-142 scope is reframed accordingly (per-repo folders + jimeshOps governance instead of central shared package) — §9 scope-change update posted.

## End of mandate.
