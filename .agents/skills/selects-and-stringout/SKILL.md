---
name: selects-and-stringout
description: The ladder from a large unsorted shoot to a cuttable timeline — ingest, transcribe, mark selects, build a stringout, then tighten. Load when the footage count is too large to watch end to end (roughly 30+ clips, or any long-form piece), before attempting an assembly.
---

# Selects and Stringout

A 15-clip shoot can be reviewed clip by clip. A 200-clip day cannot. The method
that works at small scale — contact-sheet everything, look at all of it, then
assemble — collapses quietly: it gets slower, then it gets expensive, then it
starts skipping things without saying so.

This is the ladder that scales. Each rung reduces the material before the next
rung has to look at it.

## The ladder

```
raw shoot  →  triage  →  organized bins  →  transcript  →  selects
           →  stringout  →  rough cut  →  fine cut
```

Never skip a rung to save time. Jumping from raw footage to "assemble the best
shots" is how you end up re-cutting three times, because the selection was made
on incomplete information.

### 0. Triage (see `footage-triage`)

Formats, orientation, frame rates, audio presence, delivery target. Non
negotiable, and it is cheap. Mixed orientation or a wrong timeline resolution
discovered at rung 6 costs everything built on rungs 1-5.

### 1. Organize before looking

One bin per **shoot day**, sub-bins per **camera**. Import into the current
folder deliberately — `ImportMedia` has no destination parameter and lands
wherever the current folder happens to be (`media_pool set_current_folder`
first, always).

Name bins for what they are, not what you hope they contain. `Day1/Drone`,
`Day1/Pocket`, `Day1/Audio` beats `Best Shots`.

### 2. Transcribe anything with speech — first, and before watching it

For talking-head, interview, or narration footage, the transcript is the index.
Searching text is orders of magnitude cheaper than scrubbing video, and it is
how you find "the bit where he explains the route" without watching forty
minutes.

See `captions-workflow` for the transcription route that works on this stack.
Once transcripts exist, `edit_engine(action='search_spoken_content')` searches
across every transcribed clip and returns timestamped hits with handles.

**B-roll has no transcript.** For silent footage (drone, cutaways), use contact
sheets — see `footage-triage` step 4 — but budget them: one sheet per clip,
and reading each sheet costs real tokens. Above ~50 clips, sheet only the clips
a transcript or a shot list says you need.

### 3. Selects — mark the good parts, do not trim yet

A select is an in/out on the part of a clip worth keeping. Record them as
**marks on the media pool item**, not by cutting:

- `media_pool(action='set_clip_marks', clip_ids=..., mark_in=..., mark_out=...)`
- or markers via `media_pool_item_markers` when a clip has several usable moments

Why marks rather than subclips or a timeline: marks are non-destructive, survive
re-organisation, and carry into an assembly without committing to an order.

**Be generous here.** The default on a first pass is to keep too much. Trimming
a long selects reel is fast and visible; recovering a moment you discarded at
this stage means going back to raw footage, and usually it just never happens.

### 4. Stringout — order without judgement

Put every select on one timeline, in rough narrative or chronological order,
end to end. Do not tighten. Do not grade. Do not add music.

The stringout is deliberately too long — often 3-5x the target runtime. Its job
is to make the shape of the piece visible for the first time. You cannot judge
structure from a bin.

Build it in one call with positioned `clip_infos` (see `resolve-rough-cut` for
the exact shape and the exclusive-`end_frame` trap), and name the timeline for
what it is: `<episode> stringout`.

### 5. Rough cut — the first pass with opinions

Duplicate the stringout, do not edit it in place. The stringout stays as the
reference of everything that was available.

Now cut for structure: what is the hook, what earns its place, what order tells
the story. This is where `house-style` applies — read it before making these
calls, since it carries the corrections already given.

### 6. Fine cut — pacing, then finishing

Tighten within shots, place B-roll against the dialogue track, then and only
then: color, audio balance, captions, titles.

## Scale limits worth knowing before you promise a timeline

| Limit | Number | Consequence |
|---|---|---|
| Contact-sheet review | ~30 clips comfortable, ~50 painful | Above this, sheet selectively — driven by transcript or shot list |
| `media_analysis` adaptive sampling | 80 frames per clip max | A 35-minute take is sampled to roughly its first 16 minutes; **the tail is never seen** |
| Long takes | — | Ask the shooter for short clips per subject rather than 20-35 minute takes; it is a sampling problem, not a preference |

## When footage arrives pre-culled

If the shooter already threw away the unusable takes, scale the effort **down**:
fewer frames per clip, fewer sheets, and trust the selection. The expensive part
of selects is separating usable from unusable, and that work is already done.
Re-litigating it wastes the budget that should go into structure.

## The rule

**Reduce before you examine.** Every rung exists to hand the next rung less
material. The failure mode at scale is not choosing badly — it is examining
everything at full cost and running out of budget before reaching the rungs
where the actual editing happens.
