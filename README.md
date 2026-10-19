# H3LM — a GPU lightmapper for Halo 3 and Halo 3: ODST

**H3LM bakes Halo 3 and ODST lightmaps on your graphics card, in minutes instead of hours, with no `tool.exe` and no farm.**

It writes the same lightmap tags the game already reads. No shader change, no engine change, no patched game files. Your map loads in Sapien and in MCC like any other lightmapped map.

> Free to use. Windows 10/11, 64-bit. Source code is private for now; this page holds the download and the docs.

---

## Why H3LM

Halo 3's `tool.exe` lights a map on the **CPU**, through a "farm" of worker processes. It is slow, it is hard to set up, and while it runs your PC is close to useless.

H3LM does the whole bake on the **GPU** with DirectX 12 ray tracing.

| | `tool.exe` (MCC Editing Kit) | **H3LM** |
|---|---|---|
| Where the light is computed | CPU farm | GPU (DirectX 12 ray tracing) |
| zanzibar, whole bake | **37 min** at its *lowest* quality | **11.5 min** at Optimized quality |
| Setup | farm scripts, several processes, tool commands | one window: pick a scenario, press **Start** |
| Your PC while it bakes | busy | stays usable |
| A map that was never lightmapped | run tool's setup steps first | works directly — H3LM makes the missing tags itself |
| Halo 3: ODST | separate kit, separate workflow | same window, picks the game for you |
| Lightmap page size | set per map (1024 on most) | 256 up to **4096** (`--size`) |
| Re-baking when nothing changed | starts from zero | reuses the finished steps (~21% faster, same result) |

*Times measured on an RTX 3060 Ti.*

---

## What makes it better

### Cleaner light — less noise, same look
- **Per-sample random seeds.** The stock engine gives all samples in a texel the *same* random rays, so extra samples remove no noise. H3LM gives each sample its own: about **30% less grain** for about 4% more time.
- **Even sky rays.** Sky rays are spread evenly over the sky instead of at random. No blotchy walls, and half the sky rays give the same quality.
- **Smoothed light direction.** Light direction is smoothed inside each surface, never across a shadow edge. Bump-mapped walls lose their mottled look; brightness and shadows do not change.
- **One light direction for all colours** (as Unreal does). No colour speckle, less blue tint on metal.
- **Bounce-light denoise** that respects edges.
- **Sharp shadow edges on big pages.** Extra samples go only where they matter: shadow edges, light edges and chart borders.

### Fewer artefacts
- **UV seams stitched in world space.** No seam lines across walls or where BSP pieces meet.
- **No light bleeding under walls.** The engine pushes samples 0.08 units off a surface, which carries them through thin walls. H3LM uses 0.01, which matches Bungie's own 2007 lightmaps.
- **No hot pixels, no black props.** Firefly clamp, gutter fill and per-vertex repair.
- **Fixes MCC's over-bright object probes.** MCC's `tool.exe` adds direct light to instance probes that Bungie's 2007 bake did not — the "silver discs" on 070 come out 30–46× too bright. H3LM lights them the 2007 way.

### Light that crosses BSP seams
- **Shadows continue across BSPs.** A cliff in the next BSP casts its shadow into this one, from any distance.
- **Closed doors in seam doorways block light.**
- **Lamps light both sides of a seam.** A light near a BSP border lights the neighbouring BSP too, as `tool.exe` does.
- **Lighting zone sets** are read from the scenario, so BSPs that overlap in space (citadel, 120_halo) do not shadow each other.

### Sharper lightmaps on the same page
- **A new UV packer.** Charts nest in each other's gaps and long charts are cut. Guardian goes from about **70% to over 85%** of the page used; some maps gain far more (construct +39%, a jungle BSP +65% more texels). It follows each material's lightmap resolution.
- **A better DXT5 encoder.** Less compression error, no visible texel grid, about 8× faster.
- **Uncompressed pages** (ARGB8) up to 3072; DXT5 at 4096.

### Runs on more PCs
- **Hardware ray tracing** on NVIDIA RTX, AMD RX 6000+ and Intel Arc.
- **Software ray tracing** on GPUs without it (GTX 900/10/16, AMD RX 400+): the same lightmaps within a fraction of a percent, about 5× slower.
- **Big bakes in parts.** When a page does not fit in memory, it is baked in bands. Construct at 8×8 samples needed over 41 GB before; now it peaks at 10 GB. Same result, byte for byte.

### New options tool does not have
- **Baked ambient occlusion** (`--ao`, Unreal-style).
- **Shadow fill** and **sun softness** controls.
- **Even final-gather rays** (`--even-rays`) for a faster bake.
- **Every improvement can be turned off** for an A/B test against stock.

---

## Quality presets

Low to Super Slow are Bungie's own `tool.exe` settings. Optimized and Ultra are tuned by zoephie: every setting was swept against an everything-at-maximum bake, and **Optimized stops where more stopped showing**.

| Preset | Samples | Photons | Gather rays / threshold | Sky rays | Source |
|---|---|---|---|---|---|
| Low | 1×1 | 2M | 128 / 0.1 | 1× | Bungie |
| Medium | 2×2 | 4M | 256 / 0.05 | 1× | Bungie |
| High | 3×3 | 8M | 512 / 0.01 | 1× | Bungie |
| Super Slow | 4×4 | 9M | 2048 / 0.005 | 1× | Bungie |
| **Optimized** | 4×4 | 9M | 2048 / 0.01 | 1× | zoephie — **recommended** |
| Ultra | 8×8 | 18M | 4096 / 0.0025 | 2× | zoephie — final bakes |
| MAX | 8×8 | 200M | 65536 / 0 | 128× | every field at maximum, untested |

### Rough times — guardian, Optimized, RTX 3060 Ti, whole bake

| Page size | Time |
|---|---|
| 1024 | 5.7 min |
| 3072 | 7.8 min |
| 4096 | 12.7 min |

Big campaign BSPs take longer.

---

## Requirements

- Windows 10 or 11, 64-bit
- A DirectX 12 GPU
  - **Fastest:** ray tracing GPUs (DXR 1.1) — NVIDIA RTX, AMD RX 6000+, Intel Arc
  - **Works, ~5× slower:** GTX 900/10/16, AMD RX 400+
  - The window has a GPU check that tells you which one you have.
- The **Halo 3** and/or **Halo 3: ODST** MCC Editing Kit with your map's tags

`tool.exe` is **not** needed.

---

## Quick start

### The window
1. Download the latest release and unzip it. Keep `h3lm-gui.exe` and `h3lm-lm.exe` in the same folder.
2. Run `h3lm-gui.exe`.
3. **Browse** to your `.scenario`. The window reads the game (Halo 3 or ODST) from the kit.
4. Pick one BSP or all, pick a quality, press **Start**.
5. When it finishes, open the map in Sapien.

Hover any setting for what it does. **Copy log** puts the whole log on the clipboard for a bug report.

### The command line
```
h3lm-lm lightmap --scenario="C:\H3EK\tags\levels\solo\070_waste\070_waste.scenario" --bsp=070_bsp_020 --quality=optimized --ao
```

Useful options:

| Option | What it does |
|---|---|
| `--bsp=a,b` | bake only these BSPs (default: all) |
| `--quality=...` | `low`, `medium`, `high`, `super_slow`, `optimized`, `ultra`, `max` |
| `--size=N` | lightmap page size, up to 4096 |
| `--ao` | baked ambient occlusion |
| `--even-rays` | even final-gather rays, faster |
| `--no-install` | keep the result in the work folder only |
| `--dry-run` | print the plan, run nothing |
| `--list` | list the scenario's BSPs |

---

## Safe by design

- **Your tags are backed up.** Old lightmap files are copied to a backup folder before anything is installed.
- **Cancel any time.** A cancelled bake installs nothing. Tags are installed only after every BSP finishes.
- **Nothing is patched.** No game or kit files are changed. H3LM writes only lightmap tags (and, for a map that was never lightmapped, points the scenario at them, after a backup).

---

## Status and known limits

- **Halo 3:** campaign and multiplayer maps bake, including maps with lighting regions and multi-BSP scenarios.
- **ODST:** maps bake and are lit (tested on campaign, multiplayer and never-lightmapped maps). ODST light levels do not match Bungie's ODST lightmaps exactly yet.
- Windows only.

---

## FAQ

**Will players need anything to see my map?**
No. H3LM writes the same lightmap tags `tool.exe` writes. The game reads them as normal.

**Is the source code available?**
Not for now. This repository holds the releases and the docs.

**Can I compare it with a stock bake?**
Yes. Every change H3LM makes can be turned off, so you can A/B it against stock.

**Something went wrong.**
Open an issue with the log (**Copy log** in the window), the map, the quality and your GPU.

---

## AI use

H3LM was made with the help of AI (Anthropic's Claude), used as a coding assistant. A human reviewed every change, and everything was tested by a human before release.

If you would rather not use a tool made with AI, that is your choice, and it is a fair one.

---

## Support

H3LM is free. If it helps you: [ko-fi.com/zoephloaf](https://ko-fi.com/zoephloaf)

Made by **zoephie**.

*Halo, Halo 3 and Halo 3: ODST are trademarks of Microsoft. H3LM is a fan-made tool and is not made, endorsed or supported by Microsoft, Bungie or 343 Industries / Halo Studios.*
