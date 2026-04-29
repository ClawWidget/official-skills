---
id: "11111111-1111-4111-8111-111111111111"
name: dir-to-single-file
version: "0.1.0"
enabled: true
display-name: "Dir → Single File"
task: "Returns the only file in a directory; fails if the directory has zero or multiple files."
usage-hint: "Type-coercion adapter — chain after a skill that emits a directory path when the next skill expects a file path and exactly one file is expected."
icon: file
llm-hint: "Call between a skill whose output is path-dir and a skill whose input is path-file when the directory is expected to contain one file."
tags: [adapter, path-coercion, composable]
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
  - name: single_file
    type: string
    semantic_kind: path-file
    description: "Absolute path to the only file in the directory."
---

# Dir → Single File

Type-coercion adapter for chains. Given a directory, returns the absolute path of the only file inside it.

```cwidget
1. cwidget when $dir --eq "" --stop --message "dir is required"
2. $LIST = cwidget fs list $dir
3. $FILES = cwidget text split $LIST --on "\n"
4. $COUNT = cwidget json length $FILES
5. cwidget when $COUNT --not-eq "1" --stop --message "Expected exactly one file in directory"
6. $FIRST = cwidget json get $FILES --path "0"
7. $SINGLE = cwidget concat $FIRST ""
8. $OUT = cwidget json set "{}" --path single_file --value $SINGLE
```
