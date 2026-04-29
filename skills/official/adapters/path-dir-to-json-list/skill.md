---
id: "6e7f8a9b-0c1d-4e2f-c04b-5c6d7e8f9a0b"
name: path-dir-to-json-list
version: "0.1.0"
enabled: true
display-name: "Dir → JSON List"
task: "Lists directory entries and emits a JSON array of file paths."
usage-hint: "Type-coercion adapter — chain after a skill that emits a directory path when the next skill expects a JSON array of file paths."
icon: list
llm-hint: "Call between a skill whose output is path-dir and a skill whose input is json when the directory contents must be enumerated as a JSON array."
tags: [adapter, path-coercion, json-list, composable]
composable: true
trigger:
  type: manual
mcp-input:
  properties:
    dir:
      type: string
      semantic_kind: path-dir
      description: "Absolute path to a directory under $HOME."
  required:
    - dir
permissions:
  fs:
    - list: "~/"
outputs:
  - name: entries_json
    type: string
    semantic_kind: json
    description: "JSON array of absolute file paths found in the directory."
---

# Dir → JSON List

Type-coercion adapter for chains. Lists the files in a directory and emits their paths as a JSON array.

```cwidget
1. cwidget when $dir --eq "" --stop --message "dir is required"
2. $LIST = cwidget fs list $dir
3. $FILES = cwidget text split $LIST --on "\n"
4. $OUT = cwidget json set "{}" --path entries_json --value $FILES
```
