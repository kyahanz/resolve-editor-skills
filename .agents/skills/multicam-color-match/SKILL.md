---
name: multicam-color-match
description: Bring footage from two or more cameras (drone, pocket gimbal, phone) to a common look — identify each camera from metadata, apply its correct manufacturer log LUT, then match exposure and white balance to a hero shot. Load before grading any timeline that mixes cameras, and before planning multicam sync.
---

# Multi-Camera Match

Two cameras that shot the same event will not cut together straight out of the
box, even when both are "DJI". They have different sensors, different log
curves, and different auto-exposure behaviour. This skill is the order of
operations that gets them to one look — and the traps that make a wrong guess
look plausible.

## Step 1 — Identify the camera per clip from metadata, never the filename

DJI cameras all write `DJI_<timestamp>_<n>_D.MP4`. The filename cannot tell a
Mini 5 Pro from an Osmo Pocket. The **format-level `encoder` tag** can:

```bash
ffprobe -v error -show_entries format_tags=encoder \
  -of default=noprint_wrappers=1:nokey=1 "<clip>"
```

Measured output on this project's footage: `DJI Mini5Pro`.

Bucket the clips by that string before touching color. A folder called
"Footage Drone" is a claim about intent, not a guarantee about contents — a
pocket-gimbal clip dropped in the same folder will silently receive the drone's
LUT and come out with the wrong contrast curve.

## Step 2 — One LUT per camera, and they are NOT interchangeable

| Camera | Log profile | LUT (ships in Resolve's LUT root) |
|---|---|---|
| DJI Mini 5 Pro | **D-Log M** | `DJI Mini 5 Pro D-Log M to Rec.709 LUT.cube` |
| DJI Osmo Pocket 4 | **D-Log** | `DJI OSMO Pocket 4 D-Log to Rec.709 V1.0.cube` |

**D-Log and D-Log M are different transfer curves**, not two names for one
thing. D-Log M has a shorter toe and less shadow range; pushing D-Log M through
a D-Log LUT (or vice versa) produces wrong contrast and a colour cast that is
easy to mistake for "this camera just looks different" — and then get
"corrected" with a creative grade that is really papering over a decode error.

Check what is actually installed before assuming a download is needed:

```
lut(action='list')     # search the returned set for the camera's name
```

Apply per clip with `graph(action='set_lut', node_index=1, source='item', ...)`,
matched to that clip's camera bucket.

## Step 3 — Do not trust the file's colour metadata

Measured on a D-Log M clip from this project:

```
color_space=bt709   color_transfer=bt709   color_primaries=bt709
```

The camera tags log footage as Rec.709 anyway. **There is no metadata field
that tells you whether a clip is log.** The reliable signals, in order:

1. The camera model (step 1) plus what that model was set to shoot — ask the
   user once per shoot; the answer applies to every clip from that camera.
2. The image itself: log footage looks flat and milky with lifted blacks.

Never conclude "not log" from the colour tags. That mistake leaves a LUT off
and the footage looks merely "a bit flat", which reads as a grading taste
problem rather than a missing decode step.

## Step 4 — LUTs equalise the SPACE, not the LOOK

After each camera has its correct LUT, the shots are in the same colour space
and still will not match: exposure, white balance and contrast differ per camera
and per shot. Matching is a separate pass, and it comes **before** any creative
grade.

1. **Pick a hero shot** — the one whose exposure and white balance you want
   everything else to sit against. Usually the camera with the most screen time,
   on a representative subject, in the dominant lighting.
2. **Match the others to it.** Live: `timeline_item_color(action='bulk_match_to_hero')`
   (dry-run first; it returns a confirm token). Offline, from extracted frames:
   the `drx` catalog's `match_to_reference`, `level_clips` (within-camera drift),
   or `white_balance_match` (when there is a known-neutral patch).
3. **Then** grade creatively on top, once — not per camera.

Grading each camera separately to "look nice" and hoping they meet in the middle
is how a cut ends up with a visible colour jump at every angle change.

## Step 5 — Sync: the drone probably has no audio

**Measured on this project: every drone clip reports `Audio Ch: 0`.** DJI
Mini-series drones commonly record no audio at all.

That kills the default multicam sync method. Audio-waveform alignment needs
audio on both sides; with a silent drone there is nothing to correlate.

What is left, in order of reliability:

- **Time-of-day.** DJI filenames carry **local** time (`DJI_20260906071340_...`
  = 07:13:40 local) while `format_tags.creation_time` carries **UTC**
  (`2026-09-06T00:13:41Z` for the same clip — a +7 offset here). Pick one
  convention and stay in it; mixing them puts clips hours apart.
- **A visible sync event** — a clap, a gesture, a vehicle passing frame — found
  by eye in both angles.
- **Manual placement**, which for cutaway B-roll (the usual drone role) is
  perfectly adequate: drone shots rarely need frame-accurate sync to anything.

Ask what the drone footage is *for* before spending effort on sync. If it is
B-roll over a talking-head track, it does not need syncing at all — it needs
placing.

## Step 6 — Check the build before promising multicam features

Both native multicam surfaces are **21.1+**:

- `MediaPool.CreateMulticamClip`
- `Timeline.AutoAlignClips`

On a build chosen to keep bridge scripting alive (21.0.x), neither exists.
Confirm with `resolve_control(action='check_version_support')` before offering
them. What still works there: `media_pool(action='setup_multicam_timeline')`,
which builds a stacked prep timeline (one angle per video track) — useful for
eyeballing angles, but the actual multicam-clip conversion stays a UI step.

## Order of operations, condensed

1. Bucket clips by camera (`format_tags.encoder`)
2. Apply that camera's own log LUT — per bucket
3. Pick a hero shot; match the other buckets to it
4. Creative grade once, on the matched result
5. Sync only if the content actually requires it

Getting 2 and 3 backwards — creative grading before matching — is the single
most common way a multi-camera cut ends up impossible to fix later.
