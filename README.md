# Left Door Pocket

Parametric storage pocket / organizer for the **inside of a Daytona Coupe driver-side
door**, designed against a 3D scan of the door panel. Multi-part printed assembly
(pocket + backing plate + two modular accessories) joined with M4 heat-set inserts and
screws.

Built and printed on the **Creality K2 Plus** (PLA for test, engineering plastic for
production), per the workspace standards in `CAD_STANDARDS.md`.

## Status

- **Canonical file:** `Assembly.FCStd` (FreeCAD 1.1.2 Assembly workbench).
- Geometry + assembly complete. **Active phase: K2 Plus build-plate fit** — the
  DoorPocket and BackingPlate are 392 mm in X (42 mm over the 350 mm bed).
- **Parametric backlog:** 127 unbound dimensions across the three part files; only the
  print-bed visualization is parametric so far. See `STATE.md` and `plan.md`.

## Layout

```
LeftDoorPocket/
├── Assembly.FCStd          ← canonical top-level assembly
├── Params.FCStd            ← VarSet for cross-doc parametric expressions
├── Left Door Pocket V1.3.FCStd  ← legacy single-doc fallback (do not work in)
├── parts/                  ← per-part source bodies (DoorPocketSet, Accessory_1/2)
├── archive/                ← historical V1.0–V1.2 versions (do not touch)
├── cad_files/              ← door 3D-scan reference (.stl, .mlp)
├── macros/                 ← project .FCMacro files
├── scripts/                ← audit_parametric.py and utilities
├── 3mf/  stl/              ← print deliverables (tracked)
├── gcode/                  ← slicer output (gitignored, regenerable)
├── images/                 ← screenshots / renders
├── intent.md  plan.md  STATE.md   ← goal, plan, detailed living state
└── CLAUDE.md + standards docs      ← project + workspace rules
```

## Working in this project

1. Open `Assembly.FCStd` (it pulls in the 4 source docs via App::Link) and double-click
   the `Assembly` node to make it active.
2. All geometry must stay parametric — read `CLAUDE.md` before any change.
3. After any edit: `python3 scripts/audit_parametric.py` (authoritative; fix violations
   before committing).
4. Write changes as macros under `macros/`, not by editing FCStd XML.

See `STATE.md` for assembly architecture, lessons learned, and the full next-session
checklist.
