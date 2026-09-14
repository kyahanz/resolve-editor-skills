---
name: proxy-workflow
description: Generate and link proxy media for heavy source footage (4K 10-bit HEVC, multicam) so playback and editing stay responsive. Load when a shoot is large, the codec is a hard one to decode, or playback stutters — and before promising a timeline that will be edited live.
---

# Proxy Workflow

Modern consumer cameras record in codecs that are cheap to *write* and expensive
to *read*. A drone writing 4K 10-bit HEVC is fine in the air and punishing on a
timeline with several angles stacked.

Proxies trade disk for responsiveness: edit against light files, relink to the
originals for the final render.

## When proxies are worth it

Probe first (`footage-triage`), then decide on evidence:

| Signal | Proxy? |
|---|---|
| `pix_fmt=yuv420p10le` / `profile=Main 10` (10-bit HEVC) | Likely yes above ~20 clips |
| 4K+ with multiple angles on stacked tracks | Yes |
| Long-form (10 min+) with hundreds of clips | Yes |
| A handful of short clips, one angle | No — proxy generation costs more than it saves |

Measured on this project's DJI Mini 5 Pro footage: `yuv420p10le`, `Main 10`,
3840x2160 — the heavy case.

## The hard constraint: proxy generation is NOT scriptable

There is **no API to generate proxies.** `GenerateProxy` appears in Resolve's
attribute space but does not exist as a real method — it is one of the borrowed
names the API falsely reports as present (see the `hasattr` fabrication entry in
`resolve_control api_truth`). Do not offer it, and do not conclude it works
because a capability probe said yes.

What IS scriptable is **linking** proxies that already exist:

- `media_pool(action='check_proxy_media_compatibility', clip_id, proxy_path, ...)`
- `media_pool(action='link_proxy_checked', clip_id, proxy_path, dry_run?, require_compatible?)`
- `media_pool_item(action='unlink_proxy', clip_id)`

So the workflow is: **generate outside Resolve, link inside it.**

## Generating proxies with ffmpeg

Write proxies to their own directory. **Never beside the source, never over the
source** — source media is read-only, without exception (AGENTS.md).

```bash
# per clip: 1/4 resolution, fast-decode H.264, same frame rate and timing
ffmpeg -i "<source>" \
  -vf "scale=iw/2:ih/2" \
  -c:v libx264 -preset veryfast -crf 23 \
  -c:a copy \
  "<proxy_dir>/<basename>_proxy.mp4"
```

Rules that keep a proxy usable:

- **Same frame rate and same duration as the source.** A proxy that differs in
  either will link but drift, and the drift shows up as sync error late.
- **Half or quarter resolution**, not an arbitrary size — keeps the aspect exact.
- **Do not re-encode audio** (`-c:a copy`) unless the codec cannot be copied;
  re-encoding risks a sample-count change.
- **Preserve rotation.** ffmpeg applies the display matrix on decode by default,
  so a rotated source produces an already-upright proxy — which is correct, but
  it means the proxy's stored dimensions differ from the source's. Verify the
  link rather than assuming.

## Link, verify, then edit

1. `check_proxy_media_compatibility` per clip — do not skip it; it is the step
   that catches a frame-rate or duration mismatch before it becomes sync drift.
2. `link_proxy_checked` with `require_compatible=true`.
3. Read back: confirm the clip reports a proxy. A link that silently fails
   leaves you editing the originals and wondering why it is still slow.

## Before the final render

**Unlink or switch back to the originals.** Rendering from proxies delivers a
quarter-resolution master, and nothing in the render report will say so —
`verify_output` checks duration and file presence, not whether the pixels came
from the proxy.

Check the delivered frame size against the intended target (see
`delivery-targets`) as the last verification step. A 1080x1920 deliverable that
was actually rendered from 540x960 proxies looks soft in a way that is easy to
mistake for compression.
