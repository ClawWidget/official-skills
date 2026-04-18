# Official Skills — Catalog Build Handbook

**Primary role:** **Catalog build tool contract** — behavior of `cwidget dev build-catalog` (flags, `--source` vs `category`, `--strict`, hash and URL formulas, `dist/` layout, emitted JSON fields). Use when **configuring publish CI**, **explaining wrong `category`**, or **after changing** [`catalog_builder.rs`](../../crates/cwidget/src/catalog_builder.rs).

**See also:** [`OFFICIAL_SKILLS_REPO_ALIGNMENT.md`](./OFFICIAL_SKILLS_REPO_ALIGNMENT.md) — full repo guide (skills, Actions, CDN, policies).

**Audience:** Maintainers of `ClawWidget/official-skills` (CI, CDN, layout).

**SSOT:** The **pinned `cwidget` binary** from ClawWidget — [`catalog_builder.rs`](../../crates/cwidget/src/catalog_builder.rs), [`contracts/registries.rs`](../../crates/cwidget/src/contracts/registries.rs). Do not treat old markdown or pre-repair examples as authoritative.

---

## 1. Command reference

| Item | Value |
|------|--------|
| Verb | `cwidget dev build-catalog` |
| Scope | Maintainer **dev** subcommand (not an end-user install path). |

### Flags

| Flag | Meaning |
|------|---------|
| `--source <path>` | Root scanned **recursively** for `*.md` skills. |
| `--out <path>` | Output root for `catalog.json`, `.well-known/`, `skills/`. |
| `--registry <name>` | Namespace in slugs and paths (use **`official`** for this repo). |
| `--registry-url <url>` | Public HTTPS origin **without** trailing slash. |
| `--commit <sha>` | Provenance string for `generator.commit` (optional → `"unknown"`). |
| `--strict` | **Official CI:** fail on any invalid/broken skill or TC010 violation. **Non-strict (default):** broken files may log `[skip]` and be omitted from the catalog — for local drafts only. |

### Recommended invocation (official CI)

```bash
cwidget dev build-catalog \
  --source skills/official \
  --out dist/ \
  --registry official \
  --registry-url https://agents.clawwidget.com \
  --commit "${GITHUB_SHA}" \
  --strict
```

**Pin** a `cwidget` release that implements Repair 1.5+ (categoryful slugs, `size_bytes`, hash-locked URLs, `--strict`). **Do not publish** using an older pre-repair binary.

**`--source`:** For repo layout `skills/official/<category>/<file>.md`, **`--source` must be `skills/official`** so `category` is `media`, `files`, `system`, … — not `official`.

---

## 2. Source tree expectations

### Layout

```text
skills/
└── official/
    ├── media/
    │   ├── youtube-dl.md
    │   └── transcribe-audio.md
    ├── files/
    │   └── pdf-to-text.md
    └── system/
        └── notify.md
```

### Skill files

| Rule | Detail |
|------|--------|
| Format | UTF-8 `.md` with YAML frontmatter per ClawWidget grammar. |
| Discovery | Every `*.md` under `--source`, recursive; other files ignored. |
| Parse / validation failures | **Without `--strict`:** file may be **skipped** with `[skip]` on stderr — local iteration. **With `--strict`:** build **fails** — required for official publish CI. Broken skills must **never** ship as `[skip]` in official mode. |
| Identity | **`slug` in the catalog is `{registry}/{category}/{name}`** where `category` comes from the directory under `--source` and **`name`** is frontmatter `name:`. **`name:` must equal the filename stem** (basename of the `.md`). Do not rely on diverging filenames and `name:`. |

---

## 3. Frontmatter and builder

- Parser path matches the app (`parse_skill_preview`); in-memory `id:` injection only — **does not rewrite** sources on disk.
- Per-skill validation and TC010 handling: see [`catalog_builder.rs`](../../crates/cwidget/src/catalog_builder.rs) `process_skill` — **`--strict`** fails on TC010; non-strict collects warnings.

### `llm-hint`

| Item | Behavior |
|------|-----------|
| Key | `llm-hint:` → `llm_hint` in JSON. |
| Max | 1024 UTF-8 chars; over → warning + truncate + `…`. |

---

## 4. Category and slug derivation

From [`catalog_builder.rs`](../../crates/cwidget/src/catalog_builder.rs):

| Field | Rule |
|-------|------|
| **category** | Relative path from `--source` → parent dir’s **first** component, or **`general`** if file sits directly under `--source`. |
| **name** | Frontmatter `name:` (must match file stem). |
| **slug** | `{registry}/{category}/{name}` — **always three segments** for typical tree (or `…/general/…` for flat `--source`). |

| `--source` | Example file | `category` |
|------------|----------------|------------|
| `skills/official` | `skills/official/media/youtube-dl.md` | `media` |
| `skills/official` | `skills/official/system/notify.md` | `system` |
| `skills` | `skills/official/media/youtube-dl.md` | **`official`** (wrong for intended layout) |

---

## 5. Hashes and URLs (runtime-consumed)

| Item | Rule |
|------|------|
| **Hash** | SHA-256 over **raw file bytes**. |
| **Wire form** | `sha256:` + 64 lowercase hex digits. |
| **`size_bytes`** | Byte length of that file — **runtime contract** for pre-fetch ceiling ([`install.rs`](../../crates/cwidget/src/jit/install.rs)). |
| **`raw_url`** | `{registry_url}/skills/{slug}/sha256-{hex}/skill.md` — see `CatalogSkillEntry::expected_raw_url()` in [`registries.rs`](../../crates/cwidget/src/contracts/registries.rs). The JIT layer **compares** `entry.raw_url` to this derivation before fetching; a mismatch aborts install. This is not a cosmetic builder detail. |
| **`canonical_url`** | Same path without the `sha256-<hex>/` segment — browsing; runtime **does not** use it for fetch. |

`hash_hex` in the path is **without** the `sha256:` prefix.

---

## 6. Output layout (`--out`)

```text
dist/
├── catalog.json
├── .well-known/
│   └── mcp-registry.json
└── skills/
    └── official/
        └── <category>/
            └── <name>/
                ├── skill.md
                └── sha256-<64 hex>/skill.md
```

---

## 7. `catalog.json` fields — runtime vs tooling

Top-level metadata (`schema_version`, `registry`, `registry_url`, `generated_at`, `generator`, `skill_count`) is emitted for tooling and provenance.

### Runtime-consumed contract fields (per skill)

Aligned with [`CatalogSkillEntry`](../../crates/cwidget/src/contracts/registries.rs) — the JIT installer and validators depend on these:

| Field | Role |
|-------|------|
| `slug` | Identity key — **categoryful** `official/<category>/<name>` for CDN skills. |
| `name` | Must match basename of slug and frontmatter `name:`. |
| `description` | Display / previews. |
| `raw_url` | **Fetch target** — must match hash-locked derivation. |
| `hash` | Integrity at download. |
| `requires` | Dependency URIs — categoryful official slugs in v1. |
| `trigger` | Trigger type string. |
| `composable` | Composition flag. |
| `remote_expose` | Remote exposure flag. |
| `sidecars_required` | Sidecar IDs for previews / install. |
| `human_in_loop` | Heuristic from body (`ui form` / `ui confirm`). |
| `llm_hint` | AI routing prose. |
| `size_bytes` | **Live** size gate before network I/O (non-zero). |

### Builder / tooling extras (still emitted; secondary to wire contract)

| Field | Role |
|-------|------|
| `canonical_url` | Human-readable URL; not used for JIT fetch. |
| `category` | Redundant convenience copy of path segment (also encoded in `slug`). |
| `tags` | Derived hints (e.g. sidecars + composable). |
| `added_at`, `last_modified_at` | Timestamps from build. |
| `generator` | Tool name, **Cargo package version**, commit. |

**Never hand-edit** `catalog.json` in the official repo — regenerate with **`cwidget dev build-catalog`** using a **pinned repaired** binary.

---

## 8. `.well-known/mcp-registry.json`

Emitted beside `catalog.json`. Shape is defined in [`catalog_builder.rs`](../../crates/cwidget/src/catalog_builder.rs). Branding/support URL changes belong in **ClawWidget source**, not manual JSON edits in `official-skills`.

---

## 9. CI alignment checklist

- [ ] **`cwidget` pinned** to a release that includes Repair 1.5 (categoryful slugs, `--strict`, hashed `raw_url`, real `size_bytes`).
- [ ] **`build-catalog` uses `--strict`** for official publish — failures are **CI failures**, not `[skip]`.
- [ ] **`--source skills/official`** (or equivalent) so **`category`** is correct.
- [ ] **`--registry-url`** matches production CDN (no trailing slash).
- [ ] **`--commit`** set to the source SHA.
- [ ] Deploy the **`--out`** tree so URLs line up with §5–§6.
- [ ] **`cwidget validate --catalog-mode dist/`** after build (directory containing **`catalog.json`**) — exercises CP001–CP005 ([`catalog_publish.rs`](../../crates/cwidget/src/contracts/rules/catalog_publish.rs)).
- [ ] **Never hand-edit** `catalog.json`.
- [ ] **Never publish** from an **older pre-repair** `cwidget` that emits two-segment slugs or non-canonical `raw_url`.

---

## 10. Drift and updates

| Change | Action |
|--------|--------|
| Builder URL layout / fields | Land in ClawWidget `catalog_builder.rs` + release; bump CI pin. |
| `schema_version` | Coordinated bump in ClawWidget `CATALOG_SCHEMA_VERSION`. |
| MCP manifest tweaks | Same — source change + release. |

---

## 11. Summary

| Question | Answer |
|----------|--------|
| Who owns catalog **behavior**? | ClawWidget `cwidget dev build-catalog`. |
| Official CI flags? | **`--strict`** + **`--source skills/official`** + pinned repaired binary. |
| May `catalog.json` be hand-edited? | **No.** |
| Slug identity? | **`official/<category>/<name>`**; **`name:`** = filename stem. |
