---
id: "31d0713e-895b-489d-ae57-5403f97de501"
name: downloads-quick-find
version: "0.1.0"
enabled: true
display-name: Downloads Quick Find
task: Opens a fast local chooser for files already in Downloads.
usage-hint: "Use the hotkey to search local files in ~/Downloads/ and pick one quickly."
icon: search
tags: [atomic-windows, atomic-search, local-files, downloads, hotkey]
trigger:
  type: on-hotkey
present:
  - atomic_search
permissions:
  fs:
    - list: ~/Downloads/
---

# Downloads Quick Find

Builds a local-only search checkpoint from the current contents of `~/Downloads/`.

```cwidget
1. $FILES = cwidget fs list ~/Downloads/
2. cwidget when $FILES --eq "" --stop --message "No local files found in ~/Downloads/."
3. $RESULT_IDS = cwidget text split "$FILES" --on "\n"
4. $RUN_ID = cwidget concat "downloads-quick-find-" "$NOW_ISO"
5. $HANDOFF = cwidget concat '{"layout":"atomic_search","payload":{"layout":"atomic_search","values":{"query":"","result_ids":' "$RESULT_IDS" ',"placeholder":"Search local files in Downloads"}},"run_id":"' "$RUN_ID"
6. $HANDOFF = cwidget concat "$HANDOFF" '","triggered_by":{"kind":"user_gesture","trigger":"on-hotkey"}}'
```
