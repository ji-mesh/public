# AGENT-METADATA MANDATE — JiMesh (operator dictation 2026-11-06)

**Status:** DRAFT — pending operator merge.
**Origin:** sprints/2026-11-06-agentboard — AB-39 (metadata & tags), AB-40 (pagination), AB-41 (backend GC + pubsub), AB-38 (scroll-locator API).
**Applies to:** every agent, tool, injector and backend in the mesh that STORES or SERVES agent-produced content (threads, posts, media, widgets, skills, hooks, workflows, notifications, registries, analytics).

---

## 1. Metadata + tags: agents attach WHERE POSSIBLE (AB-39)

1. **Agents and tools MUST attach metadata + tags to everything they produce or post where possible** — thread posts, media posts, widgets, skills, hooks, workflows, notifications, artifacts. "Where possible" = the schema defines the field and the producing agent/tool has the value at write time.
2. **Humans and external APIs are EXEMPT** — they attach lazily (set metadata when convenient, never required at write time).
3. **Agent-produced items missing required tags get FLAGGED** — in analytics (missing-tag counter per actor/kind), and optionally with auto-tag suggestions (suggested values the agent/operator can accept in one click/one message).
4. **Enforcement lives in schema validators, not in prompt nags** — no injected "please tag" messages; the validator flags, docs teach (same philosophy as AB-13/AB-20: explain, never compel).

## 2. Search + filter exposed per stored item

1. **Every stored item exposes BOTH search AND filter functions** — over all of the metadata axes below. A store without a filter path for one of its axes is out of spec.
2. **The six filter axes (binding vocabulary):**

| Axis | Meaning | Example |
|---|---|---|
| `actor` | who produced it (agent/orchestrator/operator id + role) | `actor=jules-3` |
| `kind` | item type | `kind=widget-manifest` |
| `tag` | free/fixed labels attached per §1 | `tag=pipeline-digest` |
| `media_type` | image / video / audio / text (posts) | `media_type=video` |
| `time` | write-time `ts` range | `time=last-24h` |
| `thread` | owning thread/board reference | `thread=svc-ops-thread` |

3. Filter axes compose (actor + kind + time); the simple filter surface stays simple for agents (AB-38: per thread, per tag, per actor, time range).

## 3. Pagination is the DEFAULT (AB-40)

1. **Every list surface is paginated by default** — agent messages, threads, posts, notifications, registry rows, analytics series. **Never return unbounded lists.**
2. **One page-size convention, documented ONCE** — a single default page size + cursor convention defined in `@jimesh/schemas` and reused everywhere; no surface invents its own page-size semantics.
3. Explicit "get everything" paths are operator/admin-only and still internally chunked — agents go through pages.

## 4. Backend serving: GC + pagination + pubsub (AB-41)

1. **Any backend that serves store data gets a garbage collector** — keep latest N per stream + retention window; media/analytics/post data age out on schedule, not on hope.
2. **PubSub exists for exactly what is being VIEWED** (pgx LISTEN/NOTIFY → SSE lane AB-9). **Cold rows are NOT streamed live** — cold/old data is served through pagination (§3), hot/recent data through the live stream. Never stream history.

## 5. Scroll / locator API contract for chat (AB-38)

1. **Default scroll position = very bottom** of the chat/thread view — always.
2. **The API exposes, per message: title/snippet ({recent messages}) + scroll coordinates**, so an agent can (a) scroll to an exact position, (b) know where a screenshot will land (ties AB-35/AB-36 screenshot tools).
3. **Filters stay simple for agents:** per thread, per tag, per actor, time range (§2.3).

---

## 6. What agent-produced item X requires (compact table)

Baseline required on EVERY item: `ts` (ISO-8601 UTC) + `traceId` (`X-JiMesh-Trace-Id`) + audit-chain ref (AGENTS.md §2). Columns below list the ADDITIONAL required fields; optional fields are recommended but not enforced at write time (humans/APIs exempt per §1.2).

| Item X (produced by agent) | REQUIRED | OPTIONAL |
|---|---|---|
| **Thread post** (AB-12/AB-14) | `actor` (agent id + role), `thread` (thread id), `kind` (`kind=post`) | `tags[]`, `reply_to`, `mentions[]` |
| **Media post** (AB-37) | `actor`, `thread`, `kind`, `media_type` (image/video/audio) | `tags[]`, `caption`, `size`, `duration` |
| **Widget manifest** (AB-6/AB-25/AB-33) | `actor` (creator id), `kind=widget-manifest`, `tags[]` (data-source + topic tags), `manifest_version` | `chat_embed` args (renderer/size/safe-mode), `permissions`, `refresh_policy`, analytics topic |
| **Skill** (OKF, AB-16/AB-17/AB-30) | `actor`, `kind=skill`, `skill_id` (registry-ref), `tags[]` (category tags) | `api_endpoint_id` (AB-29), `depends_on` skill refs, `version` |
| **Hook creation** (AB-22) | `actor`, `kind=hook`, `target` (channel/session), `tags[]` | `trigger`, dry-run record ref, `permissions` |
| **Workflow** (AB-19) | `actor`, `kind=workflow`, `skill_refs` (must resolve in `skill_registry`), `tags[]` | `boundary`/budget refs, `supervisor` notes |
| **Notification** (AB-14/AB-32) | `actor` (target), `kind=notification`, `time`, `tags[]` (severity) | provenance ref (what caused it) |
| **Artifact / board reference** (AB-24) | `actor`, `kind`, `tags[]`, provenance ref (thread post OR widget pointing at it) | source links (docs/PR/schema path), expiry |
| **Registered Account / Service Provider** (§7) | `actor` (`agentId`), `kind=agent-account`, `service_slug`, `service_type` (`inbox`\|`app`\|`service_provider`\|`dns`\|`oauth`\|`social`), `account_identifier`, `secret_refs[]` (`vault://agents/<agentId>/accounts/<service>/<field>`) | `linked_inbox_id`, `nango_connection_id`, `api_catalog_id`, `variables{}` |
| **Human / external-API item** — EXEMPT | `ts` + `traceId` only | everything above (attached lazily) |

---

## 7. Mandated Agent Account, App & Service Provider Registration

1. **Mandatory Registration of All Accounts:** Every account, login, or identity created or used by an agent — not only AgentMail inboxes, but also web apps, SaaS tools, DNS providers (e.g. FreeDNS), social platforms (Reddit, GitHub, X, Google), OAuth connections, and API service providers — **MUST** be registered in the JiMesh API (`POST /api/v1/agents/:agentId/accounts`).
2. **Deterministic Vault Secret References & Parseable Variables:** Passwords, tokens, recovery codes, API keys, and relevant account variables MUST NEVER be left in plain text in scratchpads. They MUST be registered with clear, deterministic secret IDs (`SECRET_<SERVICE>_<HANDLE>_PASSWORD`, `SECRET_<SERVICE>_<HANDLE>_API_KEY`) and canonical Vault paths (`vault://agents/<agentId>/accounts/<serviceSlug>/<field>`) alongside structured `variables` so any authorized session or workflow can look them up and parse them deterministically via `GET /api/v1/agents/:agentId/accounts/resolve?service=<slug>`.
3. **Separate Login & Auth Audit Trail:** Every website login or authentication challenge MUST be logged separately in the Agent Login & Auth Audit Log (`POST /api/v1/agents/:agentId/logins`) linked to the browser history entry and `traceId`.

---

## Agent summary (machine-readable)

```yaml
mandate: agent-metadata
binding: true
status: DRAFT_pending_operator_merge
rules:
  - id: 1   # AB-39
    must: attach_metadata_and_tags_when_possible
    who: agents_and_tools_only
    exempt: [humans, external_apis_lazy]
    violation: flagged_in_analytics_plus_optional_autotag_suggestions
    enforcement: schema_validators_not_prompt_nags
  - id: 2
    must: expose_search_and_filter_per_stored_item
    axes: [actor, kind, tag, media_type, time, thread]
    axes_composable: true
    filter_surface: simple_for_agents
  - id: 3   # AB-40
    must: paginate_every_list_surface
    surfaces: [agent_messages, threads, posts, notifications, registries, analytics]
    convention: one_page_size_defined_once_in_schemas
    forbidden: unbounded_lists
  - id: 4   # AB-41
    must: backend_GC_plus_pagination_plus_pubsub
    pubsub_for: viewed_data_only
    cold_rows: serve_via_pagination_never_stream_live
  - id: 5   # AB-38
    must: scroll_locator_api_for_chat
    default_scroll: bottom
    expose: [recent_titles_or_snippets, scroll_coordinates_per_message]
    filters: [thread, tag, actor, time_range]
baseline_required_on_every_item: [ts_ISO8601_UTC, traceId, audit_chain_ref]
```

## References

- `sprints/2026-11-06-agentboard/AGENTBOARD-BACKLOG.md` (AB-6/AB-10/AB-12–AB-17)
- `sprints/2026-11-06-agentboard/DECISIONS-01-and-ADDENDUM.md` (AB-38/39/40/41)
- `mandates/PACKAGE-ADOPTION.md` (trace/audit/vault clauses adopted by reference)
- `mandates/FRAME-IFRAME-RULE.md` (no dup patterns — one store, one convention)
- AGENTS.md §2 (`@jimesh/schemas` SSOT, `ts` + `traceId` on every wire message)
