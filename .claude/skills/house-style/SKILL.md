---
name: house-style
description: The editorial and finishing preferences this project's work is judged against — cut rhythm, shot selection, delivery conventions, and the corrections that have already been given. Load before assembling, restructuring, or refining any cut so the same note does not have to be given twice.
user-invocable: false
---

# House Style

The craft guides in `docs/guides/` describe editing in general. This file
describes **how this editor wants it done** — the accumulated, specific
corrections that would otherwise have to be repeated every session.

Read it before any edit task. Append to it whenever a correction is given.

## The capture protocol

This file is only worth what gets written into it. When the user corrects an
editorial decision — rejects a cut, changes a shot choice, adjusts a duration,
says "not like that" — do not just fix it. Fix it, then add the rule here.

A useful entry has three parts:

- **The rule**, stated as an instruction, not an observation.
- **Why**, in the user's terms — what it was in service of.
- **The trap**, if there is one: what makes it easy to get wrong.

Write rules that are falsifiable. "Cut on motion" is a rule; "make it feel
dynamic" is not. If a correction is one-off and situational, it does not belong
here — this file is for what generalizes.

When an entry turns out to be wrong or too broad, edit it. A stale rule
confidently followed is worse than no rule.

---

## Pacing and rhythm

<!-- Hold lengths, when to cut early, what "too long" means for this material. -->

- **When the user gives an explicit duration cap, hit it with a margin — trim
  hold lengths proportionally across all shots first, don't drop shots from
  the arc unless shot count itself is the problem.**
  - Why: asked to compress a 1:42 cut to under 1:00 while keeping the same
    6-shot story arc; shortening every hold (not cutting shots) preserved the
    narrative and landed comfortably under the cap (55s) rather than right at
    the edge.
  - Trap: duration-change math has to be redone by hand for every shot
    (source in/out, record position) — always re-verify total duration and
    zero gaps/overlaps after any timing change, never assume the arithmetic
    was right.

- **A beat-synced montage needs a rhythm peak, not just uniform tempo-matched
  density — build in at least one shot that holds longer than its grid at the
  piece's visual high point, so the cut has a felt arc instead of one
  unbroken cut rate.** Passing the beat-grid math (cuts land within a frame of
  every downbeat) proves the edit is *synced*; it does not prove it is
  *satisfying* to watch — those are different checks and the first does not
  clear the second.
  - Why: delivered a 29s beat-synced short-form cut that measured
    frame-accurate against the downbeats and still read as "kurang"
    (flat/underwhelming) — cutting at a uniformly fast rate for the entire
    back half never let any single moment land before the next cut arrived.
  - Trap: `beat-sync-editing`'s accelerating-grid structure (slower before the
    phrase boundary, faster after) creates energy on paper, but "faster"
    still means *every* shot in that section is equally quick — it is not
    itself a rhythm break. A rhythm break is one shot in the fast section
    deliberately held past its grid length; the grid gives permission for
    where cuts *can* land, not a mandate that they all must.

- **Check footage variety before promising cut density — if the source is a
  small number of clips of the same single subject, say so before building,
  not after the user notices the cut feels thin.** A handful of clips can
  supply a few genuinely distinct shots of one subject, not ten; committing to
  a 10-shot fast-cut structure on 3 source clips of one statue guarantees the
  piece runs out of real variety before it runs out of shots.
  - Why: the GWK short-form cut only had 3 source clips for all 10 shots;
    after fixing one batch of literally-duplicate framing (see Shot
    selection), the piece was still capped by how much genuinely different
    footage of one monument exists — no amount of re-cutting fixes a supply
    problem.
  - Trap: "different timestamp in the same clip" reads as a distinct shot in
    the edit's own metadata but often is not a distinct shot on screen (see
    `beat-sync-editing`'s Clip A orbit case) — evaluate variety by what's
    visually different (subject, framing, scale, distance), not by how many
    source files or timestamps were touched.

## Shot selection

<!-- What earns a place in the cut; what gets dropped even when it's a good shot. -->

- **A shot earns its place by carrying story/human energy and connecting to
  the piece's event, not by being visually striking in isolation.** Drop a
  static or empty "beauty" shot even when it's well composed.
  - Why: a lotus-monument shot and an empty-walkway "hero" shot both looked
    good on their own but read as "kurang" (lacking) once cut in, because
    they were visually disconnected from the crowd/event energy carrying the
    rest of the piece. Swapping them for busier, event-connected footage
    (a group kneeling near a truck, a dense boulevard) fixed the note
    immediately — no grading or pacing change needed.
  - Trap: a still frame or contact sheet cannot tell you this — a shot can
    look great as an image and still be the weak link in motion, because
    what's missing (movement, people, story) only shows up watching it in
    context with its neighbors.

- **Duplicate framings concentrate in whichever source clips the cut reuses
  most — before locking a sequence, group its shots by source clip, render the
  members of every clip used 3+ times side by side in one sheet, and look.**
  - Why: the Ragunan/Blok M cut shipped with three near-duplicate pairs (two
    civet-on-pavement closeups, one reaction shot twice, one deer-behind-fence
    framing twice). All three came from the three clips the cut leaned on
    hardest — one clip supplied 7 of the 47 shots. Grouping by source clip put
    every pair on the same row, where they were obvious in one glance.
  - Two weaker checks were tried first and BOTH missed all three pairs:
    - Listing shots by (subject + framing) from the build plan. The plan
      distinguishes shots by filename and in-point, so near-identical shots
      read as distinct, and every gapless/duration check passes.
    - Automated similarity on one mid-frame per shot. The true pairs scored
      0.20-0.33 structural similarity while unrelated shots that merely shared
      a composition scored 0.48-0.66 — the ranking was worse than useless.
      A single frame does not represent a moving shot; do not trust this.
  - So: the grouped visual sheet is the check that works. The user watching it
    once will out-detect any metric, so the goal is to look before they do.

- **Do not discard a long, mostly-empty source clip on the strength of a
  contact sheet — scan it densely (2-3s steps) before ruling it out.** A
  sparse clip's few good seconds fall between contact-sheet samples far more
  often than a busy clip's do.
  - Why: a 2:50 locked-off plaza clip was judged "90% empty, not worth using"
    from a 12-tile contact sheet and dropped. It actually contained the one
    shot the user most wanted — the group cycling past camera waving — in a
    ~4s window that no tile landed on. The user had to ask for it by name.
  - Trap: the emptier the clip, the more confident the "nothing here" read
    from a contact sheet feels, and the more wrong it is — sampling density
    should go UP for sparse clips, not down.

## Cut points

<!-- Cut on motion vs on rest, handles, how much air before and after a beat. -->

- **"Transisinya patah-patah / kurang smooth" on a hard-cut assembly usually
  means a LUMINANCE jump between neighbouring shots, not a missing dissolve —
  measure mean luma of every candidate shot window and order the sequence as a
  monotonic ramp before reaching for transitions.**
  - Why: a hazy-reservoir vertical cut was called "patah-patah dan kurang
    smooth". Measuring mean luma of each shot's in-point window exposed the
    real cause: the order ran 114 → 38 → 121, a near-black top-down shot
    sandwiched between two bright ones. Re-ordering the same nine shots into a
    dark→bright ramp (38 → 59 → 65 → 70 → 114 → 121 → 126 → 136 → 165) fixed
    the complaint with no transitions added, and read as a natural light arc.
  - Measure it, don't eyeball it, with one ffmpeg call per window:
    `ffmpeg -ss <t> -t 1 -i <clip> -vf "scale=160:-2,format=gray" -f rawvideo -`
    piped to a mean. Contact sheets are gamma-mangled and tiny; they will not
    rank shots reliably.
  - Trap: the instinct on this note is to add cross-dissolves, which on a 21.0.x
    build is a manual hand-off and does not fix the underlying flash anyway. A
    dissolve smooths a luma jump by hiding it; ordering removes it.

- **"Kurang sinkron" on a cut that already measures frame-accurate against the
  beat grid means the cuts are on the METRONOME but not on the music's ACTUAL
  events — plot the RMS energy curve and cut on where the energy changes, not
  on the nearest inferred downbeat.**
  - Why: a 4-shot cut measured 0.3–7.8 ms off every downbeat and still came
    back as "transisinya sesuain sama beat music". The track's real event — a
    2.4x energy jump, its only drop — sat at 5.43–5.52 s, while the nearest
    inferred downbeat was 4.923 s. The first cut was therefore **506 ms early**,
    almost a full beat (604 ms) at 99 BPM: the picture changed before the music
    did. Moving that one cut to 5.522 s fixed it; nothing else changed.
  - How to find it: `librosa.feature.rms` at hop 512 for a per-second energy
    bar chart to spot the section change, then re-run at hop 128 over the
    suspect 1–2 s window to get the exact frame the energy steps. Cross-check
    against `librosa.onset.onset_strength` peaks for the accent list.
  - Trap: `detect_beats` reports high confidence for the *spacing* of the pulse,
    and that is a true statement about the metronome — it says nothing about
    whether a given downbeat carries any musical weight. A uniform grid over a
    track that has one drop will happily offer nine equally-valid-looking cut
    points, only one of which the listener is actually waiting for.

- **A supplied music track sets the piece's LENGTH, not just its cut points —
  read its duration first and re-plan the shot count before touching the
  timeline.**
  - Why: a locked 52 s cut had to become 23.5 s the moment the client's track
    arrived; keeping the old structure and "syncing" it would have meant
    dropping half the shots anyway, but discovered late instead of planned.
  - Trap: the beat grid is the visible part of the job, so it is easy to jump
    straight to `detect_beats` and only notice the runtime mismatch after the
    cut points are computed.

- **`clip_infos.end_frame` in `media_pool.create_timeline_from_clips` is
  EXCLUSIVE. Pass `start_frame + duration`, never `start + duration - 1` — the
  off-by-one leaves a 1-frame gap at the tail of every shot, which renders as a
  one-frame BLACK FLASH at every cut.**
  - Why: a 9-shot cut came back as "kaya ada kedut-kedut" (twitchy). Per-frame
    luma on the render showed luma 0.0 at frames 151, 294, 437, 582, 726, 869,
    1013, 1300 — the last frame of all eight shots. The defect had been present
    since v1 and survived three re-cuts and two frame-verification passes.
  - Trap: `timeline.get_items_in_track` reads back `start:216000 end:216239
    duration:239` for a 240-frame slot. That LOOKS contiguous and it is easy to
    call it "gapless" — the tell is `duration` being one less than the spacing
    between consecutive `start` values. Compare those two numbers explicitly.
  - The check that actually catches it: decode the render to grayscale and scan
    for frames with mean luma < 5. Reading one frame per shot never finds it,
    because the black frame is a single frame at the boundary and every sampled
    frame sits inside the shot.

- **Never declare a cut, grade, or color change "done" from an API success
  response alone — render a preview and pull actual frames before reporting
  a result as final.**
  - Why: a write call (e.g. `SetCDL`) can return `success:true` on a payload
    that silently did nothing (wrong key casing produced a no-op identity
    grade); a rendered frame is the only thing that can't lie about what's
    actually in the file.
  - Trap: the obvious verification path (Resolve's still-export folder under
    `~/Documents`) can be silently blocked by Windows Controlled Folder
    Access, failing with a misleading "file not found" rather than a
    permissions error. When that happens, render to a project-owned scratch
    folder instead and pull frames with ffmpeg — don't give up on
    verification just because the default path is blocked.

## Structure and openings

<!-- How a piece starts, what the first frames have to do, how it lands. -->

- **A cold-open/highlight arc: wide establishing shot → 2-3 shots building
  energy/movement → a closer with real narrative weight, not just visual
  novelty.**
  - Why: this shape read well once the closer carried the same event-energy
    as the rest of the piece, instead of being a quiet "pretty" shot tacked
    on at the end for visual variety.
  - Trap: a visually unique "hero" shot is not automatically the right
    closer — if it's disconnected from the piece's energy it reads as a lull
    right before the cut ends, not a landing.

## Rejected by default

Things not to add unless explicitly asked. Seeded from the rough-cut deliverable
contract in the `resolve-rough-cut` skill, which exists because this work gets
thrown away:

- Titles, captions, and text cards
- Transitions (an assembly is hard cuts)
- Effects and speed ramps
- Music beds
- Grading on a cut that was asked for as an assembly

## Delivery conventions

<!-- Aspect ratios, timeline naming, versioning, where renders go. -->

- **After the user adds nodes by hand, the grade you previously set is STILL on
  the old node — re-setting it on the new node applies it TWICE. Explicitly
  neutralise every node you are not using.**
  - Why: a CDL set on NodeIndex 1 survived the user inserting two nodes (it
    became node 1 of 3). Setting the same CDL on node 3 doubled a 1.55 slope
    into ~2.4: mid grey measured 1.0 (blown) where the intended value was 0.67.
    Every API call returned success; node counts and LUT readback all looked
    right.
  - Neutral CDL is `Slope "1 1 1" / Offset "0 0 0" / Power "1 1 1" /
    Saturation "1"` — send it to each unused node index.

- **Verify a grade WITHOUT rendering: `safe_export_lut` the item, then compare
  the cube numerically against the transform you intended.** This is the check
  to use when the user has asked you not to export, and it is strictly sharper
  than looking at a frame.
  - Export (Color page only — it returns a bare False elsewhere), read the
    `.cube`, sample probe colours, and compare against camera-LUT-then-CDL
    computed by hand. A correct grade matches to ~0.0004; the doubled grade
    above showed 0.36 at mid grey, which is what exposed it.
  - Also check mean deviation from identity: 0 means the grade is a silent
    no-op. Differing values per clip prove per-clip trims actually landed.
  - Validate your own cube sampler against `ffmpeg lut3d` on a few solid
    colours before trusting a mismatch — nearest-neighbour sampling differs
    from ffmpeg's trilinear by ~0.01-0.03, and anything larger is a real defect,
    not a sampling artifact.

- **`timeline_item.set_audio` cannot set Volume on 21.0.x: it returns
  `{"success": true, "Volume": false}`.** The outer success is the CALL, the
  inner per-property flag is the WRITE. Read the inner flag; an audio level
  change has to be handed to the user.

- **There is no AddNode in Resolve's scripting API on any build — a requested
  "3 serial nodes" layout cannot be scripted. Use a COLOR GROUP instead: the
  group's Pre-Clip graph, the clip graph, and the group's Post-Clip graph are
  three independently addressable stages in guaranteed order.**
  - `graph grade_capabilities` → `graph_methods` lists only Get/SetLUT,
    node cache, node enabled, ApplyGradeFromDRX, ApplyArriCdlLut,
    ResetAllGrades. A fresh clip has exactly 1 node and stays that way.
  - Working layout for a log-normalise + look job:
    `project_settings.add_color_group` → `timeline_item_color.assign_color_group`
    (NOT `assign_to_color_group`, which is not an action) per item →
    `graph.set_lut(source="color_group_pre", group_name=...)` for the camera LUT
    → `timeline_item_color.safe_set_cdl` per item for the look. The group Pre
    graph runs before the clip graph, so the LUT is guaranteed to land before
    the CDL — which matters, because inside a SINGLE node Resolve applies the
    node LUT after the primaries by default, silently inverting the intent.
  - `safe_set_cdl` wants ONE `cdl` dict with capitalised keys and
    space-separated string triples: `{"NodeIndex":"1","Slope":"1.05 1.0 0.96",
    "Offset":"...","Power":"...","Saturation":"1.12"}`. Flat `slope=[...]`
    kwargs are refused outright, which is the good case — see the CDL casing
    no-op trap above for the bad one.
  - The only route to a real multi-node graph is `ApplyGradeFromDRX`, which
    REPLACES the graph — so it needs a 3-node .drx from somewhere, i.e. the user
    building one clip by hand first. Offer that, do not pretend to script it.

- **On a build without `TimelineItem.SetSpeed` (pre-21.1), speed changes are
  still reachable: set the media pool clip's `FPS` clip property to a fraction
  of the timeline rate, then let the source range set the item length.**
  `media_pool_item.set_clip_property(clip_id, "FPS", "29.97")` on a 59.94 clip
  makes every source frame occupy two timeline frames — a true 50% speed, no
  `SetSpeed` and no derivative media. Resolve keeps the clip's frame NUMBERING
  unchanged and only restates its duration, so source in/out stay in the
  original frame space; ask for N source frames and the item lands at
  N x (timeline_fps / clip_fps) timeline frames.
  - Pair it with `timeline_item.set_retime(process=3, motion_estimation=2)`
    (optical flow), which IS available pre-21.1 — `get_retime`/`set_retime`
    carry the retime QUALITY and are a different surface from `set_speed`.
    Without it the slow shot is frame-doubled and visibly stutters.
  - Verify from the render, not the API: per-frame mean-absolute-difference
    across the slowed shot should halve against the same shot at 100%, and the
    count of near-zero diffs (duplicate frames) must be ZERO. Measured 0.248 ->
    0.124 with 0/331 duplicates, against an unchanged 0.572 on a 100% control
    shot in the same render.
  - Limits, state them: it is UNIFORM per clip, not a ramp, so the speed change
    lands as a step — put that step on a musical accent and it reads as
    deliberate. The FPS attribute is project-wide for that pool item, so a clip
    used more than once cannot have two different speeds this way. Pick a
    fraction that divides the slot cleanly (50% needs an even frame count) or
    the shot boundary drifts a frame.
  - Keyframes are NOT a fallback on these builds — `get_keyframes` fails with
    "has no attribute 'GetKeyframeCount'", so scripted animated push-ins are
    out too.

- **Cross-dissolve transitions cannot be added via script when the project's
  Resolve build was chosen to keep bridge scripting alive (e.g. 21.0.4.5) —
  `TimelineItem.AddTransition` needs 21.1+, and 21.1 free removes scripting
  entirely. Hand transitions off to the user for manual placement in the Edit
  page rather than retrying an automated route.**
  - Why: the one automated workaround (author an offline `.drt` via
    `drt.assemble`, which needs `media_pool.capture_media_template` once per
    source file) switches the *live* Resolve project mid-session and can hang
    on an unclosable modal — this actually happened once, and took the whole
    bridge connection down until the user manually reopened the project and
    restarted the bridge script.
  - Trap: retrying an automated transition route feels more thorough than
    telling the user to do it by hand, but the manual step costs seconds and
    the automated one already caused a real outage — don't re-attempt it
    without the user explicitly asking again with the risk spelled out.

---

## Where personal grading taste lives

Colour and look preferences are **not** kept here — this file travels with the
repository. Grading taste lives in the user-level `colorist-assistant` skill and
in persistent memory. Load those for look selection, grade transfer, and the
Resolve API traps around them.

If any entry below would be specific to one person rather than to this project's
work, it belongs in the user-level skill instead, and this file should be
gitignored rather than committed.
