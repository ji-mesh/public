# WORKSPACE-RULE MANDATE (operator 01.11: "ab jetzt nur noch in ji-mesh orga arbeiten — nach finaler migration")

**Binding since 2026-11-01. Mirrors AGENTS.md §7. Applies to every agent (Jules / OpenCode / dev) and every repo.**

## 1. The rule

**ALL work happens ONLY in the `ji-mesh` GitHub org. No exceptions.**

| Repo | Role |
|---|---|
| `ji-mesh/jimeshOps` | SSOT: docs + mandates + schemas + linear bridges |
| `ji-mesh/jimesh2` | dashboard + backend |
| `ji-mesh/jimeshIO` | website + widgets |
| `ji-mesh/jimesh-docs` | articles + dev guides (private org mirror) |
| `ji-mesh/jskills` | skills + bridges (jules-linear.py, jules.sh) |
| `ji-mesh/ad1` | calendar lane audit (private fork) |

**NEVER work in `ji-podhead` repos**: no pushes, no clones, no PRs, no issues there. ji-podhead is the legacy personal namespace — the org copies are the single source of work.

## 2. Local working copies

1. Every local tree must track the `ji-mesh` remote (`origin` or explicit `jimesh` remote).
2. If a local tree still points at ji-podhead — add ji-mesh as remote and force-reset to it:
   ```bash
   git remote add jimesh https://github.com/ji-mesh/<repo>.git   # or: git remote set-url
   git fetch jimesh && git reset --hard jimesh/main
   ```
3. Fresh clone path (agents should use HTTPS; SSH if keys exist):
   ```bash
   git clone https://github.com/ji-mesh/<repo>.git && cd <repo>
   ```

## 3. Submodule URLs (jimeshOps `source/`)

`.gitmodules` in `ji-mesh/jimeshOps` must point **every** submodule URL at the ji-mesh org, and every gitlink pin must be a **current ji-mesh HEAD** — never a ji-podhead sha (those shas do not exist in the org copies → `submodule-not-found` in fresh Jules clones):

- `source/jimesh2` → `https://github.com/ji-mesh/JiMesh2.git`
- `source/jimeshIO` → `https://github.com/ji-mesh/jimeshIO.git`
- `source/jimesh-docs` → `https://github.com/ji-mesh/jimesh-docs.git`
- `source/jskills` → `https://github.com/ji-mesh/jskills.git`

Pin method (operator prescription): `git update-index --cacheinfo 160000,<ji-mesh-sha>,source/<sub>`.

⚠️ **Never `git add source/` while submodule worktrees lag behind their ji-mesh HEADs** — `git add` stages the WORKTREE gitlink and silently stomps the corrected pin (happened 01.11, commit 21d20a3: jimesh2 pin reverted to a ji-podhead sha → "upload-pack: not our ref" on every fresh clone). Always `update-index` → VERIFY the sha exists in the sub-repo (`git -C <sub> cat-file -t <sha>` must print commit) → `git ls-files -s source/` → plain `git commit` (no pathspec). A sha from the WRONG repo silently poisons fresh clones (happened 01.11: 5e2d106 = jimeshOps sha pinned as jimesh2).

## 4. Migration status (2026-11-01, verified)

| Repo | ji-mesh org repo | `.gitmodules` URL | Pin vs ji-mesh HEAD |
|---|---|---|---|
| jskills | ✅ | ✅ ji-mesh | ⚠️ stale → `03aed83` |
| jimeshIO | ✅ (HEAD `b0db0ac`) | ❌ ji-podhead | ⚠️ stale → `b0db0ac` |
| JiMesh2 | ✅ (HEAD `6d0fd12`) | ❌ ji-podhead | ⚠️ stale → `6d0fd12` |
| jimesh-docs | ✅ (HEAD `4a5dcfb`) | ❌ ji-podhead | ⚠️ stale → `4a5dcfb` |
| jimesh-docs → nested `articles` | ✅ `ji-mesh/articles` (private org copy, mirrored 01.11) | ✅ rewired (commit 3ec07ab) | ✅ pinned fc94d52 |
| ad1 | ✅ private fork (main dcfb8010) | ✅ ji-mesh (submodule source/ad1, added 01.11) | ✅ pinned dcfb8010 |

Open items: NONE — migration complete (01.11): all jimeshOps submodule URLs + pins on ji-mesh HEADs, nested articles rewired to the private org copy. Fresh clones pull ji-mesh only.

## 5. Enforcement for agents

- Jules sessions: always target `ji-mesh/<repo>` in `jules-linear.py create --repo`.
- PRs/branches: only in ji-mesh repos (PR titles per LINEAR-STRUCTURE.md §5).
- Any tooling that clones (scheduler, bridges, fresh sessions): clone from ji-mesh; if a clone hits a ji-podhead URL from an old `.gitmodules`, re-point before doing anything else.

## End of mandate.
