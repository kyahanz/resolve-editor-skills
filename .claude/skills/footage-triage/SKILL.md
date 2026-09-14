---
name: footage-triage
description: Probe and classify a folder of camera-original footage before any timeline exists — orientation, frame rate, audio presence, codec — and settle the delivery format with the user. Load at the START of any cut, before importing or choosing shots, so format mistakes are caught while they are still free to fix.
---

# Footage Triage

The first ten minutes of an edit decide whether the next two hours are wasted.
This skill is the preflight: know what the footage actually **is**, and what the
deliverable is supposed to be, before a single clip lands on a timeline.

Run it before `resolve-rough-cut`, before importing, before shot selection.

## Why this exists (the incident)

A 15-clip DJI shoot was assembled, graded, and re-cut twice before anyone noticed
the footage was **mixed orientation** — 7 clips landscape, 8 clips vertical — and
the timeline was 1080x1920. The landscape clips were being centre-cropped to ~28%
of their width for the whole session. The user felt it ("this shot is lacking")
and the shots were swapped for *content* reasons, when the real defect was
technical. Two full re-cuts and a grade pass were spent on a problem a two-minute
probe would have caught.

The probe that missed it looked like this:

```bash
ffprobe -show_entries stream=width,height,rotation ...   # WRONG
```

`rotation` is **not** a stream entry on modern ffmpeg. It lives in
`side_data_list`. The command above returns nothing for a rotated clip and reads
as "no rotation" — a silent false negative on the one field that matters most.

## Step 1 — Probe every clip, correctly

```bash
ffprobe -v error -select_streams v:0 \
  -show_entries stream=width,height,r_frame_rate,avg_frame_rate,codec_name \
  -show_entries stream_side_data=rotation \
  -show_entries format=duration \
  -of default=noprint_wrappers=1 "<clip>"
```

Read five things per clip, and report them as a table, not prose:

| Field | Why it decides something |
|---|---|
| `width`/`height` + `side_data rotation` | **Orientation.** `rotation=90` or `270` means the clip DISPLAYS transposed: stored 3840x2160 displays as 2160x3840. Resolve honours the flag; do not reframe it yourself. |
| `r_frame_rate` vs `avg_frame_rate` | They disagree on **VFR** footage (phones, some action cams). Match the timeline to what Resolve conforms to, not to `r_frame_rate`. |
| `duration` | Total available runtime vs the target length — tells you how much selection pressure there is. |
| `codec_name` | H.265/HEVC 10-bit on a weak machine means proxies or a long grind. Say so before promising a timeline. |
| audio channel count (`-select_streams a`) | **Zero audio channels is common on drones.** Find out before promising to "mix in the natural sound" — there may be nothing to mix. |

## Step 2 — Group by format and refuse to silently mix

Bucket the clips by **display** orientation and frame rate. Then:

- **One bucket** → proceed; the timeline format is decided for you.
- **More than one bucket** → STOP and report it. Do not pick for the user.

A mixed-orientation folder has no correct default. Every option costs something,
so name the cost of each:

| Option | Cost |
|---|---|
| Vertical timeline, vertical clips only | Discards the landscape clips entirely |
| Landscape timeline, landscape clips only | Discards the vertical clips entirely |
| Vertical timeline, all clips | Landscape clips lose ~70% of frame width to centre-crop |
| Landscape timeline, all clips | Vertical clips get pillarboxed, or heavily cropped top/bottom |

**Never resolve this by guessing from clip counts.** "Most clips are vertical so
I'll go vertical" silently throws away the other half of the shoot.

## Step 3 — Settle the delivery target out loud

Ask, before setting timeline resolution — it is locked in practice once an edit
exists on it:

> "This is going out as [vertical 9:16 for Reels/TikTok/Shorts] or [landscape
> 16:9 for YouTube]?"

Then set BOTH frame rates before the first timeline exists (`timelineFrameRate`
via `project_manager safe_set_project_settings`; `timelinePlaybackFrameRate`
is not writable through the API — see `resolve-rough-cut` for the exact UI
wording to hand the user).

## Step 4 — Contact sheets without a Python toolchain

Shot selection needs to see frames, and `scripts/contact_sheet.py` needs Pillow,
which is frequently absent. ffmpeg alone is enough:

```bash
# 4 frames at 8/36/64/92% of duration, tiled 2x2
ffmpeg -y -ss "$T1" -i "$CLIP" -frames:v 1 -vf "scale=480:-1" f1.jpg
# ... repeat for T2..T4, then:
ffmpeg -y -i f1.jpg -i f2.jpg -i f3.jpg -i f4.jpg \
  -filter_complex "xstack=inputs=4:layout=0_0|w0_0|0_h0|w0_h0" sheet.jpg
```

Use `xstack`, not `tile` — `tile` takes a single input pad and errors with
"More input link labels specified for filter 'tile' than it has inputs".

One sheet per clip, read as an image. Roughly 15x cheaper than reading frames
individually, and enough to judge content and framing.

## Step 5 — Report, then wait

Output a compact block:

- Clip count, total runtime
- Orientation buckets (with clip names in each)
- Frame rate(s), and whether any clip is VFR
- Clips with no audio
- Codec, and a warning if it is a heavy one
- **The format question**, unanswered, as the last line

Do not start selecting shots off the back of this. Triage ends with a decision
the user makes, not with an assembly.

## The rule this skill exists to enforce

**Technical defects masquerade as taste.** A crop-mangled shot reads as "weak
footage"; a frame-rate mismatch reads as "bad pacing"; a VFR clip reads as "the
sync feels off". When the user says a shot is *lacking* and the fix is not
obvious, re-probe the format before re-cutting for content — you may be
re-editing around a defect instead of fixing it.
