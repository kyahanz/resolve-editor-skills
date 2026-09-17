---
name: sound-atmosphere-matching
description: Read a music track's shape — energy curve, brightness, density, and where its sections begin — then match footage choice, pacing and entry point to it. Load when the client supplies the music and the edit must follow its mood, not just its beat. Pairs with beat-sync-editing, which handles the grid; this decides which part of the track to cut over at all.
---

# Sound-Atmosphere Matching

`beat-sync-editing` answers *where* the cuts land. This answers a question that
comes first and matters more: **which part of the track should the video sit on,
and can this track carry the feeling being asked for at all.**

A cut perfectly locked to the grid still fails if it is laid over the sleepiest
90 seconds of the song. That is a mood problem, and it is measurable.

## What is actually measurable

All local, all fast (seconds on a 2.5-minute track), all through `librosa` in
the **repo venv** — and all run **out-of-process** (see `beat-sync-editing`:
heavy audio work through the MCP wedges the server).

| Signal | librosa call | Reads as |
|---|---|---|
| **Energy** | `feature.rms` | Loud/full vs quiet/spacious — the main one |
| **Brightness** | `feature.spectral_centroid` | Bright and open vs dark and close |
| **Density** | `onset.onset_strength` | Busy and eventful vs sparse and still |

Normalise each against the track's own maximum and average over windows (4-8 s
works). The absolute numbers mean nothing across tracks; the **shape over time**
is the whole point.

## Step 1 — Profile the whole track before choosing anything

Do not profile only the part you planned to use. The point is to find where the
track's energy actually lives.

Measured on a real 155-second acoustic track (`bensound-newfrontier.mp3`):

| Section | Energy (normalised) | What it is |
|---|---|---|
| 0-16 s | 0.14 - 0.19 | Quiet intro |
| 16-64 s | 0.32 - 0.39 | Slow build |
| **64-128 s** | **0.49 - 0.52** | The body — where the track lives |
| 128-155 s | 0.26 → 0.09 | Outro fade |

The edit in progress at the time was cutting over **0-41 s** — the sleepiest
stretch of the song — and nobody had noticed, because the beat grid was
perfectly aligned there. Moving the entry to the body changed how the piece felt
without touching a single shot.

## Step 2 — Choose the entry point where a phrase and an energy step coincide

Do not enter at an arbitrary "good bit". Enter where **a phrase boundary and an
energy increase land together** — the music resolves and lifts at the same
moment, and the video gets both.

On the track above, `phrase_starts` were 1.09, 17.09, 33.09, 49.09, **65.09**,
81.08, 97.08, 113.08. Energy steps from 0.37 to 0.49 at ~64 s. So **65.09 s**
is the entry: a phrase start sitting exactly on the lift.

Convert to a source frame for the music item (`65.09 × fps`) and compute the
cut grid **relative to that entry**, not to the start of the file. Every
downbeat time from the detector is absolute; subtract the entry time before
turning them into timeline frames.

## Step 3 — Match pacing to energy, not to preference

Energy level should set shot length, and it should be argued from the curve:

| Energy | Shot length | Grid |
|---|---|---|
| Low (< 0.3) | Long, 8-16 s | Phrase or 4-bar |
| Medium (0.3-0.5) | 4-8 s | 4-bar |
| High (> 0.5) | 2-4 s | 2-bar or bar |

When the track *changes* level, the cut rate should change with it. A build that
keeps the same shot length through the lift wastes the lift.

## Step 4 — Say when the track cannot do what is being asked

This is the honest part, and it is the most useful thing this skill does.

A track's **dynamic range** — the spread between its quietest and loudest
sustained sections — caps what the edit can feel like. The track measured above
spans 0.09 to 0.52 in 8-second averages, and its "peak" is only half its own
transient maximum. It is a calm acoustic piece: gentle, warm, no drop.

**No amount of fast cutting makes calm music feel energetic.** Cutting a
0.5-energy acoustic track at 2-bar intervals produces a piece that looks busy
and feels wrong, because the picture is insisting on something the sound never
does.

So when the brief is "make it punchy" and the profile says the track tops out
low and flat: **say so, and ask for a different track.** Report the numbers —
the intro/body/outro table above is the argument, and it takes seconds to
produce. Do not quietly deliver a fast cut and let the mismatch be discovered
later.

## What short-form ("TikTok-style") can and cannot be built here

Worth stating plainly, because the gap is in the tooling, not in the method:

**Available:** beat-locked cutting at any grid, fast pacing, vertical 9:16,
punchy grading, music entry on an energy step, render-and-verify.

**Not available on a bridge-compatible Resolve build (21.0.x):** speed ramps and
retimes (no clip-speed API at *any* version), transitions and whip cuts (21.1+),
audio fades (21.1+), text and caption overlays (destination track cannot be
chosen from the API). Those are the "flashy" layer, and promising them is how a
toolkit gets called bullshit.

What this stack produces well is the **foundation**: shot choice, structure,
rhythm and colour. That is the part that decides whether a short-form piece
works. The garnish is a manual pass in the UI.

## Workflow

1. Profile the **whole** track — energy, brightness, density over time.
2. Report the section table. If the range is narrow and the brief wants energy,
   stop and say so.
3. Pick the entry where a phrase boundary meets an energy step.
4. Pick the grid from the energy level at that entry.
5. Hand the frame numbers to `beat-sync-editing` and build.
6. Verify by render, then measure cut positions against downbeats.
