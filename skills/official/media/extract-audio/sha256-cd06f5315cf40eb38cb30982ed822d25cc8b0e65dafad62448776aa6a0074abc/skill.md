---
id: "3f9a2c7d-1b6e-4a8f-9d3c-6e2b4f1a7c55"
name: extract-audio
version: "0.1.0"
enabled: true
display-name: Extract Audio
task: Drop video files to convert each into an .m4a audio file using ffmpeg.
icon: music
llm-hint: "Fires on file drop. Converts local video files (mp4/mov/mkv/webm) to .m4a using ffmpeg and writes <input>.m4a beside each source file. Returns {saved_path}. Local-only, no network. Commonly used before official/media/transcribe-audio."
tags: [audio-extraction, ffmpeg, video-to-audio, on-drop, m4a-output]
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
    description: Path to the extracted audio file
---

# Extract Audio

Drop a video file to convert it to `.m4a` using ffmpeg.

The output is written next to the source file as `<original>.m4a`.

```cwidget
1. cwidget when $DROPPED_FILE_EXT --not-matches "^(mp4|mov|mkv|webm)$" --stop --message "Expected a video file (.mp4, .mov, .mkv, or .webm)."
2. cwidget sidecar run ffmpeg "$DROPPED_FILE" "$DROPPED_FILE.m4a"
3. cwidget ui notify "Audio extracted."
4. $OUT = cwidget json set "{}" --path saved_path --value "$DROPPED_FILE.m4a"
```
