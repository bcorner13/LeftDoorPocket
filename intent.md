# intent.md — Left Door Pocket

## Goal

A parametric storage pocket / organizer that mounts to the **inside of a Daytona Coupe
driver-side (left) door**, designed to fit the door panel captured by 3D scan
(`cad_files/Daytona Coupe Inside door.stl`).

The deliverable is a multi-part printed assembly:
- **DoorPocket** — the main pocket body (base, walls, internal cavity)
- **BackingPlate** — mounts the pocket to the door, carries 16 M4 heat-set inserts
- **Accessory_1 / Accessory_2** — modular insert accessories that seat in the pocket
- Hardware: 16× M4×8.15 heat inserts (press-fit, permanent) + 16× M4×25 screws (removable)

## Constraints

- Must follow `CAD_STANDARDS.md` (mm units, manifold geometry, Part/PartDesign).
- Must follow the parametric rules in `CLAUDE.md` / `~/.claude/CLAUDE.md`:
  everything driven by `Params.FCStd` VarSet, no literal sketch constraints,
  datum-plane attachment, decoupled per-interface clearances.
- Printable on the **Creality K2 Plus** (350×350×350 mm build volume).
  - ⚠️ Current DoorPocket and BackingPlate are **392 mm in X — 42 mm too long** for
    the bed. Resolving this (diagonal print vs. midline split) is the active next phase.
- Peg-in-hole and screw interfaces must use print-fit clearances (FDM-tolerant),
  not geometric tightness.

## Status

Geometry and the 1.1.2 Assembly are built (`Assembly.FCStd` canonical).
Remaining work: K2 Plus build-plate fit, then the parametric refactor backlog
(127 unbound dimensions across the three part files). See `plan.md` and `STATE.md`.
