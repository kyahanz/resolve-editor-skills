---
name: vlog-story-spine
description: Structure for long-form vlog content — the retention shape of a 10-minute piece, where B-roll earns its place, how beats are separated, and the middle-sag problem. Load before cutting anything longer than about two minutes, after selects exist and before the rough cut is built.
---

# Vlog Story Spine

A 50-second cold open survives on pretty shots. A 10-minute vlog does not — it
needs a reason to keep watching at minute 4, and that reason has to be built
deliberately. Nothing about shot quality fixes a structural sag.

Load this **after** selects exist (`selects-and-stringout` rung 3) and before
the rough cut, because structure is a decision about ordering, and ordering is
cheap to change on a stringout and expensive to change later.

## The shape

| Beat | Rough share of runtime | Job |
|---|---|---|
| **Hook** | first 10-20s | Show the most compelling thing that happens, or state the question the video answers. Not a greeting, not a logo, not "hey guys" |
| **Promise** | next ~20s | What this video is and why it is worth the remaining minutes. Can be spoken, or implied by a strong establishing sequence |
| **Body beats** | the bulk | 3-6 distinct segments, each with its own small arc. A beat that does not change something is a beat to cut |
| **Payoff** | last ~60-90s | The thing the promise implied. Arriving, finishing, the result, the view |
| **Outro** | last ~10-15s | Short. Land and stop |

Treat the shares as proportions, not rules. The falsifiable part is the
**order** and the **presence** of each beat — a vlog missing a promise plays as
aimless even when every shot is good.

## Beats, and why they are the unit

A beat is a segment with one subject and a small internal change: arrive
somewhere, attempt something, react to it. Cut between beats, not between shots,
when deciding structure.

Two tests for whether a beat earns its place:

1. **Does something change?** Location, understanding, stakes, mood. A beat
   where nothing changes is footage, not story.
2. **Would removing it be noticed?** If the piece still makes sense without it,
   it is a candidate for the cut — regardless of how good the footage is.

The second test is the one people refuse to apply to their best-looking shots.
Apply it anyway; see `house-style` (Shot selection), where exactly this call
was already made once on this project.

## The middle sag

The predictable failure of long-form: minutes 3-7 flatten. Energy from the hook
is spent, the payoff is not close enough to pull.

What actually works against it:

- **Front-load the second-best thing.** Do not save every strong moment for the
  end; the audience leaves before it.
- **Change the texture.** If three beats in a row are talking-head, the fourth
  must not be. Alternate: talking → action → wide/B-roll → talking.
- **Shorten beats as the piece progresses.** A beat that would have run 90s at
  minute 2 should run 60s at minute 6.
- **Cut a beat entirely.** The most reliable fix for a sag is less material, not
  better transitions.

## B-roll is a problem-solver, not decoration

Reach for B-roll when it does a job:

| Job | Example |
|---|---|
| Cover a cut | Hide a jump cut in the dialogue track |
| Compress time | "Then we drove two hours" over 4 seconds of road |
| Show what is described | He mentions the crowd; cut to the crowd |
| Give a beat a breath | A held wide before the next segment starts |

B-roll laid over dialogue **because it looks nice** is the most common way a
vlog becomes visually busy and narratively flat. If a cutaway does none of the
four jobs above, it belongs in the stringout, not the cut.

Drone footage is almost always B-roll in this role — which means it rarely needs
frame-accurate sync, only placement (see `multicam-color-match` step 5).

## Practical order for building the rough cut

1. Lay the **dialogue/narration spine** first — the talking track, tightened
   (`resolve-tighten-recording` for dead-air removal on long takes).
2. Mark the **beat boundaries** on that spine with timeline markers.
3. Check the **shape**: is there a hook, is there a promise, do beats change
   something, where is the sag.
4. Restructure at the beat level, while it is still cheap.
5. **Then** place B-roll against the locked spine.
6. Then music, then color, then captions.

Placing B-roll before the spine is locked means re-placing all of it after every
structural change.

## What this file still needs

Everything above is general long-form craft. **The part that makes a cut feel
like this channel's cut is not here yet**, because it has to come from real
edits rather than theory.

Capture it the way `house-style` does — when a structural note is given
("terlalu lama di bagian ini", "hook-nya kurang kena", "jangan mulai dari
salam"), write the rule down here with:

- **The rule**, as an instruction
- **Why**, in the user's own terms
- **The trap**, if there is one

### Captured rules

_Not yet captured — no long-form piece has been cut on this project yet._
