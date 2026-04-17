# ClawWidget Official Skills

Curated, PR-reviewed atomic skills for the [ClawWidget](https://clawwidget.com)
runtime. Served at `agents.clawwidget.com`.

---

## Install a skill

Slugs follow `official/<category>/<name>` (see paths under `skills/official/`).

```bash
cwidget skill install official/system/notify
cwidget skill install official/media/youtube-dl
```

---

## Browse the catalog

```
https://agents.clawwidget.com/catalog.json
```

Or install ClawWidget and run:

```bash
cwidget skill search
```

---

## What is a skill?

A skill is a single Markdown file with a frontmatter contract. It runs
sandboxed in the ClawWidget runtime — no arbitrary shell access, explicit
permissions only, local-first. Each skill file is inspectable before
installation.

---

## Seed skills

| Slug | What it does |
|---|---|
| `official/system/notify` | Send a macOS system notification |
| `official/media/youtube-dl` | Download YouTube audio to Desktop |
| `official/media/transcribe-audio` | Transcribe a local audio file |
| `official/files/pdf-to-text` | Extract plain text from a PDF |
| `official/system/clipboard-summary` | Summarize clipboard contents and notify |

---

## Submit a skill

See [CONTRIBUTING.md](./CONTRIBUTING.md). Open a PR — CI validates
automatically, a CODEOWNER reviews, and on merge the catalog deploys within
~2 minutes.

---

## Trust model

- Every skill is reviewed by a CODEOWNER before merge.
- CI enforces the full ClawWidget contract validator on every PR.
- Skill files are content-hashed at publish time. The hash in `catalog.json`
  must match the bytes you download, or the runtime rejects installation.
- No telemetry. No analytics. No client IP collection.

---

## MCP auto-discovery

MCP-aware clients can discover this registry at:

```
https://agents.clawwidget.com/.well-known/mcp-registry.json
```

---

## License

MIT
