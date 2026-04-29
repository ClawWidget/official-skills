---
id: "4c5d6e7f-8a9b-4c0d-be2f-3a4b5c6d7e8f"
name: text-to-dir
version: "0.1.0"
enabled: true
display-name: "Text → Dir"
task: "Interprets a text string as a directory path, creates it if needed, and returns the directory path."
usage-hint: "Type-coercion adapter — chain after a skill that emits a path string as text when the next skill expects a typed directory path."
icon: folder-plus
llm-hint: "Call between a skill whose output is text (a path string) and a skill whose input is path-dir when the directory must exist before the next step."
tags: [adapter, path-coercion, composable]
composable: true
trigger:
  type: manual
mcp-input:
  properties:
    text:
      type: string
      semantic_kind: text
      description: "A path string (must start with ~/) to interpret as a directory."
  required:
    - text
permissions:
  fs:
    - mkdir: "~/"
outputs:
  - name: resolved_dir
    type: string
    semantic_kind: path-dir
    description: "Absolute path to the created or pre-existing directory."
---

# Text → Dir

Type-coercion adapter for chains. Interprets the input text as a directory path, creates the directory (including parents) if it does not exist, and returns the path.

```cwidget
1. cwidget when $text --eq "" --stop --message "text is required"
2. cwidget when $text --not-matches "^~/" --stop --message "path must be under home directory"
3. cwidget fs mkdir $text --parents
4. $OUT = cwidget json set "{}" --path resolved_dir --value $text
```
