# ToS & Usage Rules — Engine Wrappers (OpenCode · AGY · Camoufox)

> **Purpose:** Technical/compliance orientation for the JiMesh CLI wrapper / provider dashboard.
> It states, per engine, what our wrapper may and may not do, which source backs each rule, and the
> operator-facing warning strings the dashboard must show in its account-management section.
>
> **Date:** 2026-11-11 (all sources retrieved 2026-11-11)
> **Related docs (same directory):** `README.md`, `provider-auth-and-quota.md`, `agy-protocol.md`, `opencode-v2-api.md`
> **Language:** EN master strings. Operator-facing UI copy may be localized to German later — keep this file EN-only.
>
> **Disclaimer:** This document is **not legal advice**. It is a technical/compliance orientation compiled from
> publicly accessible vendor pages, with exact quotes marked as such. Where a claim could not be verified against
> a fetched source it is explicitly marked *unverified* or *community-reported* (see §6). Vendor terms change;
> re-verify the URLs in §5 before relying on a quote.

---

## §1 Per-technology rules

### 1.1 OpenCode (V2, `opencode serve` by Anomaly) — our own engine

**Governing documents**

- OpenCode **Terms of Use**, effective Nov 8, 2026: <https://opencode.ai/legal/terms-of-service>
- OpenCode **Legal index** (Privacy Policy, Terms, MIT license pointer): <https://opencode.ai/legal>
- **MIT License** of the open-source repository: <https://github.com/anomalyco/opencode/blob/dev/LICENSE>

**What is allowed (verified)**

- Self-hosting / running your own instances is governed by the repository's open-source license, not by the hosted-services terms. Exact quotes:
  > "These Terms cover the website, the inference product and the hosted software. Open source software you run yourself is governed by its own license." *(Terms of Use, Summary)*
  > "our open source software that is not provided to you on a hosted basis is subject to the open source license and terms set forth on the applicable repository where you access such open source software, and such license and terms will exclusively govern your use of such open source software." *(Terms of Use, introduction)*
  - The repository license is the **MIT License** ("Permission is hereby granted, free of charge … to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software"). Verified from the raw `LICENSE` file of `anomalyco/opencode`.
- **No clause was found that prohibits running your own OpenCode server/instances.** The Terms restrict misuse of the *hosted* Services; self-hosted use falls under MIT. (Absence-of-clause statement based on a full read of the fetched Terms of Use, 2026-11-11.)
- Programmatic use of the product (SDK / `opencode serve`) is an intended use case: the official site states "Drive OpenCode from your own scripts, CI jobs and tools" (SDK) and the Console docs describe "Harnesses for using Console models from Claude Code, Codex, and other agents". <https://opencode.ai/docs/console/>
- Bringing your own provider keys is by design: "Your ability to access Third Party Models is contingent on you having API keys or otherwise having the right to access such Third Party Models." *(Terms of Use, Third Party Models)*

**What is prohibited / restricted (verified)**

- Bulk or limit-circumventing accounts — Terms of Use restriction #4:
  > "creates, maintains, or uses accounts in bulk, or creates, maintains, or uses multiple accounts to circumvent usage limits, access restrictions, billing obligations, promotions, suspensions, or any other restriction or policy applicable to the Services"
- Programmatic extraction from the hosted Services — restriction #5: "automatically or programmatically extracts data or Output (defined below)". Enforcement: "A violation of any of the foregoing is grounds for termination of your right to use or access the Services."
- Internal-use framing: "You will only use the Services for your own internal use, and not on behalf of or for the benefit of any third party, and only in a manner that complies with all laws that apply to you."
- Note: restrictions #4/#5/#10/#11 target the **hosted Services** (website, inference product, hosted software) — not MIT-licensed self-hosted code. Reselling or embedding *Anomaly's hosted* inference as a customer-facing feature would fall under the Terms; running our own instance does not.

**OpenCode Console (Zen/Go) API keys**

- The Console is "an optional service that provides additional benefits to using OpenCode, on your own or as a team" (`/docs/console/`). Subscriptions: OpenCode Go $10/month, Go Plus $40/month (opencode.ai homepage, 2026-11-11). Console API keys are governed by the same Terms of Use (hosted Services) + Privacy Policy; no separate API-key terms page was found on <https://opencode.ai/legal>.
- **Dashboard stance:** use Console API keys as ordinary BYOK credentials; they are quota-bearing secrets → vault-only (see `mandates/PACKAGE-ADOPTION.md`), never logged.

**Bottom line for our wrapper:** OpenCode is our own engine; self-hosted `opencode serve` is unrestricted by anything we found. The only OpenCode-related ToS risk is (a) abusing Anomaly's hosted Console/inference with pooled accounts, and (b) **using OpenCode as a client for the Antigravity login** — which Google's FAQ names explicitly (see §1.2).

---

### 1.2 AGY (Google Antigravity CLI, official `agy` binary)

**Governing documents**

- Google Antigravity **Additional Terms of Service**: <https://antigravity.google/terms/> (incorporates the Google Universal Terms, <https://policies.google.com/terms>)
- Antigravity **FAQ**: <https://antigravity.google/docs/faq/>
- Antigravity **Plans / quota**: <https://antigravity.google/docs/plans/>
- Official binary download: <https://antigravity.google/download>
- Official forum (linked from the Antigravity Terms as the support channel): <https://discuss.ai.google.dev/c/antigravity/64>

**What is prohibited (exact quotes, verified 2026-11-11)**

- **Additional Terms, Section 6** — third-party access is a breach:
  > "You must not abuse, harm, interfere with, or disrupt the Service. This includes, but is not limited to, using the Service in connection with products not provided by us. Using third party software, tools, or services to access the Service (e.g. using OpenClaw with Antigravity OAuth) is a breach of this Agreement. Such actions may be grounds for suspension or termination of your Antigravity and/or Gemini CLI accounts."
- **FAQ** — third-party tools, and *OpenCode is named in the question heading* ("Why can't I use third-party software (such as Claude Code, OpenClaw, or OpenCode) with my Antigravity login?"):
  > "Using third-party software, tools, or services to access Antigravity is a violation of our Terms of Service and severely degrades the experience for legitimate product users. Such actions can result in suspension or termination of your account. To use a third-party coding agent with Gemini, we recommend using a Gemini Enterprise or Google AI Studio API key."
- **FAQ** — headless wrapping and account rotation:
  > "Antigravity CLI headless mode (`agy -p`) is intended for direct scripting and CI workflows. Wrapping `agy` inside third-party coding agents or applications, or automating account rotation, is not supported under consumer plans."
- **Google Universal Terms** (apply via the Antigravity Terms §1): "You must not abuse, harm, interfere with, or disrupt our services or systems" — abuse list includes "spamming, hacking, or bypassing our systems or protective measures" and "hiding or misrepresenting who you are in order to violate these terms". Suspension clause: "Google may suspend or terminate your access to the services or delete your Google Account if any of these things happen: - you materially or repeatedly breach these terms, service-specific additional terms, or policies".

**What is allowed / tolerated (verified, with caveats)**

- **Manual switching between your own active, paid accounts** to manage token usage — official forum reply by @Siddharth_Naik (2026-07-15):
  > "While we cannot share the exact details of our abuse detection protocol, switching between two active, paid accounts to manage token usage is not considered a violation of our Terms of Service."
  > "However, please keep in mind that issues often arise when users attempt to use secondary accounts that are part of a discounted or shared pool. To ensure uninterrupted access and avoid triggering any system flags, please make sure you are only using paid accounts that you personally own."
  - *Caveat:* forum reply, not contract text; the responder's staff badge was not independently verified (the wording "our abuse detection protocol" indicates team affiliation). A later question in the same thread (2026-10-28) about a *personal tool* calling Antigravity and *automatic* switching was **still unanswered** at retrieval — treat automated switching as unresolved/risky.
- **Headless `agy -p` for direct scripting and CI** is the stated intended use (FAQ quote above) — the line is crossed when it is wrapped inside third-party coding agents/applications or when account rotation is automated.
- **Third-party coding agents should use API keys instead:** "we recommend using a Gemini Enterprise or Google AI Studio API key" (FAQ quote above). This is Google's own compliant pattern for wrappers/SaaS.

**Quota model (verified, Plans page)**

- Google AI Ultra: "The highest, most generous quota, refreshed every five hours" + "The highest weekly rate limits".
- Google AI Pro: "A high, generous quota, refreshed every five hours until the weekly limit is reached" + "A higher weekly rate limit".
- Free/other: "A meaningful quota, refreshed weekly" + "A weekly rate limit".
- "The baseline rate limits are primarily determined by available capacity and exist to prevent abuse."
- Overage: "Users on Google AI Pro or Ultra plans can use purchased AI credits … for additional overage usage above the baseline quota." (user setting: *Never* / *Always*).
- **No BYOK on consumer Antigravity:** "There is currently no support for the following options: - Bring-your-own-key (BYOK) or bring-your-own-endpoint for additional rate limits".

**Multi-account language**

- The often-quoted exact phrase *"using multiple accounts to circumvent service limits"* could **not** be found verbatim on any official Google page fetched on 2026-11-11 — it circulates in community/forum posts as a paraphrase. *Unverified as an official quote.*
- The closest **verified** official analogue is Google's **Circumvention** program policy (<https://support.google.com/docs/answer/148505>):
  > "Do not engage in actions intended to bypass our policies, subvert restrictions placed on your account, or distribute content removed in other Google products."
  > "Creating or using multiple accounts or other methods intended to engage in a behavior that was previously prohibited."
  - *Scope caveat:* that page states "The program policies below apply to Drive, Docs, Sheets, Slides, Forms, Vids, Sites, and Pics" — it is not the Antigravity contract. Use it as evidence of Google's general posture, not as the Antigravity rule.

**Bottom line for our wrapper:** official `agy` binary only, driven by the user's own login on their own machine; no third-party OAuth client (OpenCode + Antigravity OAuth = named violation), no token extraction, no automated account rotation. For any commercial/multi-user product surface, route via **Gemini API key / Gemini Enterprise (Google Cloud)** as the FAQ recommends — *not* via consumer Antigravity OAuth. ("Vertex AI" is Google Cloud's enterprise model-API platform and is commonly used for this; the FAQ wording verified here is "Gemini Enterprise or Google AI Studio API key" — Vertex AI itself was not quoted from the fetched pages.)

---

### 1.3 Camoufox (anti-detect Firefox fork)

**Governing documents**

- Official repository: <https://github.com/daijro/camoufox> (the earlier lead "gcamoufox/camoufox" / "camoufox.io" was wrong — correct is **daijro/camoufox**, site **camoufox.com**)
- Official docs site: <https://camoufox.com/>
- License: **MPL-2.0** (shown as the repo license badge; `LICENSE` file present in the repository)

**What it is (verified quotes)**

- "Camoufox is an open source anti-detect browser built for webscraping & AI agents. 🦊" *(repository README)*
- "Camoufox is a Firefox build for web scraping and AI agents. Every launch gets a fresh, internally consistent device identity … and the browser reports it from C++, so a page sees native values rather than JavaScript overrides." *(repository README, Introduction)*
- "Playwright drives it, through an automation layer the page cannot see." *(repository README)*
- Site: "An open source anti-detect browser built for AI agents. 🦊" with the feature "Undetectable by design 🎭" and "Human-like mouse movement 🖱️" *(camoufox.com)*
- Stability caveat (quote): "⚠️ This project is under development. It may not be suitable for stable production use. ⚠️" *(repository README)*
- Proxy intent (quote): "Camoufox is intended to be used with rotating proxies (preferably residential IPs)." *(repository README)*

**License terms**

- **MPL-2.0**: copyleft for modified *files*; using/combining it in a larger product is allowed as long as modified MPL-licensed files stay MPL and the license notice is preserved. No usage/AUP restrictions beyond the license text were found in the repository.
- **No terms-of-service / acceptable-use page was found** for Camoufox as of 2026-11-11 (repo, README, camoufox.com all lack one). The license is therefore the only binding document from the vendor.

**What is allowed**

- Using it as an automation browser for scraping, testing and AI-agent workflows (its stated purpose), including headless server use and Playwright-driven sessions.

**What is prohibited / out of scope (our rules, plus source backing)**

- **Never at logins/OAuth flows** — our own JiMesh rule (see `provider-auth-and-quota.md` and §2). The vendor makes no exception for account logins; fingerprint rotation at a login screen directly engages the target site's account-security controls.
- **Target-site ToS compliance is the operator's/user's responsibility** — no source in Camoufox's own docs addresses third-party site terms, so this is assigned to the user by us, in our AUP.
- Vendor's own framing ("anti-detect", "undetectable by design", proxy-oriented) means large-scale automated access can look like bot traffic to target sites → keep volumes humane and stay inside each target site's rules (our rule; not a vendor quote).

---

## §2 The compliant wrapper stance (our position)

Each rule → the source that backs it. Rules marked *(internal)* are JiMesh engineering/policy controls with no external vendor quote; they are enforced by us regardless.

| # | Rule | Backing source |
|---|------|----------------|
| R1 | **Official binaries only.** Install `agy` and `opencode` from their official channels (`antigravity.google/download`, `opencode.ai` install); never patched/repacked builds. | Antigravity Terms §6 ("using the Service in connection with products not provided by us"); Antigravity FAQ (third-party software = violation) |
| R2 | **No token/credential extraction.** Never read out, export or replay `oauth_creds.json` or any OAuth token file between machines or profiles. | Antigravity Terms §6 (third-party access via Antigravity OAuth = breach → suspension/termination); Google Universal Terms ("You're responsible for what you do with your Google Account, including taking reasonable steps to keep your Google Account secure") *(security expectation)* |
| R3 | **No third-party OAuth client.** Do not wire OpenCode (or any other agent) to the Antigravity login. AGY runs only as the official binary with its own sign-in. | Antigravity FAQ — OpenCode named explicitly in the question heading; Terms §6 example ("using OpenClaw with Antigravity OAuth") |
| R4 | **Profile isolation via `HOME` / `XDG_*`.** One profile = one isolated environment directory (HOME, XDG_CONFIG_HOME, XDG_DATA_HOME, XDG_CACHE_HOME, XDG_STATE_HOME). | *(internal)* Prevents credential bleed; aligns with R2 and with Google's account-security expectation |
| R5 | **Manual profile switching only.** Switching is a user action in the dashboard; never automatic. | Forum reply: manual switching between own paid accounts is not a violation; FAQ: "automating account rotation" is not supported under consumer plans |
| R6 | **Single active profile per user.** Exactly one AGY profile active at a time; no pools, no shared/discounted accounts. | Forum reply: "make sure you are only using paid accounts that you personally own"; Google Circumvention policy (multiple accounts to engage in previously prohibited behavior); OpenCode ToS restriction #4 (same principle on our own hosted surface) |
| R7 | **No auto-fallback on quota errors.** On HTTP 429 / `RESOURCE_EXHAUSTED` the task must **fail or pause** — the user clicks to continue or switches profile manually. Never silently rotate to another account. | FAQ: "automating account rotation" not supported under consumer plans; Terms §6 (abuse/suspension risk); OpenCode ToS restriction #4 (circumventing usage limits) |
| R8 | **AGY in our wrapper = personal, local, direct-scripting use.** Headless `agy -p` only as the FAQ intends (direct scripting/CI on the user's own machine); no wrapping into a hosted multi-tenant agent service. | FAQ headless quote; Terms §6; FAQ recommendation to use an API key for third-party coding agents |
| R9 | **Commercial/multi-user surfaces use API keys** (Gemini API key / Gemini Enterprise on Google Cloud; OpenCode Console/API keys or provider BYOK; never consumer AGY OAuth). | FAQ: "we recommend using a Gemini Enterprise or Google AI Studio API key"; Antigravity Plans: no BYOK on consumer plans; OpenCode Terms: access to third-party models "contingent on you having API keys" |
| R10 | **No credential copying between instances** (no `oauth_creds.json` injection, no profile cloning of auth state). | R2 backing; *(internal)* token-hijacking/abuse-detection signal per forum warning about flagged account patterns |
| R11 | **Camoufox never at login/OAuth/checkout flows** of target sites. | *(internal JiMesh rule)* — vendor docs contain no exception; anti-detect use at logins engages target-site account controls |
| R12 | **Our own AUP forbids account pooling/cycling** for JiMesh customers. | OpenCode ToS restriction #4 as the model clause; Google Circumvention policy; forum warning about "discounted or shared pool" accounts |

---

## §3 Multi-user / SaaS posture

1. **Per-user isolation.** One user = one isolated runner environment (own `HOME`/`XDG_*`, own profile dir, own OAuth callback port). No cross-user sharing of profiles, tokens or browser storage. *(internal control; supports R2/R4/R6)*
2. **Shared-IP risk.** Many users' AGY traffic exiting one datacenter IP can pattern-match bot/abuse traffic at scale. **Mitigation:** teams and multi-tenant features go through **API keys (Gemini API / Gemini Enterprise, OpenCode Console, provider BYOK)** — Google's FAQ explicitly recommends an API key for third-party coding agents, and consumer Antigravity has no BYOK (Plans page). *(IP-reputation reasoning is engineering judgment, not a quoted vendor statement.)*
3. **OAuth callback port isolation.** Each concurrent local OAuth flow must bind a distinct localhost callback port (and distinct state/PKCE verifier) so simultaneous logins cannot collide or cross-deliver codes. *(internal control)*
4. **Cloud-SaaS vs BYOC split.**
   - **Cloud-SaaS:** only API-key-based engines (OpenCode Console keys, Gemini API/Gemini Enterprise keys, other provider keys as BYOK). No consumer OAuth profiles in the cloud tier.
   - **On-Prem / BYOC local runner:** the customer's own machine runs the official binaries with their own profiles under R1–R11; JiMesh never receives or copies their tokens.
   - **BYOK = customer responsibility:** the customer warrants their keys/quotas; JiMesh does not pool, share or rotate them. *(Terms backstop: OpenCode Terms #4, Google Circumvention policy, forum "personally own" wording)*
5. **No auto-retry/rotation on quota errors in any tier** (R7). Paused tasks surface a dashboard warning (§4) instead of silently hopping accounts.

---

## §4 Dashboard account-management section — terms & warning copy

EN master strings for the provider dashboard. IDs are stable; body text is plain language, 2–4 sentences.
Operator-facing UI may be localized to German later — this file stays EN.

### Info-box strings (shown when adding/editing a profile)

**`warn.agy.tos`** — shown when adding an AGY profile
- **Title:** Google Antigravity terms apply
- **Body:** Accessing Antigravity through third-party software or tools is a breach of Google's Additional Terms and can suspend or terminate your Antigravity and/or Gemini CLI accounts. This dashboard therefore drives the official `agy` binary only, with its own sign-in on your machine. Read the terms before you continue.
- **Sources:** <https://antigravity.google/terms/> · <https://antigravity.google/docs/faq/>

**`warn.agy.no-cycling`** — shown on the profile list / quota screen
- **Title:** No account cycling
- **Body:** Never rotate automatically between accounts to dodge quota limits — automating account rotation is not supported on consumer plans, and Google's circumvention policy prohibits using multiple accounts to bypass restrictions. On a quota error this dashboard pauses the task and waits for you. Use only paid accounts that you personally own.
- **Sources:** <https://antigravity.google/docs/faq/> · <https://discuss.ai.google.dev/t/is-switching-google-antigravity-accounts-a-tos-violation/174378> · <https://support.google.com/docs/answer/148505>

**`warn.agy.headless-wrap`** — shown when a job schedules `agy -p`
- **Title:** Headless mode is for your own scripts and CI
- **Body:** `agy -p` is intended for direct scripting and CI workflows on your own machine. Wrapping it inside third-party agents or applications, or letting software switch accounts for you, is not supported on consumer plans. For product or team usage, switch to a Gemini API key.
- **Sources:** <https://antigravity.google/docs/faq/>

**`warn.oauth.user-browser-only`** — shown in any OAuth sign-in flow
- **Title:** Sign in yourself, in your own browser
- **Body:** Complete Google sign-in manually in your own browser on this machine. Never paste, import or copy OAuth token files between profiles or hosts — moving credentials around is treated as third-party access and can get the account suspended.
- **Sources:** <https://antigravity.google/terms/> (§6) · <https://antigravity.google/docs/faq/>

**`warn.profile.single-active`** — shown when activating a second profile
- **Title:** One active profile at a time
- **Body:** Keep exactly one AGY profile active per user. Switching between two of your own paid accounts manually is fine, but pooled, shared or discounted accounts trigger abuse flags. The dashboard blocks simultaneous activation to protect your accounts.
- **Sources:** <https://discuss.ai.google.dev/t/is-switching-google-antigravity-accounts-a-tos-violation/174378>

**`warn.quota.no-auto-fallback`** — shown when a task pauses on 429 / `RESOURCE_EXHAUSTED`
- **Title:** Quota reached — paused, not rotated
- **Body:** This task stopped because the provider returned a quota error. The dashboard will not switch accounts for you; wait for the refresh (5-hour/weekly on Antigravity plans) or switch profile manually with a click. Your API-key based engines are unaffected.
- **Sources:** <https://antigravity.google/docs/plans/> · <https://antigravity.google/docs/faq/>

**`warn.camoufox.no-logins`** — shown when a browser flow targets a login/OAuth page
- **Title:** No anti-detect browser at sign-in
- **Body:** Camoufox must never be used for logins, OAuth consent screens or checkout flows — our rule, so credentials are only ever entered by a human in a normal browser. Camoufox is for scraping, testing and agent workflows on public pages. The Terms of each website you visit are your responsibility.
- **Sources:** <https://github.com/daijro/camoufox> · <https://camoufox.com/> *(rule itself is JiMesh policy; no vendor page addresses logins)*

**`warn.saas.byok-responsibility`** — shown on the team/SaaS key settings page
- **Title:** Your keys, your responsibility
- **Body:** API keys you connect (BYOK) belong to you: their quotas, billing and ToS compliance are your responsibility. JiMesh never pools, shares or rotates keys between customers, and cloud features use API keys only — never shared consumer logins. Do not connect keys from accounts you do not personally own.
- **Sources:** <https://opencode.ai/legal/terms-of-service> · <https://discuss.ai.google.dev/t/is-switching-google-antigravity-accounts-a-tos-violation/174378>

### Link strings (footer / "Legal" drawer of the account section)

- **`link.tos.antigravity`** — label: "Google Antigravity Additional Terms" → <https://antigravity.google/terms/>
- **`link.faq.antigravity`** — label: "Antigravity FAQ (third-party tools & headless mode)" → <https://antigravity.google/docs/faq/>
- **`link.plans.antigravity`** — label: "Antigravity Plans & quotas" → <https://antigravity.google/docs/plans/>
- **`link.tos.google`** — label: "Google Terms of Service" → <https://policies.google.com/terms>
- **`link.forum.account-switching`** — label: "Forum: switching accounts & token usage" → <https://discuss.ai.google.dev/t/is-switching-google-antigravity-accounts-a-tos-violation/174378>
- **`link.tos.opencode`** — label: "OpenCode Terms of Use" → <https://opencode.ai/legal/terms-of-service>
- **`link.legal.opencode`** — label: "OpenCode Legal (Privacy, Terms, license)" → <https://opencode.ai/legal>
- **`link.camoufox.license`** — label: "Camoufox license (MPL-2.0)" → <https://github.com/daijro/camoufox>
- **`link.aup.jimesh`** — label: "JiMesh Acceptable Use (no account pooling)" → *(internal AUP page — add when published; the clause to publish: "Creating, maintaining or using multiple accounts to circumvent usage limits is prohibited.")*

---

## §5 Source table

| # | URL | What was verified from it (2026-11-11) | Status |
|---|-----|----------------------------------------|--------|
| S1 | <https://antigravity.google/terms/> | §6 third-party-software breach quote; suspension/termination clause; Agreement composition (Universal Terms + Antigravity Terms + Privacy Policy) | Fetched ✓ |
| S2 | <https://antigravity.google/docs/faq/> | Third-party-tools violation + "Gemini Enterprise or Google AI Studio API key" recommendation; `agy -p` headless/wrapping/rotation quote; OpenCode named in FAQ heading | Fetched ✓ |
| S3 | <https://antigravity.google/docs/plans/> | 5-hour refresh (Pro/Ultra), weekly limits, AI-credit overage, "no BYOK" statement, "exist to prevent abuse" | Fetched ✓ |
| S4 | <https://discuss.ai.google.dev/t/is-switching-google-antigravity-accounts-a-tos-violation/174378> | Staff-style reply: manual switching between own paid accounts not a violation; shared-pool warning; 2026-10-28 follow-up question unanswered | Fetched ✓ (forum, not contract) |
| S5 | <https://policies.google.com/terms> | "You must not abuse, harm, interfere with, or disrupt our services or systems"; abuse examples; suspension/termination conditions; account-security responsibility | Fetched ✓ |
| S6 | <https://support.google.com/docs/answer/148505> | "Circumvention" policy: bypassing restrictions + "Creating or using multiple accounts …"; scope = Docs Editors/Drive family (caveat noted) | Fetched ✓ |
| S7 | <https://opencode.ai/legal> | Legal index: Privacy Policy, Terms of Use links, "Open source license (MIT)" | Fetched ✓ |
| S8 | <https://opencode.ai/legal/terms-of-service> | Effective Nov 8 2026; self-hosted = own license; restriction #4 (multiple accounts to circumvent usage limits); restriction #5 (programmatic extraction); internal-use clause; third-party models require API keys; termination clause | Fetched ✓ |
| S9 | <https://opencode.ai/docs/console/> | Console = optional service; harnesses for Claude Code/Codex/other agents; Go/Go Plus/Go Teams subscriptions | Fetched ✓ |
| S10 | <https://raw.githubusercontent.com/anomalyco/opencode/dev/LICENSE> | MIT License text ("use, copy, modify, merge, publish, distribute, sublicense, and/or sell") | Fetched ✓ |
| S11 | <https://github.com/daijro/camoufox> | Repo identity (daijro/camoufox, not gcamoufox), MPL-2.0 badge, README quotes (anti-detect browser for webscraping & AI agents; under-development warning; rotating-proxies intent; Playwright/Juggler automation hiding) | Fetched ✓ |
| S12 | <https://camoufox.com/> | Official docs site; "open source anti-detect browser built for AI agents"; "Undetectable by design"; no terms/AUP page present | Fetched ✓ |
| S13 | <https://antigravity.google/download> | Official binary download entry point (site navigation link) | Link verified in fetched nav (not opened) |
| S14 | <https://opencode.ai/> | SDK "Drive OpenCode from your own scripts, CI jobs and tools"; Go $10 / Go Plus $40; Legal footer link | Fetched ✓ |

---

## §6 Claims downgraded to *unverified* / community-reported

1. **"Google policy prohibits 'using multiple accounts to circumvent service limits'"** — the exact phrase was **not** found on any official Google page fetched (2026-11-11). It appears in community/forum posts as a paraphrase. Verified official analogue: the Circumvention policy (S6), whose stated scope is the Docs Editors/Drive family. → Treat the exact wording as *community-reported paraphrase*.
2. **"Wrapping `agy -p` … pure local personal use is low-risk"** — the FAQ states the headless/wrapping rule (S2); the "low-risk" assessment for purely local personal use is *our own risk judgment*, not a Google statement.
3. **"Official Google forum clarification" (S4)** — the forum `discuss.ai.google.dev` is linked by Google's own Terms as the support channel, but the individual reply is a forum post; the responder's staff badge was not independently verified. *Forum guidance, not contract text.*
4. **"Commercial SaaS should route via GCP / Vertex AI"** — the verified FAQ wording is "Gemini Enterprise or Google AI Studio API key". "Vertex AI" as the Google Cloud route is general knowledge, *not quoted* from the fetched pages.
5. **Profile isolation via HOME/XDG, OAuth port isolation, no `oauth_creds.json` copying, shared-IP bot-pattern risk, BYOK split** — JiMesh engineering/policy controls *(internal)*; the vendor-side backing (third-party-access breach, account-ownership warning, API-key recommendation) is quoted above, but the specific mechanisms are ours.
6. **Camoufox login prohibition** — JiMesh's own rule; Camoufox's vendor pages contain no statement about logins/OAuth flows or target-site ToS. No vendor ToS/AUP exists to quote.
7. **OpenCode "no restrictions on running your own instances"** — an absence-of-clause finding from a full read of the fetched Terms (S8): self-hosted use is governed by the MIT license (S10). Absence of a prohibition is not an affirmative grant beyond the license.

*End of document. Modify only via a new revision; re-verify §5 URLs before quoting externally.*
