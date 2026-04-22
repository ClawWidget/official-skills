---
id: "d8e4a6b2-3c5f-4d7a-9e0f-1b2c3d4e5f6a"
name: youtube-dl
version: "0.1.0"
enabled: true
display-name: YouTube Audio Downloader
task: Saves audio from YouTube links for offline listening and reuse.
usage-hint: "Copy a YouTube link to your clipboard to save its audio to Downloads."
icon: download
llm-hint: "Pulls audio from copied YouTube links for downstream listening or processing. Chain when the user wants a YouTube video's audio saved before transcription, summarization, or clipping."
tags: [youtube, video-download, audio-download, clipboard-url, media-ingest]
trigger:
  type: on-clipboard
  match: url
  match-regex: '^(https?://)?((www|m)\.)?(youtube\.com|youtu\.be)/'
permissions:
  clipboard: read
  network: domains
  network-domains:
    - youtube.com
    - "*.youtube.com"
    - youtu.be
  sidecars: [yt-dlp]
  fs:
    - write: ~/Downloads/
composable: true
outputs:
  - name: saved_dir_path
    type: string
    description: Directory path where audio was written (~/Downloads/); this is a directory path, not a file path
---

# YouTube Audio Downloader

Watches the clipboard for YouTube URLs. On match, downloads the audio track to `~/Downloads/` using `yt-dlp` and returns the download directory path.

To build a download-then-transcribe workflow: install both `official/media/youtube-dl` and `official/media/transcribe-audio`, then choose a concrete file from `saved_dir_path` and pass that file path to transcription. `saved_dir_path` is a directory path (`~/Downloads/`), not the final mp3 file path.

```cwidget
1. $URL = cwidget clip read
2. $VALID = cwidget text match $URL --regex "^(https?://)?(www\.)?youtu"
3. cwidget when $VALID --eq "false" --stop --message "No valid YouTube URL in clipboard."
4. cwidget sidecar run yt-dlp $URL --extract-audio --audio-format mp3 --output "~/Downloads/%(title)s.%(ext)s"
5. cwidget ui notify "Audio saved to ~/Downloads/"
6. $OUT = cwidget json set "{}" --path saved_dir_path --value "~/Downloads/"
```
