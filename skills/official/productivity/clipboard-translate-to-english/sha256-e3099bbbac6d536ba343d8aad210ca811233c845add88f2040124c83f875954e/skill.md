---
id: "d8a1c4e9-7f2b-4d6e-9c8a-3e5f1b7d9c42"
name: clipboard-translate-to-english
version: "0.1.0"
enabled: true
display-name: Clipboard Translate to English
task: Translates copied text into clear, natural English.
usage-hint: "Copy any text to your clipboard to instantly translate it into English."
icon: globe
llm-hint: "Turns copied foreign-language text into natural English. Chain when the user wants messages, notes, or documents translated before reading, rewriting, or forwarding."
tags: [clipboard-text, translation, english, on-clipboard, llm]
trigger:
  type: on-clipboard
  match: text
permissions:
  network: llm-only
requires:
  - "official/utilities/notify"
outputs:
  - name: english_text
    type: string
    description: English translation produced by the LLM
---

# Clipboard Translate to English

Watches the clipboard for text. On trigger, translates the content into natural English (source language auto-detected when possible). Then calls `official/utilities/notify` to signal completion (same JIT `requires:` pattern as `official/productivity/clipboard-summary`).

```cwidget
1. $ENGLISH = cwidget llm ask --system "You are a professional translator. Translate the user's text into clear, natural English. Preserve meaning, tone, and approximate length. If the text is already English, lightly polish grammar only. Output only the translated English text with no preamble, labels, or explanation." --prompt $CLIPBOARD
2. cwidget ui notify $ENGLISH
3. $DONE = cwidget skill call official/utilities/notify
4. $OUT = cwidget json set "{}" --path english_text --value $ENGLISH
```
