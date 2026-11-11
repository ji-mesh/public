# AGENTS.md — rules for agents working in this repository

This repository (`ji-mesh/public`) is **public**. Everything committed here is
world-readable forever.

## 1. ZERO sensitive data — hard rule

**Never commit** any of the following, in any file (code, docs, commits, PR
descriptions, images):

- **Secrets**: API keys, tokens, passwords, private keys, session cookies,
  `vault://` payloads, `.env` contents, OAuth credentials/tokens.
- **Personal data (PII)**: private addresses, phone numbers, personal email
  addresses of team members, ID documents, billing data.
- **Internal infrastructure**: private/internal IP addresses, internal hostnames,
  VPN endpoints, non-public URLs, customer names or data.
- **Developer paths**: absolute home-directory paths (e.g. `/home/<user>/...`) —
  use `${HOME}/...` placeholders instead.
- **Chat/prompt logs**: raw internal prompt transcripts or unverified AI output.

Violations leak data irreversibly (git history). When in doubt: **don't commit it**.

## 2. What belongs here

- `legal/` — licenses, commercial terms, published ToS/usage research
  (published masters only — no internal annotations, no PII).
- `mandates/` — published copies of the JiMesh company mandates.
- `AGENTS.md` — this file.
- Install scripts, logos, public user docs (future).

## 3. Workflow

- The Impressum / team pages of the website are **not** stored here (contains
  PII by design) — they live with the website source only.
- Sanitize before pushing: run a grep for `password|token|key|/home/|@gmail|IP`
  patterns over your diff; redact anything that looks real.
- EN-only for docs and commits (AGENTS.md of `ji-mesh/jimeshOps` §3).
- Feature branch → PR → review → merge; never push straight to `main`.
