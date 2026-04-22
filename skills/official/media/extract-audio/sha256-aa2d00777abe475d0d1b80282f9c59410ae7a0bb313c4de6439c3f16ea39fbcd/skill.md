---
id: "3f9a2c7d-1b6e-4a8f-9d3c-6e2b4f1a7c55"
name: extract-audio
version: "0.1.0"
enabled: true
display-name: Extract Audio
task: Pulls a clean audio track from any dropped video.
usage-hint: "Drop a video into ClawWidget to extract an .m4a audio file beside the original."
icon: music
llm-hint: "Pulls audio tracks out of videos as a clean intermediate step. Chain when the user wants speech, interviews, or podcasts extracted before transcription, clipping, or reuse."
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
