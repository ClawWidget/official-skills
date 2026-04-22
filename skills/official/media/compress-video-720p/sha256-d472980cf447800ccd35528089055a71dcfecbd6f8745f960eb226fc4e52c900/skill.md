---
id: "5a8d2f3b-9e41-4c6a-b7d9-1f2e3a4b5c66"
name: compress-video-720p
version: "0.1.0"
enabled: true
display-name: Compress Video to 720p
task: Creates a smaller 720p MP4 copy for easier sharing.
usage-hint: "Drop a video into ClawWidget to create a smaller 720p MP4 beside the original."
icon: clapperboard
llm-hint: "Creates smaller 720p MP4 share copies from videos. Chain when the user wants videos compressed for sending, uploading, or easier playback across devices."
tags: [video-compression, ffmpeg, mp4-output, on-drop, share-copy]
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
    description: Path to the compressed 720p video file
---

# Compress Video to 720p

Drop a video file to create a smaller MP4 share copy.

The output is written next to the source file as `<original>-720p.mp4`.

```cwidget
1. cwidget when $DROPPED_FILE_EXT --not-matches "^(mp4|mov|mkv|webm)$" --stop --message "Expected a video file (.mp4, .mov, .mkv, or .webm)."
2. cwidget sidecar run ffmpeg "$DROPPED_FILE" "$DROPPED_FILE-720p.mp4"
3. cwidget ui notify "720p video generated."
4. $OUT = cwidget json set "{}" --path saved_path --value "$DROPPED_FILE-720p.mp4"
```
