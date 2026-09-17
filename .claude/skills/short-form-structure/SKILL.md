---
name: short-form-structure
description: Build a vertical short-form cut (Reels/TikTok/Shorts) from B-roll and a supplied audio clip — read what the audio is, choose the cut-rate curve, pivot on the phrase boundary, and know which techniques this toolchain can and cannot deliver. Load when the deliverable is under about a minute and the audio is the spine.
---

# Short-Form Structure

## Why there is no "current trends" section in this file

Trends rot in weeks. A skill file listing this month's transitions, text styles
or meme formats is stale before it is useful, and writing one from recollection
produces confident nonsense — the exact failure this whole skill set exists to
avoid.

What does not rot: how a piece is **built against its audio**, and which
techniques the toolchain can actually execute. That is what is written here, and
every number in it was measured.

If a specific current reference is wanted, the honest route is to look at the
reference directly — and the strongest available signal is already in hand: a
**TikTok audio clip the user supplies is itself the trend**, and it can be
measured rather than guessed.

## Read what the supplied audio actually is

TikTok/Reels audio is usually the **viral section** of a track, already trimmed.
That makes it behave very differently from a full song. Two real tracks measured
the same way:

| | Full track (`bensound-newfrontier`) | TikTok clip (`ssstik.io_…`) |
|---|---|---|
| Length | 155 s | **29 s** |
| Energy at 0-2 s | **0.14** (sleepy intro) | **0.64** (already full) |
| Energy range | 0.09 - 0.52, slow arc | **0.57 - 0.73, flat** |
| Phrase starts | 9 of them | **2** (0.14 s, 16.58 s) |
| Tempo | 117.45 BPM | 117.45 BPM |

The consequences are structural, not cosmetic:

- **Length is decided for you.** A 29-second clip is the piece. Do not pad it
  and do not loop it.
- **There is no intro to cover.** The picture must be at full strength from
  frame one — no slow establishing drift while the music warms up, because it
  never warms up, it starts hot.
- **There may be only one phrase boundary.** That single pivot is the piece's
  entire structural spine (below).

## When the music is flat, the CUTTING builds

A viral clip is usually energy-flat — it has no build and no drop, just
sustained hook. A piece cut at one constant rate over flat music flatlines with
it.

The fix is to put the build in the edit: **accelerate the cut rate at the phrase
boundary.**

Measured, on the 29-second clip above (downbeats every 2.07 s, one phrase
boundary at 16.58 s):

| Section | Grid | Shot length | Shots |
|---|---|---|---|
| 0 → 16.58 s | 2-bar | ~4.1 s | 4 |
| 16.58 → 28.89 s | **1-bar** | **~2.05 s** | 6 |

Ten shots, 28.88 s. The rate doubles exactly where the music turns over, so the
back half lifts without the track doing anything different. Verified: the nine
real cuts landed a mean of **3.3 ms** from their downbeats — a fifth of a frame
at 59.94 fps.

Reverse the shape (fast → slow) only for a deliberate wind-down ending.

## Density decides whether hard cuts feel right

Energy says how full the track is. **Onset density says how punchy it is**, and
the two do not move together.

The TikTok clip measured energy 0.57-0.73 with onset density only **0.06-0.08** —
loud but smooth, melodic rather than percussive. On material like that, very
fast cutting (every beat, ~0.5 s) reads as arbitrary because there is no
transient for the cut to land against. Bar-level (~2 s) is as fast as smooth
music supports.

Check density before choosing a grid finer than one bar.

## Getting enough shots

Ten shots from five usable clips means **taking several moments from the same
clip**. This is normal and invisible to the viewer when the moments are
genuinely different — a crowd at 4 s and the same crowd at 28 s are different
shots.

It is only obvious when two moments share framing *and* subject. Verify by
pulling frames from the render, not by trusting the source timecodes to be far
apart.

## What this toolchain can and cannot do

Stating it plainly, because promising the second column is how a toolkit earns
the word "bullshit":

| Available | Not available on a bridge-compatible 21.0.x build |
|---|---|
| Beat-locked cutting at any grid | Speed ramps / retimes (no clip-speed API at **any** version) |
| Accelerating cut-rate structure | Transitions, whip cuts (21.1+) |
| Vertical 9:16, correct orientation | Audio fades (21.1+) |
| Punchy grading (LUT + CDL) | Text / caption overlays (destination track not selectable) |
| Render-and-verify against the beat grid | Zoom / punch-in animation |

The left column is the part that decides whether a short-form piece works —
shot choice, rhythm, structure, colour. The right column is garnish, and it is a
manual pass in the UI.

## Workflow

1. Profile the supplied audio (`sound-atmosphere-matching`) — length, energy
   shape, density, phrase boundaries.
2. Take the piece's length from the audio.
3. Place the pivot at the phrase boundary; choose a slower grid before it and a
   faster one after.
4. Pick shots that are strong from frame one — no warm-up.
5. Build, grade, then render and **measure** the cuts against the downbeats.
6. Pull frames from the render for any moment not previously seen in context.
