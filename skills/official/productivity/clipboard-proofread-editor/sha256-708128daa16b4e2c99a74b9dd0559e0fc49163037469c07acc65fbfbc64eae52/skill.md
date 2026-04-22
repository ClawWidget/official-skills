---
id: "e8046ae1-8386-4b95-8b89-7cb370d47fc6"
name: clipboard-proofread-editor
version: "0.1.0"
enabled: true
display-name: Clipboard Proofread Editor
task: Opens copied text beside an editable proofread draft before you confirm it.
usage-hint: "Copy any draft to your clipboard to open an editable proofread window."
icon: clipboard-check
llm-hint: "Fires on copied text, drafts a proofread version with llm.ask, then opens source_editor with the original clipboard text on the left and the editable proofread draft on the right. Use it when the user wants grammar cleanup without losing visibility into the source."
tags: [clipboard-text, grammar-fix, proofread, on-clipboard, llm, source-editor]
trigger:
  type: on-clipboard
  match: text
present:
  - source_editor
permissions:
  network: llm-only
---

# Clipboard Proofread Editor

Turns copied text into an editable proofread draft while keeping the original text visible in `source_editor`.

```cwidget
1. $SOURCE_TEXT = cwidget text trim $CLIPBOARD
2. cwidget when $SOURCE_TEXT --eq "" --stop --message "Copy some text before running Clipboard Proofread Editor."
3. $DRAFT = cwidget llm ask --system "You are a careful proofreader. Fix grammar, spelling, and punctuation only. Preserve meaning, voice, and structure. Output only the corrected text with no preamble or explanation." --prompt $SOURCE_TEXT
4. $RUN_ID = cwidget concat "clipboard-proofread-editor-" "$NOW_ISO"
5. $SOURCE = cwidget json parse '{"kind":"text"}'
6. $SOURCE = cwidget json set $SOURCE --path text --value $SOURCE_TEXT
7. $VALUES = cwidget json parse '{"left_title":"Original text","right_title":"Proofread draft"}'
8. $VALUES = cwidget json set $VALUES --path source --value $SOURCE
9. $VALUES = cwidget json set $VALUES --path draft_text --value $DRAFT
10. $VALUES = cwidget json set $VALUES --path summary --value "Edit the proofread draft before you confirm it."
11. $VALUES = cwidget json set $VALUES --path draft_instructions --value "Return plain text only. Preserve the original meaning and tone."
12. $PAYLOAD = cwidget json parse '{"layout":"source_editor"}'
13. $PAYLOAD = cwidget json set $PAYLOAD --path values --value $VALUES
14. $HANDOFF = cwidget json parse '{"layout":"source_editor","run_id":"","triggered_by":{"kind":"user_gesture","trigger":"on-clipboard"}}'
15. $HANDOFF = cwidget json set $HANDOFF --path payload --value $PAYLOAD
16. $HANDOFF = cwidget json set $HANDOFF --path run_id --value $RUN_ID
```
