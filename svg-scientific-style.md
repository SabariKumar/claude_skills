---
name: svg-scientific-style
description: >
  Style reference for generating SVG scientific diagrams that match Sabari's
  established figure aesthetic. Use this skill whenever creating, editing, or
  extending any SVG figure for grant proposals, papers, or presentations —
  including flowcharts, molecular graph diagrams, pipeline schematics,
  message-passing illustrations, or computational scheme figures. Trigger on
  requests like "make an SVG figure", "draw a flowchart", "make a diagram like
  the others", "add a new figure", or any figure generation task involving
  scientific content. Supports two detail levels: "overview" (high-level data
  flow for a computational chemistry audience, no hyperparameters) and
  "detailed" (architecture reference with layer counts, dimensions, and training
  config, for readers working on the code). Always read this skill before
  writing any SVG code, and determine the detail level and orientation
  before drawing. Default orientation is horizontal landscape.
---

# SVG Scientific Diagram Style Guide

All figures in this style were created in **Inkscape 1.4**, exported at
**600 DPI** for LaTeX inclusion (`inkscape:export-xdpi="600"`). Figures live
on A4 portrait canvases (210×297 mm, `viewBox="0 0 210 297"`), often as
multi-page Inkscape documents (multiple `<inkscape:page>` elements). The
coordinate system is millimeters throughout.

---

## Global Defaults

```svg
<!-- Set on the root <svg> or via repeated <style> blocks -->
* { stroke-linejoin: round; stroke-linecap: butt; }
```

All figures share this global join/cap default. Individual elements may
override `stroke-linecap` to `round` for arrows and soft paths.

---

## Detail Levels

Every figure request must be classified as one of two detail levels before
drawing begins. Ask the user if not specified.

---

### `overview` — pipeline data flow for a scientific audience

**Target reader**: computational chemist with some ML background. Assumes
familiarity with concepts like graph neural networks and transfer learning, but
not with specific implementation choices.

**Canvas**: A4 landscape preferred (297×210 mm, `viewBox="0 0 297 210"`), or
wider if the pipeline is long. Two-lane horizontal layout (encoding top,
decoding/finetuning bottom) is the standard pattern.

**What to include**:
- Named pipeline stages as labelled boxes or dataset icons
- Data types flowing between stages (e.g. "SMILES CSV", "mol. graph", "latent z")
- Conceptual operation labels inside boxes (e.g. "Graph Transformer", "Task Head")
- Loss equations in italic below decoder boxes (e.g. `ℒ_geo = MSE(dist_pred, dist_true)`) — full form is fine
- Arrows indicating data flow direction; dashed arrows for optional/cached flows
- Section swim-lane labels, but **only for lanes that benefit from a bounding box** (see below)
- Small source/script annotations in `Courier New` above dataset icons where useful (e.g. `confsweeper`)

**What to omit**:
- Layer counts, hidden dimensions, head counts
- Activation functions, normalization types
- Dropout rates, learning rates, batch sizes
- Specific aggregator lists (e.g. "sum · mean · max · std")
- Optimizer settings, warmup schedules
- Internal sub-block structure within a model stage

**Swim-lane bounding boxes**: use sparingly. The encoding pipeline at the top
often looks cleaner without a background rect — let the boxes and arrows speak
for themselves. Reserve the `fill="#f2f2f2"` panel for lanes that group
multiple parallel sub-heads (e.g. a curriculum decoder section with three
parallel branches). When a lane panel is used, keep `fill-opacity: 0.47`,
`stroke="#b7b7b7"`, `stroke-width: 0.2`, `stroke-opacity: 0.73`.

**Phase sub-panels**: when a decoder lane has discrete curriculum phases, shade
each phase column with a distinct very-light pastel background rect
(`fill-opacity: 0.55`, `stroke-width: 0.13`). Phase label sits at the left
edge of each column (`text-anchor="middle"` over the column centre), bold,
coloured to match its phase tint. Phase colours:
- Phase 1: fill `#e8f5e9`, stroke `#b2dfb2`, label `#2d7a4f`
- Phase 2: fill `#e3eef9`, stroke `#b2c8e8`, label `#3d6fa6`
- Phase 3: fill `#f3e8f7`, stroke `#d4b8e4`, label `#7a4a8c`

**Dataset icons**: always use the canonical stacked-disc icon from `Database.svg`
(see Dataset / CSV icons section). Never use a plain rect or a hand-drawn
ellipse/rect cylinder. Scale: `~0.475` for primary input datasets (~13 mm tall),
`~0.34` for secondary cache icons (~9 mm tall). Fill colour encodes data role —
extend the table in the Dataset section as needed; two confirmed additions:
- Raw/unlabelled input dataset: `#ffaaaa` (pink-red, full opacity)
- Precomputed geometric cache: `#f6ffd5` / `#ddffee` (yellow-green to mint)

**Arrow edge labels**: keep to a single short noun phrase on one line (e.g.
"ensemble", "node emb."). Two stacked lines read as clutter at this scale;
prefer dropping the second line over using two.

**Loss footer**: place a combined loss equation as a single centred line at the
bottom of the figure (no horizontal rule separator above it). Use an inline
`Courier New` `<tspan>` for the mathematical expression itself, leaving the
surrounding descriptor text in Arial. Example:
```svg
<text font-family="Arial" font-size="2.8" fill="#555555" text-anchor="middle">
  <tspan font-family="Courier New">train/loss = Σᵢ λᵢ · ℒᵢ / EMAᵢ</tspan>
  (per-head EMA normalisation keeps contributions comparable)
</text>
```

**Font sizes**: primary box labels `3.8px` bold; sub-labels `2.8px`; tertiary
detail `2.5px` in `#666666`; annotations and cache labels `2.2–2.5px`;
lane headings `3.8px` bold in `#444444`; phase labels `3.5px` bold.

**Compactness**: figures should be tightly laid out with minimal whitespace.
Concretely:
- Fit content to the canvas — do not pad the viewBox to a round number if
  the content only fills half of it. Crop the viewBox height to the actual
  content extent plus ~5 mm margin.
- Inter-box gaps: 7–10 mm between horizontally adjacent boxes; 6–8 mm
  between a box and its downstream arrow target.
- Vertical lane padding: 3–4 mm between the lane border and the nearest box.
- Avoid blank rows or columns. If a lane is mostly empty, shrink it or remove it.
- Do not add a legend unless the figure actually uses more than two distinct
  arrow or fill conventions that would otherwise be ambiguous.

---

### `detailed` — architecture reference for code-level understanding

**Target reader**: someone working on or reading the code who needs to map
config parameters to model components.

**Canvas**: A4 portrait (210×297 mm, `viewBox="0 0 210 297"`) or wider as
needed. Vertical flow (top-to-bottom within a model) is the standard pattern,
with separate side panels for training config and output heads.

**What to include** (in addition to everything in `overview`):
- Layer / block counts (e.g. "×3 layers")
- Hidden dimensions at each stage (e.g. "d=256")
- Number of attention heads and per-head dimension
- Normalization type (BN / LN) and activation function (GELU / ReLU)
- Aggregator lists (e.g. "sum · mean · max · std")
- Dropout rates on individual components
- Residual skip connections shown as dashed bypass arrows
- Training hyperparameters (lr, batch size, warmup, weight decay, epochs,
  seed) in a dedicated annotation panel — use a light gray bordered rect
- Loss weights and temperature parameters for alignment objectives
- Input/output dimensionality on arrows where it aids comprehension

**Box labels**: same noun-phrase style, but sub-labels may be 2–3 lines to
carry parameter details. E.g.:
```
Multi-Head Self-Attention
16 heads · dim=16/head · no QKV bias
```

**Font sizes**: primary box labels `3.2–3.8px`; parameter sub-labels `2.5–2.8px`;
training config annotation text `2.5px`; section labels `3.2px` bold.

**Training config panel recipe**:
```svg
<rect fill="none" stroke="#cccccc" stroke-width="0.25" rx="2"/>
<text font-family="Arial" font-size="2.8" font-weight="bold"
      fill="#555555" text-anchor="middle">Training</text>
<!-- one line per key hyperparameter, font-size 2.5, fill #888888 -->
```

---

## Canonical Marker Definitions

Paste this `<defs>` block verbatim into every new figure. It defines all four
arrow/marker types used across the figure set. Reference them by ID in path
`marker-start` / `marker-end` attributes.

```svg
<defs>
  <!-- Triangle: solid filled triangle. Use for standard flowchart arrows
       and dashed data-flow connectors.
       Reference: marker-end="url(#Triangle)" -->
  <marker
     id="Triangle"
     refX="0" refY="0"
     orient="auto-start-reverse"
     markerWidth="1" markerHeight="1"
     viewBox="0 0 1 1"
     preserveAspectRatio="xMidYMid">
    <path
       transform="scale(0.5)"
       style="fill:context-stroke;fill-rule:evenodd;
              stroke:context-stroke;stroke-width:1pt"
       d="M 5.77,0 -2.88,5 V -5 Z" />
  </marker>

  <!-- Triangle-sm: smaller triangle for tight-space arrows (markerWidth 0.3).
       Use when marker-end="url(#Triangle)" feels too large.
       Reference: marker-end="url(#Triangle-sm)" -->
  <marker
     id="Triangle-sm"
     refX="0" refY="0"
     orient="auto-start-reverse"
     markerWidth="0.3" markerHeight="0.3"
     viewBox="0 0 1 1"
     markerUnits="userSpaceOnUse"
     preserveAspectRatio="xMidYMid">
    <path
       transform="scale(0.5)"
       style="fill:context-stroke;fill-rule:evenodd;
              stroke:context-stroke;stroke-width:1pt"
       d="M 5.77,0 -2.88,5 V -5 Z" />
  </marker>

  <!-- ArrowTriangleStylized: curved/swept triangle for message-passing arrows.
       Always used bidirectionally (marker-start + marker-end).
       Reference: marker-start="url(#ArrowTriangleStylized)"
                  marker-end="url(#ArrowTriangleStylized)" -->
  <marker
     id="ArrowTriangleStylized"
     refX="0" refY="0"
     orient="auto-start-reverse"
     markerWidth="1" markerHeight="1"
     viewBox="0 0 1 1"
     preserveAspectRatio="xMidYMid">
    <path
       transform="scale(0.5)"
       style="fill:context-stroke;fill-rule:evenodd;
              stroke:context-stroke;stroke-width:1pt"
       d="m 6,0 c -3,1 -7,3 -9,5 0,0 0,-4 2,-5 -2,-1 -2,-5 -2,-5 2,2 6,4 9,5 z" />
  </marker>

  <!-- ColoredDot: filled circle dot marker, inherits fill from the path.
       Use for marking waypoints on molecule-structure paths.
       Reference: marker-start="url(#ColoredDot)"
                  marker-mid="url(#ColoredDot)"
                  marker-end="url(#ColoredDot)" -->
  <marker
     id="ColoredDot"
     refX="0" refY="0"
     orient="auto"
     markerWidth="1" markerHeight="1"
     viewBox="0 0 1 1"
     preserveAspectRatio="xMidYMid">
    <path
       transform="scale(0.45)"
       style="fill:context-fill;fill-rule:evenodd;
              stroke:context-stroke;stroke-width:2"
       d="M 5,0 C 5,2.76 2.76,5 0,5 -2.76,5 -5,2.76 -5,0
          c 0,-2.76 2.3,-5 5,-5 2.76,0 5,2.24 5,5 z" />
  </marker>
</defs>
```

### Which marker to use

| Situation | Marker IDs |
|---|---|
| Flowchart step → step | `marker-end="url(#Triangle)"` |
| Dashed data-flow / input connector | `marker-end="url(#Triangle)"` |
| Tight-space or small-scale arrow | `marker-end="url(#Triangle-sm)"` |
| Message-passing (bidirectional, curved) | `marker-start="url(#ArrowTriangleStylized)" marker-end="url(#ArrowTriangleStylized)"` |
| Molecule bond path with atom dots | `marker-start="url(#ColoredDot)" marker-mid="url(#ColoredDot)" marker-end="url(#ColoredDot)"` |

---

## Typography

| Context | Font | Size (mm units) | Weight | Color |
|---|---|---|---|---|
| Primary labels (flowchart boxes) | `Arial` | `10px` / `4.05022px` | bold `700` | `#333333` or `#000000` |
| Secondary/sublabels | `Arial` | `9px` / `2.82222px` | normal | `#4a4820`, `#555555` |
| Section/aim labels | `Arial` | `7.76px` | normal | `#000000` |
| Legend / annotation | `Arial` | `8.5px` | normal | `#888888` |
| Database cylinder labels | `Arial` | `4.69–5.74px` | normal | `#000000` |
| Scatter plot / embedding labels | `Arial` | `11.4809px` | normal | `#000000` |
| Caption text (inline figures) | `Arial` | `2.82222px` / `sans-serif` fallback | normal | `#000000` |

Text is always center-anchored (`text-anchor: middle`) unless it is an edge
annotation, in which case `text-anchor: start`. `line-height: 0.9` is used
for multi-line tspans. Multi-line labels use `<tspan sodipodi:role="line">`
children, spaced at ~3.5px increments.

---

## Color Palette

### Node / atom fills (molecular graph diagrams)

These are the primary pastel node colors. Use them for graph nodes, atom
circles, and categorical highlights.

| Name | Hex | Typical use |
|---|---|---|
| Teal | `#b2e6dc` | Default atom / primary node |
| Teal bright | `#48dbd6`, `#9cebe9`, `#6acacb` | Active / highlighted atom |
| Rose | `#e3c9c4` | Secondary node type |
| Sage | `#cdcca7` | Tertiary / experimental data box |
| Blue-lavender | `#adc6f1`, `#cfdbed`, `#a9c0de` | Edge-type or bond-order node |
| Lavender | `#f4d7ee`, `#ebb8e1` | Hyperedge or fragment node |
| Pink/salmon | `#ffd5d5`, `#ffb3ad`, `#e9afaf` | Highlighted/target node |
| Coral/red | `#ff675a`, `#cf6e61` | High-flux / error node |
| Steel blue | `#5e81ac` | Embedding space scatter (opacity 0.5–0.7) |
| Cyan | `#88c0d0` (opacity 0.7), `#8fbcbb` | Embedding anchor point |
| Brick rose | `#bf616a` | LLM embedding scatter (opacity 0.5) |

### Structural / background fills

| Element | Fill | Stroke |
|---|---|---|
| Section panel (light) | `#f2f2f2`, `opacity: 0.47` | `#999999`, `stroke-width: 0.25` |
| Section panel (medium) | `#e6e6e6` | `#999999`, `stroke-width: 0.5` |
| Flowchart box (neutral) | `#e6e6e6` | `#999999`, `stroke-width: 0.5` |
| Flowchart box (sage) | `#e8e6c8` (pastel sage) | `#aaa880`, `stroke-width: 0.5` |
| Flowchart box (output) | `#e6e6e6` | `#999`, `stroke-width: 0.5` |
| Database cylinder (blue) | `#d5e5ff` | `#000000`, `stroke-width: 0.595` |
| Database cylinder (mauve) | `#efe3e8` | `#000000`, `stroke-width: 0.595` |
| NN node (pink) | `#e9afaf` | none |
| Graph background panel | `#cccccc` | `#b3b3b3`, `stroke-width: 0.63` |
| Chat bubble / annotation | `#ffffff` | `#666666`, `stroke-width: 0.13`, `rx` rounded |
| Hyperedge blob | `#62aa2b` / `#19a5b8` | same color, `opacity: 0.22–0.36`, dashed |

### Molecular graph atom nodes (small circles, r ≈ 1.8mm)

Small atom nodes use **self-stroke** (fill = stroke color), `stroke-width:
0.800804`, `stroke-linecap: round`. They carry no separate outline.

---

## Shape Specifications

### Flowchart boxes

```svg
<rect rx="4" fill="#e6e6e6" stroke="#999999" stroke-width="0.5"
      width="118" height="48" />
```

- Corner radius: `rx="4"` (standard), `rx="1.35–3.72"` (panel borders)
- No drop shadow, no gradient
- Text vertically centered inside box

### Section panels (Aim groupings)

```svg
<rect fill="none" stroke="#b7b7b7" stroke-width="0.17–0.29"
      stroke-opacity="0.72549" rx="1.35–3.72" />
```

Light gray outline only, no fill (or very faint `#f2f2f2` with low opacity).

### Molecular graph panels

```svg
<rect fill="#cccccc" stroke="#b3b3b3" stroke-width="0.63"
      stroke-linecap="butt" stroke-linejoin="round" />
```

No border radius. Used as backdrop for embedded graph visualizations.

### Dataset / CSV icons

All dataset inputs (SMILES CSVs, labelled property CSVs, cached descriptor
files) use a **stacked-disc cylinder** icon derived from the canonical
`Database.svg` asset. The geometry is fixed; only the fill color changes to
reflect the dataset's role in the pipeline.

Use pastel fills from the main palette — never the dark sage `#cdcca7` /
`#888060` pair for dataset icons (too dark; reserved for annotation boxes).
Preferred fills for datasets:

| Dataset type | Fill | Stroke |
|---|---|---|
| Unlabelled SMILES (pretrain input) | `#d5e5ff` (light blue) | `#000000` |
| Labelled CSV (finetune input) | `#cfdbed` (blue-lavender) | `#000000` |
| Cached descriptors | `#efe3e8` (mauve) | `#000000` |
| Generic / neutral dataset | `#e6e6e6` (light gray) | `#000000` |

**Canonical dataset icon snippet** — paste and adjust `transform` to position.
The icon's native size is ~28×31 mm; scale down with a `transform="matrix(...)"` 
wrapper. At typical figure scale (≈7–10 mm tall) use `matrix(0.25,0,0,0.25,x,y)`.

The fill color is set on all four `<path>` elements that form the cylinder body
(the paths with `id` containing `path286xx`). The wireframe outline path
(`id="path27044"`) always has `fill="none" stroke="#000000"`.

```svg
<!-- Dataset icon — wrap in <g transform="matrix(0.25,0,0,0.25,X,Y)"> to position -->
<!-- Replace FILL_COLOR with a pastel hex from the table above -->
<g>
  <!-- Wireframe outline (always black, no fill) -->
  <path
     style="fill:none;stroke:#000000;stroke-width:0.577;stroke-linecap:square;
            stroke-linejoin:round;stroke-miterlimit:10"
     d="m 320.36736,53.821484 0.0273,14.753317
        m -29.04333,-14.753317 0.0275,14.753317 v 0
        m 29.01583,0 c 0.0969,0 -0.003,-0.09619 -3e-5,-10e-7
          4e-5,3.51864 -6.4952,6.371097 -14.50764,6.371187 h -2e-5
          c -8.01264,3.6e-5 -14.5082,-2.852458 -14.50816,-6.371187
          0.003,-0.09619 -0.0969,10e-7 2e-5,10e-7
        m 29.0158,-4.917781 c 0.0969,0 -0.003,-0.09619 0,0
          4e-5,3.51864 -6.4952,6.371097 -14.50764,6.371187 h -2e-5
          c -8.01264,3.6e-5 -14.5082,-2.852458 -14.50816,-6.371187
          0.003,-0.09619 -0.0969,0 0,0
        m 29.01582,-4.91779 c 0.0969,0 -0.003,-0.09619 0,0
          4e-5,3.51864 -6.4952,6.371097 -14.50764,6.371187 h -2e-5
          c -8.01264,3.6e-5 -14.5082,-2.852458 -14.50816,-6.371187
          0.003,-0.09619 -0.0969,0 0,0
        m 28.98852,-4.917747 c 0,3.51869 -6.49546,6.371151 -14.50801,6.371151
          -8.01256,2e-6 -14.50802,-2.85246 -14.50802,-6.371151
          0,-3.518691 6.49546,-6.371153 14.50802,-6.371151
          8.01255,0 14.50801,2.852461 14.50801,6.371151 z"
     transform="matrix(0.26458333,0,0,0.26458333,247.27,42.42)" />

  <!-- Disc 1 (bottom) -->
  <path style="fill:FILL_COLOR;fill-opacity:0.65;stroke:#000000;stroke-width:0.595;
               stroke-linecap:square;stroke-linejoin:round;stroke-miterlimit:10"
     d="m -71.910511,-177.01445 c -24.454467,-1.72927 -43.119429,-10.24225
        -45.572069,-20.78515 -1.33634,-5.74442 3.98141,-12.43372 13.4812,-16.95825
        10.631431,-5.06351 24.136723,-7.65986 39.8439,-7.65986
        12.391865,0 22.952708,1.51871 32.500481,4.67375
        10.308225,3.40634 17.575922,8.40836 20.172486,13.8838
        0.746421,1.57399 0.841211,2.0513 0.841211,4.23577
        0,2.22178 -0.08541,2.63433 -0.875739,4.23003
        -4.358453,8.79987 -19.128821,15.51024 -39.377773,17.88984
        -3.521835,0.41387 -17.485877,0.73954 -21.013697,0.49007 z"
     transform="matrix(0.26458333,0,0,0.26458333,323.53853,95.240205)" />

  <!-- Disc 2 -->
  <path style="fill:FILL_COLOR;fill-opacity:0.65;stroke:#000000;stroke-width:0.595;
               stroke-linecap:square;stroke-linejoin:round;stroke-miterlimit:10"
     d="m -72.046964,-158.4486 c -0.600391,-0.0582 -2.503905,-0.23891 -4.23003,-0.40147
        -4.612984,-0.43442 -10.480039,-1.47741 -15.13629,-2.69079
        -10.038016,-2.61583 -17.406896,-6.27952 -21.984166,-10.93017
        -3.74772,-3.80781 -4.3382,-5.87528 -4.35374,-15.244 l -0.007,-4.45922
        1.04446,1.32134 c 6.17506,7.81204 20.678774,13.63246 39.351416,15.79196
        5.247328,0.60685 15.821762,0.81662 21.417145,0.42486
        19.282248,-1.35004 35.190051,-6.70425 42.773788,-14.39671
        1.010907,-1.0254 2.023912,-2.10998 2.251119,-2.41017
        0.37186,-0.49132 0.406325,0.13601 0.345227,6.28381 l -0.06788,6.82963
        -0.929719,1.88727 c -0.640517,1.30021 -1.574311,2.53668 -3.001957,3.975
        -6.664707,6.71456 -19.27798,11.4772 -35.776025,13.50866
        -3.482401,0.42879 -18.81742,0.78927 -21.695961,0.51 z"
     transform="matrix(0.26458333,0,0,0.26458333,323.53853,95.240205)" />

  <!-- Disc 3 -->
  <path style="fill:FILL_COLOR;fill-opacity:0.65;stroke:#000000;stroke-width:0.595;
               stroke-linecap:square;stroke-linejoin:round;stroke-miterlimit:10"
     d="m -74.093753,-140.00674 c -17.869033,-1.6346 -32.165467,-6.69096
        -39.331447,-13.91073 -3.72102,-3.74895 -4.31996,-5.86925 -4.31734,-15.28373
        l 0.001,-4.50294 0.58043,0.81872 c 1.11195,1.56842 4.27679,4.44307
        6.53939,5.93979 7.96961,5.2719 18.97474,8.74498 33.116403,10.45111
        4.8901,0.58997 16.683689,0.81191 22.105318,0.41599
        19.400976,-1.41677 34.875351,-6.74205 42.65514,-14.67915
        l 2.265176,-2.31098 -0.08193,6.64838 -0.08193,6.64837
        -0.874235,1.77453 c -4.128689,8.38045 -18.789834,15.05481
        -38.697014,17.61646 -3.840698,0.49422 -19.828326,0.74474
        -23.879202,0.37418 z"
     transform="matrix(0.26458333,0,0,0.26458333,323.53853,95.240205)" />

  <!-- Disc 4 (top) -->
  <path style="fill:FILL_COLOR;fill-opacity:0.65;stroke:#000000;stroke-width:0.595;
               stroke-linecap:square;stroke-linejoin:round;stroke-miterlimit:10"
     d="m -74.503111,-121.47298 c -14.165434,-1.32255 -26.427729,-4.92557
        -34.386049,-10.10364 -3.09119,-2.01127 -6.35958,-5.37397 -7.52748,-7.74468
        l -0.93258,-1.89303 -0.086,-6.6443 -0.086,-6.64429 2.2692,2.28384
        c 7.16173,7.20794 20.152522,12.17293 37.337515,14.27014
        4.950917,0.60419 17.482542,0.83912 22.924034,0.42976
        19.411583,-1.46035 35.434362,-7.1485 42.810003,-15.19771
        l 1.614414,-1.76185 0.105191,5.26063
        c 0.127065,6.35447 -0.02634,7.6839 -1.149945,9.96628
        -4.291101,8.71647 -20.55894,15.70433 -41.152724,17.67719
        -4.596687,0.44036 -17.467353,0.50055 -21.739659,0.10166 z"
     transform="matrix(0.26458333,0,0,0.26458333,323.53853,95.240205)" />
</g>
```

**Positioning note**: The four disc `<path>` elements all use
`transform="matrix(0.26458333,0,0,0.26458333,323.53853,95.240205)"` in their
native coordinate space, which places the icon at approximately (8.5, 3.5) mm
in the SVG canvas. To reposition the icon, wrap the entire `<g>` in an
outer `<g transform="translate(dx,dy)">` — do not modify the inner transform
values on each path.

**Sizing note**: To resize, wrap in `<g transform="matrix(s,0,0,s,x,y)">` where
`s` is your scale factor relative to the native ~28×31 mm. For a ~7 mm tall
icon use `s ≈ 0.23`.

### Hyperedge blobs (heterograph figures)

```svg
<circle fill="#62aa2b" fill-opacity="0.220472"
        stroke="#69ad91" stroke-width="0.192023"
        stroke-dasharray="0.576068 0.576068" />
```

Dashed outline, very low fill opacity, softly colored.

### Hexagonal fragment outlines

```svg
<path stroke="#000000" stroke-width="0.483203"
      stroke-linecap="round" fill="none" />
```

Used in hypergraph message-passing figures to delineate fragments.

---

## Arrows and Connectors

**Always use `<path>` for arrows** — never `<line>`. Path objects have
editable start and end nodes in Inkscape and carry `sodipodi:nodetypes`
metadata, which makes post-generation editing much easier. Even straight
arrows must be written as paths:

```svg
<!-- Straight arrow — use path, not line -->
<path d="M X1,Y1 L X2,Y2"
      stroke="#999999" stroke-width="0.5" stroke-linecap="round"
      fill="none" marker-end="url(#Triangle)"/>

<!-- Curved arrow — cubic bezier, nodetypes csc -->
<path d="M X1,Y1 C CX1,CY1 CX2,CY2 X2,Y2"
      sodipodi:nodetypes="csc"
      stroke="#999999" stroke-width="0.5" stroke-linecap="round"
      fill="none" marker-end="url(#Triangle)"/>
```

### Standard flowchart arrows

```svg
<path d="M X1,Y1 L X2,Y2"
      stroke="#999999" stroke-width="0.5" stroke-linecap="round"
      fill="none" marker-end="url(#Triangle)"/>
```

Stroke width `0.5` for main flow arrows; `0.37–0.43` for drop arrows
inside branching trees (scale with visual weight of the branch).

### Dashed / data-flow arrows

```svg
<path d="M X1,Y1 L X2,Y2"
      stroke="#888060" stroke-width="0.5"
      stroke-dasharray="2 1.5" stroke-linecap="round"
      fill="none" marker-end="url(#Triangle)"/>
```

Sage-colored (`#888060`) for cached / precomputed data dependencies;
gray (`#999999`) for standard flow. `stroke-dasharray="2 1.5"` is the
preferred dash rhythm (matches hand-edited figures).

### Message-passing arrows (bidirectional, curved)

```svg
<path stroke="#000000" stroke-width="0.264583px"
      stroke-linecap="butt"
      marker-start="url(#ArrowTriangleStylized)"
      marker-end="url(#ArrowTriangleStylized)"
      fill="none" />
```

Marker: stylized curved triangle (`m 6,0 c -3,1 -7,3 -9,5 …`), bidirectional.
Paths are gentle cubic curves (`sodipodi:nodetypes="csc"`).

### Molecular bond lines

```svg
<path stroke="#000000" stroke-width="0.528575"
      stroke-linecap="butt" fill="none" />
<!-- dashed bonds -->
<path stroke="#cccccc" stroke-width="0.255"
      stroke-dasharray="0.51 1.53" fill="none" />
```

---

## Molecular Graph Nodes

### Large nodes (graph overview, r ≈ 2.59mm)

```svg
<circle fill="#6acacb" stroke="#b3b3b3" stroke-width="0.352"
        stroke-linecap="round" r="2.592284" />
```

These appear in pipeline/overview figures. Border is always `#b3b3b3` at
`stroke-width: 0.352`, regardless of fill color.

### Small atom nodes (r ≈ 1.8mm, self-stroked)

```svg
<circle fill="#48dbd6" stroke="#48dbd6"
        stroke-width="0.800804" stroke-linecap="round" r="1.8022451" />
```

### Ghost / context nodes (faded)

Apply `opacity: 0.355241` to the enclosing group to show a faded prior-state
graph.

---

## Embedded Plot Elements (Computational Scheme figures)

When a figure contains embedded scatter plots or potential energy surfaces:

- **Axes**: `stroke="#000000"`, `stroke-width="0.8"`, `stroke-linecap="square"`
- **Scatter points**: small circles via `<use>` referencing a shared `<path>`
  in `<defs>`, colored per category (see palette above)
- **Scatter opacity**: `fill-opacity: 0.5–0.7` for background points; `1.0`
  for anchor/highlighted points
- **NN layer nodes**: pink circles `#e9afaf`, no stroke, `r ≈ 6mm`
- **NN edges**: `stroke="#808080"`, `stroke-width="1"`, `stroke="#cccccc"`
  outlines

---

## Export Settings

When finalizing figures for LaTeX:

```
inkscape:export-xdpi="600"
inkscape:export-ydpi="600"
```

Export as PNG via Inkscape; alternatively include directly with `\includesvg`
(requires the `svg` LaTeX package).

---

## Quick Reference: Common Element Recipes

### Flowchart process box
```svg
<rect x="X" y="Y" width="118" height="48" rx="4"
      fill="#e6e6e6" stroke="#999999" stroke-width="0.5" />
<text font-family="Arial" font-size="10" font-weight="bold"
      fill="#333333" text-anchor="middle" x="Xc" y="Yc">Label</text>
```

### Dataset icon (CSV / data input)
Use the canonical stacked-disc icon (see Dataset / CSV icons section above).
Never use a plain rect for dataset nodes. Wrap and position with:
```svg
<!-- s ≈ 0.23 for ~7 mm tall icon; adjust translate to place correctly -->
<g transform="matrix(0.23,0,0,0.23,X,Y)">
  <!-- paste full icon <g> block here, with FILL_COLOR replaced -->
</g>
<text font-family="Arial" font-size="3.6" font-weight="bold"
      fill="#333333" text-anchor="middle" x="Xc" y="Ylabel">Label</text>
```

### Graph node (large, teal)
```svg
<circle fill="#b2e6dc" stroke="none" r="2.9411876" />
```

### Graph node (small, self-stroked)
```svg
<circle fill="#48dbd6" stroke="#48dbd6"
        stroke-width="0.800804" stroke-linecap="round" r="1.8022451" />
```

### Section panel outline
```svg
<rect fill="none" stroke="#b7b7b7" stroke-width="0.17"
      stroke-opacity="0.72549" rx="1.35" />
```

### Arrows (all types — always `<path>`, never `<line>`)
```svg
<!-- Straight flow arrow -->
<path d="M X1,Y1 L X2,Y2"
      stroke="#999999" stroke-width="0.5" stroke-linecap="round"
      fill="none" marker-end="url(#Triangle)"/>

<!-- Dashed dependency / cache arrow -->
<path d="M X1,Y1 L X2,Y2"
      stroke="#888060" stroke-width="0.5" stroke-dasharray="2 1.5"
      stroke-linecap="round" fill="none" marker-end="url(#Triangle)"/>
```
