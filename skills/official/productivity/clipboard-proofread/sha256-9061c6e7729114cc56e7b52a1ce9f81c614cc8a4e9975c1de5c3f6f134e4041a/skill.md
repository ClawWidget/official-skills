---
id: "c4e8f1a2-9b6d-4e7c-a3f5-1d2e8b9c0a41"
name: clipboard-proofread
version: "0.1.0"
enabled: true
display-name: Clipboard Proofread
task: Polishes copied text for perfect grammar and professional clarity.
usage-hint: "Copy any draft to your clipboard to instantly generate a proofread version."
icon: clipboard-check
llm-hint: "Polishes copied writing without changing the meaning. Chain when the user wants emails, notes, or drafts proofread for grammar, spelling, punctuation, or clarity."
tags: [clipboard-text, grammar-fix, proofread, on-clipboard, llm]
trigger:
  type: on-clipboard
  match: text
permissions:
  network: llm-only
requires:
  - "official/utilities/notify"
outputs:
  - name: corrected_text
    type: string
    description: Grammar-corrected plain text from the LLM
---

# Clipboard Proofread

Watches the clipboard for text. On trigger, returns a lightly edited version with grammar, spelling, and punctuation fixes while preserving meaning and tone. Then calls `official/utilities/notify` to signal completion (same JIT `requires:` pattern as `official/productivity/clipboard-summary`).

```cwidget
1. $CORRECTED = cwidget llm ask --system "You are a careful proofreader. Fix grammar, spelling, and punctuation only. Preserve meaning, voice, and structure. Output only the corrected text with no preamble or explanation." --prompt $CLIPBOARD
2. cwidget ui notify $CORRECTED
3. $DONE = cwidget skill call official/utilities/notify
4. $OUT = cwidget json set "{}" --path corrected_text --value $CORRECTED
```
