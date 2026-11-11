# FRAME / IFRAME RULE — Mandate (operator 10-31: "write it as a mandate")

## Ground rule (binding for all website lanes, demos, and dashboard components)

1. **No important feature hides inside an iframe.** Everything interactive (files/editor, git diff/history, CRM kanban, helpdesk inbox, vault/audit, calendar triggers, console) lands as a real component in the dashboard (jimesh2) or as a React component in `site-ui` — never as a frozen HTML iframe where events are reachable only through extra clicks.
2. **Iframes are allowed only for animated/pure-visual HTML elements** — visualization without hidden features (e.g. carousel slides, canvas animations). If an iframe is used anyway: minimized — one choice — media/"view fullscreen" CTA in the card, no scenery markup, no second file browser, no duplicate sidebars.
3. **Single frames, divs or html elements stacked** — multiple views (e.g. editor+git) live in ONE split pane: browser/preview left/top, editor center, git/changelog right — no throttle duplication (no second code editor, no duplicate file-browser design, no duplicate sidebar mock).
4. **Unified design (DEV-119)**: token sheet from the demo series + all old demos adopt those tokens.
5. **Real packages instead of slop** (see PACKAGE-ADOPTION.md): ra-core (CRM), chatwoot (helpdesk), open-pencil (scene-graph), monaco, mui-x-grid, react-player, payload (CMS), xyflow.
6. **Execution gate before delivery**: page review in the planner (DEV-118) — any 'iframe with hidden feature' finding gets NO until the work type was replaced (React component or adopted package).

---

# MANDATE: LINEAR ↔ REPO DOC-SYNC (operator 11-01: "all docs in linear must also be in the repo, in a clear structure")

**Every Linear document mirrors one REPO file — 1:1, fixed structure:**

## Structure (binding)

```
jimeshOps/
├── AGENTS.md                      # Entry point (required reading + rules)
├── docs/INDEX.md                  # Master index (everything hangs there)
├── mandates/                      # Binding rules (package-adoption, frame-rule, dev-process)
├── schemas/                       # @jimesh/schemas SSOT (TS + py + proto + sql + linear)
│   └── src/…
└── docs/development/
    ├── process/                   # How we work (EN!): docstring-and-apidocs, deliverables-and-scope, templates/
    ├── research/                  # Lane research (one file per lane)
    ├── security/                  # Key rotation + security audits
    ├── sprints/                   # Sprint plans + gap analyses
    └── handoffs/                  # Prompt exports + handovers
```

## Sync rules

1. **One Linear document = exactly one repo file** (path linked in the doc header). No duplicates with drift.
2. Change → always FIRST in the repo (commit), THEN `documentUpdate` into the Linear document (this direction, never reverse).
3. New docs category → folder in repo first, then INDEX.md entry, then Linear document.
4. **Language:** EN (team in India) — only operator-facing UI copy may be DE.

## For Jules prompts

Jules reads the repo files (not the Linear docs) — the Linear docs are the readable review layer for the devs. Jules PRs reference the file paths.

---

# MANDATE: MILESTONE / PROJECT UPDATE RULES (operator 01.11: "be wichtigen updates zu milestones projekt updates schreiben und die changes refrenzieren")

**Every state change in Linear (milestone, issue, project) must be written as an UPDATE with references:**

1. **Milestone update (any state change):** reference the milestone name, the PR link, the commit sha + repo, the linked issue IDs, the docs that changed, the package(s) used.
2. **Issue update:** reference the milestone it belongs to + the related issues + the PR + the docs.
3. **Project update:** reference the milestones affected + what changed + the linked issues.
4. **Never silent:** if a feature is blocked/scope-changes/gets-done — WRITE the update in Linear. No empty milestones. No silent scope changes.
5. **Template** has been provided in the linear-project skill (see §9 for the format).

## End of mandate update.
