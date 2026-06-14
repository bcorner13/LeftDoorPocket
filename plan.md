# plan.md — Left Door Pocket

> **Note:** This is a *retrofit* plan. The geometry and assembly already exist
> (built across sessions through 2026-05-10). This document records the realized
> design plus the remaining roadmap, so future sessions don't re-derive it.
> `STATE.md` holds the detailed, living state; this is the bootstrap-required plan view.

## PARAMETERS

Current `Params.FCStd` VarSet (3 properties — build-volume visualization only):

| Name | Type | Default | Purpose |
|---|---|---|---|
| `PrintBedX` | App::PropertyLength | 350 mm | K2 Plus build volume X |
| `PrintBedY` | App::PropertyLength | 350 mm | K2 Plus build volume Y |
| `PrintBedZ` | App::PropertyLength | 350 mm | K2 Plus build volume Z |

Parametric backlog — Params to add during the refactor (per `CLAUDE.md`):
- Pocket envelope: `LengthX`, `LengthY`, `HeightZ`, `WallThick`, `BaseThick`
- Pegs: `PegD`, `PegLength`, `PegOffsetX`, `PegOffsetY`
- Inserts/screws: `InsertBoreD`, `InsertDepth`, `ScrewClearD`
- Clearances (decoupled, one per interface):
  - `PegFitClearance` — peg-into-hole print fit
  - `BoltHoleClearance` — screw-through clearance
  - `ClearanceSide` — geometric tightness (~0.05–0.1 mm), kept separate from print fits

## FEATURE TREE

```
Assembly.FCStd  (Assembly::AssemblyObject — top level)
├── L_DoorPocket          → DoorPocketSet#Body      (GROUNDED)
├── L_Accessory_1         → Accessory_1#Body002
├── L_Accessory_2         → Accessory_2#Body003
├── SubAsm_BackingPlate   (nested Assembly — rigid unit)
│   ├── L_BackingPlate    → DoorPocketSet#Body001
│   └── Insert × 16       (M4×8.15 heat inserts)
├── Screw × 16            (M4×25, removable group)
└── PrintBed_K2Plus       (Part::Box, parametric, hidden)

Per-part source docs (parts/):
- DoorPocketSet.FCStd → Body (DoorPocket) + Body001 (BackingPlate, cross-body SubShapeBinder)
- Accessory_1.FCStd   → Body002
- Accessory_2.FCStd   → Body003
```

## CONSTRAINT STRATEGY

- Each per-part FCStd is refactored body-by-body (start simplest: Accessory_1).
- Every dimensional constraint bound via `setExpression` to `<<Params>>#VarSet.<name>`.
- Sketches attached to **datum planes** (convert any feature-face attachments / Deactivated
  MapMode peg datums to origin-plane-offset datums with parametric offsets).
- Clearances decoupled per interface — never overload one clearance variable.
- App::Link components use `LinkTransform=True` so intrinsic placements compose.

## VALIDATION

1. `python3 scripts/audit_parametric.py` — must report zero issues per file
   (current backlog: 127 unbound across the three part docs).
2. Geometry watertight / manifold (CAD_STANDARDS.md).
3. Build-plate fit: each printable body ≤ 350 mm on the K2 Plus, OR a validated
   diagonal orientation / midline split.
4. Assembly recompute clean; exploded view resolves; no DAG warnings.

## ROADMAP (active → deferred)

1. **K2 Plus fit** (active) — diagonal-print test of the 392 mm DoorPocket/BackingPlate;
   fall back to a parametric midline split if artifacts are unacceptable.
2. **Parametric refactor** — body-by-body, audit-clean each before moving on.
3. **Assembly polish** — explicit fixed joints, per-step instruction screenshots.
