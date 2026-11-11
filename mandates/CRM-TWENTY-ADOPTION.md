# CRM/TWENTY-ADOPTION — birth of the real CRM lane

**Status:** ACTIVE — fork landed today on ji-mesh/twenty.
**Author:** operator-directive 2026-11-01 @ operator quote: "Twenty (https://github.com/twentyhq/twenty) oder NextCRM (…) eins nehmen wir jetzt und forken das und bauen das dann ein statt dem temu CRM aber mit unserer email und wir integrierte es in die sidebar und dann mocke für die demo und public website".

---

## 0. Operator decision (2026-11-01, evening) — CONFIRMED: Twenty is THE lane

Two parallel real-CRM lanes briefly existed after the vendored-session commit. Resolved by operator answer:

- **OFFICIAL lane: Twenty** (`ji-mesh/twenty` fork of twentyhq/twenty) — per this mandate (§1 criteria). All sidebar/marketing integration steps below apply.
- **NOT official: `feat/real-nextcrm`** (ji-mesh/jimesh2, vendored nextcrm-app under `src/crm-app`, 2712 files, pushed 2026-11-01 19:58). Kept **read-only as a reference implementation** (useful for email-per-installation patterns and RA-core adapter reuse); no merges to main, no pushes, no mounts from it. Do not wire it into the sidebar.
- No new commits on `feat/real-nextcrm` beyond upstream reference sync; anything it needs (mail layouts, RA adapter learnings) gets ported into the Twenty lane, not maintained in parallel.

---

## 1. Why Twenty

| Criteria | Twenty (chosen) | NextCRM (fallback) |
|---|---|---|
| ✅ Community size | 60k+ stars; actively shipped | ~1k stars |
| ✅ Email | native Outlook / SMTP connectors (Settings → Mail) — matches jimesh "eine mailbox per installation" story | manual Next Mail module (less depth) |
| ✅ Stack | NestJS + Postgres + GraphQL (real product) | Next.js + Prisma + Postgres (light) |
| ✅ Fits jimesh mesh | runs containerized; Ozone can drive it (like code-server / adminer resolve) | fine but re-codes a UI we already own |
| ✅ License | **AGPL-3.0** (some `@license Enterprise` files — respect their license header; don't reuse enterprise-marked files unless the plan licenses them) | MIT |

If both run locally demos later aren't harmed — Twenty inside jimesh2 sidebar gets the direct real product link (see lane step 4).

---

## 2. Adoption lane (7 steps, in order)

1. Fork → `ji-mesh/twenty` — **DONE 2026-11-01** (`gh repo fork twentyhq/twenty --org ji-mesh --clone=false`).
2. Self-host build at the GCP dev-VM lane (operator provisioned gcloud project earlier) — run the upstream repo's `infra/` compose with a Postgres mesh-side service; documentterraform + gcloud config as a separate jules-lane ticket. No docker-compose.yml committed at repo root — check `packages/server`/`infra/` upstream docs.
3. Mail: Configure Twenty connectivity to the existing **jimesh mail** (Postmark/SendGrid + catch-all) via Twenty's "Settings → Mail" connectors; map to jimesh `jimesh-configuration` keys (documented in jimeshWorkspaces.md). No vendor lock-in (our email layer is already on the mesh).
4. jimesh2 dashboard sidebar — the real steps (must be executed via the directive pattern, i.e. at least one new nav `key + aria-label + test` for `twenty` in `HierarchySidebar.tsx`; plus i18n `nav.*` entry; plus App.tsx route registrar: `'/workspaces/twenty'` mounting `TwentyAppFrame` iframe-less via MeshProxy route alias). **Lane key:** "twenty-sidebar-integration" (HIGH).
5. Demo (marketing site): mount the **real rendered route** when live — via MeshProxy `/apps/twenty` alias; if demo-data only, mount the **MeshCrmBoard ra-core** with `NEXT_PUBLIC_JIMESH_API_BASE` pointing to the REALTwenty's REST/GraphQL provider on the dev VM (adapter inside `mesh-crm-board.tsx`).
6. Public website: replace `/crm-demo` demo-theme panel with a real embedded panel that routes to its own contract per mandate `FRAME-IFRAME-RULE` (no iframe-mocking; the app's original mesh view remains on instance domain, link to dashboards demo of `/workspaces/designs#/twenty`).
7. Test and acceptance: route resolves on mesh, agent mail flow works (jimesh mail → Twenty connector), serial demo acceptance on both site and dashboard mount.

## 3. Rag-on-state today

The display today only shows MutiCrm's **demo** pattern (MeshCrmBoard with demoDeals-Label). After step 3 (Twenty up), all display becomes real data — and jimesh secures a real CRM section on /workspaces via MeshProxy.

## 4. Legal

AGPL-3.0 — internal self-host has no surface obligations. Public marketing pages: linking to a live route is compliant; do NOT copy Twenty's marketing copy/screenshots to our site (no concern raised) — the mandate is a self-hosted internal tool, source yours.

---

## 5. Reason not to jam the sidebar mesh entry today

`HierarchySidebar.tsx` is directive-driven per the mandate tree ("Navigation entry" rules with tests, i18n nav.* keys, aria rules, and the Ozone app-discovery). Adding Twenty without a real route and without i18n = instant dead link = "dead-element"; we hard-bind it to real data only when step 2 (service rails up) is complete.
