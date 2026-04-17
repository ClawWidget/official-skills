---
id: "d8e4a6b2-3c5f-4d7a-9e0f-1b2c3d4e5f6a"
name: youtube-dl
version: "0.1.0"
enabled: true
display-name: YouTube Audio Downloader
task: Downloads audio from a YouTube URL copied to the clipboard into ~/Downloads/.
icon: download
llm-hint: "Fires when a YouTube URL is copied. Downloads audio to ~/Downloads/ via yt-dlp. Returns {saved_path} — the download directory (~/Downloads/); the exact filename is not surfaced without registry flags not in the contract. Compose with official/transcribe-audio by dropping the saved file onto the ClawWidget drop zone to produce a transcript. Single-video only — not for playlists. Requires yt-dlp sidecar."
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
  - name: saved_path
    type: string
    description: Directory where audio was written (~/Downloads/); exact filename is not in the contract surface for v1
---

# YouTube Audio Downloader

Watches the clipboard for YouTube URLs. On match, downloads the audio track to `~/Downloads/` using `yt-dlp` and returns the download directory path.

To build a download-then-transcribe workflow: install both `official/youtube-dl` and `official/transcribe-audio`, then drop the saved mp3 onto the ClawWidget drop zone.

```cwidget
1. $URL = cwidget clip read
2. $VALID = cwidget text match $URL --regex "^(https?://)?(www\.)?youtu"
3. cwidget when $VALID --eq "false" --stop --message "No valid YouTube URL in clipboard."
4. cwidget sidecar run yt-dlp $URL --extract-audio --audio-format mp3 --output "~/Downloads/%(title)s.%(ext)s"
5. cwidget ui notify "Audio saved to ~/Downloads/"
6. $OUT = cwidget json set "{}" --path saved_path --value "~/Downloads/"
```
