---
id: "3b4c5d6e-7f8a-4b9c-ad1e-2f3a4b5c6d7e"
name: text-to-file
version: "0.1.0"
enabled: true
display-name: "Text → File"
task: "Writes a string to an isolated temporary file and returns the file path."
usage-hint: "Type-coercion adapter — chain after a skill that emits a text string when the next skill expects a file path."
icon: file-plus
llm-hint: "Call between a skill whose output is text and a skill whose input is path-file when the text must be written to a temporary file first."
tags: [adapter, file-write, composable]
composable: true
trigger:
  type: manual
mcp-input:
  properties:
    text_content:
      type: string
      semantic_kind: text
      description: "Text string to write to a temporary file."
  required:
    - text_content
permissions:
  fs:
    - write: "~/"
outputs:
  - name: file_path
    type: string
    semantic_kind: path-file
    description: "Absolute path to the temporary file containing the text."
---

# Text → File

Type-coercion adapter for chains. Writes the input text to an isolated temporary file and returns the file path.

```cwidget
1. cwidget when $text_content --eq "" --stop --message "text_content is required"
2. $TMPDIR = cwidget fs tmpdir
3. $FILE_PATH = cwidget concat $TMPDIR "/adapter-output.txt"
4. cwidget fs write $FILE_PATH --content $text_content
5. $OUT = cwidget json set "{}" --path file_path --value $FILE_PATH
```
