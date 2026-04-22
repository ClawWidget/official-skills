---
id: "e9f2d5c8-4a1b-4f7e-9d6c-2e8f3a5b7c91"
name: clipboard-action-items
version: "0.1.0"
enabled: true
display-name: Clipboard Action Items
task: Extracts clear, actionable next steps from any copied text.
usage-hint: "Copy any text to your clipboard to instantly extract action items."
icon: list-check
llm-hint: "Turns copied notes, emails, and meeting transcripts into concrete next steps. Chain when the user wants action items, follow-ups, or commitments extracted from messy text."
tags: [clipboard-text, action-items, task-extraction, on-clipboard, llm]
trigger:
  type: on-clipboard
  match: text
permissions:
  network: llm-only
requires:
  - "official/utilities/notify"
outputs:
  - name: action_items
    type: string
    description: Plain-text bullet list of action items from the LLM
---

# Clipboard Action Items

Watches the clipboard for text. On trigger, extracts concise, actionable items (tasks, decisions, follow-ups implied by the text). Then calls `official/utilities/notify` to signal completion (same JIT `requires:` pattern as `official/productivity/clipboard-summary`).

```cwidget
1. $ACTIONS = cwidget llm ask --system "You extract action items only. Given meeting notes, email, or prose, list concrete next steps or commitments. Use plain-text lines starting with '- ' (dash space). One item per line. Be specific and imperative where possible. If there are no actions, output a single line: '- No explicit action items found.' Output nothing else — no title, preamble, or explanation." --prompt $CLIPBOARD
2. cwidget ui notify $ACTIONS
3. $DONE = cwidget skill call official/utilities/notify
4. $OUT = cwidget json set "{}" --path action_items --value $ACTIONS
```
