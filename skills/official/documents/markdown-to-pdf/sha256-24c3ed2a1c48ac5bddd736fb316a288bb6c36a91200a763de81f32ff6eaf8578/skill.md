---
id: "7b3e6a91-8dcb-4d3d-9f57-3c4f0a2f8d11"
name: markdown-to-pdf
version: "0.1.0"
enabled: true
display-name: Markdown to PDF
task: Turns any dropped Markdown document into a clean, shareable PDF.
usage-hint: "Drop a Markdown file into ClawWidget to create a PDF beside the original."
icon: file-output
llm-hint: "Turns finished Markdown drafts into shareable PDFs. Chain when the user wants notes, reports, or generated Markdown rendered for sending, printing, or handoff."
tags: [markdown, pdf-generation, pandoc, on-drop, document-render]
trigger:
  type: on-drop
permissions:
  fs:
    - read: ~/Documents/
    - write: ~/Documents/
  sidecars: [pandoc]
  input-types: [md]
composable: true
outputs:
  - name: saved_path
    type: string
    description: Path where the generated PDF was written
---

# Markdown to PDF

Drop a `.md` or `.markdown` file onto the ClawWidget drop zone to render it to PDF with Pandoc.

The output path uses the input path with `.pdf` appended.

```cwidget
1. cwidget when $DROPPED_FILE_EXT --not-matches "^(md|markdown)$" --stop --message "Expected a Markdown file (.md or .markdown)."
2. cwidget sidecar run pandoc $DROPPED_FILE --to pdf --output "$DROPPED_FILE.pdf"
3. cwidget ui notify "PDF generated."
4. $OUT = cwidget json set "{}" --path saved_path --value "$DROPPED_FILE.pdf"
```
