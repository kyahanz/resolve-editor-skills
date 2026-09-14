---
name: captions-workflow
description: Produce captions/subtitles for footage on a Resolve build without Studio's auto-subtitle — transcribe locally with Whisper, write SRT, import it, and place it. Load when a cut needs captions or when a transcript is needed to drive selects on speech footage.
---

# Captions Workflow

Captions matter twice: as the deliverable (most social video is watched muted)
and as the **index** that makes long-form selects possible — see
`selects-and-stringout`, where the transcript is how you find content without
scrubbing.

## Why not Resolve's own subtitle generator

`Timeline.CreateSubtitlesFromAudio` and `MediaPoolItem.TranscribeAudio` are
**Studio-only**. On the free edition they return `False` *and* raise a modal
upsell dialog — and while that dialog is up, **unrelated API calls also fail**
until a human clicks it away. An automated caller sees a cascade of unexplained
failures and misattributes them to whatever it called next.

So on a free build: do not call them. Detect the edition first
(`resolve_control(action='get_version')` — product name is `DaVinci Resolve` on
free, `DaVinci Resolve Studio` on Studio) and route to local Whisper instead.

## Step 1 — Transcribe locally

Whisper is not installed by default. It needs a real Python (not the Windows
Store stub):

```bash
pip install -U openai-whisper
```

Then, per clip with speech:

```bash
whisper "<clip>" --model small --output_format srt --output_dir "<out_dir>"
```

Model choice is a speed/accuracy trade: `base` for a rough index, `small` for
usable captions, `medium`+ when names and jargon matter. Models download on
first use — say so before starting, the first run is not instant (`tiny` is
72 MB; larger models are substantially bigger).

**Verified on this machine** (Python 3.12, `openai-whisper-20250625`,
`torch-2.14.0`): install succeeds, the CLI resolves, ffmpeg decode works, and a
12-second clip produced a well-formed SRT. On instrumental music with no speech
it emitted a single `.` rather than hallucinating lyrics — near-empty output on
a silent or musical clip is correct behaviour, not a failure.

### The stale-PATH trap (bites twice)

Two separate processes need to find these binaries, and they fail differently:

- **A shell started before the install** keeps its old PATH. `pip` even warns
  that `whisper.exe` "is not on PATH" while the *persisted* user PATH already
  contains it — the warning reads the process environment, not the registry.
  Check with `[System.Environment]::GetEnvironmentVariable("Path","User")`
  before concluding anything is missing.
- **The MCP server process** is worse: it caches its PATH at launch, so
  `media_analysis(action='capabilities')` keeps reporting
  `whisper_cli: false`, `ffmpeg: false`, `ffprobe: false` long after the tools
  are installed and working. **Measured immediately after a successful install
  and a successful whisper run.**

  The server needs a **restart** to see them. Until then, any MCP action routed
  through its internal ffmpeg/whisper detection (frame capture with `max_width`,
  transcription, scene detection) will refuse with an install prompt for
  software that is already installed.

So: verify tools by running them directly, not by asking `capabilities()`. A
`false` there means "this server has not restarted since the install", which is
a different problem from "not installed" and has a different fix.

**Check the clip has audio before transcribing.** Drone footage frequently has
zero audio channels (measured on this project: every DJI Mini clip reported
`Audio Ch: 0`). Running Whisper on a silent clip wastes minutes and returns
nothing or hallucinated text.

```bash
ffprobe -v error -select_streams a -show_entries stream=channels \
  -of default=noprint_wrappers=1:nokey=1 "<clip>"
```

Empty output means no audio track at all.

## Step 2 — Import the SRT (this works)

**Measured on Resolve 21.0.4.5 free, over the bridge:** an `.srt` imported with
`media_pool(action='import_media', paths=['.../file.srt'])` succeeds and lands
in the media pool as a clip of type **`Subtitle`**. Confirmed by readback in
`project_summary` → `by_type: {Subtitle: 1}`.

## Step 3 — Placing it is a UI step, not an API one

**Measured, same build:** appending that Subtitle clip with
`media_pool(action='append_to_timeline', clip_ids=[...])` **fails**
(`execution.success: false`, `count: 0`) — and took the in-Resolve bridge down
with it (`connection forcibly closed`), requiring a manual restart from
Workspace ▸ Scripts.

So:

- Add the subtitle track by API if needed —
  `timeline(action='add_track', track_type='subtitle')` works.
- **Then hand off:** the user drags the Subtitle clip from the Media Pool onto
  the subtitle track. Do not retry the append; it is not a transient failure and
  it costs a bridge restart each time.

## Step 4 — Style and position

Caption **styling** (font, size, position) is not reachable through the
scripting API. The offline route is
`project_read`/`project_db(action='set_subtitle_style')` on the advanced server,
which patches the project database directly — whole-track only, and it requires
the project **closed** and Resolve fully quit and relaunched afterwards. Treat
that as a deliberate maintenance operation, not a mid-session tweak.

For anything lighter, styling is the user's UI pass.

## Step 5 — Where captions may sit

On vertical deliverables, the platform overlays its own UI. Captions placed at
the conventional "lower third" end up behind TikTok's or Instagram's own caption
block. Keep them inside the middle ~70% vertically — see `delivery-targets` for
the full safe-margin budget.

## Using the transcript for editing, not just captions

Once transcripts exist, they drive the parts of editing that are otherwise
manual:

- `edit_engine(action='search_spoken_content', query, mode)` — phrase /
  all_words / regex across every transcribed clip, returning timestamped hits
  with handles. This is how you find a moment in a 40-minute take.
- `edit_engine(action='plan_transcript_tighten', clip_ref)` — fillers, false
  starts and long pauses at word boundaries, with a stated reason per removal
  so the plan can be argued with.

Both need `transcript_words` present: run transcription, then the strata
backfill. They say so rather than returning an empty plan.

## Order, condensed

1. Check edition — free means local Whisper, never the Studio call
2. Check the clip actually has audio
3. `whisper … --output_format srt`
4. `import_media` the SRT (works, lands as type `Subtitle`)
5. `add_track` subtitle, then **hand the drag-to-timeline to the user**
6. Styling and position: UI, or the offline DB route with Resolve closed
