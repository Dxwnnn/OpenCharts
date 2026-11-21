# OpenCharts

A free chart and monitoring tool for DaVinci Resolve, implemented as a single **ResolveFX DCTL** effect. It provides test charts and output analysis tools that work in a manual color-managed workflow (DaVinci Wide Gamut / DaVinci Intermediate).

> **Status:** v1.0.0 – tested by the author in DaVinci Resolve 20.2.
> **Not affiliated** with Blackmagic Design or any commercial chart product.

<!-- Add screenshots here -->

---

## Features

### Input charts

| Chart | Description |
|---|---|
| Mid Gray | 18 % linear gray, encoded with the selected transfer function; shows the encoded value |
| Gray Ramp | Smooth code-value ramp plus an 11-step wedge (0.0 – 1.0) |
| Exposure Ramp | Stop-based ramp around 18 % gray (2–9 stops per side), stepped and continuous |
| Gradient Chart | Code-value ramps for gray, R, G, B, C, M, Y – reveals banding and clipping |
| Color Chart | 24-patch ColorChecker layout, converted into the selected primaries and transfer function |
| Skin Chart | 8 skin tones (L\* 80 → 24) in three chroma rows (×1.0, ×0.6, ×1.4) |
| LUT Cube | 9×9×9 grid shown as 3×3 slices (one blue level per slice) for testing LUTs and transforms |
| Test Patches | Mid gray, Rec.709 R/G/B patches converted into the selected primaries, and white – for verifying the color math |

### Output monitoring

| Tool | Description |
|---|---|
| False Color | Stop-based colors relative to 18 % gray, with a legend bar |
| Saturation False Color | Colors by CIELAB chroma C\*, with a legend bar |
| Negatives Indicator | Magenta marks pixels that fall outside a chosen target gamut (negative linear component) |
| Highlight Neutrals | Marks near-neutral pixels (CIELAB chroma below a tolerance) |
| Skin Hue Indicator | Marks pixels near a skin-tone hue line and shows the direction of deviation |

### Color management

- **Color Space Primaries:** Rec.709, DaVinci Wide Gamut (default), P3-D65, Rec.2020 (all D65)
- **Transfer Function:** Linear, DaVinci Intermediate (default), Rec.709 (OETF), sRGB

The selected settings are used in every calculation: charts are generated in the chosen space, and all monitoring tools decode the image to linear light and compute luminance and CIELAB from the chosen primaries.

---

## Requirements

- DaVinci Resolve 20.2 (the target version). The ResolveFX DCTL plugin may require Resolve Studio – please state your edition when reporting issues.
- A graphics card and driver supported by Resolve. The effect works per pixel and needs no extra resources.

---

## Installation

1. Copy `OpenCharts.dctl` into Resolve's LUT folder:
   - **Windows:** `C:\ProgramData\Blackmagic Design\DaVinci Resolve\Support\LUT`
   - **macOS:** `/Library/Application Support/Blackmagic Design/DaVinci Resolve/LUT`
   - **Linux:** `/opt/resolve/LUT`
2. Open the Color page.
3. Open **Effects → ResolveFX → DCTL** and apply it to a node.
4. Click **Reload DCTL List** and choose `OpenCharts`.

> **Important:** OpenCharts is a *ResolveFX* DCTL, not a LUT-style DCTL. Loading it through the LUT or CST menus produces errors such as `wrong argument int p_Width in Transform DCTL`.

---

## Quick start

1. Set **Color Space Primaries** and **Transfer Function** to match the signal at this node (for the recommended workflow: DaVinci Wide Gamut and DaVinci Intermediate).
2. Choose an **Input Chart** and a **Layout** (`Side Panel` keeps the image on the left; `Full Frame` shows only the chart).
3. Turn on any monitoring tool you need. Monitoring runs on the image even if the chart is off.
4. Use **Bypass** to disable all processing.

Place the effect at the point of the node tree where you want to measure. Monitoring tools do not modify your grade; they replace the output of the node they are on.

---

## Parameters

| Parameter | Default | Range / options |
|---|---|---|
| Input Chart | Off | Off, Mid Gray, Gray Ramp, Exposure Ramp, Gradient Chart, Color Chart, Skin Chart, LUT Cube, Test Patches |
| Layout | Side Panel | Side Panel, Full Frame |
| Panel Width | 0.38 | 0.15 – 0.9 (Side Panel only) |
| Exposure Stops Each Side | 6 | 2 – 9 (Exposure Ramp) |
| Chart Exposure Stops | 0.0 | −4 – +4 (Color Chart, Skin Chart) |
| Hide Legends | off | Hides all numbers and legend bars |
| Legend Size | 1.0 | 0.5 – 3.0 |
| Flip Vertical | off | Use if charts appear upside down |
| False Color | off | |
| Saturation False Color | off | |
| Negatives Indicator | off | |
| Negatives Target Gamut | Rec.709 | Rec.709, DaVinci Wide Gamut, P3 D65, Rec.2020 |
| Highlight Neutrals | off | |
| Neutral Chroma Tolerance | 4.0 | 1 – 15 (C\*) |
| Skin Hue Indicator | off | |
| Skin Hue Tolerance Degrees | 20 | 5 – 40 |
| Indicator Background Dim | 0.5 | 0 – 0.9 (Negatives, Neutrals, Skin Hue) |
| Color Space Primaries | DaVinci Wide Gamut | Rec.709, DaVinci Wide Gamut, P3 D65, Rec.2020 |
| Transfer Function | DaVinci Intermediate | Linear, DaVinci Intermediate, Rec.709, sRGB |
| Bypass | off | Bypasses everything |

False Color and Saturation False Color replace the image (False Color wins if both are on). Negatives, Highlight Neutrals, and Skin Hue are overlays on top; if they overlap, Negatives wins over Skin Hue, which wins over Neutrals.

---

## Reference

### Exposure Ramp

Each patch is `0.18 · 2^stop` in linear light, encoded with the selected transfer function. With Rec.709 or sRGB the upper stops clip at 1.0 (above about +2.5 stops over mid gray). That is correct for display-referred encodings; DaVinci Intermediate has far more headroom.

### False Color scale

Stops are `log2(Y / 0.18)`, where `Y` is the luminance of the decoded linear image in the selected primaries.

| Stops | Color | DaVinci Intermediate code (approx.) |
|---|---|---|
| < −6 | Violet | < 0.029 |
| −6 … −4 | Blue | 0.029 – 0.093 |
| −4 … −2 | Cyan | 0.093 – 0.201 |
| −2 … −0.5 | Gray (original luma) | 0.201 – 0.301 |
| −0.5 … +0.5 | Green (mid gray) | 0.301 – 0.371 |
| +0.5 … +1.5 | Pink | 0.371 – 0.443 |
| +1.5 … +3 | Yellow | 0.443 – 0.552 |
| +3 … +4 | Orange | 0.552 – 0.625 |
| > +4 | Red | > 0.625 |

This scale is a design choice of this project and is **not** identical to any camera vendor's or other tool's false color.

### Saturation False Color scale (CIELAB C\*)

| C\* | Color |
|---|---|
| < 3 | Gray |
| 3 – 10 | Blue |
| 10 – 25 | Green |
| 25 – 45 | Yellow |
| 45 – 70 | Orange |
| 70 – 100 | Red |
| > 100 | Magenta |

Chroma is absolute, so it depends on exposure: the same color looks less saturated when it is darker.

### Negatives Indicator

The linear image is converted to XYZ and then into the **target gamut**. A pixel is flagged when any component is below −0.0005. Use it to see what falls outside Rec.709 while grading in a wider space.

### Highlight Neutrals

A pixel is highlighted if `Y > 0.005` (about −5 stops, to ignore noisy shadows) and its CIELAB chroma is below the tolerance.

### Skin Hue Indicator

The pixel's CIELAB `(a*, b*)` is projected onto a skin line at a hue angle of **55°**. A pixel counts as skin-like if `L* > 20`, `6 ≤ C* ≤ 70`, and its angular deviation from the line is within the tolerance.

- Green: close to the line (within 35 % of the tolerance)
- Yellow: deviates toward yellow/green
- Pink: deviates toward red/magenta

### Color Chart and Skin Chart references

- **Color Chart:** the widely published sRGB renditions of the 24 ColorChecker patches (D65), decoded to linear Rec.709, converted to XYZ and then into the selected primaries. The neutral patches come out at reflectances of roughly 0.90, 0.59, 0.36, 0.19, 0.09, 0.03. These are approximations, not spectrally calibrated measurements.
- **Skin Chart:** constructed in CIELAB (D65) along a 55° hue line, with L\* from 80 to 24 and chroma from 20 to 28. It is a practical visual aid, not a measured skin-tone reference.

### LUT Cube

A 9×9×9 grid of code values in the working space. Slices are blue levels 0 to 8 (`B = level / 8`), laid out 3×3, left to right and top to bottom. Within a slice, red increases to the right and green increases upward. Place the chart before a LUT or transform to see how it deforms the grid.

### Legends

Legends are numbers only (a built-in 3×5 pixel font; there are no letters):

- Mid Gray: encoded value
- Gray Ramp: code value per step
- Exposure Ramp: stops relative to 18 % gray
- Color Chart: patch number 1–24
- Skin Chart: L\* per column and chroma factor per row
- LUT Cube: blue level 0–8
- False Color and Saturation False Color: legend bar at the bottom of the image, labeled with each band's lower limit

---

## Color science details

### Primaries (CIE xy, white point D65 = 0.3127, 0.3290)

| Space | R | G | B |
|---|---|---|---|
| Rec.709 | 0.640, 0.330 | 0.300, 0.600 | 0.150, 0.060 |
| DaVinci Wide Gamut | 0.8000, 0.3130 | 0.1682, 0.9877 | 0.0790, −0.1155 |
| P3-D65 | 0.680, 0.320 | 0.265, 0.690 | 0.150, 0.060 |
| Rec.2020 | 0.708, 0.292 | 0.170, 0.797 | 0.131, 0.046 |

The RGB→XYZ matrices were computed from these primaries and the D65 white point, and the XYZ→RGB matrices are their numerical inverses. All spaces share the D65 white point, so no chromatic adaptation is needed.

### Transfer functions

**DaVinci Intermediate** (`x` linear, `y` encoded)

```
A = 0.0075      B = 7.0          C = 0.07329248
M = 10.44426855
LIN_CUT = 0.00262409    LOG_CUT = 0.02740668

encode:  y = (log2(x + A) + B) * C     for x > LIN_CUT
         y = x * M                      otherwise
decode:  x = 2^(y / C - B) - A         for y > LOG_CUT
         x = y / M                      otherwise
```

**Rec.709 OETF** (scene-referred, not Gamma 2.4)

```
encode:  y = 4.5 x                                  for x < 0.018053968510807
         y = 1.09929682680944 x^0.45 - 0.09929682680944   otherwise
```

**sRGB**

```
encode:  y = 12.92 x                      for x <= 0.0031308
         y = 1.055 x^(1/2.4) - 0.055      otherwise
```

The linear segments are also applied to negative inputs (no clamping), so encode and decode stay invertible and out-of-gamut values survive.

### Verification values (DaVinci Wide Gamut / DaVinci Intermediate)

Use the **Test Patches** chart. Values are computed independently in Python.

| Patch | R | G | B |
|---|---|---|---|
| Mid gray (linear 0.18) | 0.3360 | 0.3360 | 0.3360 |
| Rec.709 red × 0.18 | 0.2785 | 0.1070 | 0.0948 |
| Rec.709 green × 0.18 | 0.2252 | 0.3070 | 0.1780 |
| Rec.709 blue × 0.18 | 0.1349 | 0.1689 | 0.3061 |
| White (linear 1.0) | 0.5138 | 0.5138 | 0.5138 |

For other settings the mid gray patch should read 0.1800 (Linear), 0.4088 (Rec.709), and 0.4614 (sRGB).

---

## Known limitations and assumptions

- A DCTL processes **one pixel at a time**, so histograms, waveforms, and vectorscopes are not possible.
- The Color Chart and Skin Chart references are approximations (see above).
- The False Color scale, chroma bands, and the 55° skin line are design choices of this project.
- "Rec.709" is the scene-referred OETF; Gamma 2.4 (BT.1886) is not implemented.
- CIELAB chroma is absolute, so Saturation False Color, Highlight Neutrals, and the Skin Hue Indicator depend on exposure.
- The DaVinci Intermediate constants and DaVinci Wide Gamut primaries come from Blackmagic's published values. They are self-consistent, but it is worth cross-checking them against a Color Space Transform node in your own Resolve version.
- Legends contain numbers only.
- Y-axis orientation can differ between setups; use **Flip Vertical** if charts appear upside down.

---

## Troubleshooting

| Problem | Likely cause / fix |
|---|---|
| `wrong argument int p_Width in Transform DCTL` | The file was loaded as a LUT/CST DCTL. Load it through **ResolveFX → DCTL**. |
| The effect does not appear in the list | Wrong folder, or the list is stale. Check the LUT folder and press **Reload DCTL List**. |
| Compiler errors in a modified file | See development notes below. |
| Charts are upside down | Enable **Flip Vertical**. |
| Top stops of the Exposure Ramp are flat | Expected with Rec.709 or sRGB (clipping at 1.0). Use DaVinci Intermediate or Linear. |

---

## Development notes (DCTL pitfalls found while building this)

- The ResolveFX transform signature is
  `transform(int p_Width, int p_Height, int p_X, int p_Y, float p_R, float p_G, float p_B)`.
- UI parameters (`DEFINE_UI_PARAMS`) are only visible inside `transform()`. Pass them as arguments to helper functions. Combo box enumerator names are defined by the macro and can be used globally.
- Keep UI labels free of special characters (parentheses broke nothing in testing but were avoided on purpose; `P3 D65` is used instead of `P3-D65`).
- Do not rely on `float3` operators. Use `make_float3` and component-wise math.
- No global arrays; lookup tables are written as `if` chains.
- Use only the documented math functions (`_log2f`, `_exp2f`, `_powf`, `_fminf`, `_fmaxf`); trig is avoided via constants and a series approximation.

### Source layout

1. UI parameters
2. Transfer functions
3. Primaries and RGB↔XYZ
4. Chart generators, monitoring, legends
5. Chart selection and compositing
6. Main transform

---

## Roadmap ideas

- Optional Gamma 2.4 (BT.1886) transfer function
- Additional transfer functions for camera log curves
- Alternative false color scales
- Alternative skin-tone reference sets

---

## Changelog

**1.0.0**
- All listed input charts and output monitoring tools
- Primaries and transfer function used in every calculation
- Stop-based Exposure Ramp, ColorChecker-based Color Chart, LUT Cube test pattern
- Number legends with Hide Legends, Legend Size, Flip Vertical, Bypass

---

## License

MIT

## Credits

- Author: _your name_
- Built for DaVinci Resolve (Blackmagic Design). Color science references: Blackmagic Design's DaVinci Wide Gamut / Intermediate documentation, ITU-R BT.709 / BT.2020, IEC 61966-2-1 (sRGB), CIE 1931 / CIELAB.
