# YouTube AnyCaption Summarizer

A reliable YouTube summarizer for the real world.

This skill turns YouTube videos into clean markdown transcripts and polished summaries, even when captions are messy, incomplete, auto-generated, or missing entirely.

## Why people like this skill

Most YouTube summary tools break down when subtitle quality is bad. This one is built to keep going.

It tries the best available source in this order:
- **Manual captions first** for the cleanest result
- **Auto-captions next** when manual captions are missing
- **Local Whisper transcription** when the video has no usable captions

That means it works well for:
- founder videos
- tutorials and walkthroughs
- product demos
- technical explainers
- private or restricted videos
- batch research workflows

## What you get

For each video, the skill produces:
- a **raw markdown transcript**
- a **polished markdown summary**
- a **session-ready result block** for OpenClaw workflows

Output files:
- `SANITIZED_VIDEO_NAME_transcript_raw.md`
- `SANITIZED_VIDEO_NAME_Summary.md`

## What makes it special

- Works even when captions are weak or missing
- Handles manual CC, auto-captions, and Whisper fallback automatically
- Supports **private / restricted YouTube videos** with cookies or browser-cookie extraction
- Supports **single-video** and **batch** workflows
- Creates durable markdown files you can save, search, and reuse later
- Gives cleaner completion reporting for OpenClaw sessions
- Better suited for serious research and implementation notes than one-off chat summaries

## Best use cases

Use this skill when you want to:
- summarize a YouTube video into something you can actually keep
- process private/internal YouTube uploads
- batch process multiple videos in one run
- turn video content into research notes or implementation docs
- get a summary in the source language or a chosen target language

## Quick install on macOS

```bash
brew install yt-dlp ffmpeg whisper-cpp
MODELS_DIR="$HOME/.openclaw/workspace"
MODEL_PATH="$MODELS_DIR/ggml-medium.bin"
mkdir -p "$MODELS_DIR"
if [ ! -f "$MODEL_PATH" ]; then
  curl -L https://huggingface.co/ggerganov/whisper.cpp/resolve/main/ggml-medium.bin \
    -o "$MODEL_PATH.part" && mv "$MODEL_PATH.part" "$MODEL_PATH"
else
  echo "Model already exists at $MODEL_PATH, leaving it unchanged."
fi
command -v python3 yt-dlp ffmpeg whisper-cli
ls -lh "$MODEL_PATH"
```

This setup is intentionally safe:
- it does not overwrite your workspace
- it does not edit your OpenClaw config
- it does not replace the model if it already exists

## Quick start

### Single video

```bash
python3 scripts/run_youtube_workflow.py "https://www.youtube.com/watch?v=VIDEO_ID"
```

### Choose a target summary language

```bash
python3 scripts/run_youtube_workflow.py "https://www.youtube.com/watch?v=VIDEO_ID" \
  --summary-language zh-CN
```

### Private or restricted video

```bash
python3 scripts/run_youtube_workflow.py "https://www.youtube.com/watch?v=VIDEO_ID" \
  --cookies-from-browser chrome
```

### Batch mode

If you have multiple videos, put them in a text file first, one URL per line:

```text
https://www.youtube.com/watch?v=VIDEO_ID_1
https://www.youtube.com/watch?v=VIDEO_ID_2
```

Then run:

```bash
python3 scripts/run_youtube_batch_end_to_end.py --batch-file ./youtube-urls.txt
```

## How it works

At a high level, the skill:
1. fetches video metadata first
2. creates safe output paths
3. tries captions before transcription
4. falls back to local Whisper when needed
5. writes the raw transcript
6. generates the final summary
7. returns a clean result block for the session

## Why this is better than a basic transcript tool

A lot of tools can extract subtitles when everything is perfect.

This skill is better when reality is messy:
- some videos have great captions
- some only have auto-generated captions
- some have no usable captions at all
- some are private or restricted

Instead of failing early, this skill is designed to keep moving and still produce useful output.

## Example requests

- “Summarize this YouTube video into markdown.”
- “Generate a transcript and polished summary for this YouTube link.”
- “Process this private YouTube video with my browser cookies.”
- “Batch summarize these YouTube links.”
- “Use subtitles when available, otherwise transcribe locally.”
- “Create a Chinese summary from this English YouTube video.”

## Notes

- Default output folder: `~/Downloads`
- Default Whisper model: `ggml-medium`
- Works best when `yt-dlp`, `ffmpeg`, `whisper-cli`, and `python3` are available on your machine
- For deeper workflow details, see `references/detailed-workflow.md`

## ClawHub

- https://clawhub.ai/arthurli202602-commits/youtube-anycaption-summarizer

## License

MIT-0
