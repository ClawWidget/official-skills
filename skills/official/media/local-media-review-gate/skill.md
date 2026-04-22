---
id: "2e2114bc-f88b-43ff-92d9-66c686b31163"
name: local-media-review-gate
version: "0.1.0"
enabled: true
display-name: Local Media Review Gate
task: Routes local media through an approval card or a trim checkpoint before follow-up work.
usage-hint: "Right-click a local file for a quick review choice, or drop media to set a local trim range."
icon: clapperboard
tags: [atomic-windows, action-feed, media-trimmer, local-media, approval]
triggers:
  - type: on-context-menu
    label: Review in ClawWidget
  - type: on-drop
present:
  - action_feed
  - media_trimmer
permissions:
  fs:
    - read: ~/
  input-types: [mp3, m4a, wav, flac, mp4, mov, mkv, webm]
---

# Local Media Review Gate

Uses an Atomic Window checkpoint before media follow-up: quick approval from Finder, or a local trim-range selection when media is dropped in.

```cwidget
1. cwidget when $EVENT_TRIGGER_TYPE --eq "on-drop" --skip-to 7
2. $SUMMARY = cwidget concat "Choose the next local step for " "$CONTEXT_FILE_STEM"
3. $ACTION_VALUES = cwidget concat '{"summary":"' "$SUMMARY" '","action_ids":["approve","trim_later","skip"]}'
4. $RUN_ID = cwidget concat "local-media-review-action-feed-" "$NOW_ISO"
5. $HANDOFF = cwidget concat '{"layout":"action_feed","payload":{"layout":"action_feed","values":' "$ACTION_VALUES" '},"run_id":"' "$RUN_ID"
6. $HANDOFF = cwidget concat "$HANDOFF" '","triggered_by":{"kind":"user_gesture","trigger":"on-context-menu"}}'
7. cwidget when $DROPPED_FILE_EXT --not-matches "^(mp3|m4a|wav|flac|mp4|mov|mkv|webm)$" --stop --message "Expected a local audio or video file."
8. $PAYLOAD = cwidget concat '{"layout":"media_trimmer","values":{"media_path":"' "$DROPPED_FILE" '","start_seconds":0,"end_seconds":15}}'
9. $RUN_ID = cwidget concat "local-media-review-trimmer-" "$NOW_ISO"
10. $HANDOFF = cwidget concat '{"layout":"media_trimmer","payload":' "$PAYLOAD" ',"run_id":"' "$RUN_ID"
11. $HANDOFF = cwidget concat "$HANDOFF" '","triggered_by":{"kind":"user_gesture","trigger":"on-drop"}}'
```
