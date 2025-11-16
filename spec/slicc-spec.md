# SLICC Specification

## Overview
SLICC is an open, modular enhancement framework for existing storage systems (Host Platforms). 
This specification defines terminology, compatible geometry, hierarchy, dimensional codes, derivation rules.

## Terminology

### Host Platform Terms

- **Chassis** – outer structure holding bins  
- **Bin** – host container inserted into the chassis, composed of:
  - **Bottom** – lower interface  
  - **Core** – usable internal space  
  - **Top** – upper interface

### SLICC Layer Terms

- **Rack** – Harnesses the bin’s core region, reproduces bottom for seating and provides Slots for cartridge Rails.
- **Cartridge** – a modular storage element that typically consists of two main physical parts: **Shell** and **Lid**.
  These include the following functional regions:
  - **Rails** – slide into the Rack’s **Slots**, constraining lateral motion.
  - **Pull Tab** – engages the Bin’s **Top** envelope for retention and extraction.
  - **Body** – The usable volume of a Cartridge, defined by:
    - **Body Width / Height:** canonical dimensions set by the host Family (`body_width_mm`, `body_height_mm`).
    - **Span:** physical length occupied in the host’s front–back direction (`slot_count × slot_pitch − rack_clearance`).

## Hierarchy

SLICC is organized into three tiers:

### Platform (P)
A **Platform** corresponds to a real, pre-existing storage ecosystem (e.g. Gridfinity, Festool Organizer). 

A platform defines:

- The host bin geometry  
- The choice of dimensional modes (incremental or explicit)  
- Host unit sizes (if applicable)  
- Slot pitch  
- Rack envelope rules

Each platform is described by one YAML file (`<platform_id>.yaml`).

```
/platfroms/<platform_id>.yaml
```

Examples:

- `gf.yaml` → Gridfinity Family  
- `fto.yaml` → Festool Organizer Family

### Class (H-axis)
A **Class** defines the **height** domain for a platform.

Two types exist:
- **Incremental (H)** – uses host-defined height increments  
- **Explicit (h)** – uses direct mm values  

A Class is *not* a separate file; it is derived from a platform’s defined height mode.

Examples:
- `H9` = Gridfinity height class  
- `h81` = explicit height (in mm)

### Family (W-axis)
A **Family** is a concrete, buildable variant within a platform.

A Family defines:
- Canonical cartridge **body width** and **height**  
- The number of host width units (`W`) or direct width (`w`)  
- Optional length units (`L`) or explicit length (`l`)  
- Rack envelope (corner radius, rack width, etc.)  
- Slot pitch

## Dimensional System

### Code Types
SLICC uses:

| Code | Meaning |
|------|---------|
| **H / W / L** | Incremental host units |
| **h / w / l** | Explicit millimetres |
| **S** | SLICC slot units |

Each axis must use either incremental or explicit mode.

---

### 3.2 Slot Axis (S)
The Slot axis defines modularity and cartridge span (Host Length direction).

```
span_mm = slot_count × slot_pitch_mm − span_clearance_mm
```

Slot pitch is defined per platform.

---

## Geometry Rules

### Height Axis
```
bin_core_height_mm =
    bin_height_mm
  − bin_top_height_mm
  − bin_bottom_height_mm
```

**Incremental (H):**
```
bin_height_mm =
    height_units × chassis_height_increment_mm
  + bin_height_constant_mm
```

**Explicit (h):**
```
bin_height_mm = chosen from bin_height_mm list
```

---

### Width Axis
```
body_width_mm =
    core_width_mm
  + body_width_constant_mm
```

**Incremental (W):**
```
core_width_mm =
    width_units × chassis_width_increment_mm
  + core_width_constant_mm
```

**Explicit (w):**
```
core_width_mm = chosen explicit value
```

---

### Length Axis
**Incremental (L):**
```
core_length_mm =
    length_units × chassis_length_increment_mm
  + core_length_constant_mm
```

**Explicit (l):**

```
core_length_mm = chosen explicit value
```

---

### Rack Geometry
```
rack_slot_height_mm =
    bin_core_height_mm
  + rack_slot_constant_mm
```

`rack_corner_radius_mm` matches host bins fillet.

---

## ID Rules

Family ID is:
```
<platform_id>-(<H/h<class>>-)<W/w<family> # omit H/h for platforms with only one height.
```

### File Naming
Cartridge/STEP files follow:

```
<platform_id>_<H/h<class>>_<W/w<family>\<element>_S##.stl
```

Where `<element>` is one of:
- `shell`
- `lid`
- optional: `hub`, `rim`, etc. (family-defined)

Examples:
```
shell_S03.stp
shell_S05.stp
rack_L1_S12.stp
```

---

### Folder Naming

Family folder:
```
GF_H9_W1\
FTO_W1\ # class h68 can be omitted, because its the only one
```

Families may contain:
```
rack/
cartridge/
hingelid/
smd_06/
spool/
```

---

## Platform Definition Schema (YAML v0.2)

```yaml
name: "<Human-readable Name>"
platform_id: "<Short Code>"

# HEIGHT AXIS (choose H or h)
chassis_height_increment_mm: <value>
bin_height_constant_mm: <value>
bin_height_mm:
  - <value>

bin_top_height_mm: <value>
bin_bottom_height_mm: <value>

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

# SLICC LAYER
rack_slot_constant_mm: <value> # optional, default: -16
rack_corner_radius_mm: <value> # optional, default:   1
slot_pitch_mm: <value>

# OPTIONAL MASKS
mask_height: "00.0" # default H: "00"   / h: "000"
mask_width:   "0.0" # default W: "0.0"  / w: "000"
mask_length:  "0.0" # default L: "0.0"  / l: "000"

spec_version: "0.2"
spec_repo: "https://github.com/DCEM/slicc"
license: "CC BY 4.0"
license_url: "https://slicc.eu/license"
canonical_platform: "https://slicc.eu/platforms/#<platform_id>"
```

## Distribution & Publication
- Platform definition files: https://github.com/DCEM/slicc/platforms/<platform_id>
- SLICC Family index: https://slicc.eu/families/

## License
SLICC is released under **CC BY 4.0**.
See `LICENSE.md` or https://slicc.eu/license
