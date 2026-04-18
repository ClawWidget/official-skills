# ClawWidget — `official-skills` Repository Alignment Guide

**Primary role:** End-to-end **repository** guide for `ClawWidget/official-skills` — layout, skill authoring rules, seed skills, CI/CD YAML, CDN settings, operational runbook, acceptance checklist, and runtime constraints. Use this when **bootstrapping or operating** the GitHub repo and Cloudflare deploy.

**See also:** [`Official_Skills_Catalog_Build_Handbook.md`](./Official_Skills_Catalog_Build_Handbook.md) — **catalog builder mechanics** (`cwidget dev build-catalog`, `--source`, `--strict`, hashes, URLs, `dist/` layout). Read that doc when **debugging build output** or **changing** [`catalog_builder.rs`](../../crates/cwidget/src/catalog_builder.rs) / CI flags.

**Audience:** Maintainer of the `ClawWidget/official-skills` GitHub repository.

**Status:** This guide is **authoritative only when paired with a pinned `cwidget` binary from the repaired Trusted Toolchain** (Repair 1.5 and later). Bump the CI pin when contract or builder behavior changes; **never** treat older prose or pre-repair examples as source of truth.

**SSOT:** ClawWidget `crates/cwidget/` — especially [`catalog_builder.rs`](../../crates/cwidget/src/catalog_builder.rs), [`contracts/registries.rs`](../../crates/cwidget/src/contracts/registries.rs), [`contracts/rules/catalog_publish.rs`](../../crates/cwidget/src/contracts/rules/catalog_publish.rs), [`jit/install.rs`](../../crates/cwidget/src/jit/install.rs), [`jit/planner.rs`](../../crates/cwidget/src/jit/planner.rs).

---

## 1. What this repository is

`ClawWidget/official-skills` is the **source of truth** for the official curated skill registry. It:

- Stores `.md` skill files under `skills/official/<category>/<name>.md`
- Runs CI on every PR to validate skills before merge
- On merge to `main`, runs `cwidget dev build-catalog … --strict` and deploys the output to `agents.clawwidget.com` via Cloudflare Pages
- Serves two artifacts: `catalog.json` (JIT install + catalog validation) and `.well-known/mcp-registry.json` (MCP discovery)

---

## 2. Repository layout

```
official-skills/
├── README.md                        # Public-facing: what this is, how to submit
├── CONTRIBUTING.md                  # PR rules; validator + strict build-catalog must pass
├── LICENSE                          # MIT
├── CODEOWNERS                       # Founder + trusted reviewers — required PR approval
├── .github/
│   ├── workflows/
│   │   ├── validate.yml             # PR: strict build + catalog-mode validate
│   │   └── publish.yml             # main: strict build + deploy
│   └── pull_request_template.md
├── skills/
│   └── official/
│       ├── media/
│       │   ├── youtube-dl.md
│       │   └── transcribe-audio.md
│       ├── files/
│       │   └── pdf-to-text.md
│       └── system/
│           ├── notify.md
│           └── clipboard-summary.md
└── tools/
    └── catalog-builder/             # Optional wrapper; invokes `cwidget dev build-catalog`
```

**Branch protection on `main`:** PR required + green CI + CODEOWNER approval + no force-push.

---

## 3. Skill file format (frontmatter contract)

Every `.md` file under `skills/` must have a YAML frontmatter block. Runtime and CI enforce these rules.

### 3.1 Filename and `name:` (hard rule)

- **`name:` must equal the filename stem** (basename without `.md`).
- Example: file `skills/official/system/notify.md` → `name: notify`.
- Do **not** author skills where `name:` could “drift” from the path — the builder and publish rules assume **basename match** (`RULE_CATALOG_PUBLISH_SLUG_BASENAME_MATCHES_NAME` / CP003 in code).

### 3.2 Required fields

| Field | Type | Rules |
|-------|------|--------|
| `id` | UUID v4 (hyphenated) | Unique across the entire `skills/` tree. |
| `name` | String | Path-safe slug; **must match filename stem** (§3.1). |
| `version` | String | Semver, e.g. `"0.1.0"`. |
| `trigger` | Block | See §3.3. |

### 3.3 Optional fields (recommended)

| Field | Type | Notes |
|-------|------|--------|
| `display-name` | String | Human-readable label. |
| `task` | String | One sentence; becomes `description` in `catalog.json`. |
| `icon` | String | Lucide slug, kebab-case. |
| `enabled` | Boolean | Default `true`. |
| `priority` | Integer | Dispatch priority; lower runs first; default `50`. |
| `composable` | Boolean | When `true`, `outputs:` is mandatory. |
| `outputs` | List | Required when `composable: true`. |
| `remote-expose` | Boolean | `false` for MVP; Phase 7 ClawBridge gates remote exposure. |
| `read-only` / `idempotent` | Boolean | Planner hints. |
| `requires` | List of strings | JIT dependency URIs — **categoryful** `official/<category>/<name>` only in v1 (§3.4). |
| `llm-hint` | String | **Mandatory for all official skills.** AI-facing prose (§3.5). |

### 3.3 Trigger types

```yaml
trigger:
  type: manual

trigger:
  type: on-clipboard
  match: url
  match-regex: '^https?://...'

trigger:
  type: on-drop
  match: any

trigger:
  type: on-hotkey
  combo: "Cmd+Shift+D"

trigger:
  type: on-schedule
  cron: "0 9 * * 1-5"

trigger:
  type: on-context-menu
  label: "Process with ClawWidget"
  file-types: [".pdf", ".txt"]
```

### 3.4 `requires:` — dependency declarations

Skills that call other skills via `cwidget skill call` **must** declare each target in `requires:`.

```yaml
requires:
  - "official/system/notify"
  - "official/media/transcribe-audio"
```

**Rules (match runtime + `catalog_publish.rs`):**

| Rule | Detail |
|------|--------|
| **Categoryful v1 URIs** | Every entry **`official/<category>/<name>`** — three segments after `official/`. **Not** `official/<single-segment-slug>` for published catalog skills (CP004). |
| **Closed graph** | Every referenced slug must resolve to another skill **in the same catalog** produced from this repo. |
| **Flat graph** | A skill listed in `requires:` must itself have **`requires:` empty** — no transitive chains (CP002); runtime also enforces at install plan time ([`planner.rs`](../../crates/cwidget/src/jit/planner.rs)). |
| **Trusted namespace** | Only `official/` URIs in v1 (CP001). |
| **Length** | Bounded (see contract / validator for `MAX_REQUIRES_LEN`). |

### 3.5 `llm-hint:`

Write a **system-prompt fragment**: when to use the skill, what it returns, how it composes, limits. Target ~150–800 characters; hard cap **1024** UTF-8 (truncate + `…` in catalog).

### 3.6 Permissions block

```yaml
permissions:
  clipboard: read
  fs:
    - read: ~/Downloads/
    - write: ~/Desktop/
  network: domains
  network-domains:
    - youtube.com
    - "*.youtube.com"
  sidecars: [yt-dlp]
  credentials:
    - name: my-api-key
      domains: [api.example.com]
```

**Guardrails:** `delete` never auto-generated; `remote-expose: true` + outbound `credentials:` / broad `network-domains:` combinations follow product policy; `sidecars:` IDs must exist in the runtime.

---

## 4. Seed skills (reference set)

Five skills exercise the contract surface. **Slugs below are categoryful** (post–Repair 1.5).

| # | Slug | Category | Key contract features |
|---|------|----------|------------------------|
| 1 | `official/system/notify` | system | Zero deps, zero sidecars, `composable: true`. |
| 2 | `official/media/youtube-dl` | media | `sidecars: [yt-dlp]`, network, `on-clipboard`, `composable: true`. |
| 3 | `official/media/transcribe-audio` | media | `sidecars: [whisper]`, `manual`, `composable: true`. |
| 4 | `official/files/pdf-to-text` | files | `on-drop`, `fs:` scope, no sidecars. |
| 5 | `official/system/clipboard-summary` | system | `requires: ["official/system/notify"]` — exercises flat + closed graph with a leaf dependency. |

Reference implementations for testing live under `crates/cwidget/tests/fixtures/` in the main repo; **official-skills** authoring must follow the **category directory layout** in §2.

### Minimal excerpts (identity + requires)

**`skills/official/system/notify.md` — frontmatter snippet**

```yaml
name: notify
# ...
composable: true
outputs:
  - name: sent
    type: boolean
    description: True when the notification was dispatched.
llm-hint: "Use when you need a desktop notification. Returns { sent: boolean }."
```

**`skills/official/system/clipboard-summary.md` — requires + call target**

```yaml
name: clipboard-summary
requires:
  - "official/system/notify"
```

```cwidget
3. cwidget skill call official/system/notify
```

**`skills/official/media/youtube-dl.md` — cross-skill hint**

```yaml
llm-hint: "… Compose with official/media/transcribe-audio for a download-then-transcribe chain …"
```

---

## 5. How the catalog is built — `cwidget dev build-catalog`

**Official CI and publish must use:**

```bash
cwidget dev build-catalog \
  --source skills/official \
  --out dist/ \
  --registry official \
  --registry-url https://agents.clawwidget.com \
  --commit "$GITHUB_SHA" \
  --strict
```

| Mode | Behavior |
|------|-----------|
| **`--strict`** (required for official) | Any **broken** skill file or **TC010** (`skill.call` without matching `requires:`) **fails the process** — no `[skip]`, no partial catalog for production. |
| Default (no `--strict`) | Local iteration: broken files may log `[skip]` and continue ([`catalog_builder.rs`](../../crates/cwidget/src/catalog_builder.rs)). |

**Why `--source skills/official`:** Category is the **first path component under `--source`**. With `--source skills/`, a file at `skills/official/media/foo.md` is misclassified as category `official` instead of `media`. Use the **registry namespace directory** as `--source`.

### What the builder does (summary)

1. Discovers `*.md` under `--source`.
2. Parses each skill; **`--strict` → failure on parse/TC010**; non-strict may skip broken files with `[skip]`.
3. Computes SHA-256 and **`size_bytes`** for each raw file ([`CatalogSkillEntry`](../../crates/cwidget/src/contracts/registries.rs)).
4. Emits slug **`{registry}/{category}/{name}`** ([`catalog_builder.rs`](../../crates/cwidget/src/catalog_builder.rs) ~250–256).
5. Writes hashed + canonical copies under `--out/skills/…`.
6. Writes `catalog.json` and `.well-known/mcp-registry.json`.

### `catalog.json` — identity and URLs

- **`slug`:** `official/<category>/<name>` (three segments for official CDN entries).
- **`raw_url`:** `{registry_url}/skills/{slug}/sha256-{64-hex}/skill.md` — must match `CatalogSkillEntry::expected_raw_url()`; the JIT installer **refuses** a mismatched URL ([`install.rs`](../../crates/cwidget/src/jit/install.rs)).
- **`hash`:** `sha256:` + 64 lowercase hex chars — integrity at **download** only.
- **`size_bytes`:** Live **runtime** field — early reject before fetch when over ceiling (non-zero values; `0` means “unknown” for legacy rows).
- **Never hand-edit** `catalog.json` or hashes in git as authoring — generate with **`build-catalog`**.

Example skill row (trimmed):

```json
{
  "slug": "official/media/youtube-dl",
  "name": "youtube-dl",
  "description": "…",
  "raw_url": "https://agents.clawwidget.com/skills/official/media/youtube-dl/sha256-<hex>/skill.md",
  "canonical_url": "https://agents.clawwidget.com/skills/official/media/youtube-dl/skill.md",
  "hash": "sha256:<64 hex chars>",
  "size_bytes": 1343,
  "requires": [],
  "category": "media"
}
```

---

## 6. GitHub Actions

Pin **`CWIDGET_VERSION`** (or equivalent) to a **released** binary that includes Repair 1.5 — **not** `latest`. After bumping the pin in Xcode-driven releases, follow repo version-sync policy for the main ClawWidget repo; this doc does not duplicate those mechanics.

### `validate.yml` — pull requests

Produce a catalog with **`--strict`**, then run **`cwidget validate --catalog-mode`** on the directory that contains **`catalog.json`** ([`validate_cmd.rs`](../../crates/cwidget/src/validate_cmd.rs)).

```yaml
name: Validate Catalog
on:
  pull_request:
    paths: ['skills/**', '.github/workflows/validate.yml']

jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Install cwidget
        env:
          CWIDGET_VERSION: "x.y.z"
        run: |
          curl -sSL "https://releases.clawwidget.com/cwidget-${CWIDGET_VERSION}-linux-x64.tar.gz" | tar -xz
          install -m 755 cwidget /usr/local/bin/

      - name: Build catalog (strict — broken skills fail CI)
        run: |
          cwidget dev build-catalog \
            --source skills/official \
            --out dist/ \
            --registry official \
            --registry-url https://agents.clawwidget.com \
            --commit "${GITHUB_SHA}" \
            --strict

      - name: Catalog publish-time rules (CP001–CP005)
        run: cwidget validate --catalog-mode dist/
```

`validate --catalog-mode` loads `catalog.json` and runs the full **`catalog_publish::run`** rule set ([`catalog_publish.rs`](../../crates/cwidget/src/contracts/rules/catalog_publish.rs)) — trusted-only `requires`, flat graph, basename match, categoryful official slugs, **`raw_url` vs slug+hash** consistency.

### `publish.yml` — merge to `main`

Same **`build-catalog … --strict`** step as validate, then deploy **`dist/`** (so `catalog.json` is at site root).

```yaml
      - name: Build catalog + content-hashed skill copies (strict)
        run: |
          cwidget dev build-catalog \
            --source skills/official \
            --out dist/ \
            --registry official \
            --registry-url https://agents.clawwidget.com \
            --commit "${{ github.sha }}" \
            --strict
```

---

## 7. CDN: Cloudflare Pages

| Setting | Value |
|---------|--------|
| Domain | `agents.clawwidget.com` |
| TLS | Full Strict; HSTS |
| `/catalog.json` | Short TTL + `stale-while-revalidate` |
| `/skills/**/sha256-*/skill.md` | `immutable`, long TTL |
| `/skills/**/skill.md` (canonical) | Short TTL |
| `/.well-known/mcp-registry.json` | Moderate TTL |
| CORS | `Access-Control-Allow-Origin: *` for GET (if policy allows) |

---

## 8. Operational runbook

### Publish a new skill

1. Add `skills/official/<category>/<name>.md` with **`name:` matching the stem**.
2. PR → strict CI → merge → Pages deploy refreshes `catalog.json`.

### Update a skill

1. Edit the `.md`; merge as above.
2. Old **`sha256-…/skill.md`** URLs remain immutable; new installs use the new hash from `catalog.json`.

### Retract a skill

Remove the file; new installs fail resolution for that slug; local **Fork on Edit** copies remain per product model.

---

## 9. Critical constraints (runtime)

| Constraint | Detail |
|------------|--------|
| Slug shape | Official CDN skills: **`official/<category>/<name>`** (CP004). |
| `name:` vs slug | **`name`** equals **last segment of slug** (basename); must match filename stem (CP003). |
| Flat + closed `requires` | As enforced in catalog + JIT plan ([`planner.rs`](../../crates/cwidget/src/jit/planner.rs)). |
| `raw_url` | Must match hash-locked URL derived from slug + hash; otherwise install aborts ([`install.rs`](../../crates/cwidget/src/jit/install.rs)). |
| `size_bytes` | Non-zero → pre-fetch ceiling enforcement. |
| No hand-edited catalog for production | Regenerate with **pinned** `cwidget dev build-catalog --strict`. |

---

## 10. Acceptance checklist (first / ongoing deploy)

- [ ] Seed set (or successor) lives under `skills/official/<category>/` with **categoryful** slugs in docs and `requires:`.
- [ ] `publish.yml` and PR workflow use **`build-catalog … --strict`**.
- [ ] **`cwidget` version pinned** — includes Repair 1.5+; not a pre-repair binary.
- [ ] **`cwidget validate --catalog-mode dist/`** passes after strict build.
- [ ] Spot-check: fetch one `raw_url`, verify **`sha256`** matches `hash` in `catalog.json`.
- [ ] CODEOWNERS + branch protection enforced.
- [ ] **Never** commit hand-edited `catalog.json` for production.

---

## 11. Pointers into ClawWidget

| Topic | Path |
|-------|------|
| Wire types / `expected_raw_url` | `crates/cwidget/src/contracts/registries.rs` |
| Builder + `--strict` | `crates/cwidget/src/catalog_builder.rs` |
| CP001–CP005 | `crates/cwidget/src/contracts/rules/catalog_publish.rs` |
| JIT URL + size checks | `crates/cwidget/src/jit/install.rs` |
| Install plan / flat deps | `crates/cwidget/src/jit/planner.rs` |
