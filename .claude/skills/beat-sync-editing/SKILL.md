---
name: beat-sync-editing
description: Cut music-driven montage to the music — detect beats, group them into bars and phrases, and place shot boundaries on musical structure instead of arbitrary durations. Load when assembling any cut carried by music rather than dialogue (B-roll montage, travel, drone, sport), before choosing shot lengths.
---

# Beat-Synced Editing

Everything else in this toolkit finds edit points in **speech** — word
boundaries, pauses, fillers. Music has none of those. It has pulse, and pulse is
a different measurement entirely.

For a montage carried by music, shot length is not a taste decision made in
seconds. It is a decision made in **bars**. A cut that lands on the music reads
as deliberate; the same cut 300 ms early reads as sloppy, and the viewer feels
it without being able to name it.

## Install: it must go in the REPO venv, not the system Python

`librosa` is an optional extra, deliberately not in `requirements.txt`.

The MCP server runs `venv/Scripts/python.exe` (see `.mcp.json`), and its
`sys.prefix` resolves to the repo venv — **measured**, despite `PYTHONHOME`
pointing at the system Python. So:

```bash
venv/Scripts/python.exe -m pip install librosa
```

Installing into the system Python looks successful and changes nothing.

This differs from PATH-discovered tools: ffmpeg and whisper are found via the
CLI on PATH, so a server **restart** picks them up. librosa is found by
**import**, so it must live in the venv the server actually runs.

## What the detector gives you

```python
from src.utils import beat_detection as bd
bd.detect_beats(media_path)          # -> tempo_bpm, beats[], confidence
bd.group_into_phrases(beats, beats_per_bar=4, bars_per_phrase=8)
bd.snap_to_frames(times, fps)        # -> integer frame numbers
```

Measured on a real 2:35 track (`bensound-newfrontier.mp3`):

| Field | Value |
|---|---|
| `tempo_bpm` | 117.45 |
| BPM from median beat spacing | 120.0 |
| `beat_count` | 280 |
| `confidence.band` | `high` (jitter 0.0272 — "the tracker locked on") |
| Bar (4 beats) | every ~2.0 s |
| 4-bar group | every **8.0 s** |
| Phrase (8 bars) | every **16.0 s** |

**Trust the beat times, not the BPM number.** The reported 117.45 and the
spacing-derived 120.0 disagree slightly; the beat *times* are what you cut on,
and they were dead regular. A tempo figure is an estimate — a cut point is not.

## Choosing the grid: bars, not seconds

| Grid | Interval here | Feel |
|---|---|---|
| Every beat | 0.5 s | Relentless. Almost always wrong |
| 2 bars | 4 s | Fast montage, high energy |
| **4 bars** | **8 s** | The workhorse for B-roll montage |
| Phrase (8 bars) | 16 s | Slow, cinematic. Long for a single shot |

Phrase boundaries are where music *resolves* — an edit there feels inevitable
rather than merely synchronised. But 16 s is a long time to hold one shot, so
the usual shape is: **cut on 4-bar marks, and put your strongest shot changes on
the phrase boundaries.** Structure follows the music's own structure.

## The traps, in the order they bite

**1. `timeline_fps` defaults to 24.0.** `edit_engine(action='plan_beat_cuts')`
falls back to 24 fps if you do not pass one. On a 59.94 timeline every returned
frame number is then wrong by a factor of 2.5, and nothing in the result says
so. Always pass the real timeline rate.

**2. Downbeats are inferred, not detected.** The first detected beat is assumed
to start a bar. A track with a pickup (an anacrusis) is off by one beat for its
entire length. Symptom: the cuts feel consistently *early*. Fix with
`beat_offset`, do not re-cut by hand.

**3. Frame-snap, always.** A cut two frames off the beat reads as a mistake
rather than as syncopation. Use `snap_to_frames(times, fps)` and build the
timeline from those integers — never from the raw float seconds.

**4. Confidence is reported; read it.** Tempo tracking degrades on rubato, live
performance without a strong downbeat, and heavily swung material. When
`confidence.band` is not `high`, say so and fall back to manual placement rather
than delivering cuts that are confidently, rhythmically wrong.

**5. Do NOT call `plan_beat_cuts` through the MCP — it wedges the server.**

This is the worst trap in this file, and it was measured twice.

| Route | Result |
|---|---|
| `venv/Scripts/python.exe` calling `edit_engine.plan_beat_cuts` directly | **2.2 s**, 18 cut points |
| `edit_engine(action='plan_beat_cuts')` through the MCP | **Hung for 1800 s**, then aborted |

Worse than the hang: the server did not recover. The next call — a plain
`timeline(action='get_current')` that had answered in 125 ms moments earlier —
also hung for the full 1800 s. The whole server was dead, and only a Claude Code
restart brought it back.

The function itself is trivial: resolve a path, `detect_beats`, group, snap.
Nothing loops. The cost is `librosa.load` plus numba-backed beat tracking, and
running that inside the MCP server's process blocks it completely.

**So: always run beat detection out-of-process.**

```bash
venv/Scripts/python.exe -c "
import sys; sys.path.insert(0, r'<repo>')
from src.utils import edit_engine as ee
r = ee.plan_beat_cuts(r'<analysis_root>', media_path=r'<audio>',
                      timeline_fps=59.94, mode='bar', min_shot_seconds=7)
print(r['cut_frames'])
"
```

Take the returned frame numbers and build the timeline with the ordinary
`media_pool` calls. Those are cheap and do not block.

**6. `plan_beat_cuts` needs an `analysis_root`, not an open project.** The error
text says "open a Resolve project", but an open project is not enough — measured
with `Cold Open v04` open and readable, the call still refused. What
`_project_context` actually wants is a versioning provider or an explicit
`analysis_root=` pointing at a directory that exists. Pass one; any existing
directory works, and `.davinci-resolve-mcp-analysis/` is the gitignored
convention.

## What "off the beat" actually looks like

Measured on a real 51-second cut assembled to arbitrary ~10 s shot lengths
against the track above:

| Cut | Time | Nearest downbeat | Off by |
|---|---|---|---|
| 1 | 0.00 s | 1.09 s | 1091 ms |
| 2 | 9.99 s | 9.06 s | 938 ms |
| 3 | 19.99 s | 19.09 s | 900 ms |
| 4 | 29.98 s | 29.09 s | 885 ms |
| 5 | 38.97 s | 39.08 s | 107 ms |

At 120 BPM one beat is 500 ms, so four of five cuts sat nearly **two full beats**
off the pulse. The footage, grade and track were all fine; the cut fought the
music anyway. This is the failure this skill exists to prevent, and it is
invisible until measured.

## Workflow

1. Detect beats on the **final** music track — changing the track invalidates
   every cut point.
2. Group into bars and phrases; pick a grid (4-bar is the default worth
   defending).
3. Snap to frames at the **real** timeline fps.
4. Build shot durations from the gaps between those frames, not from round
   seconds.
5. Put the strongest shot changes on phrase boundaries.
6. Verify: render, then measure the actual cut positions against the downbeats
   the way the table above does. "It looks like it lines up" is not verification.

## When NOT to use this

Dialogue-led pieces cut on meaning, not pulse — music under narration should
duck, not dictate. And a montage with a deliberately loose, drifting feel is a
real choice; lock it to the grid only when the music is the spine.
