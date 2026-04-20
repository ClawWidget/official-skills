---
id: "7347260e-899c-4496-a00d-4179166b7a8f"
name: screenshot-design-review
version: "0.1.0"
enabled: true
display-name: Screenshot Design Review
task: Critique the UI in a screenshot and put markdown on the clipboard. Fires on new screenshot files on the Desktop, or when you copy an image to the clipboard (people often say screenshots or UI feedback).
icon: palette
llm-hint: "Fires on on-screenshot (Desktop file watcher) or on-clipboard-image (copied image in clipboard). Runs llm.ask --image on your vision route, writes markdown UI critique to the clipboard, then a short notify. Returns {critique_markdown}. Requires runtime.vision route + BYOK LLM key; fs read is scoped to ~/Desktop/ only. Output is LLM-dependent (non-deterministic)."
tags:
  - featured
  - design
  - ui
  - screenshot
  - feedback
  - vision
triggers:
  - type: on-screenshot
  - type: on-clipboard-image
llm-route: vision
permissions:
  fs:
    - read: ~/Desktop/
  network: llm-only
  clipboard:
    - write
composable: true
outputs:
  - name: critique_markdown
    type: string
    description: Markdown UI/UX critique written to the clipboard for pasting
---

# Screenshot Design Review

Runs when the screenshot watcher sees a new image file (typically under `~/Desktop/`). Sends the image to your **vision** LLM route (`llm.ask --image`), writes the markdown critique to the clipboard, and shows a short notification so you can paste into your IDE.

Requires a configured vision-capable model on the `runtime.vision` route and LLM credentials in Keychain.

```cwidget
1. $CRITIQUE = cwidget llm ask --image $SCREENSHOT_PATH --system "You are an expert UI/UX designer. Analyze this screenshot and produce a concise markdown critique: one summary sentence, 3-5 specific improvement bullets ordered by impact, and a closing priority note." --prompt "Review this UI."
2. cwidget clip write $CRITIQUE
3. cwidget ui notify "UI Simplifier: Markdown ready"
4. $OUT = cwidget json set "{}" --path critique_markdown --value $CRITIQUE
```
