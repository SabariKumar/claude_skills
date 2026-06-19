---
name: pymol-render
description: >
  Workflow for generating ray-traced molecule images in PyMOL using Sabari's
  custom pymolrc. Use this skill whenever asked to render a molecule, produce
  a PyMOL image, generate a ball-and-stick figure, or write a PyMOL script.
  Trigger on phrases like "render this molecule", "make a PyMOL image",
  "ball and stick figure", "ray trace", or "pymol script". Always read this
  skill before writing any PyMOL commands or scripts.
---

# PyMOL Render Workflow

All molecule renders use the custom pymolrc from the CSU Theory Suite:
`20231024_sabari_pymolrc.txt`. This file must be loaded at the start of every
PyMOL session before any rendering commands are issued. It sets global render
quality settings and defines the `BallnStick` command used for all renders.

---

## Setup: Loading the pymolrc

The pymolrc is located at:
```
https://raw.githubusercontent.com/CSU-Theory-Suite/theorysuitescripts/refs/heads/main/pymol/20231024_sabari_pymolrc.txt
```

To load it in a PyMOL session, run from the PyMOL command line:
```python
run /path/to/20231024_sabari_pymolrc.txt
```

Or source it automatically by placing it (or symlinking it) at `~/.pymolrc`.

---

## Global Settings Applied by pymolrc

The pymolrc sets the following workspace defaults on load. Do not override
these unless specifically asked:

| Setting | Value | Effect |
|---|---|---|
| `bg_color` | `white` | White background |
| `ray_opaque_background` | `off` | Transparent background for PNG export |
| `orthoscopic` | `0` | Perspective projection |
| `ray_trace_mode` | `1` | Advanced ray tracing |
| `ray_texture` | `2` | Matte texture |
| `antialias` | `3` | High-quality antialiasing |
| `ambient` | `0.5` | Ambient light level |
| `spec_count` | `5` | Specular highlights |
| `shininess` | `50` | Surface shininess |
| `specular` | `1` | Specular reflection on |
| `reflect` | `0.1` | Low reflection |
| `space` | `cmyk` | CMYK colour space for print |

The pymolrc also applies **Bondi VDW radii** to all elements via `cmd.alter`
calls. These override PyMOL's defaults and should not be changed.

---

## Standard Render: BallnStick

`BallnStick` is the primary render command for all molecule images. It applies:

```python
cmd.color("gray30", "elem C")          # dark gray carbons
cmd.set("dash_gap", 0.01)
cmd.set("dash_radius", 0.035)
cmd.set("surface_quality", 2)
cmd.set("surface_type", 4)
cmd.set("depth_cue", "off")
preset.ball_and_stick(mode=1)          # PyMOL ball-and-stick preset
```

**Always call `BallnStick()` as the representation step** — never use
`show sticks`, `show spheres`, or `util.cbaw` directly for standard renders.

---

## Standard Render Script Template

Use this template for any molecule render. Fill in `INPUT_FILE` and
`OUTPUT_FILE`. Adjust `orient` and `zoom` as needed.

```python
# Standard BallnStick render script
# Assumes pymolrc has been sourced (either via ~/.pymolrc or explicit run)

from pymol import cmd
import pymol

# Load structure
cmd.load("INPUT_FILE.xyz")      # or .pdb, .mol2, .sdf, etc.

# Apply BallnStick representation
BallnStick()

# Orient and frame
cmd.orient()
cmd.zoom("all", buffer=2)

# Ray trace and save
cmd.ray(2400, 2400)             # width x height in pixels; 2400×2400 for print quality
cmd.png("OUTPUT_FILE.png", dpi=300)
```

**Resolution guidance**:
- Print / publication: `ray(2400, 2400)`, `dpi=300`
- Slide / presentation: `ray(1600, 1600)`, `dpi=150`
- Quick preview: `ray(800, 800)`, `dpi=72`

---

## Colour Conventions

The pymolrc sets `space cmyk` and `ray_trace_mode 1`. Colour choices should
be compatible with both screen and print:

| Element | Default colour after BallnStick |
|---|---|
| C | `gray30` (dark gray) |
| N | PyMOL default blue (from `preset.ball_and_stick`) |
| O | PyMOL default red |
| H | PyMOL default white/light gray |
| S | PyMOL default yellow |
| Halogens | PyMOL element defaults |

To apply CPK colouring instead of the gray-carbon default, call
`cmd.util.cbaw()` after `BallnStick()`. To apply colorblind-friendly colours,
call `CB()` (also defined in pymolrc) after `BallnStick()`.

---

## Other Available Commands (from pymolrc)

These are defined in the pymolrc and available after loading it:

| Command | Usage |
|---|---|
| `BallnStick()` | **Primary render** — ball and stick with gray carbons |
| `AO()` | Ambient occlusion spheres render, white background, opaque |
| `AOsolv()` | AO render including solvent molecules |
| `AOD()` | AO render with dark (gray20) carbon atoms |
| `CB()` | Load colorblind-friendly named colors (`cb_red`, `cb_blue`, etc.) |
| `CBSS()` | Colorblind-friendly cartoon/ribbon coloring by secondary structure |
| `colorh1(sel)` | Eisenberg hydrophobicity coloring, scheme 1 (red scale) |
| `colorh2(sel)` | Eisenberg hydrophobicity coloring, scheme 2 (green scale) |
| `yrb(sel)` | YRB hydrophobicity/charge surface coloring (Hagemans 2015) |
| `nci(arg1,arg2,arg3)` | NCI isosurface from density + gradient cube files |
| `mo_plot(arg1,arg2)` | MO isosurface (positive=red, negative=blue) from cube file |
| `spin_density_plot(arg1,arg2)` | Spin density isosurface from cube file |
| `density_plot(arg1,arg2)` | Electron density isosurface from cube file |
| `esp_plot(arg1,arg2,arg3)` | ESP-mapped density surface from two cube files |

---

## Writing Render Scripts

When asked to write a PyMOL render script:

1. Always start with a comment block noting that the pymolrc must be loaded.
2. Use `BallnStick()` as the representation step unless a different style is
   explicitly requested.
3. Include `cmd.orient()` and `cmd.zoom("all", buffer=2)` before ray tracing.
4. Default to `ray(2400, 2400)` and `dpi=300` unless the use case suggests
   lower resolution.
5. Save as PNG (`.png`). Use `ray_opaque_background off` (already set by
   pymolrc) for transparent backgrounds suitable for compositing into figures.
6. If multiple views are needed, loop over orientations using
   `cmd.rotate("y", angle)` between `cmd.ray()` / `cmd.png()` calls.

---

## Example: Render from an XYZ File

```python
# Requires pymolrc to be loaded first (e.g. place in ~/.pymolrc)
from pymol import cmd

cmd.load("molecule.xyz")
BallnStick()
cmd.orient()
cmd.zoom("all", buffer=2)
cmd.ray(2400, 2400)
cmd.png("molecule_ballnstick.png", dpi=300)
```

## Example: Multiple Views (front, side, top)

```python
from pymol import cmd

cmd.load("molecule.xyz")
BallnStick()
cmd.orient()
cmd.zoom("all", buffer=2)

for name, rot in [("front", (0,0,0)), ("side", (0,90,0)), ("top", (90,0,0))]:
    cmd.reset()
    cmd.orient()
    cmd.rotate("x", rot[0])
    cmd.rotate("y", rot[1])
    cmd.rotate("z", rot[2])
    cmd.zoom("all", buffer=2)
    cmd.ray(2400, 2400)
    cmd.png(f"molecule_{name}.png", dpi=300)
```

---

## pymol_style.py: Additional Commands

A second script, `pymol_style.py`, provides an alternative `BallnStick`
implementation plus `drawgridbox` and `nci`. Load it alongside the pymolrc:

```python
run /path/to/pymol_style.py
```

Source:
```
https://raw.githubusercontent.com/CSU-Theory-Suite/theorysuitescripts/refs/heads/main/pymol/pymol_style.py
```

### BallnStick (pymol_style.py version — takes a selection argument)

This version differs from the pymolrc version: it takes an explicit selection
string and applies its own colour scheme rather than using
`preset.ball_and_stick`. Use this when you need per-object control or the
specific colour settings below.

```python
BallnStick("molecule_name")
```

Applied settings:
| Setting | Value |
|---|---|
| C colour | `gray85` (light gray) |
| H colour | `gray98` (near white) |
| N colour | `slate` (blue-gray) |
| stick_radius | `0.07` |
| sphere_scale | `0.18` (heavy atoms), `0.13` (H) |
| stick_color | `black` |

**Example — render a single loaded object:**
```python
from pymol import cmd

cmd.load("molecule.xyz")
BallnStick("molecule")      # pass the object name, not "all"
cmd.orient()
cmd.zoom("molecule", buffer=2)
cmd.ray(2400, 2400)
cmd.png("molecule_ballnstick.png", dpi=300)
```

---

### drawgridbox — Draw a CGO Grid Box Around a Selection

Draws a coloured wireframe grid box (CGO object) around any selection.
Useful for visualising binding sites, unit cells, or spatial regions of
interest alongside a BallnStick render.

**Requires** these imports at the top of the script:
```python
from pymol.cgo import *
from random import randint
```

**Signature:**
```python
drawgridbox(selection="(all)", nx=10, ny=10, nz=10,
            padding=0.0, lw=2.0, r=1.0, g=1.0, b=1.0, dw=5.0)
```

| Parameter | Default | Meaning |
|---|---|---|
| `selection` | `"(all)"` | Selection to box around |
| `nx, ny, nz` | `10, 10, 10` | Grid subdivisions per axis |
| `padding` | `0.0` | Extra space (Å) around selection extent |
| `lw` | `2.0` | Line width |
| `r, g, b` | `1.0, 1.0, 1.0` | Box colour (white by default) |
| `dw` | `5.0` | CGO dot width |

Returns the name of the created CGO object (e.g. `"gridbox_4821"`).

**Example — white grid box around entire molecule:**
```python
from pymol import cmd
from pymol.cgo import *
from random import randint

cmd.load("molecule.xyz")
BallnStick("molecule")

# Draw a 5×5×5 white grid box with 1Å padding
drawgridbox("molecule", nx=5, ny=5, nz=5, padding=1.0,
            lw=2.0, r=1.0, g=1.0, b=1.0)

cmd.orient()
cmd.zoom("molecule", buffer=3)
cmd.ray(2400, 2400)
cmd.png("molecule_gridbox.png", dpi=300)
```

**Example — coloured box around a binding site selection:**
```python
from pymol import cmd
from pymol.cgo import *
from random import randint

cmd.load("complex.pdb")
BallnStick("complex")

# Highlight binding site residues within 5Å of ligand
cmd.select("site", "byres (resn LIG around 5)")
cmd.show("sticks", "site")

# Draw a cyan box around the binding site only
drawgridbox("site", nx=8, ny=8, nz=8, padding=2.0,
            lw=1.5, r=0.0, g=0.8, b=0.8)

cmd.orient("site")
cmd.zoom("site", buffer=3)
cmd.ray(2400, 2400)
cmd.png("binding_site_grid.png", dpi=300)
```

---

### nci — Non-Covalent Interaction (NCI) Isosurface

Renders an NCI isosurface from a pair of cube files produced by
[NCIplot](https://github.com/rmera/ncipy) or similar: a reduced density
gradient (`{stem}-grad`) and a signed density (`{stem}-dens`). The surface
is coloured by a rainbow ramp mapped to the density range `[arg2, arg3]`.

**Signature:**
```python
nci(arg1, arg2, arg3)
```

| Argument | Meaning |
|---|---|
| `arg1` | Filename stem (without `-dens`/`-grad` suffix or `.cube`) |
| `arg2` | Lower bound of density ramp (integer, e.g. `-3`) |
| `arg3` | Upper bound of density ramp (integer, e.g. `3`) |

Expects two cube files in the working directory:
- `{arg1}-dens.cube` — signed electron density
- `{arg1}-grad.cube` — reduced density gradient

**Example — NCI surface on a hydrogen-bonded dimer:**
```python
from pymol import cmd

# Load the molecule for context
cmd.load("dimer.xyz")
BallnStick("dimer")

# Load the cube files (NCIplot output, stem = "dimer_nci")
cmd.load("dimer_nci-dens.cube")
cmd.load("dimer_nci-grad.cube")

# Render NCI surface, density ramp from -3 to 3
nci("dimer_nci", -3, 3)

# Set isosurface transparency for visibility
cmd.set("transparency", 0.3, "grad")

cmd.orient()
cmd.zoom("all", buffer=2)
cmd.ray(2400, 2400)
cmd.png("dimer_nci.png", dpi=300)
```

**Colour interpretation of the NCI ramp:**
- **Blue** (negative density): strong attractive interactions (H-bonds, ion pairs)
- **Green** (near-zero density): weak/van der Waals interactions
- **Red** (positive density): steric repulsion / ring strain

**Tip**: adjust `arg2`/`arg3` to contract or expand the ramp. Typical ranges:
- `nci("stem", -3, 3)` — standard
- `nci("stem", -5, 5)` — wider, more uniform colouring
- `nci("stem", -2, 2)` — tighter, emphasises strong interactions
