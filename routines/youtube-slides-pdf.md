---
name: youtube-slides-pdf
purpose: Extract the presentation slides from a YouTube video (talk, webinar, lecture) and assemble them into a single PDF.
inputs:
  - "Video URL (e.g. https://www.youtube.com/watch?v=<id>)"
  - "Scene-change threshold (default: 0.08; lower = more frames)"
  - "Crop region if slides fill only part of the frame (default: none)"
  - "Output directory (default: C:\\Users\\Andreas Tinhofer\\Downloads)"
when-to-use: When a recorded talk, webinar, or lecture shows slides on screen and you want them as a browsable PDF instead of scrubbing through the video.
---

# YouTube Slides to PDF

Download a YouTube video, detect the moments where the slide on
screen changes, save one clean frame per slide, remove near-duplicate
frames, and bind the result into one PDF named after the video.
Requires `yt-dlp`, `ffmpeg`, and Python with `img2pdf`, `Pillow`, and
`imagehash`.

Unless the user names a different output directory, `{{OUTDIR}}` is
`C:\Users\Andreas Tinhofer\Downloads`. The path contains a space:
always quote it in commands. Work in a temporary subfolder
(`{{OUTDIR}}\slides-<video-id>\`) so intermediate frames never
clutter the output directory; only the finished PDF stays. After
step 3, run all commands from inside that subfolder so the relative
paths (`video.mp4`, `frames/`) resolve.

## Steps

1. **Check the tools.** Verify `yt-dlp`, `ffmpeg`, and the Python
   packages are installed. If not, install them first. Windows
   (PowerShell):

   ```powershell
   winget install Gyan.FFmpeg yt-dlp.yt-dlp
   pip install img2pdf pillow imagehash
   ```

   (macOS/Linux: `brew install ffmpeg yt-dlp`, then the same `pip`
   line.) Confirm the versions afterwards.
2. **Probe the video.** Run:

   ```bash
   yt-dlp --skip-download \
     --print "%(title)s | %(duration)s | %(width)sx%(height)s" "{{URL}}"
   ```

   Note the title (it names the PDF) and the duration (sanity check
   for the frame count later). If the probe fails with 403/login,
   retry with `--cookies-from-browser firefox` — the user must be
   logged in and entitled to watch. Stop if the video is not
   accessible.
3. **Download video only** — audio is not needed and skipping it is
   faster:

   ```bash
   yt-dlp -f "bestvideo[height<=1080]" \
     -o "{{OUTDIR}}/slides-%(id)s/video.%(ext)s" "{{URL}}"
   ```

4. **Extract one frame per slide change** with ffmpeg's scene
   detection. `cd` into the temp subfolder first; ffmpeg does not
   create output directories, so create `frames/` before the call:

   ```bash
   mkdir frames
   ffmpeg -i video.mp4 \
     -vf "select='gt(scene,{{THRESHOLD}})',showinfo" \
     -fps_mode vfr frames/%04d.png
   ```

   Also grab the very first frame — scene detection only fires on
   *changes*, so the opening slide is missed otherwise. Write it to
   `frames/0000.png` (the detection run starts numbering at `0001`,
   so this sorts first without overwriting anything):

   ```bash
   ffmpeg -i video.mp4 -vf "select='eq(n,0)'" \
     -frames:v 1 frames/0000.png
   ```
 Compare the frame count against the video: a 45-minute
   talk has maybe 30–80 slides. Hundreds of frames → the video has
   animations or an embedded webcam; raise the threshold (0.2–0.3)
   or crop first (step 5). Only a handful → lower it (0.03–0.05).
   Fallback if scene detection stays unusable: sample steadily with
   `-vf "fps=1/10"` (one frame every 10 s) and let step 6 dedupe.
5. **Crop if needed.** If the slides occupy only part of the frame
   (speaker video beside the slides, letterboxing), find the slide
   region by inspecting one extracted frame, then re-run step 4 with
   the crop in front of the select filter:
   `-vf "crop=W:H:X:Y,select='gt(scene,{{THRESHOLD}})'"`.
   Cropping before detection also stops the webcam window from
   triggering false slide changes.
6. **Remove near-duplicates.** Animated bullet points and progress
   bars produce several frames of the same slide. Dedupe with a
   perceptual hash, keeping the *last* frame of each group (the
   fully built slide):

   ```python
   from pathlib import Path
   from PIL import Image
   import imagehash

   frames = sorted(Path("frames").glob("*.png"))
   keep, prev = [], None
   for f in frames:
       h = imagehash.phash(Image.open(f))
       if prev is not None and h - prev <= 6:
           keep[-1] = f          # same slide, keep the later frame
       else:
           keep.append(f)
       prev = h
   ```

   Tune the distance (6) if slides are wrongly merged (lower it) or
   duplicates survive (raise it). Note that only *adjacent* frames
   are compared: a very slow animated build can drift past the
   threshold across many frames and still leave a duplicate — raise
   the distance if that happens.
7. **Assemble the PDF** from the kept frames, in order:

   ```python
   import img2pdf
   pdf = Path(r"{{OUTDIR}}") / f"{title}.pdf"
   pdf.write_bytes(img2pdf.convert([str(f) for f in keep]))
   ```

   Sanitize the title for use as a filename (strip `\ / : * ? " < > |`).
8. **Verify and report.** Open the PDF and spot-check it against 2–3
   points in the video (beginning, middle, end): are those slides in
   the PDF, sharp, uncropped, and in the right order? Then delete the
   temporary subfolder (video + frames) and report using the output
   format below.

## Output format

```
## Slides extracted — <date>

- Source: <URL>
- Title: <video title>
- Video: <duration>, <resolution>
- Frames extracted: <n> (threshold <t>) → after dedupe: <pages>
- Crop applied: <W:H:X:Y | none>
- PDF: <path> (<pages> pages, <file size>)

### Quality notes
- <spot-check results, blurry pages, missed or merged slides, …>

### Open items
- <e.g. "pages 12-14 show the speaker overlay", or "none">
```

## Guardrails

- Only process videos the user is entitled to watch. If the video is
  private, members-only, DRM-protected, or the probe indicates
  restricted access, stop and ask instead of trying to bypass it.
- The slides remain the presenter's work. Extract them for the
  user's personal reference only — do not publish, redistribute, or
  upload the PDF anywhere without the user asking explicitly.
- Never commit the video, frames, or PDF to a repository — keep them
  in the output directory only.
- Do not overwrite an existing PDF of the same name without asking;
  suffix with the video id instead.
- One download per run — if it fails, resume with yt-dlp's continue
  behavior rather than starting parallel downloads.
- Delete the downloaded video and intermediate frames after the PDF
  is verified (step 8); ask first only if the user said they want to
  keep the video.
