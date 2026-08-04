This is a collection of HTML test patterns gathererd from around the web. These adaptations are applied:
1. UI elements are hidden/removed.
1. Scenes needing manual input are adapted to "run on rails".
1. The background is made transparent to allow their use as a test of keying.
1. They will run fullscreen, filling the whole viewport available to them.

Licenses and attributions are maintained as appropriate.

## Chips

Each card on the index page carries chips describing what the pattern is good for:

| Chip | Meaning |
| --- | --- |
| `key` | Transparent background — usable as a key source |
| `flags` | Behaviour is configurable with query-string flags |

To add another, give it a colour block in `index.css` next to `.chip-key` and `.chip-flags` —
the base `.chip` class carries the shape, so a new type only restates three custom properties:

```css
.chip-yourtag {
    --chip-fg: #fbbf24;
    --chip-bg: rgba(251, 191, 36, 0.12);
    --chip-border: rgba(251, 191, 36, 0.3);
}
```

then add `<li class="chip chip-yourtag" title="…">yourtag</li>` to the `<ul class="chips">` of
each card it applies to.

Original sources:
1. [Dark City Ambience](https://codepen.io/atzedent/pen/zxKmgpj)
1. [Chromophilic](https://codepen.io/atzedent/pen/bNgKVvN)
1. [Untitled (Sakura?)](https://codepen.io/at80/pen/kyOdeK)
1. [Birds of a Feather](https://codepen.io/tmrDevelops/pen/dMdNvy)
1. [Seascape](https://www.shadertoy.com/view/Ms2SD1)
1. [Mystify](https://www.shadertoy.com/view/MsKcRh)
1. [Marching cubes](https://github.com/mrdoob/three.js/blob/master/examples/webgl_marchingcubes.html)
1. [Physical clearcoat](https://github.com/mrdoob/three.js/blob/master/examples/webgl_materials_physical_clearcoat.html)

# Test Card

[`test-card/`](test-card/index.html) is original to this repository rather than sourced from the
web, and is the one pattern that is deliberately **opaque** — it is a reference, not a key source.
The whole card is generated as SVG at the exact device-pixel resolution of the viewport, so the
high-frequency elements land on real pixel boundaries, and it rebuilds on resize or a change of
display scaling.

## Standards stack

| Function | Standard | What it contributes |
| --- | --- | --- |
| Master architecture | ITU-R BT.1729 | 4:3 centre-cut-safe layout inside a wider frame, aspect markers, resolution-dependent detail |
| Colour bars & PLUGE | SMPTE RP 219-1 (= ARIB STD-B28) | 75% bars, chroma set, Y-ramp, black/white references, PLUGE |
| A/V sync & stereo ID | EBU Tech 3325 / 3304 | Sweep bar with a coincident 1 kHz blip; 1 kHz channel-identification tones |

## Query string

Bare flags, order-independent — e.g. `test-card/index.html?2020&sync`.

| Flag | Effect |
| --- | --- |
| `601`, `601-625` / `ebu`, `709`, `2020` | Force the colourimetry. Default follows the resolution: ≤1024×576 → BT.601, ≥3840×2160 → BT.2020, otherwise BT.709. `601` is SMPTE 170M (525-line); `601-625` / `ebu` is EBU 3213. |
| `sync` | EBU 3325 sync-flash variant: a sweep bar crosses the frame every 2 s and the strip flashes white for exactly one frame as it passes the centre, coincident with a 40 ms 1 kHz blip. Replaces the stereo ident. |
| `legal` | Map 0–100% video to codes 16–235 so the sub-black PLUGE bar is representable. Without it the −2% bar clips to black — the ID strip says which mode is active. |
| `noosd` | Drop every burned-in caption — the identification strip, detail-band labels, aspect-ratio labels and sync caption. The strip's twelfth returns to the colour bars (5.5/12) and the gratings grow to fill their cells. Marker lines and all measurement elements stay. |
| `nomarkers` | Omit the safe-area / centre-cut / geometry overlay. |
| `srgb` | Force sRGB output even on a wide-gamut display. |
| `mute` | Build the picture only, no audio. |

## Layout

Row heights are twelfths of picture height, as RP 219 specifies. The 4:3 centre is exactly 4h/3
wide and carries every element that must survive a centre cut; the flanks are extra-wide-only. All
block boundaries are snapped to whole device pixels, so abutting blocks neither overlap nor leave a
seam. At any viewport narrower than 4:3 the centre shrinks to the full width and the flanks vanish.

| Rows (of 12) | Content |
| --- | --- |
| 0.5 | Identification strip: resolution, DPR, aspect, colourimetry, output space, range, audio mode, measured refresh rate, uptime. Removed by `noosd`, which gives the half-twelfth to the bars row. |
| 1.5 | BT.1729 detail band: 1-pixel checkerboard, multiburst gratings at fs/2, fs/4, fs/6, fs/8, fs/16, and a Fresnel zone plate. Flanks carry horizontal gratings for the vertical-resolution check. |
| 5 | RP 219 row a — 75% colour bars, 40% grey flanks |
| 1 | RP 219 row b — 100% cyan / 100% white / 75% white / 100% blue |
| 1 | RP 219 row c — 100% yellow / black / Y-ramp / 100% white / 100% red |
| 3 | RP 219 row d — 15% grey flanks, black and 100% white references, PLUGE at −2%, +2%, +4% |

RP 219's row a is 7/12; here it is 5/12, with the 2/12 released to the identification strip and the
detail band. Every other dimension matches ARIB STD-B28 — at 1920×1080 the seven bar columns come
out as five of 206 px and two of 205 px, and row d as 309, 411, 171, 69, 68, 69, 68, 69, 206 px, the
sample counts tabulated in the standard. Widths are derived from exact fractions and rounded per
boundary, so which cell absorbs a rounding pixel can differ from the standard's table by one; the
totals are identical. With `sync`, row d gives up its bottom twelfth to the sweep strip.

## Colourimetry

Selecting a colour space converts the bars' primaries; it does not change what the neutrals mean.
Chromatic elements are linearised with a pure 2.4 curve, matrixed from the selected primaries into
the output space in linear light, and re-encoded with the same curve — so BT.709 in a BT.709 card is
a bit-exact identity and greys never drift. Neutral levels (greyscale, references, PLUGE) are
emitted verbatim, which keeps the PLUGE steps exact.

On a wide-gamut display the bars are emitted as `color(rec2020 …)` so BT.2020 primaries are not
clipped into sRGB on the way to the panel; the identification strip reports which output space is in
use. Pass `srgb` to force the narrow path.

## Audio

Both modes run at −18 dBFS (EBU alignment level) and start automatically where autoplay permits;
otherwise any click or keypress starts them, and the identification strip shows the audio state.

- **Stereo ident** (default): continuous 1 kHz on the left channel, 1 kHz on the right interrupted
  by a 250 ms gap once per second. Inverted channels or a sum-to-mono phase cancellation are audible
  immediately.
- **Sync flash** (`sync`): a 40 ms 1 kHz blip every 2 s, scheduled on the audio clock and issued
  early by the reported output latency so it leaves the speaker as the flash is presented. The
  sweep position is derived from the same clock, one frame ahead, so bar, flash and blip coincide.
