---
id: "a1b7d9e5-6f8c-4a0d-8e3f-4a5b6c7d8e9f"
name: clipboard-summary
version: "0.1.0"
enabled: true
display-name: Clipboard Summary
task: Turns any copied text into a fast, readable bulleted summary.
usage-hint: "Copy any text to your clipboard to instantly generate a bulleted summary."
icon: clipboard-list
llm-hint: "Condenses copied text into a fast bullet summary. Chain when the user wants notes, articles, emails, or transcripts reduced to the main points."
tags: [clipboard-text, llm-summary, bullet-summary, on-clipboard, requires-chain]
trigger:
  type: on-clipboard
  match: text
permissions:
  network: llm-only
requires:
  - "official/utilities/notify"
outputs:
  - name: summary
    type: string
    description: The 2-3 bullet summary produced by the LLM
---

# Clipboard Summary

Watches the clipboard for text. On trigger, summarizes the content into 2-3 bullet points via LLM, then calls `official/utilities/notify` to signal completion.

This is the first `requires:` chain in the catalog. Installing this skill automatically JIT-fetches `official/utilities/notify` from the registry at install time — no manual step needed.

```cwidget
1. $SUMMARY = cwidget llm ask --system "You are a concise summarizer. Respond with exactly 2-3 bullet points using plain-text dashes. No intro, no outro." --prompt $CLIPBOARD
2. cwidget ui notify $SUMMARY
3. $DONE = cwidget skill call official/utilities/notify
4. $OUT = cwidget json set "{}" --path summary --value $SUMMARY
```
