# PACKAGE-ADOPTION MANDATE — JiMesh (since 2026-10-31, EN)

Applies to EVERY package we adopt externally and modify ("we take the packages and modify them").
Reference research: `docs/development/research/cms_helpdesk_integrations.md`, `docs/development/research/video_media_integrations.md`, backlog: `docs/development/roadmap/POLISH-BACKLOG-2026-10-31.md`.

---

## 1. Scope uniformity for all packages

Whatever package comes in, it gets the same treatment — no special paths:

1. **Self-host only.** No cloud-only path moves into the UI or provisioning. Every feature lane that upstream runs "only with SaaS/Cloud" is either replaced by our own lane or stays out.
2. **Branding & design tokens via the mesh building block.** Colors, type, radii, icons come from `assets/brand` + the tokens — never from the upstream theme.
3. **Agents ONLY via the AI-Gateway proof.** No native "Captain"-/canned-lane responses from the package (example: Chatwoot Captain is cloud-bound, not officially self-hosted). Agent calls run exclusively via `/v1` with `X-JiMesh-Trace-Id` — agents never call packages directly.

---

## 2. Security clauses

1. **Secrets ONLY in the Vault.** No `.env` routes for adopted packages; no API keys in the repo, in the UI bundle, or in Helm values in plaintext.
2. **Audit-chain binding:** every visible state change (create/update/delete/reveal) writes into the audit chain — no silent mutations.
3. **Session/trace headers:** package calls that cross service boundaries carry `X-JiMesh-Trace-Id` + session context (H1 cookie or `jimesh-sk-` token) — never a raw key.
4. **Unicode-safe:** all label/text fields NFC-normalized UTF-8, no control chars (<0x20 except \n \t).

## 3. Package portfolio (2026-11-01, complete)

The binding list: **every package has its adoption status in the inventory note** (`docs/development/research/PACKAGE-SVG-GCP-INVENTORY-2026-10-31.md`). Short form:

| Package | Version/Lane | Status | Sprint |
|---|---|---|---|
| `@open-pencil/scene-graph` + `dom-css` | 0.16 (real npm dep + vendored ESM) | ✅ wired in (18-design real model, DEV-93 follows with dashboard canvas) | DEV-93 |
| `monaco-editor` | self-hosted `/vendor/monaco/vs` | ✅ except DEV-92 wrapper | DEV-92 |
| `@xyflow/react` | dashboard + site console | ✅ | — |
| `@mui/x-data-grid` | via esm.sh, mesh-sql in 07-project | ✅ demo lane | prod lane after DEV-118/119 |
| `ra-core` + `react-hook-form` | installed (atomic-crm pattern) | ✅ installed, DEV-117 builds MeshCrmBoard | DEV-117 |
| `chatwoot` | self-host stack + Helm `chatwoot-tenant` | ⬜ install pending | DEV-98/116 |
| `react-player` | 3.4.0 | ✅ installed, MediaLane live | DEV-109 (YT feed) |
| `@tanstack/react-query` | v5 in jimesh2 | ✅ | — |
| `payload` | planned | ⬜ DEV-97 | gate: Next-16 spike |
| `strapi` | phase-2 spike | ⬜ DEV-110 | multi-tenant customer CMS |
| `@gitgraph/js` | installed, unused | ⚠️ | DEV-94: use or remove |
| `@react-three/fiber` | 3D hub canvas (animation-states.json) | ✅ | DEV-118/119 grouping |
| `fast-whisper` self-host (+ Deepgram/OpenAI) | MED-1/5 route planned | ⬜ | DEV-99/102 |
| `A2A gateway` (pure Go) | JSON-RPC 2.0 agent-card + tasks/send/get/verify | 🔧 DEV-132 session (Harness² sandbox flag + AutoGen delegation) | DEV-132 |

### Own product lanes (not a third-party product, built on adopted packages)

- **mesh-crm** (DEV-125): own CRM product (kanban/contacts/timeline/audit chain) on the ra-core pattern — the dashboard is the primary product; the atomic-crm adoption delivers the object model + form patterns, the UI is built with our own design tokens (`docs/development/research/PACKAGE-SVG-GCP-INVENTORY-2026-10-31.md` → slop matrix).
- **mesh-cdn / content-delivery** (DEV-124): own content-delivery lane — origin lanes: Ozone-S3 (self-hosted, sealed) + Cloudflare-R2-Edge (creds after rotation), served via react-player; cache rules, pre-signed seals, bandwidth metrics in the dashboard lane.

## 4. Docstring / API-docs rules (DEV-150 mandate cross-ref)

- Python: Google-style docstrings → API docs via autodoc; TS: TSDoc → typedoc; OpenAPI 3.1 for REST routes.
- Rendering: typedoc/next-docs lane + markdown rendering in the public-site blog/MDX lane.
- Detail spec: `docs/development/process/DOCSTRING-AND-APIDOCS.md` (DEV-150 in flight).

## 5. Delivery/scope rules (DEV-150 cross-ref)

- Acceptance-criteria-first; package-lane-first (the package defines the boundary); operator approval gate; review gates (DoD = reviewed).
- Detail spec: `docs/development/process/DELIVERABLES-AND-SCOPE.md` (in flight).

## 6. Forbidden

- No invisible re-branding of third-party UIs (our own CRM UI uses the atomic-crm pattern, not its logos).
- No external data silos without the vault-seal lane.
- No cloud-only features smuggled into the UI.
