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

- **Two shots of the same subject in the same framing read as a repeat even
  when they come from different timestamps or different source clips — before
  locking a sequence, list its shots by (subject + framing) and check for
  collisions, not just by clip name.**
  - Why: a Ragunan/Blok M cut shipped with two "civet on a girl's shoulder"
    shots (same person, same framing, ~90s apart); the user spotted the
    duplication immediately on first watch even though the two came from
    source timestamps 0:59 and 3:15 of the same clip.
  - Trap: the build plan lists shots by clip filename and in-point, so two
    near-identical shots look distinct in the plan and in every gapless/
    duration check. Only a per-shot frame render exposes it — and only if you
    compare shots against each other rather than judging each one alone.

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
