---
id: "7347260e-899c-4496-a00d-4179166b7a8f"
name: screenshot-design-review
version: "0.1.0"
enabled: true
display-name: Screenshot Design Review
task: Critique the UI in a screenshot, save a markdown file to `~/Documents/authoring/`, and copy the critique to the clipboard. Fires on new screenshot files on the Desktop, or when you copy an image to the clipboard (people often say screenshots or UI feedback).
usage-hint: "Take a screenshot or copy an image; the critique saves to Documents and you get a notification."
icon: palette
llm-hint: "Fires on on-screenshot (Desktop file watcher) or on-clipboard-image (copied image in clipboard). Runs llm.ask --image on your vision route, writes a markdown UI critique to ~/Documents/authoring/latest-critique.md and to the clipboard, then emits a completion notification with file actions. Returns {critique_file}. Requires runtime.vision route + BYOK LLM key; fs read is scoped to ~/Desktop/ only and fs write is scoped to ~/Documents/authoring/. Output is LLM-dependent (non-deterministic)."
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
    - write: ~/Documents/authoring/
  network: llm-only
  clipboard:
    - write
composable: true
outputs:
  - name: critique_file
    type: file-path
    description: Markdown UI/UX critique written to disk
notify:
  on-success: true
  title: "Screenshot Design Review"
  body: "Markdown critique ready"
  actions:
    - label: "Show in Finder"
      reveal: critique_file
    - label: "Open"
      open: critique_file
---

# Screenshot Design Review

Runs when the screenshot watcher sees a new image file (typically under `~/Desktop/`). Sends the image to your **vision** LLM route (`llm.ask --image`), writes the markdown critique to `~/Documents/authoring/latest-critique.md`, copies it to the clipboard, and emits a completion notification with Finder/Open actions.

Requires a configured vision-capable model on the `runtime.vision` route and LLM credentials in Keychain.

```cwidget
1. $CRITIQUE = cwidget llm ask --image $SCREENSHOT_PATH --system "You are an expert UI/UX designer. Analyze this screenshot and produce a concise markdown critique: one summary sentence, 3-5 specific improvement bullets ordered by impact, and a closing priority note." --prompt "Review this UI."
2. cwidget clip write $CRITIQUE
3. cwidget save $CRITIQUE --as latest-critique.md --to ~/Documents/authoring/
4. cwidget ui notify "Screenshot Design Review: Markdown ready"
5. $OUT = cwidget json set "{}" --path critique_file --value ~/Documents/authoring/latest-critique.md
```
