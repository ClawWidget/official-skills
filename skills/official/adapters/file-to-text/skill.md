---
id: "2a3b4c5d-6e7f-4a8b-9c0d-1e2f3a4b5c6d"
name: file-to-text
version: "0.1.0"
enabled: true
display-name: "File → Text"
task: "Reads a UTF-8 file and returns its plain-text contents as a string."
usage-hint: "Type-coercion adapter — chain after a skill that emits a file path when the next skill expects a text string."
icon: file-text
llm-hint: "Call between a skill whose output is path-file and a skill whose input is text when the file's UTF-8 content is needed as a plain string."
tags: [adapter, file-read, composable]
composable: true
trigger:
  type: manual
mcp-input:
  properties:
    file_path:
      type: string
      semantic_kind: path-file
      description: "Absolute path to a UTF-8 file under $HOME."
  required:
    - file_path
permissions:
  fs:
    - read: "~/"
outputs:
  - name: text_content
    type: string
    semantic_kind: text
    description: "UTF-8 text contents of the file."
---

# File → Text

Type-coercion adapter for chains. Reads a file at the given path and returns its UTF-8 contents as a plain string.

```cwidget
1. cwidget when $file_path --eq "" --stop --message "file_path is required"
2. $TEXT = cwidget fs read $file_path
3. $OUT = cwidget json set "{}" --path text_content --value $TEXT
```
