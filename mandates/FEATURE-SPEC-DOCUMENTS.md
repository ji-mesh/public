# FEATURE-SPEC-DOCUMENTS MANDATE (Operator Rule: 2026-11-03)

**Applies to every Feature and Core Pillar Milestone in the JiMesh project. Enforced across all AI sessions and dev teams.**

---

## 1. Concept: One Blueprint Document per Major Feature / Milestone

For every major feature or milestone track in the **`jiMesh`** Linear project (e.g., `Encrypted Docs Editor`, `Unified Chat & Helpdesk`, `CRM & Pipeline Engine`, `MeshCalendar`), a dedicated **Linear Document** must be created and linked directly to the project container.

Linear Documents provide an authoritative, reviewable, visual blueprint for operators, developers (Siddharth, Kushal), and AI agents before and during execution.

---

## 2. Mandatory Document Structure (SSOT Template)

Every Feature Specification Document in Linear must contain the following 5 sections:

### 1. Visual Design Reference
- Screenshot, Figma export, or Stitch mockup embedded at the very top (`![Design Reference](https://uploads.linear.app/...)`).
- Visual ground truth of what the UI layout, components, and panels look like.

### 2. Architectural Locations & File Paths
- Explicit mapping of which repository and files host the feature:
  - Public marketing surface (`ji-mesh/jimeshIO`)
  - Operator dashboard surface (`ji-mesh/jimesh2`)
  - Shared packages (`@jimesh/schemas`, `@jimesh/site-ui`)
  - Backend routes, gateways, and provisioning Helm charts

### 3. Core Functional Requirements & Behavioral Contract
- Explicit default view behavior (e.g., single-page code editor vs. split-view for Markdown/SVG).
- Actions, toolbar buttons, and triggers (Upload, Download, Split toggle, Git Diff).
- Fallback behaviors for binary files, non-previewable formats, and edge cases.
- Package-first doctrine (`mandates/FRAME-IFRAME-RULE.md`): genuine packages and components only, zero iframe slop.

### 4. Security & Cryptographic Boundaries
- Storage encryption mechanisms (e.g., AES-256-GCM envelope encryption into Apache Ozone).
- Secret injection rules (`Vault-only`, `mandates/PACKAGE-ADOPTION.md`).
- Isolation boundaries (gVisor container sandbox, OPA/Rego capability checks).
- Audit trail logging and cryptographic hash verification.

### 5. Roadmap, Issue Wiring & Deliverables
- Status (`Draft`, `In Progress`, `Shipped / Done`).
- Direct links to associated Linear issues (e.g., `DEV-134`, `DEV-135`, `DEV-130`).
- Git commit hashes and PR links verifying deployment and test results.

---

## 3. Synchronization Rule (Repo ↔ Linear)

- **Repo File as SSOT:** Every Linear Document must mirror a markdown document committed to `docs/specs/<FEATURE>-SPEC.md` in `jimeshOps`.
- **Direction:** Commit to repo first → sync to Linear Document via Linear GraphQL API (`documentUpdate` / `documentCreate`).
- **Never Out of Sync:** When acceptance criteria or layouts change, both the repo file and the Linear Document are updated together.

---

## 4. Current Feature Specification Documents

| Feature | Repo SSOT File | Linear Document Link |
|---|---|---|
| **Encrypted Docs Editor** | `docs/specs/ENCRYPTED-DOCS-EDITOR-SPEC.md` | [Encrypted Docs Editor (d0bbb48a4749)](https://linear.app/jimesh/document/encrypted-docs-editor-d0bbb48a4749) |
| **Unified Helpdesk & Chat** | `docs/specs/UNIFIED-HELPDESK-SPEC.md` | [Helpdesk Architecture Spec (5d9a7b36b011)](https://linear.app/jimesh/document/unified-helpdesk-and-agent-chat-engine-architecture-spec-5d9a7b36b011) |
| **MeshCalendar Lane** | `docs/specs/MESHCALENDAR-SPEC.md` | (in flight DEV-130) |

---
## End of Mandate.
