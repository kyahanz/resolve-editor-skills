---
name: delivery-targets
description: Platform delivery specs and the render-and-verify loop — aspect ratio, resolution, frame rate, loudness, and the safe margins captions must sit inside. Load when setting up a timeline for a known destination, and again before rendering a deliverable.
---

# Delivery Targets

The format question belongs at the **start** of an edit, not the end. Timeline
resolution is locked in practice once an edit exists on it, and frame rate is
locked by the API the moment a timeline is created. Choosing wrong means
rebuilding.

See `footage-triage` for settling this with the user before the first timeline
exists.

## Specs by destination

| Destination | Frame | Aspect | Notes |
|---|---|---|---|
| Reels / TikTok / Shorts | 1080x1920 | 9:16 | The dominant vertical target. Source shot vertical (rotation=90) maps 1:1 |
| YouTube (standard) | 1920x1080 or 3840x2160 | 16:9 | 4K only if the source is genuinely 4K and the upload budget allows |
| YouTube Shorts | 1080x1920 | 9:16 | Same as Reels; a separate cut, not a crop of the landscape edit |
| Square (feed posts) | 1080x1080 | 1:1 | Rare now; confirm before assuming |

**Frame rate:** match the source unless there is a reason not to. Source at
59.94 delivers fine at 59.94 for motion-heavy content (drone, sport); halve to
29.97 only if the platform or file size demands it — never mix.

**Never crop one master into another aspect.** A 16:9 edit centre-cropped to
9:16 loses ~70% of frame width and destroys every composition in it. If both
orientations are needed, they are two cuts built from orientation-appropriate
footage — see `footage-triage`.

## Safe margins — the part most edits get wrong

Vertical platforms overlay their own UI on top of the video. Anything important
inside those bands is covered on the actual phone, even though it looks fine in
Resolve.

Budget, as a fraction of a 1080x1920 frame:

- **Top ~10%** — account name, follow button
- **Bottom ~20%** — caption text, music credit, action buttons
- **Right ~12%** — like/comment/share column

So: **keep burned-in captions and any key subject inside the middle ~70%
vertically.** A caption sitting at the visual "bottom third" like a 16:9 lower
third will be behind the platform's own caption on a phone.

For 16:9, standard title-safe (inner 90%) and action-safe (inner 93%) still
apply for anything destined for a TV or an embed with controls.

## Loudness

Ask for the named standard rather than guessing a number —
`render(action='list_loudness_standards')` returns the supported set:
`web`, `podcast`, `ebu_r128`, `atsc_a85`, `ott_dialogue_gated`.

For social/streaming, `web` (around -14 LUFS integrated) is the normal target.
Platforms normalise anyway, so mastering hot gains nothing and costs headroom.

Attach the standard to the delivery target and hand its emitted
`loudness_target.target` to the advanced server's `loudness_qc` for
verification. Dialogue-gated standards assert true peak only — see
`resolve-delivery` for why.

## The render loop that works on this stack

Measured working sequence:

```
render(action='set_format_and_codec', {format: 'mp4', codec: 'H264'})
render(action='set_settings', {settings: {TargetDir: <writable dir>, CustomName: <name>}})
render(action='add_job')            → job_id
render(action='start', {job_ids: [job_id]})
render(action='get_job_status', {job_id})   → poll to Complete
render(action='verify_output', {job_id, expected_duration_seconds: <n>})
```

Two things that matter:

- **`TargetDir` must be a writable directory that exists.** Create it first.
  Do not target `~/Documents` — on Windows with Controlled Folder Access it
  fails with a misleading "file not found" rather than a permissions error.
  A project-owned scratch folder is the safe choice.
- **Always `verify_output`.** `JobStatus` reports `Complete` even when the
  render produced a near-empty stub, and Resolve rewrites the job's own
  MarkIn/MarkOut to match the collapsed extent — so the job metadata agrees
  with itself and is still wrong. `verify_output` checks the actual file and
  cross-checks the timeline items. Pass `expected_duration_seconds` so a
  deliberate short render is not mistaken for a collapse, and run it **before**
  deleting the job — a deleted job carries no `TargetDir`.

## Renders are also how you verify the picture

The same render is the only reliable way to see what is actually in the cut:
render, then pull frames with local ffmpeg and look at them. This is the
workaround when Resolve's own still-export path is blocked, and it is the rule
`house-style` (Cut points) requires before calling any grade or cut finished.

## Before delivering

- Timeline resolution and frame rate match the target
- Captions/subject inside the platform's safe margins
- Loudness measured against the named standard, not eyeballed
- `verify_output` passed with a stated expected duration
- Frames pulled from the render and actually looked at
