# Contributing to ClawWidget Official Skills

This registry is the single authoritative source for skills served at
`agents.clawwidget.com`. Every file here is a reviewed, sandboxed, composable
skill written for the ClawWidget runtime.

---

## Before you open a PR

- Read the [skill format spec](#skill-file-format) below.
- Run `cwidget validate --catalog-mode skills/` locally against your file.
- Every rule violation printed is a `RULE_*` ID. Look it up — it will tell you
  exactly what to fix.

---

## Skill file format

Skills live at `skills/official/<category>/<slug>.md`.

```
skills/
└── official/
    ├── media/
    │   └── your-skill.md
    ├── files/
    └── system/
```

The frontmatter block at the top of each `.md` file is the contract. Required
fields:

| Field | Type | Notes |
|---|---|---|
| `name` | string | Matches the filename without `.md` |
| `description` | string | 1–2 sentences, human-readable |
| `llm_hint` | string | **Required for all official skills.** See below. |
| `trigger` | string | `on-demand`, `on-clipboard`, `on-schedule`, etc. |
| `composable` | bool | `true` if other skills may call this one |
| `remote_expose` | bool | `false` for MVP; no remote-expose PRs accepted yet |
| `human_in_loop` | bool | |
| `sidecars_required` | list | Declared sidecar binaries |
| `requires` | list | Slugs this skill depends on. Must be `official/*` only. Max depth: 1. |

---

## Writing a good `llm_hint`

`llm_hint` is a **meta-prompt fragment written for AI consumers** (Cursor,
Claude Code, the ClawWidget Router). It is not shown to end users; `description`
is. Think of it as the tool description you'd write for an LLM tool-use API.

**Checklist (CODEOWNER will verify):**

- [ ] Describes *when* to use this skill (not just *what* it does)
- [ ] Names the return shape: `Returns { field: type, … }`
- [ ] Notes composition partners if any: `Compose with official/X to …`
- [ ] Notes hard limits: single-item only, local-only, etc.
- [ ] 100–800 characters. Under 100 is too thin; over 800 will be truncated by
      the validator with a warning.

**Example:**

```
llm_hint: >
  Use when the user pastes a YouTube URL and wants the audio saved locally.
  Returns { saved_path: string, duration_ms: integer }.
  Compose with official/transcribe-audio to get a transcript chain.
  Single video only in v1 — do not use for playlists.
```

---

## Dependency rules

- `requires:` may only reference `official/*` slugs.
  → Violations fail with `RULE_CATALOG_PUBLISH_REQUIRES_TRUSTED_ONLY`.
- `requires:` depth is capped at 1. A skill you depend on must have an empty
  `requires:` list.
  → Violations fail with `RULE_CATALOG_PUBLISH_REQUIRES_FLAT`.
- Every slug in `requires:` must exist as a file in this repo.
  → Missing slugs fail the `--catalog-graph` check.

---

## Slug namespace policy

- Slugs are `official/<category>/<name>`. The `official/` namespace is
  reserved for this curated registry.
- Do not submit slugs that impersonate tools, companies, or trademarks you do
  not own.
- Do not submit `remote_expose: true` skills — this is blocked by policy until
  ClawBridge (Phase 7) prerequisites are validated.

---

## PR checklist

Your PR description will auto-populate from `.github/pull_request_template.md`.
Complete every item. PRs with unchecked boxes will not be reviewed.

---

## Review & merge

1. CI runs `cwidget validate --catalog-mode` and `--catalog-graph`. Both must
   be green.
2. A CODEOWNER approves. See `CODEOWNERS` for who that is.
3. On merge to `main`, the catalog auto-deploys to
   `agents.clawwidget.com/catalog.json` within ~2 minutes.

You do not need to compute hashes or update `catalog.json` manually. The
publish pipeline does that.

---

## Local preview

```bash
# Validate your skill before pushing
cwidget validate --catalog-mode skills/
cwidget validate --catalog-graph skills/

# Build the full catalog locally (requires cwidget dev build)
cwidget dev build-catalog --source skills/ --out dist/ --registry official \
  --registry-url https://agents.clawwidget.com --commit local
```

After `build-catalog`, copy Cloudflare headers into `dist/` (same as CI):

```bash
install -m 644 tools/catalog-builder/cloudflare-pages-headers.txt dist/_headers
```
