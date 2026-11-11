# LINEAR-STRUCTURE MANDATE (operator 03.11: "neues jimesh project aufsetzen mit ordentlicher Struktur: Epics/Issues statt 24 Mikro-Milestones; Milestones = Core Pillars/Releases wie frontend-public, chat-helpdesk, crm-pipeline")

**Applies to every issue in the JiMesh Linear projects. Enforced since 2026-11-03.**

## 1. jiMesh project = the SSOT container

- The **jiMesh** project ([linear.app/jimesh/project/jimesh-b732161b9718](https://linear.app/jimesh/project/jimesh-b732161b9718)) is the single active project container for all feature and architecture issues.
- The legacy **jiMesh-MVP** project ([linear.app/jimesh/project/archived-jimesh-mvp-migrated-to-jimesh-ea3dcfcabd63](https://linear.app/jimesh/project/archived-jimesh-mvp-migrated-to-jimesh-ea3dcfcabd63)) is archived and frozen as of 2026-11-03.
- Sprints are one week, Mon→Sun (first: **Sprint 1, 2026-11-02 → 2026-11-08** — "Night run & widget chain").
- ⚠️ The GraphQL API rejects `cycleCreate` ("Cycle creation is not supported") for this team — sprint cycles are selected via `cycleId` on `issueUpdate`.

## 2. Milestones = Core Pillars / Release Tracks (Not micro-features)

Milestones are NOT created for individual PRs or cosmetic fixes (e.g., no "Feature: Footer rewrite" or "Feature: Mouse animation"). Micro-features are **Issues / Sub-issues** underneath the core milestones.

The 6 canonical milestones in `jiMesh`:
1. `M1: Public Frontend (frontend-public)` — Marketing site, landing page carousels, responsive hero animations, interactive widget previews on `/workspaces`, footer/nav polish.
2. `M2: Unified Chat & Helpdesk Engine (chat-helpdesk)` — Native omnichannel shared inbox, private notes, multi-mailbox sender picker, canned response macros, AI copilot & agent-human handover.
3. `M3: CRM & Pipeline Engine (crm-pipeline)` — Deal pipeline management, contact history, lead workflows, `ra-core` DealList, gateway CRM routes, integrated CRM conversation drawer.
4. `M4: Admin Dashboard & Tenancy (frontend-admin)` — Operator cockpit (`jimesh2`), Keycloak SSO & realm federation, tenant provisioning, seat management, audit log viewer.
5. `M5: Workspaces & Content Engines (workspaces-content)` — Encrypted FileBrowser, Monaco Markdown split-editor, OpenPencil CanvasKit scene graph, media & voice lane.
6. `M6: Core Mesh Infrastructure & Gateway (mesh-infra-gateway)` — Agent-to-Agent (A2A) gateway, reverse proxy, Tailscale/Headscale mesh network, Vault secrets engine, cryptographic audit chain.

## 3. Sub-issues + labels/tags are MANDATORY

1. **Sub-issues:** every work item under a feature hangs as a sub-issue of the feature issue (`parentId`), e.g. DEV-136 + DEV-92 under DEV-131 (EditorFlow); DEV-89/90/91/94 under DEV-137 (version control). No orphan work items in descriptions/comments.
2. **Labels on every issue** (naming: `type/…`, `status/…`, `area/…`, `needs/…` — slash style, not colon):
   - one `type/*`: feature · docs · schema · research · infrastructure · security · social · case-study · mcp · spike · chore
   - `status/in-flight` or `status/blocked` when applicable
   - `apply/urgent` for night-run/urgent work, `needs:decision` for operator-gated items
3. No issue ships without: description with acceptance criteria · priority · start/due date · assignee (owner) · labels · milestone · sprint membership (see the linear-project skill §6 anatomy).

## 4. Maintenance rules

- Restructure scripts live in the repo (e.g. `linear-restructure*.sh` pattern) — idempotent, EN comments, keys only from `.env` (`LINEAR_API_KEY`, never printed).
- Duplicate milestones get deleted, never silently kept. Scope changes → §9 update template (linear-project skill).
- Legacy Done issues (DEV-26–53) may stay label-less — they are outside the sprint and frozen.

## 5. PR ↔ issue linking is MANDATORY, in BOTH directions (operator 01.11, later: "die prs müssen auch immer die features referenzieren und der name muss dann auch stimmen und verlinkt sein und vice versa")

Every PR and every Linear issue must reference each other — no unlinked work:

1. **PR title** = `DEV-<id>: <short title>` — the correct numeric issue ID, matching the issue's subject. Placeholder names like `DEV-SVG` are forbidden.
2. **PR body** contains `Refs: DEV-<id> (linear.app/issue/DEV-<id>)` plus the acceptance-criteria checklist.
3. **Branch name** contains `DEV-<id>` (GitHub auto-associates the PR with the issue).
4. **Linear issue** gets the PR URL as a comment the moment the PR exists (via `jules-linear.py` bridge — automatic for bridge-launched sessions; manual `jules-linear.py comment DEV-<id> "🔗 PR: …"` otherwise).
5. **Vice versa:** if an issue gains a PR later (manual push, dev work), the issue comment is added the same day — an open PR without an issue comment or an issue whose PR is unlinked = hygiene violation, fix on sight.
6. Audit (2026-11-01): 9 open jimeshIO PRs — all titles corrected (PR #22 `DEV-SVG` → `DEV-151: …`), `Refs:` lines added to 7 PR bodies, PR-link comments posted on all 9 issues.

## 6. Feature Specification Documents in Linear (MANDATORY per operator 03.11)

Every major feature track or core pillar milestone must maintain a dedicated **Linear Document** inside the `jiMesh` project:
- Top: Visual design reference / UI screenshot
- Section 1: Architectural locations & file mapping across `jimeshIO` and `jimesh2`
- Section 2: Functional requirements & default view behaviors
- Section 3: Security & cryptographic boundaries (AES-256-GCM Ozone, Vault)
- Section 4: Roadmap, associated issues (`DEV-<id>`), and verification commits
- SSOT mirror in repo: `docs/specs/<FEATURE>-SPEC.md`
- Details: `mandates/FEATURE-SPEC-DOCUMENTS.md`

## End of mandate.
