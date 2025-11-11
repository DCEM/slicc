# SLICC Specification

This specification defines how SLICC enhances existing storage **Host Platforms** ( e.g. *Gridfinity*,  *Festool Organizer*) with modular, printable components while keeping everything compatible, open, and consistent.
This way SLICC integrates with what you already have and adds what you need.

## Taxonomy

This section defines the physical and logical structure of the SLICC framework.

### *Host Platform* terms (pre-existing platform)

- **Chassis** – the outer host structure that holds and constrains Bins
- **Bin** – the host container that installs in the Chassis
  - A Bin is described in three functional regions:
    - **Bottom** – lower interface zone (seating/base features).
    - **Top** – upper interface zone (lid/locking/stacking-lip features).
    - **Core** – the **usable region** between **Bottom** and **Top**

### *SLICC* terms (what SLICC adds)

- **Rack** – occupies the **Core** region of the Bin, reproduces the **Bottom** interface for seating, and provides **Slots** that accept Cartridges.
- **Cartridge** – a modular storage unit that interfaces with the Rack.
   It consists of two main physical parts: **Shell** and **Lid**.
   These include the following functional regions:
  - **Rails** – slide into the Rack’s **Slots**, constraining lateral motion.
  - **Pull Tab** – engages the Bin’s **Top** envelope for retention and extraction.
  - **Body** – the usable volume of a Cartridge, defined by:
    - **Body Width / Height:** canonical dimensions set by the host Family (`body_width_mm`, `body_height_mm`).
    - **Span:** physical length occupied in the host’s front–back direction (`slot_count × slot_pitch − rack_clearance`).

> **In short:**
>  The **Chassis** and its **Bins** form the *host platform*.
>  The **Rack** and its **Cartridges** form the *SLICC layer* that upgrades it —
>  sized precisely to the Bin’s external envelope (**Bottom**, **Core**, and **Top**) while providing a standardized slotted interface.

## Dimensions and Host Units

SLICC distinguishes between **continuous dimensions** and **host system unit references**.

| Concept        | Representation                   | Definition                                   | Example           |
| -------------- | -------------------------------- | -------------------------------------------- | ----------------- |
| **Dimensions** | `height × width × length / span` | Continuous, measured in mm                   | 81 × 108 × 162 mm |
| **Host Units** | `H / W / L`                      | Discrete host-defined classes (if available) | H3 W2 L1          |
| **Slot Count** | `S`                              | SLICC-defined slot units (always discrete)   | S3                |

### Host Unit Base Dimensions

`host_unit_width_mm`, `host_unit_length_mm`, `host_unit_height_mm`
Real-world size (in millimetres) of one Host Unit along each axis.
Used when the host platform defines discrete grid-based units (e.g. Gridfinity).

### Body Dimensions

`body_width_mm`, `body_height_mm`
Define the canonical cartridge body size for this Family — typically corresponding to one host width and the Family’s standard height class.

If the host system defines discrete height or width units (`H` / `W`), these values correspond to those host unit counts as given in the Family YAML (`host_height_units`, `host_width_units`).

### Rack Envelope
Define the outer geometry of the SLICC Rack within the Bin’s **Core** region.

- `rack_corner_radius_mm` — outer corner radius (matches the Bin’s internal fillet).
- `rack_width_mm` — total rack width at the Core interface.
- `rack_length_mm` — optional; define only when the host platform constrains length explicitly.


### Host Units

| Code  | Meaning                                               | Defined by  | Example        |
| ----- | ----------------------------------------------------- | ----------- | -------------- |
| **H** | Height Units (**SLICC Class**) – host vertical module | Host system | `H3 Class`     |
| **W** | Width Units (**SLICC Family**) – host lateral module  | Host system | `H3 W2 Family` |
| **L** | Length Units (**Rack length**) – front-to-back module | Host system | `Rack L1`      |

- When the host system supports discrete unitization, these codes must follow the host’s convention.  
- Fractional values (e.g., `W1.5`, `L0.5`) are valid when SLICC provides partial modules.  
- If the host has only one fixed height, the **H** code may be omitted.  
- If a host does **not** provide discrete units, use lowercase **h w l** followed by the **dimension in mm** instead (e.g., `h81 w108 l162`).  

### Slot Count

| Code  | Meaning                                                      | Defined by | Example         |
| ----- | ------------------------------------------------------------ | ---------- | --------------- |
| **S** | **Slot Count** – number of SLICC slots along the rack’s span (length) or Rails along a Cartridge | SLICC      | `Cartridge S03` |

- `S` is always defined within the SLICC system
- The **slot count** determines the maximum physical **span** a cartridge can occupy:
  `span = S * slot_pitch - 2 * clearance`

### Summary Rules

| Category                | Code(s)   | Defined by      | When to use                 |
| ----------------------- | --------- | --------------- | --------------------------- |
| **Host Units**          | H / W / L | Host system     | When discrete modules exist |
| **Measured Dimensions** | h / w / l | SLICC (derived) | When host units don’t exist |
| **Slot Units**          | S         | SLICC           | Always present              |



## Slot Count and Span

- **Slot Count** (`slot_count`) – number of slot positions along the rack’s **Length**.  (Rails will also be counted in slot units)
- **Span** (`span`) – physical distance a cartridge occupies along the same axis (`span = slot_count × slot_pitch - clearance`).  

User-facing models use the single-letter code **S** for both concepts:  
`S03` → requires 3 slots in a rack and **spans** three slot pitches.

---

## Choosing Slot Pitch

The **slot pitch** defines modularity and labeling granularity. A practical default is **≈ 3.5 mm**, which balances usable storage, printable resolution, and label compatibility (6 / 9 / 12 mm divisions).

**Rule of thumb:** target a ratio of about **1 slot : 1/12 host Length Unit**.

| Platform              | Host L1 Rack → Slot Count                   | Slot Pitch | Notes                                              |
| --------------------- | ------------------------------------------- | ---------- | -------------------------------------------------- |
| **Festool Organizer** | **S12** (for **L1** rack)                   | **3.5 mm** | 1 Rack L1 → S12                                    |
| **Gridfinity**        | **S24** (for **L2**) / **S11** (for **L1**) | **3.4 mm** | L2 → S24 is ideal; L1 → S11 due to geometry limits |

This ratio is the “sweet spot” for granularity, label sets, and divisibility. Future platforms should aim for similar slot_pitch / L ratios.

## Distribution & Publication

Each **Family** is defined by a single YAML file in a public, version-controlled repository.

For the canonical reference implementation, all Families are stored in:

- Git repository: https://github.com/DCEM/slicc  
- YAML files: `/families/<platform_id>.yaml`  
- Human-facing index: https://slicc.eu/families/

### YAML Schema (v0.1 – Draft)

This schema represents the first version of the SLICC Family definition format.
It is **not yet complete** and may expand in later versions.

All numeric or measured fields **must include a unit suffix** where applicable
(e.g. `_mm` for millimetres, `_um` for micrometres).

```yaml
# SLICC Family Definition (YAML v0.1)
family_name: "<Descriptive Name>"
platform_id: "<Short Code>"

# Canonical cartridge body for this Family
body_width_mm: <value>
body_height_mm: <value>

# Host unit classification (optional, omit if value = 1)
host_height_units: <int>   # e.g. 9 for H9
host_width_units: <int>    # e.g. 1 for W1

# Host base unit sizes (optional, for grid-based systems)
host_unit_width_mm: <value>
host_unit_length_mm: <value>
host_unit_height_mm: <value>

# Rack envelope (platform-specific)
rack_corner_radius_mm: <value>
rack_width_mm: <value>
# rack_length_mm: <value>   # optional

slot_pitch_mm: <value>

spec_version: "0.1"
spec_repo: "https://github.com/DCEM/slicc"

license: "CC BY 4.0"
license_url: "https://slicc.eu/license"
canonical_family: "https://slicc.eu/families/#<platform_id>"
```

> **Note:** v0.1 is not yet complete and may expand in future versions as new host platforms are integrated.

### License

- **See:** [`LICENSE.md`](LICENSE.md)
- **Canonical version:** https://slicc.eu/license

