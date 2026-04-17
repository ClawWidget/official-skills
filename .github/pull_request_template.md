## Skill PR checklist

**Slug:** `official/<category>/<slug>`

### Format
- [ ] File is at `skills/official/<category>/<slug>.md`
- [ ] All required frontmatter fields are present and valid
- [ ] `cwidget validate --catalog-mode skills/` passes locally with no errors

### `llm_hint` quality
- [ ] Describes *when* to invoke this skill (not just what it does)
- [ ] Names the return shape (`Returns { … }`)
- [ ] Notes composition partners, if any
- [ ] Notes hard limits (single-item, local-only, etc.)
- [ ] Between 100–800 characters

### Dependencies
- [ ] `requires:` contains only `official/*` slugs (or is empty)
- [ ] Every slug in `requires:` exists as a file in this repo
- [ ] None of the skills in `requires:` themselves have non-empty `requires:`
      (flat dep graph only)
- [ ] `cwidget validate --catalog-graph skills/` passes locally

### Policy
- [ ] `remote_expose: false` (remote-expose PRs are not accepted yet)
- [ ] Slug does not impersonate a trademark or tool you do not own

---

> **CI will re-run the same checks above.** If CI is red, fix the errors before
> requesting review. Each error message includes a `RULE_*` ID — search it in
> `help_doctor_explain.json` for the exact fix.
