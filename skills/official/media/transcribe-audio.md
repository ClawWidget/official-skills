---
id: "e9f5b7c3-4d6a-4e8b-ac1d-2e3f4a5b6c7d"
name: transcribe-audio
version: "0.1.0"
enabled: true
display-name: Transcribe Audio
task: Drop an audio file to transcribe it locally via Whisper. Returns a plain-text transcript.
icon: mic
llm-hint: "Fires on file drop. Transcribes a local audio file (mp3/m4a/wav/flac) using Whisper — fully local, no network. Returns {transcript}. Whisper runs the audio and returns text to stdout. Pairs with official/media/youtube-dl: download audio, then drop the saved file to transcribe. Requires whisper sidecar installed."
tags: [audio-transcription, whisper, speech-to-text, on-drop, local-processing]
trigger:
  type: on-drop
permissions:
  fs:
    - read: ~/Downloads/
  sidecars: [whisper]
  input-types: [mp3, m4a, wav, flac]
composable: true
outputs:
  - name: transcript
    type: string
    description: Plain-text transcription produced by Whisper
---

# Transcribe Audio

Drop an audio file onto the ClawWidget drop zone to transcribe it with Whisper (local, no network). Whisper returns the transcript directly to stdout.

Pairs with `official/media/youtube-dl`: download a YouTube video's audio, then drop the saved mp3 here to produce a full transcript.

```cwidget
1. cwidget when $DROPPED_FILE_EXT --not-matches "^(mp3|m4a|wav|flac)$" --stop --message "Expected an audio file (mp3, m4a, wav, or flac)."
2. $TRANSCRIPT = cwidget sidecar run whisper $DROPPED_FILE
3. cwidget ui notify "Transcription complete."
4. $OUT = cwidget json set "{}" --path transcript --value $TRANSCRIPT
```
