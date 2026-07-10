---
name: livestream-transcribe
purpose: Record a livestream (Vimeo, YouTube, or any HLS source) with yt-dlp/ffmpeg and produce a transcript with Whisper.
inputs:
  - "Stream URL (e.g. https://vimeo.com/<id>)"
  - "Spoken language of the stream (default: de)"
  - "Whisper model size (default: medium; use large-v3 for best quality)"
  - "Output directory for recording + transcript (default: ./recordings)"
when-to-use: When a webinar, hearing, conference talk, or other livestream must be captured and turned into a text transcript — live while it runs, or afterwards from the replay/VOD.
---

# Livestream: Record and Transcribe

Capture a livestream to a local file and transcribe it to text with
timestamps. Works for streams that are upcoming, currently live, or
already finished (replay/VOD). Requires `yt-dlp`, `ffmpeg`, and a
Whisper CLI (`whisper-ctranslate2` recommended — the faster-whisper
backend with a command line — or `openai-whisper` / `whisper.cpp`).

## Steps

1. **Check the tools.** Verify `yt-dlp`, `ffmpeg`, and a Whisper CLI
   are installed. If not, install them first. Windows (PowerShell):

   ```powershell
   winget install Gyan.FFmpeg yt-dlp.yt-dlp
   pip install whisper-ctranslate2
   ```

   (macOS/Linux: `brew install ffmpeg`, `pipx install yt-dlp
   whisper-ctranslate2`.) Confirm the versions afterwards.
2. **Probe the stream.** Run:

   ```bash
   yt-dlp --skip-download \
     --print "%(title)s | %(live_status)s | %(duration)s" "{{URL}}"
   ```

   `live_status` decides the path: `is_upcoming` → step 3,
   `is_live` → step 4, `was_live`/`not_live` (VOD) → step 5.
   If the probe fails with 403/login, retry with
   `--cookies-from-browser <browser>` (the user must be logged in and
   entitled to watch) and, for embedded players, add
   `--referer "<page the player is embedded on>"`. On Windows prefer
   `--cookies-from-browser firefox` — Chrome/Edge encrypt their
   cookie store in a way yt-dlp often cannot read.
3. **Upcoming stream:** report the scheduled start time and wait, or
   tell the user when to re-run. `yt-dlp --wait-for-video 60 "{{URL}}"`
   polls every 60 s and starts recording automatically at go-live.
4. **Live stream:** record from *now* until the stream ends:

   ```bash
   yt-dlp -o "{{OUTDIR}}/%(title)s.%(ext)s" "{{URL}}"
   ```

   Note: `--live-from-start` only works for YouTube. On Vimeo and
   generic HLS, recording begins at the moment yt-dlp connects —
   start it before the stream begins (combine with
   `--wait-for-video`). Fallback if the extractor struggles:
   `yt-dlp -g "{{URL}}"` to get the HLS manifest, then
   `ffmpeg -i "<m3u8-url>" -c copy recording.mp4`.
5. **Finished stream (VOD/replay):** a plain
   `yt-dlp -o "{{OUTDIR}}/%(title)s.%(ext)s" "{{URL}}"` downloads it.
   If only the transcript matters, save bandwidth with
   `yt-dlp -f bestaudio -x --audio-format m4a`.
6. **Extract audio** (skip if step 5 already produced audio-only):

   ```bash
   ffmpeg -i recording.mp4 -vn -ac 1 -ar 16000 audio.wav
   ```

7. **Transcribe** with Whisper, pinning the language:

   ```powershell
   whisper-ctranslate2 audio.wav --language {{LANG}} --model {{MODEL}} --output_format all
   # or: whisper audio.wav --language {{LANG}} --model {{MODEL}}
   ```

   For multi-hour recordings prefer `whisper-ctranslate2` or
   `whisper.cpp` (much faster on CPU than openai-whisper). Keep both
   the `.txt` (plain text) and `.srt` (timestamps) outputs.
8. **Verify and report.** Spot-check the transcript against 2–3 random
   points in the recording (names, numbers, technical terms). Report
   using the output format below.

## Output format

```
## Livestream capture — <date>

- Source: <URL>
- Title: <stream title>
- Status at capture: <is_live | was_live | recorded from start? >
- Recording: <path> (<duration>, <file size>)
- Audio: <path>
- Transcript: <path>.txt / <path>.srt
- Whisper model: <model> (language: <lang>)

### Quality notes
- <spot-check results, sections with poor audio, speaker overlap, …>

### Open items
- <e.g. "stream still running, recording continues", or "none">
```

## Guardrails

- Only record streams the user is entitled to watch and permitted to
  record. If the stream is password-protected, DRM-protected, or the
  probe indicates restricted access, stop and ask instead of trying
  to bypass it.
- Never commit recordings, audio files, transcripts, or browser
  cookies to a repository — they are large and often confidential.
  Keep them in the output directory only.
- Do not upload the recording or transcript to any external service
  (including cloud transcription APIs) without explicit approval;
  default to local Whisper.
- Do not hammer the source: one probe, one recording session. If the
  connection drops, resume with yt-dlp's continue behavior rather
  than starting parallel downloads.
- Transcripts are raw ASR output — mark them as unreviewed and never
  present them as verbatim quotes without the spot-check in step 8.
