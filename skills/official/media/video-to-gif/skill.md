---
id: "4c9a7e2d-1f3b-4d8c-9a6e-2b7f5c1d8e44"
name: video-to-gif
version: "0.1.0"
enabled: true
display-name: Video to GIF
task: Drop a video file to convert it into a GIF for easy sharing.
icon: film
llm-hint: "Fires on file drop. Converts a local video file (mp4/mov/mkv/webm) to a GIF using ffmpeg and writes <input>.gif beside the source file. Returns {saved_path}. Local-only, no network. Use for simple clip sharing when animation format is preferred."
tags: [gif-conversion, ffmpeg, video-processing, on-drop]
trigger:
  type: on-drop
permissions:
  fs:
    - read: ~/Downloads/
    - write: ~/Downloads/
  sidecars: [ffmpeg]
  input-types: [mp4, mov, mkv, webm]
composable: true
outputs:
  - name: saved_path
    type: string
    description: Path to the generated GIF file
---

# Video to GIF

Drop a video file to convert it to a GIF.

The output is written next to the source file as `<original>.gif`.

```cwidget
1. cwidget when $DROPPED_FILE_EXT --not-matches "^(mp4|mov|mkv|webm)$" --stop --message "Expected a video file (.mp4, .mov, .mkv, or .webm)."
2. cwidget sidecar run ffmpeg "$DROPPED_FILE" "$DROPPED_FILE.gif"
3. cwidget ui notify "GIF generated."
4. $OUT = cwidget json set "{}" --path saved_path --value "$DROPPED_FILE.gif"
```
