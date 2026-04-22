---
id: "e2f5bc53-2930-4c2d-8a7f-1bcbdbf8bb4b"
name: screenshot-design-review-editor
version: "0.1.0"
enabled: true
display-name: Screenshot Design Review Editor
task: Opens an editable screenshot critique beside the exact image that triggered it.
usage-hint: "Copy an image or take a screenshot to open an editable design-review draft."
icon: palette
llm-hint: "Fires on on-clipboard-image immediately or on-screenshot through the background queue. Runs llm.ask --image on the vision route to draft a UI critique, then opens source_editor with the local screenshot on the left and an editable plain-text critique on the right. The draft stays local-first and inspectable."
tags:
  - featured
  - design
  - screenshot
  - vision
  - atomic-windows
  - source-editor
triggers:
  - type: on-clipboard-image
  - type: on-screenshot
present:
  - source_editor
llm-route: vision
permissions:
  fs:
    - read: ~/Desktop/
    - read: ~/.cwidget/tmp/
  network: llm-only
---

# Screenshot Design Review Editor

Builds a design-review draft from a copied image or screenshot, then opens `source_editor` so the original image stays visible while you refine the critique before confirming it.

```cwidget
1. $CRITIQUE = cwidget llm ask --image $SCREENSHOT_PATH --system "You are an expert UI/UX designer. Analyze this screenshot and produce a concise plain-text critique: one summary sentence, 3-5 specific improvement bullets ordered by impact, and a closing priority note." --prompt "Review this UI and draft an editable critique."
2. $RUN_ID = cwidget concat "screenshot-design-review-editor-" "$NOW_ISO"
3. $SOURCE = cwidget json parse '{"kind":"image"}'
4. $SOURCE = cwidget json set $SOURCE --path image_path --value $SCREENSHOT_PATH
5. $VALUES = cwidget json parse '{"left_title":"Screenshot","right_title":"Editable critique"}'
6. $VALUES = cwidget json set $VALUES --path source --value $SOURCE
7. $VALUES = cwidget json set $VALUES --path draft_text --value $CRITIQUE
8. $VALUES = cwidget json set $VALUES --path summary --value "Refine the critique before you confirm it."
9. $VALUES = cwidget json set $VALUES --path draft_instructions --value "Return plain text only. Keep the critique concise and actionable."
10. $PAYLOAD = cwidget json parse '{"layout":"source_editor"}'
11. $PAYLOAD = cwidget json set $PAYLOAD --path values --value $VALUES
12. $HANDOFF = cwidget json parse '{"layout":"source_editor","run_id":"","triggered_by":{"kind":"user_gesture","trigger":"on-clipboard-image"}}'
13. $HANDOFF = cwidget json set $HANDOFF --path payload --value $PAYLOAD
14. $HANDOFF = cwidget json set $HANDOFF --path run_id --value $RUN_ID
```
