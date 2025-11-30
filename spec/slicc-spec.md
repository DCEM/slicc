# SLICC Specification

## Overview

SLICC is an open, modular enhancement framework for existing storage systems (Host Platforms). 
This specification defines terminology, compatible geometry, hierarchy, dimensional codes, and derivation rules.

Platform-agnostic reference diagrams (e.g. rail/slot geometry) live in:

`/spec/diagrams/`

## Terminology

### Host Platform Terms

- **Chassis** – outer structure holding bins  

- **Bin** – host container inserted into the Chassis, composed of:

  - **Bottom** 
    Lower interface region. 
    Includes geometry such as seating features, base transitions, and lower blends.

  - **Top** 
    Upper interface region. 
    Includes lips, locking features, stacking-lip geometry.

  - **Core** 
    The main prismatic external envelope between Bottom and Top. 
    Defined as the **largest axis-aligned prismatic region** of the Bin’s external side-wall envelope whose walls are orthogonal to the Bottom plane **and whose vertical edges share a constant radius**. 
    This makes the Core a **rectangular prism with vertical rounded corners**.

    Core parameters:  

    - `core_width_mm`  
    - `core_length_mm`  
    - `core_height_mm`  
    - `core_corner_radius_mm`

  - **Peel** 
    All external transition geometry *outside* the Core envelope. 
    Peel consists of non-orthogonal features such as:

    - tapers  
    - chamfers  
    - curved blends between Core ↔ Bottom  
    - curved blends between Core ↔ Top  

    > **Note:** 
    > Peel is defined for future Platforms and should only be used when non-orthogonal transitions materially affect the Rack–Chassis interface.

### SLICC Layer Terms

- **Rack** – harnesses the Bin’s Core region, reproduces Bottom for seating and provides Slots for Cartridge Rails.
- **Cartridge** – a modular storage element that typically consists of two main physical parts: **Shell** and **Lid**. 
  These include the following functional regions:
  - **Rails** – slide into the Rack’s Slots, constraining lateral motion.
  - **Pull Tab** – engages the Bin’s Top envelope for retention and extraction.
  - **Body** – the usable volume of a Cartridge, defined by:
    - **Body Width / Height:** canonical dimensions set by the host Family (`body_width_mm`, `body_height_mm`).
    - **Span:** physical length occupied in the host’s Length (front–back) direction (`slot_count × slot_pitch_mm − span_clearance_mm`).

## Hierarchy

SLICC is organized into three tiers:

### Platform (P)

A **Platform** corresponds to a real, pre-existing storage ecosystem (e.g. Gridfinity, Festool Organizer). 

A Platform defines:

- The host bin external geometry (Bottom, Core, Top)  
- The choice of dimensional modes (incremental or explicit)  
- Host unit sizes (if applicable)  
- Slot pitch  
- Core corner radius  
- Rack envelope rules

Each Platform is described by one YAML file (`slicc.yaml`) stored at:

```text
/platforms/<platform_id>/slicc.yaml
```

Examples:

- `platforms/gf/slicc.yaml` → Gridfinity Platform  
- `platforms/fto/slicc.yaml` → Festool Organizer Platform

Platform-specific diagrams (pull-tab envelope, bottom interface, external core envelope) live in:

```text
/platforms/<platform_id>/diagrams/
```

### Class (H-axis)

A **Class** defines the **height** domain for a Platform.

Two types exist:

- **Incremental (H)** – uses host-defined height increments  
- **Explicit (h)** – uses direct mm values  

A Class is *not* a separate file; it is derived from a Platform’s defined height mode.

Examples:

- `H9` = Gridfinity height class  
- `h81` = explicit height (in mm)

### Family (W-axis)

A **Family** is a concrete, buildable variant within a Platform.

A Family defines:

- Canonical cartridge **body width** and **height**  
- The number of host width units (`W`) or direct width (`w`)  
- Optional length units (`L`) or explicit length (`l`)  
- Rack envelope (width, length, and optionally overrides)  
- Slot pitch (usually inherited from Platform)

Implementation files for Families (STEPs, label sheets, etc.) are not part of this specification repository and are typically linked from [slicc.eu](slicc.eu).

For Platforms that require an explicit **Family definition YAML** (e.g. multiple discrete heights, non-iterated geometries), those definition files SHOULD be stored as:

```text
/platforms/<platform_id>/families/<family_id>.yaml
```

#### Family ID

```text
<platform_id>-(<H/h<class>>-)<W/w<family>>
```

(omit `H/h` for Platforms with only one height.)

## Dimensional System

### Code Types

SLICC uses:

| Code          | Meaning                |
| ------------- | ---------------------- |
| **H / W / L** | Incremental host units |
| **h / w / l** | Explicit millimetres   |
| **S**         | SLICC slot units       |

Each axis must use either incremental or explicit mode.

### Slot Axis (S)

The Slot axis defines modularity and cartridge span (Host Length direction).

```text
span_mm = slot_count × slot_pitch_mm − span_clearance_mm
```

### Slot Pitch

Slot pitch is defined per platform.

The **slot pitch** defines modularity and labeling granularity. A practical default is **≈ 3.5 mm**, which balances usable storage, printable resolution, and label compatibility (6 / 9 / 12 mm divisions).

## Geometry Rules

### Height Axis

The external Core height is derived from the Bin height and the Top/Bottom regions:

```text
core_height_mm =
    bin_height_mm
  − top_height_mm
  − bottom_height_mm
```

**Incremental (H):**

```text
bin_height_mm =
    height_units × chassis_height_increment_mm
  + bin_height_constant_mm
```

**Explicit (h):**

```text
bin_height_mm = chosen from bin_height_mm list
```

### Width Axis

```text
body_width_mm =
    core_width_mm
  + body_width_constant_mm
```

**Incremental (W):**

```text
core_width_mm =
    width_units × chassis_width_increment_mm
  + core_width_constant_mm
```

**Explicit (w):**

```text
core_width_mm = chosen explicit value
```

### Length Axis

**Incremental (L):**

```text
core_length_mm =
    length_units × chassis_length_increment_mm
  + core_length_constant_mm
```

**Explicit (l):**

```text
core_length_mm = chosen explicit value
```

### Rack Geometry

#### Rack Slot height

```text
rack_slot_height_mm =
    core_height_mm
  + rack_slot_constant_mm
```

Default: `rack_slot_constant_mm = -16` 

#### Rack envelope

```
rack_width_mm        = core_width_mm
rack_length_mm       = core_length_mm
rack_height_mm       = bottom_height_mm + rack_slot_height_mm
rack_corner_radius_mm = core_corner_radius_mm
```

### File Naming

Cartridge/STEP files follow:

```text
<platform_id>_<H/h<class>>_<W/w<family>\<element>\<part>_S##.stp
```

Where `<element>` is one of:

- `shell`
- `lid`
- optional: `hub`, `rim`, etc.

Examples:

```text
GF_H9_W1\cartridge\lid_C350.stp
GF_H9_W1\cartridge\shell_S03.stp
GF_H9_W1\cartridge\shell_S05.stp
FTO_W1\rack\rack_L1_S12.stp
```

## Platform Definition Schema (YAML v0.3)

```yaml
name: "<Human-readable Name>"
platform_id: "<Short Code>"

# HEIGHT AXIS (choose H or h)
chassis_height_increment_mm: <value>
bin_height_constant_mm: <value>
bin_height_mm:
  - <value>

top_height_mm: <value>
bottom_height_mm: <value>

# WIDTH AXIS (choose W or w)
chassis_width_increment_mm: <value>
core_width_constant_mm: <value>
core_width_mm:
  - <value>
body_width_constant_mm: <value>

# LENGTH AXIS (choose L or l)
chassis_length_increment_mm: <value>
core_length_constant_mm: <value>
core_length_mm:
  - <value>

# CORE GEOMETRY
core_corner_radius_mm: <value> # canonical external core corner radius

# SLICC LAYER
rack_slot_constant_mm: <value> # optional, default: -16
slot_pitch_mm: <value>

# OPTIONAL MASKS
mask_height: "00.0" # default H: "00"   / h: "000"
mask_width:  "0.0"  # default W: "0.0"  / w: "000"
mask_length: "0.0"  # default L: "0.0"  / l: "000"

spec_version: "0.3"
spec_repo: "https://github.com/DCEM/slicc"
license: "CC BY 4.0"
license_url: "https://slicc.eu/license"
canonical_platform: "https://slicc.eu/platforms/#<platform_id>"
```

## Distribution & Publication

- Platform definition files: 
  `https://github.com/DCEM/slicc/tree/main/platforms/<platform_id>/slicc.yaml`
- SLICC Family index (printable implementations): 
  `https://slicc.eu/families/`

---

## License

SLICC is released under **CC BY 4.0**. 
See `LICENSE.md` or https://slicc.eu/license
