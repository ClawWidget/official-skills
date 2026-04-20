---
id: "c4e8f1a2-9b6d-4e7c-a3f5-1d2e8b9c0a41"
name: clipboard-proofread
version: "0.1.0"
enabled: true
display-name: Clipboard Proofread
task: Proofreads copied text for grammar and clarity, then notifies via official/utilities/notify.
icon: clipboard-check
llm-hint: "Fires when text is copied. Proofreads pasted clipboard text (emails, notes, drafts) for grammar, spelling, and punctuation via LLM (meaning preserved), shows the result in a notification, then calls official/utilities/notify (JIT via requires). Returns {corrected_text}. Requires LLM API key; network is llm-only. Same chain shape as official/productivity/clipboard-summary."
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
