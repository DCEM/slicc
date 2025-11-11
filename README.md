# SLICC Specification

**SLICC** is an open modular storage specification. 
It defines how printable **Racks** and **Cartridges** integrate with existing **Host Platforms** such as Gridfinity or Festool Organizers.

This repository contains only the **official specification**,
**family definitions**, and **reference diagrams**.

For releases, community links, and project updates see
 [https://slicc.eu](https://slicc.eu)

---

##  Specification

The canonical SLICC specification is located in:

[`/spec/slicc-spec.md`](spec/slicc-spec.md)

This document defines:
- Host geometry terms (Chassis, Bin, Bottom / Core / Top)
- SLICC components (Rack, Cartridge, Body, Slots)
- Dimensional conventions and slot logic
- Family YAML schema and publication rules

---

##  Families

Host-specific **Family** definitions are stored in:

[`/families/`](families)

Each YAML file describes one host platform:

| File       | Description                  |
| ---------- | ---------------------------- |
| `FTO.yaml` | Festool Organizer compatible |
| `GF9.yaml` | Gridfinity H9                |

These files define the canonical body dimensions and slot pitch for that Family.

