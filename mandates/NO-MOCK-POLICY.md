# MANDATE: Strict No-Mock & Database/Store Hygiene Policy (`NO-MOCK-POLICY`)

**Status:** BINDING across all JiMesh repositories (`ji-mesh/jimeshOps`, `ji-mesh/jimesh2`, `ji-mesh/jimeshIO`, `ji-mesh/jskills`, `ji-mesh/ad1`)
**Effective Date:** 2026-11-09
**Related Logbooks:**
- [`provisioning/docker/browser-webrtc-hub/data/default/MOCK_AND_DB_AUDIT_LOGBOOK.md`](../provisioning/docker/browser-webrtc-hub/data/default/MOCK_AND_DB_AUDIT_LOGBOOK.md)
- [`docs/development/handoffs/MOCK-AND-DB-ENTRIES-INVENTORY-2026-11-09.md`](../docs/development/handoffs/MOCK-AND-DB-ENTRIES-INVENTORY-2026-11-09.md)

---

## 1. Core Mandate (Zero Seeded Mocks in Runtime Databases & JSON Stores)

1. **No Seeded Mock Database Rows or Fallback Arrays:**
   - Backend services (`server.js`, `chat-store.js`, `opencode.db`, `replay.db`, PostgreSQL) MUST NEVER auto-seed fictional demo records when a table or JSON store is empty.
   - If a store (`calendar_events_store.json`, `schedules_store.json`, `agent_profiles_registry.json`, `experts_state.json`) has no operator- or agent-created records, the API MUST return an empty array `[]` and the UI MUST render a truthful empty state (`"No cron lambdas registered..."`).
2. **No Fictional Agent Personas or Ghost Sessions:**
   - Only real runtime OpenCode / Camoufox sessions (`default`, `agent-orchestrator`, and active `ses_*` rows verified in `opencode.db` `session_v2` and `replay.db` `sessions`) may appear in sidebars, calendars, chat threads, or agent registries.
   - Helper tags (`#helper_*`), plugins (`plugin-*`), and workflow hooks (`workflow-*`) are **execution tools**, NOT standalone agent profiles. They must resolve to the owning session (`default` or `agent-orchestrator`) via `ensureAgentProfileRecord()`.
3. **No Fabricated Infrastructure, Network, or CRM Telemetry:**
   - Never hardcode fake external IPs (e.g., `185.220.101.42` / `Hetzner Online GmbH`), fake OAuth/Nango connection IDs (`nango-gh-*`, `nango-gws-*`), fake Google Meet URLs (`https://meet.google.com/jsh-cam-hitl`), or fake CRM deal values (`€32,000`, `€48,000`).
   - All network metadata must reflect actual container inspection (`host-bridge (127.0.0.1)` or live proxy config), and all account bindings must reference real Vault-backed or AgentMail-backed credentials.
4. **Mandatory Audit Logbook Co-Location:**
   - Every database table (`opencode.db`, `replay.db`) and JSON store file (`provisioning/docker/browser-webrtc-hub/data/*.json`) must be documented in [`provisioning/docker/browser-webrtc-hub/data/default/MOCK_AND_DB_AUDIT_LOGBOOK.md`](../provisioning/docker/browser-webrtc-hub/data/default/MOCK_AND_DB_AUDIT_LOGBOOK.md) (in the exact same directory as `CHANGES_2026-11-06_session_log.md`) and [`docs/development/handoffs/MOCK-AND-DB-ENTRIES-INVENTORY-2026-11-09.md`](../docs/development/handoffs/MOCK-AND-DB-ENTRIES-INVENTORY-2026-11-09.md).
   - Any test/verification row created during E2E testing (`ses_test_*`, `evt_e2e_*`) MUST be cleaned up immediately after verification or explicitly logged with its origin and lifecycle status.

---

## 2. Enforcement & Sanitization Gates

| Store / Database | Sanitization Function / Gate | Policy |
| :--- | :--- | :--- |
| `data/.opencode-data/opencode.db` (`session_v2`, `schedule`) | OpenCode SQLite SSOT | Only real OpenCode sessions; zero synthetic seed rows. |
| `data/replay.db` (`sessions`, `session_traces`) | `purgeStaleMockSessions()` | Purges any `ses_test_*`, `test-session-*`, or unbacked mock IDs. |
| `data/calendar_events_store.json` | `loadCalendarEventsStore()` + `FAKE_CALENDAR_MOCK_EVENT_IDS` | Strips all legacy demo events (`evt-okf-watchdog-nov9`, `evt-reddit-osint-nov9`, `evt-browser-som-verify-nov9`, `evt-whatsapp-route-nov9`, `evt-datahub-lead-sync-nov9`, etc.); returns `[]` when empty. |
| `data/schedules_store.json` | `loadSchedulesStore()` + `FAKE_SCHEDULE_MOCK_IDS` | Strips all legacy demo schedules (`sched_watchdog_okf_5m`, `sched_osint_reddit_hourly`, `sched_wa_session_health`); returns `[]` when empty. |
| `data/agent_profiles_registry.json` | `ensureAgentProfileRecord()` | Blocks `helper_*`, `plugin-*`, `workflow-*` from creating fake profiles; strips fake Hetzner IP `185.220.101.42` and fake `nango-*` IDs. |
| `data/agent_injections_store.json` | `loadAgentInjectionsStore()` | Strips unresolved `{{agent.id}}` template keys and deduplicates tooltip injections to max 1 per turn. |
| `data/experts_state.json` | `loadExpertsState()` | Retains only real session state drawers and live `exp_default_2911`; purges static mock expert profiles (`exp_camoufox_stealth_01`, `exp_ddns_agentmail_02`, `exp_osint_lead_03`). |
