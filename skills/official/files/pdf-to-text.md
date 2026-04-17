---
id: "f0a6c8d4-5e7b-4f9c-bd2e-3f4a5b6c7d8e"
name: pdf-to-text
version: "0.1.0"
enabled: true
display-name: PDF to Text
task: Drop a PDF file to extract its plain text via Pandoc. Local-only, no network.
icon: file-text
llm-hint: "Fires on file drop. Extracts plain text from a PDF using Pandoc (--to plain). Returns {text}. Local-only — no LLM, no network. Requires pandoc sidecar. Chain text output into an LLM step for analysis or summarization."
trigger:
  type: on-drop
permissions:
  fs:
    - read: ~/Downloads/
  sidecars: [pandoc]
  input-types: [pdf]
composable: true
outputs:
  - name: text
    type: string
    description: Plain text extracted from the dropped PDF file
---

# PDF to Text

Drop a PDF file onto the ClawWidget drop zone to extract its plain text using Pandoc. Output is returned as a string suitable for piping into an LLM summarizer.

Adjust `read: ~/Downloads/` in the permissions to match the directory your PDFs live in.

```cwidget
1. cwidget when $DROPPED_FILE_EXT --not-eq pdf --stop --message "Expected a .pdf file."
2. $TEXT = cwidget sidecar run pandoc $DROPPED_FILE --to plain
3. cwidget ui notify "PDF text extracted."
4. $OUT = cwidget json set "{}" --path text --value $TEXT
```
