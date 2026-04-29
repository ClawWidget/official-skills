---
id: "5d6e7f8a-9b0c-4d1e-bf3a-4b5c6d7e8f9a"
name: text-to-json
version: "0.1.0"
enabled: true
display-name: "Text → JSON"
task: "Parses a JSON-formatted text string and emits structured JSON."
usage-hint: "Type-coercion adapter — chain after a skill that emits a JSON string as text when the next skill expects structured JSON."
icon: braces
llm-hint: "Call between a skill whose output is text (containing valid JSON) and a skill whose input is json when the text must be parsed and validated as JSON."
tags: [adapter, json-parse, composable]
composable: true
trigger:
  type: manual
mcp-input:
  properties:
    text:
      type: string
      semantic_kind: text
      description: "A JSON-formatted text string to parse."
  required:
    - text
permissions: {}
outputs:
  - name: json_value
    type: string
    semantic_kind: json
    description: "The parsed JSON value emitted as structured output."
---

# Text → JSON

Type-coercion adapter for chains. Parses a JSON-formatted text string and emits structured JSON.

```cwidget
1. cwidget when $text --eq "" --stop --message "text is required"
2. $PARSED = cwidget json parse $text
3. $OUT = cwidget json set "{}" --path json_value --value $PARSED
```
