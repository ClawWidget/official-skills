---
id: "7f8a9b0c-1d2e-4f3a-d15c-6d7e8f9a0b1c"
name: json-to-file
version: "0.1.0"
enabled: true
display-name: "JSON → File"
task: "Serializes a JSON value to a scoped temporary file and returns the file path."
usage-hint: "Type-coercion adapter — chain after a skill that emits JSON when the next skill expects a file path."
icon: file-json
llm-hint: "Call between a skill whose output is json and a skill whose input is path-file when the JSON must be written to a temporary file first."
tags: [adapter, json-write, composable]
composable: true
trigger:
  type: manual
mcp-input:
  properties:
    json_value:
      type: string
      semantic_kind: json
      description: "JSON value to serialize and write to a temporary file."
  required:
    - json_value
permissions:
  fs:
    - write: "~/"
outputs:
  - name: file_path
    type: string
    semantic_kind: path-file
    description: "Absolute path to the temporary file containing the serialized JSON."
---

# JSON → File

Type-coercion adapter for chains. Serializes a JSON value to an isolated temporary file and returns the file path.

```cwidget
1. cwidget when $json_value --eq "" --stop --message "json_value is required"
2. $JSON_STR = cwidget json stringify $json_value
3. $TMPDIR = cwidget fs tmpdir
4. $FILE_PATH = cwidget concat $TMPDIR "/adapter-json-output.json"
5. cwidget fs write $FILE_PATH --content $JSON_STR
6. $OUT = cwidget json set "{}" --path file_path --value $FILE_PATH
```
