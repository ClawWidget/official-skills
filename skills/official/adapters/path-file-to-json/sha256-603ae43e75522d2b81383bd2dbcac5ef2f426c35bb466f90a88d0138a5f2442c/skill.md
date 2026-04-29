---
id: "8a9b0c1d-2e3f-4a4b-e26d-7e8f9a0b1c2d"
name: path-file-to-json
version: "0.1.0"
enabled: true
display-name: "File → JSON"
task: "Reads a file and parses its contents as JSON."
usage-hint: "Type-coercion adapter — chain after a skill that emits a file path when the next skill expects structured JSON parsed from that file."
icon: file-code
llm-hint: "Call between a skill whose output is path-file and a skill whose input is json when the file contains JSON that must be parsed into a structured value."
tags: [adapter, json-parse, file-read, composable]
composable: true
trigger:
  type: manual
mcp-input:
  properties:
    file_path:
      type: string
      semantic_kind: path-file
      description: "Absolute path to a JSON file under $HOME."
  required:
    - file_path
permissions:
  fs:
    - read: "~/"
outputs:
  - name: json_value
    type: string
    semantic_kind: json
    description: "Structured JSON parsed from the file contents."
---

# File → JSON

Type-coercion adapter for chains. Reads a file at the given path and parses its UTF-8 contents as JSON.

```cwidget
1. cwidget when $file_path --eq "" --stop --message "file_path is required"
2. $TEXT = cwidget fs read $file_path
3. $PARSED = cwidget json parse $TEXT
4. $OUT = cwidget json set "{}" --path json_value --value $PARSED
```
