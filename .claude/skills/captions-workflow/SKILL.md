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

## Step 3 — Placing it: the LIVE route is broken, the OFFLINE route is not

**The live append fails, and costs a bridge restart.** Measured, same build:
`media_pool(action='append_to_timeline', clip_ids=[<subtitle clip>])` returns
`execution.success: false`, `count: 0` — and took the in-Resolve bridge down
with it (`connection forcibly closed`), requiring a manual restart from
Workspace ▸ Scripts. Do not retry it; it is not a transient failure.

`timeline(action='add_track', track_type='subtitle')` does work, so the track
can be created live. Only the *placement* is blocked.

**The offline route places cues without touching the bridge at all.** The
advanced (Node) server authors subtitle items straight into a `.drp`/`.drt` —
`drp-format/place-subtitles`, reached through the `drp` / `drt` actions. A
subtitle is the simplest item in the schema (a `Sm2TiGenerator` with
`PrettyType: Subtitle` and the cue text in `Name`, on a `Type 2` track), and
the module's shape was harvested from a real Studio project with an SRT
appended by the API and re-exported — so this writes what Resolve itself
writes. It takes `{startFrame, durationFrames, text}` per cue, with
`startFrame` **timeline-absolute** (origin 86400 on the bundled templates,
108000 on a 30 fps hour-start timeline — read the real start frame, do not
assume). Overlapping cues are refused rather than silently merged: one
subtitle track cannot hold two at once.

So the automated path is: SRT → cue list (convert SRT timecodes to timeline-
absolute frames at the real timeline fps) → `place-subtitles` into the project
file → reopen. Nothing is dragged by hand.

## Step 4 — Style and position (also offline, also automatable)

Caption **styling is not reachable through the scripting API** — subtitle
TimelineItems expose only the 21 transform/composite properties, and
`GetSetting()` returns null for every subtitle-style key.

The offline route handles it:
`project_read`/`project_db(action='list_subtitle_styles' | 'set_subtitle_style')`
on the advanced server, which patches the style Resolve keeps on the subtitle
track itself (font family / size / weight / italic, plus normalised position
`[x, y]` with origin top-left). It is **whole-track, not per-caption**.

## The operational catch that applies to BOTH offline steps

The project must be **CLOSED** and Resolve fully quit, then relaunched after
the patch. That is the entire cost of the automated route — not a manual
drag-and-drop pass, just a close/patch/reopen cycle.

Consequences worth stating to the user before starting:

- It cannot run while they are working in that project. Schedule it, do not
  surprise them mid-session.
- Anything unsaved in the open project is the user's to save first. Ask.
- After relaunch, **verify by reading the placed cues back** — subtitle text is
  API-visible (the payload is the item `Name`), so readback is meaningful here,
  unlike the Fusion comp-cache cases. Do not report captions as placed on the
  strength of the patch returning success.

**Not yet exercised on this project** (as of 2026-09-17): the offline placement
path is documented and its shape is harvested from live ground truth, but no
piece here has had speech to caption yet, so it has not been run end to end in
this repo. Run it and verify the readback the first time rather than promising
the outcome — then replace this paragraph with what actually happened.

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
4. Convert SRT cues to timeline-absolute frames at the real timeline fps
5. Close the project + quit Resolve, then patch offline:
   `place-subtitles` for the cues, `set_subtitle_style` for font/position
6. Relaunch, reopen, and **read the cues back** before reporting it done

Live-only fallback if the project cannot be closed: `import_media` the SRT
(works, lands as type `Subtitle`) + `add_track` subtitle, then the user drags
it onto the track. Never `append_to_timeline` a subtitle clip — that is the
call that kills the bridge.
