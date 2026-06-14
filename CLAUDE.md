# Project rules — Left Door Pocket

**Everything in this project must be parametric.** No exceptions. Read the full rules below before making any geometry change. The global rules in `~/.claude/CLAUDE.md` apply on top of these — this file documents project-specific state and any project-only conventions.

## Canonical file

**`Assembly.FCStd`** is the canonical working file (FreeCAD 1.1.2 Assembly workbench). It
pulls in the four per-part source docs via App::Link — `parts/DoorPocketSet.FCStd`,
`parts/Accessory_1.FCStd`, `parts/Accessory_2.FCStd`, and `Params.FCStd`. Open
`Assembly.FCStd` and double-click the `Assembly` node to make it active.

`Left Door Pocket V1.3.FCStd` is the **legacy single-document fallback** — do not work in
it. The `Left Door Pocket V1.0/V1.1/V1.2` files now live in `archive/` and should be
ignored for new work. See `STATE.md` for the full assembly architecture and current state.

## Hard rules (project-specific reminders of the global rules)

1. **No literal numbers in sketch dimensional constraints.** Every dimensional constraint (DistanceX, DistanceY, Distance, Length, Radius, Diameter, Angle, etc.) must bind via `setExpression` to a Params variable — `<<Params>>#VarSet.VarName`. If the variable doesn't exist, **add it to `Params.FCStd` first**, then bind.

2. **No fixing geometry by editing raw sketch coordinates.** When geometry is misaligned, wrong-sized, or in the wrong position, fix it by:
   - Adjusting the driving Params value, OR
   - Adding a dimensional constraint that binds to the right Params variable.

   Do not edit `LineSegment StartX/StartY/EndX/EndY` or `Circle CenterX/CenterY` directly. The Spade Connector project lost three days to a Claude session that "fixed" misalignment by coordinate edits — don't repeat that here.

3. **Attach sketches to datum planes, not feature faces.** Create `PartDesign::Plane` datums (offset from origin/principal planes as needed) and attach sketches to those. Attaching to `Pocket_*.FaceN`, `Pad_*.FaceN`, or `Body_*.FaceN` creates DAG-cycle risk.

4. **Decoupled clearance variables.** Each printed mating interface gets its own Params variable — do not overload one clearance for multiple concepts. Likely candidates for this assembly:
   - `PegFitClearance` — peg-into-hole fit (the `Sketch_PegL`/`Sketch_PegR` and `Sk2_PegL`/`Sk2_PegR` interfaces between the pocket and backing plate)
   - `BoltHoleClearance` — if any bolt-through holes exist
   - Add more per interface as the design evolves.

   Keep geometric tightness (`ClearanceSide` ~0.05–0.1mm) separate from print-fit clearances (~0.2–0.5mm per side, material-dependent).

## Assembly architecture (current understanding from V1.3 structure)

V1.3 contains 4 PartDesign Bodies, 21 sketches, 9 pads, 2 pockets, 8 fillets, and several datum planes. Object naming suggests two main parts plus pegs:

- **Body / Body001 / Body002** — main pocket body (base, walls, internal pocket cut)
  - Sketches: `Sketch`, `Sketch001` … `Sketch014`
  - Features: `Pad`, `Pad001`, `Pad002`, `Pad003`, `Pocket`, `Pocket001`
- **Pegs (in `Body001` or its own Body)** — left/right alignment pegs
  - `Sketch_PegL`, `Sketch_PegR`, `Pad_PegL`, `Pad_PegR`
- **Body003** — backing plate / second part
  - `Sk2_Base`, `Sk2_Spline`, `Sk2_Rim`, `Sk2_PegL`, `Sk2_PegR`, `Pad2_Base`, `Pad2_PegL`, `Pad2_PegR`

The pegs on the second body mate with the peg holes on the main pocket — this is a printed peg-in-hole interface, so it needs a `PegFitClearance` Param (per-side).

This architecture description should be revised as you learn more by opening V1.3. Update this section when you understand the assembly better.

## Current state

The detailed, living state lives in **`STATE.md`** — read it first. Summary:

- `Assembly.FCStd` (canonical) + `parts/` source bodies are built; the 1.1.2 Assembly is complete.
- `Params.FCStd` exists with the print-bed visualization vars (`PrintBedX/Y/Z`). The
  per-part geometry is **not yet parametric** — 127 unbound dimensions remain (the refactor backlog).
- Historical `V1.0`–`V1.2` files live in `archive/` (excluded from the audit). `V1.3` is the legacy fallback at root.
- **Active phase:** K2 Plus build-plate fit (DoorPocket/BackingPlate are 392 mm in X vs the 350 mm bed).

## How to verify your change didn't break parametric

After any FreeCAD edit, before considering the task done, run:

```bash
python3 scripts/audit_parametric.py
```

It flags:
- Sketches with 0 constraints
- Sketches with dimensional constraints that lack expression bindings
- Sketches attached to feature faces (DAG risk)
- Pad/Pocket/Fillet/Chamfer features with literal numeric Length/Offset/Radius/Size

The audit is authoritative. If it reports violations, fix them before saving or committing.

## Workflow notes

- Write changes as `.FCMacro` files under `macros/`. Reasons: reviewable before running, re-runnable, idempotent-friendly, and uses FreeCAD's own serialization (safer than editing FCStd XML directly).
- For FreeCAD to find a macro from the **Macro → Macros…** menu, copy or symlink it to `~/Library/Application Support/FreeCAD/v1-1/Macro/`.
- Cross-document expressions: use `<<Params>>#VarSet.VarName`. The shorter `<<Params>>.VarName` form sometimes fails with "Params not found" errors.
- The user runs FreeCAD 1.1 on macOS. The MCP bridge auto-starts — that is not an error.
- Idempotency: macros should check `if name in vs.PropertiesList: skip` before adding properties, and replace constraints rather than append.

## Memory files (per-project context)

`~/.claude/projects/-Users-bradleycorner-Documents-3dPrinting-Commercial-License-LeftDoorPocket/memory/MEMORY.md` indexes the persistent memories for this project.

## Suggested first refactor steps (to make V1.3 parametric)

1. Run `macros/create_params.FCMacro` to create `Params.FCStd` with an empty VarSet.
2. Open V1.3 in FreeCAD with Params.FCStd.
3. Run the audit — it will report every unbound dimension and unconstrained sketch.
4. For each Body in turn (start with the simplest — likely the pegs):
   - Identify the driving dimensions of each sketch.
   - Add corresponding Params variables (`PegD`, `PegLength`, `PegOffsetX`, `PegOffsetY`, etc.).
   - Bind every dimensional constraint via `setExpression`.
   - Convert any sketch attached to a feature face to a datum plane attachment.
5. Re-run the audit between bodies. Don't move on until that body is clean.

This is multi-session work. Don't try to parametrize the whole assembly in one pass.
