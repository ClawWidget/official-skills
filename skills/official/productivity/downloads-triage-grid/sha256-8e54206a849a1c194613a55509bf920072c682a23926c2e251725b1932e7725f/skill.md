---
id: "d14ab16f-e1dc-4ec8-93b0-1e77f6bb5451"
name: downloads-triage-grid
version: "0.1.0"
enabled: true
display-name: Downloads Triage Grid
task: Opens a local multi-select triage grid for files in Downloads.
usage-hint: "Run it manually when Downloads needs a quick local review pass."
icon: table-properties
tags: [atomic-windows, omni-grid, local-files, downloads, triage]
trigger:
  type: manual
present:
  - omni_grid
permissions:
  fs:
    - list: ~/Downloads/
---

# Downloads Triage Grid

Turns the current contents of `~/Downloads/` into a local triage grid so you can review multiple files together.

```cwidget
1. $FILES = cwidget fs list ~/Downloads/
2. cwidget when $FILES --eq "" --stop --message "No local files found in ~/Downloads/."
3. $ROW_IDS = cwidget text split "$FILES" --on "\n"
4. $RUN_ID = cwidget concat "downloads-triage-grid-" "$NOW_ISO"
5. $HANDOFF = cwidget concat '{"layout":"omni_grid","payload":{"layout":"omni_grid","values":{"title":"Downloads inbox triage","row_ids":' "$ROW_IDS" ',"allow_multi_select":true}},"run_id":"' "$RUN_ID"
6. $HANDOFF = cwidget concat "$HANDOFF" '","triggered_by":{"kind":"user_gesture","trigger":"manual"}}'
```
