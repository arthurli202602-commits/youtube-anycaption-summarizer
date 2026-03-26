---
name: youtube-anycaption-summarizer
description: "Turn YouTube videos into dependable markdown transcripts and polished summaries — even when caption coverage is messy. This skill works with manual closed captions (CC), auto-generated subtitles, or no usable subtitles at all by using subtitle-first extraction with local Whisper fallback. Supports private/restricted videos via cookies, batch processing, transcript cleanup, language backfill, source-language or user-selected summary language, and end-to-end completion reporting. Ideal for YouTube research, technical walkthroughs, founder content, tutorials, private/internal uploads, and batch video summarization workflows."
homepage: "https://github.com/arthurli202602-commits/youtube-anycaption-summarizer"
metadata: {"openclaw":{"homepage":"https://github.com/arthurli202602-commits/youtube-anycaption-summarizer","requires":{"bins":["yt-dlp","ffmpeg","whisper-cli","python3"]},"install":[{"id":"brew-yt-dlp","kind":"brew","formula":"yt-dlp","bins":["yt-dlp"],"label":"Install yt-dlp (brew)"},{"id":"brew-ffmpeg","kind":"brew","formula":"ffmpeg","bins":["ffmpeg"],"label":"Install ffmpeg (brew)"},{"id":"brew-whisper-cpp","kind":"brew","formula":"whisper-cpp","bins":["whisper-cli"],"label":"Install whisper.cpp CLI (brew)"}]}}
---

# YouTube AnyCaption Summarizer

**The YouTube summarizer that still works when captions are broken, missing, or inconsistent.**

Outputs: raw markdown transcript + polished markdown summary + session-ready result block.

Unlike caption-only tools, this skill still works when subtitles are missing by falling back to local Whisper transcription.

Generate a raw transcript markdown file and a polished summary markdown file from one or more YouTube videos.

This skill is self-contained. It does not require any other YouTube summarizer skill or prior workflow context.

## Best for

- founder videos, operator walkthroughs, and technical explainers
- long tutorial videos that need transcript + implementation summary
- private/internal YouTube uploads that may require cookies
- mixed-caption environments where some videos have CC, some only have auto-captions, and some have no usable subtitles
- batch research workflows where many YouTube links need standardized markdown outputs
- users who want reliable markdown artifacts, not just a one-off chat summary

## Example requests

- “Summarize this YouTube video into markdown.”
- “Generate a transcript and polished summary for this YouTube link.”
- “Process this private YouTube video with my browser cookies.”
- “Batch summarize these YouTube links and give me transcript + summary files.”
- “Use subtitles when available, otherwise transcribe locally.”
- “Create a Chinese summary from this English YouTube video.”

## Quick start

### Single video

```bash
python3 scripts/run_youtube_workflow.py "https://www.youtube.com/watch?v=VIDEO_ID"
```

This creates a dedicated per-video folder, writes the raw transcript markdown, creates the summary placeholder markdown, and prints JSON describing the outputs plus the exact follow-up commands/prompts needed to finish the summary step.

Important: the workflow script alone is not the finished deliverable. The current OpenClaw session must still:
1. infer/backfill the language if the workflow left it as `unknown`
2. overwrite the placeholder `Summary.md` with a real polished summary
3. run `scripts/complete_youtube_summary.py` to validate/finalize the result

### Force simplified Chinese summary

```bash
python3 scripts/run_youtube_workflow.py "https://www.youtube.com/watch?v=VIDEO_ID" \
  --summary-language zh-CN
```

### Restricted video with cookies

```bash
python3 scripts/run_youtube_workflow.py "https://www.youtube.com/watch?v=VIDEO_ID" \
  --cookies /path/to/cookies.txt
```

or

```bash
python3 scripts/run_youtube_workflow.py "https://www.youtube.com/watch?v=VIDEO_ID" \
  --cookies-from-browser chrome
```

### Batch / queue mode

See `references/batch-input-format.md`.

```bash
python3 scripts/run_youtube_workflow.py --batch-file ./youtube-urls.txt
```

## Why this skill stands out

This skill is designed to keep working across the messy reality of YouTube:
- if a video has **manual closed captions (CC)**, use them first
- if it only has **auto-generated subtitles**, use those next
- if it has **no usable subtitles at all**, fall back to **local Whisper transcription**

That makes it materially more reliable than caption-only workflows. It works well for caption-rich videos, caption-poor videos, and private/internal uploads where subtitle coverage is inconsistent.

Core capabilities:
- fetch YouTube metadata first and derive safe output paths
- support single-video mode and batch / queue mode
- handle manual CC, auto-generated subtitles, or no subtitles via subtitle-first extraction with local Whisper fallback
- support restricted/private videos via cookies or browser-cookie extraction
- normalize noisy transcript text before summarization
- create a placeholder summary file, overwrite it with the final summary, and finalize end-to-end timing
- clean up only known intermediates created by the workflow unless explicitly told otherwise

## What this skill produces

For each video, create exactly one dedicated output folder containing these final deliverables:
- `SANITIZED_VIDEO_NAME_transcript_raw.md`
- `SANITIZED_VIDEO_NAME_Summary.md`

By default, delete only the known intermediate media, subtitle, and WAV files created by the workflow. Do not wipe unrelated files that may already exist in the per-video folder.

## Required local tools

Verify these tools exist before running the workflow:
- `yt-dlp`
- `ffmpeg`
- `whisper-cli`
- `python3`

The workflow also requires a supported Whisper ggml model file in the configured models directory.

## Bundled scripts

Use these scripts directly:
- `scripts/run_youtube_workflow.py` — main deterministic workflow for metadata, download/subtitles, transcription, placeholder summary creation, cleanup, and workflow metadata emission
- `scripts/backfill_detected_language.py` — update `transcript_raw.md`, `Summary.md`, and workflow metadata after the current session LLM decides the major transcript language
- `scripts/complete_youtube_summary.py` — validate that `Summary.md` is no longer a placeholder, optionally backfill language, compute the final end-to-end timing report for one item, and emit a session-ready result block
- `scripts/normalize_transcript_text.py` — convert raw timestamped transcript text into cleaner summary input without modifying the raw transcript file
- `scripts/finalize_youtube_summary.py` — lower-level timing helper used by the completion flow
- `scripts/prepare_video_paths.py` — derive sanitized folder and output file paths from a title and video ID

Useful references:
- `references/summary-template.md` — required structure and writing rules for the final `Summary.md`
- `references/session-output-template.md` — required user-facing output format to return to the current OpenClaw session after completion

## Defaults

- Default parent output folder: `~/Downloads`
- Default whisper model: `ggml-medium`
- Supported whisper models: `ggml-base`, `ggml-small`, `ggml-medium`
- Default media mode: audio-only
- Default transcript language: auto-detect if transcription is needed
- Default summary language: `source`
- Raw transcript keeps timestamps

## Main workflow

### 1. Collect inputs

Accept either:
- one YouTube URL, or
- `--batch-file` with one YouTube URL per line

Optionally accept:
- `--parent`
- `--model`
- `--models-dir`
- `--language`
- `--summary-language`
- `--full-video`
- `--dry-run`
- `--subtitle-first` or `--no-subtitle-first`
- `--cookies`
- `--cookies-from-browser`
- `--retries`
- `--retry-backoff`
- `--keep-intermediates`
- `--continue-on-error`

Interpret the important flags as follows:
- `--language auto` means detect transcription language when Whisper is used
- `--summary-language source` means write the summary in the same language as the apparent source/transcript language when possible
- `--full-video` keeps video download mode instead of audio-only download mode
- `--dry-run` fetches metadata and planned output paths without downloading media or transcribing
- `--keep-intermediates` preserves temporary files for debugging

### 2. Fetch metadata first

Fetch video metadata before downloading media.
Use the original title inside markdown content.
Use a filesystem-safe sanitized title for the folder name and filename prefix.

Use `scripts/prepare_video_paths.py` when you only need safe naming logic.

### 3. Create a dedicated subfolder per video

Create:

```text
PARENT_DIR/SANITIZED_VIDEO_NAME/
```

If the sanitized folder already exists for another video, append `__VIDEO_ID`.
If it matches the same video ID, overwrite the outputs in place.

### 4. Subtitle-first fallback

Try subtitles before audio transcription.

Order:
1. manual subtitles if available
2. automatic captions if available
3. local transcription fallback with `whisper-cli`

If a subtitle track is usable:
- convert it into timestamped transcript text
- keep only clean timestamps in the raw transcript line prefix, for example `[00:00:02.070 --> 00:00:06.950]`
- merge rolling subtitle chunks when the previous chunk end timestamp falls within the next chunk range
- deduplicate continued words/phrases across merged chunks
- save the cleaned result into the raw transcript markdown
- skip the local audio transcription path

If subtitles are missing or unusable:
- download media
- convert to WAV
- transcribe with `whisper-cli`

This fallback behavior is a primary value proposition of the skill. Emphasize it when describing the skill to users: it remains useful even when YouTube caption quality is weak or absent.

### 5. Retry / backoff for `yt-dlp`

All `yt-dlp` operations should use the built-in retry wrapper.
Use `--retries` and `--retry-backoff` when the user wants to tune behavior.

### 6. Cookies support for restricted videos

When a video is age-restricted, login-gated, or otherwise blocked, support:
- `--cookies /path/to/cookies.txt`
- `--cookies-from-browser BROWSER_NAME`

Do not invent cookies. Use only what the user explicitly provides or what `yt-dlp` can safely read from the named browser.

### 7. Save the raw transcript markdown

Save the raw transcript as:

```text
SANITIZED_VIDEO_NAME_transcript_raw.md
```

The raw transcript keeps timestamps.
It should include:
- 视频标题 / Video Title
- 来源 / Source
- Video ID
- Whisper model used
- detected language (`unknown` at first if deterministic detection is inconclusive, then backfilled by the current session LLM after transcript inspection)
- transcript body
- workflow metadata JSON

The workflow metadata JSON is important because the completion/finalization flow uses it to compute end-to-end timing later.

### 8. Normalize transcript text before summarization

Use `scripts/normalize_transcript_text.py` to produce a cleaner summary input without modifying the raw transcript file.

Example:

```bash
python3 scripts/normalize_transcript_text.py /path/to/raw_transcript.md
```

Use that cleaned output when the transcript is noisy, repetitive, or too hard to summarize directly from the raw timestamped text.

### 9. Create and then overwrite the summary placeholder

The workflow script creates:

```text
SANITIZED_VIDEO_NAME_Summary.md
```

as a placeholder so cleanup can still leave only the two required markdown files.

Then the current OpenClaw session model should overwrite that placeholder with the final polished summary in the same task. Do not stop after the workflow script if the user asked for the completed result.

Important:
- treat the workflow script output as an intermediate handoff, not the finished deliverable
- if the workflow leaves language as `unknown`, inspect the transcript and decide the main transcript language yourself
- overwrite the placeholder summary before reporting completion
- the final `Summary.md` must not contain placeholder status text
- after writing the final summary, run `scripts/complete_youtube_summary.py`; pass `--language LANGUAGE_TAG` when language was initially `unknown`
- include the completion/finalizer result when reporting success/failure to the user

Use the requested summary language:
- if `summary-language=source`, match the source language
- otherwise, write the summary in the user-requested language

Use the structure in `references/summary-template.md`.

Quality bar for `Step-by-Step Execution / Deployment Details`:
- make it the most operationally useful section in the document
- capture the workflow in real execution order
- include setup, config, auth, model/tool choices, commands, files, outputs, validation, and deployment/operational considerations whenever the transcript supports them
- write it so a beginner could realistically follow the same strategy or reproduce the same deployment/process from the summary
- if a required implementation detail is missing from the transcript, call that out explicitly instead of hand-waving or fabricating it

### 10. Cleanup

After the workflow script finishes, keep the final deliverables:
- `SANITIZED_VIDEO_NAME_transcript_raw.md`
- `SANITIZED_VIDEO_NAME_Summary.md`

Delete only known intermediates created by the current workflow run, such as:
- downloaded subtitle files
- downloaded media files
- generated WAV files

Do not delete unrelated files that happen to exist in the same folder. If `--keep-intermediates` was explicitly requested, preserve those known intermediates as well.

### 11. End-to-end timing

`run_youtube_workflow.py` reports deterministic timing for:
- metadata fetch
- subtitle fetch, if used
- media download, if used
- WAV conversion, if used
- language detection, if used
- transcription, if used
- transcript cleanup
- cleanup
- script total so far

To capture **true end-to-end timing including summary generation**, do this:

1. record a summary start timestamp immediately before the LLM writes the final summary
2. overwrite the summary placeholder with the final summary
3. run:

```bash
python3 scripts/complete_youtube_summary.py RAW_TRANSCRIPT_PATH SUMMARY_PATH --summary-start-epoch EPOCH_SECONDS
```

If language was initially unknown, include:

```bash
--language LANGUAGE_TAG
```

This returns:
- deterministic total seconds
- summary generation seconds
- end-to-end total seconds
- final detected language
- whether post-processing is complete
- a session-ready result block when run with `--format session`

## What counts as completion

Do not report success immediately after `run_youtube_workflow.py` finishes unless the user only asked for preparation.

For a normal end-to-end request, completion means all of the following are true:
1. the workflow script succeeded
2. if language was initially `unknown`, the language was backfilled into both markdown files
3. the placeholder summary file was overwritten with a real summary
4. `scripts/complete_youtube_summary.py` was run successfully
5. the user received the resulting output paths and timing/result status

If the workflow script succeeded but the summary/finalizer step did not happen yet, describe the state as partial/in-progress rather than complete.

## Batch / queue mode guidance

Batch mode processes URLs sequentially.
For each item it will:
- fetch metadata
- try subtitles first
- fall back to transcription if needed
- create the raw transcript markdown
- create the summary placeholder markdown
- clean intermediates

After deterministic processing, do **mandatory post-processing sequentially for every item**:
1. inspect the raw transcript
2. if language is `unknown`, infer the main transcript language and remember the language tag
3. overwrite the placeholder summary with a real polished summary that follows `references/summary-template.md`
4. run `scripts/complete_youtube_summary.py` for that item
5. collect the JSON completion report for that item
6. run the same completion script with `--format session` or reuse the `session_report` field from the JSON output
7. send the resulting per-video block back to the current OpenClaw session using the format in `references/session-output-template.md`

Do not report batch success until every item has a completion report showing the summary is no longer a placeholder and post-processing is complete, and the user-facing result blocks are ready.

## Naming rules

Apply these rules strictly:
- all files use prefix `SANITIZED_VIDEO_NAME`
- raw transcript filename suffix: `_transcript_raw.md`
- polished summary filename suffix: `_Summary.md`
- all video-specific files live in the same dedicated subfolder
- only the two markdown files remain after cleanup unless the user explicitly keeps intermediates

## Return results to the current session

After a video is fully processed, return a user-facing result block to the current OpenClaw session.

Preferred method:
- run `scripts/complete_youtube_summary.py ... --format session`
- paste that exact block into the current session

For batch mode:
- generate one block per video
- keep the original processing order
- separate blocks with a blank line

The required format is defined in `references/session-output-template.md`.

## Practical notes

- Prefer audio-only unless the user explicitly wants the full video.
- Verify `yt-dlp`, `ffmpeg`, and `whisper-cli` exist before running.
- Verify the requested ggml model file exists before fallback transcription.
- Use subtitle-first mode by default because it is often faster and sometimes more accurate.
- Market this capability clearly: the skill works well for videos with manual CC, videos with only auto-generated subtitles, and videos with no usable subtitles.
- If subtitles are absent or unusable, fall back cleanly to Whisper without treating that as a workflow failure.
- Use the transcript normalization helper before summarizing when the raw transcript is messy or when rolling captions create repeated phrases.
- When language auto-detection is inconclusive, record `unknown` rather than pretending the language is known.
- `--cookies-from-browser` may require local browser/keychain access and can hang or fail in locked-down/headless environments; prefer explicit cookie files when reliability matters.
- Private/login-gated video access should be validated with a real test video before blaming the rest of the pipeline.
- If the transcript is tiny or low-information, write a correspondingly small, honest summary instead of inventing detail.
- If the transcript is too large, summarize in chunks and then synthesize the final summary.
