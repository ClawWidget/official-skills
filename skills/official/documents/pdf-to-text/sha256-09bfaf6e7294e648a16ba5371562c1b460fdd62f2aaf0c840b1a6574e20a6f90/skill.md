---
id: "f0a6c8d4-5e7b-4f9c-bd2e-3f4a5b6c7d8e"
name: pdf-to-text
version: "0.1.0"
enabled: true
display-name: PDF to Text
task: Extracts clean, editable text from any dropped PDF.
usage-hint: "Drop a PDF into ClawWidget to extract its text."
icon: file-text
llm-hint: "Pulls editable text out of PDFs. Chain when the user wants a PDF summarized, searched, quoted, rewritten, or passed into a downstream LLM step."
tags: [pdf-extraction, document-text, pandoc, on-drop, text-output]
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
