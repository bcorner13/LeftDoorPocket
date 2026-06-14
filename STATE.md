# LeftDoorPocket — Project State

**Last updated:** 2026-05-10 (end of restructure session)
**Canonical working file:** `Assembly.FCStd`
**Status:** Restructure complete. **Next phase: K2 Plus build-plate fit.**

## File layout

```
LeftDoorPocket/
├── Assembly.FCStd                              ← top-level Assembly (1.1.2 workbench)
├── Params.FCStd                                ← VarSet for cross-doc parametric expressions
├── Left Door Pocket V1.3.FCStd                 ← legacy fallback (DO NOT WORK IN)
├── Left Door Pocket V1.3.Session-2026-05-09.FCStd  ← snapshot taken before LCS strip
├── parts/
│   ├── DoorPocketSet.FCStd                     ← contains Body (DoorPocket) + Body001 (BackingPlate)
│   ├── Accessory_1.FCStd                       ← Body002
│   └── Accessory_2.FCStd                       ← Body003
├── archive/
│   ├── Left Door Pocket.FCStd
│   ├── Left Door Pocket V1.0/V1.1/V1.1_backup/V1.2.FCStd
├── assembly_placements.json                    ← sidecar: body Y=27.39 record
├── scripts/audit_parametric.py                 ← parametric debt auditor
├── macros/create_params.FCMacro                ← bootstrap macro for Params.FCStd
├── CLAUDE.md                                   ← project rules
├── CAD_STANDARDS.md                            ← commercial design rules
├── MCP_TOOLS_REFERENCE.md                      ← FreeCAD MCP tool reference
└── STATE.md                                    ← this file
```

## Assembly architecture

```
Assembly  (Assembly::AssemblyObject, internal name "AssemblyV2")
├── L_DoorPocket               (App::Link → DoorPocketSet#Body, GROUNDED)
├── L_Accessory_1              (App::Link → Accessory_1#Body002)
├── L_Accessory_2              (App::Link → Accessory_2#Body003)
├── SubAsm_BackingPlate        (Assembly::AssemblyObject — sub-assembly, rigid unit)
│   ├── L_BackingPlate         (App::Link → DoorPocketSet#Body001)
│   └── Insert × 16            (M4x8.15 heat inserts, Part::FeaturePython)
├── Screw × 16                 (M4x25 screws — selectable as group)
├── Joints / GroundedJoint     (L_DoorPocket grounded)
└── PrintBed_K2Plus            (Part::Box, parametric, hidden by default)
```

**Component counts** (verified via UtilsAssembly):
- Main assembly: 36 components total
- Sub-assembly: 17 components (1 BackingPlate + 16 inserts)

**Why this architecture:**
- `SubAsm_BackingPlate` is its own `Assembly::AssemblyObject` so it can have its **own nested exploded view** (separate the 16 inserts from the BackingPlate as a sub-step).
- Heat inserts are press-fit (permanent) → rigid sub-assembly with BackingPlate.
- Screws are removable → kept as individual main-assembly components so they can be selected as a group and dragged out.
- All link Placements composed correctly via `LinkTransform=True`. BackingPlate's intrinsic Y=27.39mm carries through automatically.

## Parametric state

`Params.FCStd` VarSet currently has **3 properties**:
- `PrintBedX`, `PrintBedY`, `PrintBedZ` — all 350mm (K2 Plus build volume)

Bound to `Assembly.FCStd:PrintBed_K2Plus` for build-volume visualization.

**Per-part files have NO parametric bindings yet**:
- `parts/DoorPocketSet.FCStd`: 72 unbound issues
- `parts/Accessory_1.FCStd`: 24 unbound issues
- `parts/Accessory_2.FCStd`: 31 unbound issues
- Total: **127 unbound dimensional constraints + DAG-risk attachments + unconstrained sketches**

These are the parametric refactor backlog. Run `python3 scripts/audit_parametric.py` for the full list.

## Known cosmetic issues (not blocking)

1. **Internal Name `AssemblyV2`** vs Label "Assembly" — caused by name-collision avoidance during the V1.3 App::Part → Assembly::AssemblyObject conversion. User sees correct label in the tree; only matters for scripting.
2. **`*.bin (0 bytes)` warnings on doc load** — empty SuppressedShape/InternalShape cache files left over from the per-part prune passes. Geometry resolves fine via recompute. To clean up: open each per-part FCStd, force a full recompute, save.
3. **`Assembly001` may reappear** if user runs `Assembly_CreateAssembly` again (creates a new empty Assembly with internal name "Assembly" since that's free). Just delete it from the tree.

## Lessons learned this session (worth remembering)

- **FreeCAD's RunInitGuiPy exec context** does NOT set `__file__` and does NOT share module-level names with class methods or deferred callbacks. Stub-loader pattern (separate `_workbench.py` exec'd from `InitGui.py` with explicit shared namespace) is the correct fix. Applied to `FreeCADMCPAgent` addon at both local + 5TB locations.
- **`App::Link` `LinkTransform`** defaults to `False` in 1.1, meaning the linked object's intrinsic Placement is NOT applied — only the link's own Placement. Set to `True` to compose properly. We hit this with the BackingPlate Y=27.39 ghost gap.
- **Assembly workbench requires `Assembly::AssemblyObject` typeId**, not `App::Part`. V1.3-era "Assembly" objects were App::Part shells — they LOOK assembly-like but `Assembly_CreateView`, `Assembly_CreateBom`, and joint commands stay grayed out.
- **Tree clicks don't register with the exploded-view dialog.** Selection paths must be rooted at the Assembly object name; clicking the App::Link in the tree generates a path rooted at the link, which `getComponentReference` rejects. **Always click on a face in the 3D view.**
- **`Assembly_InsertLink` GUI command and `addObject("App::Link")` produce equivalent components.** No metadata difference. The workbench identifies components by type + parent + LinkedObject.
- **Cross-body `SubShapeBinder`** in BackingPlate (binds outline to DoorPocket.Fillet001.Face27) is why we kept those two bodies in one FCStd file (`DoorPocketSet.FCStd`) rather than splitting them. Cross-doc binders work but add complexity; we'll revisit in the parametric pass.

## ⚠️ Build-plate fit assessment (next phase)

K2 Plus build volume: **350×350×350 mm**

| Part | Size (X×Y×Z mm) | Fits K2 Plus? |
|---|---|---|
| **DoorPocket** | **392 × 126 × 157** | ❌ **42mm too long in X** |
| **BackingPlate** | **392 × 9 × 157** | ❌ **42mm too long in X** |
| Accessory_1 | 69.7 × 70 × 35 | ✅ |
| Accessory_2 | 70 × 70 × 60 | ✅ |

**The two main bodies do not fit on the K2 Plus.** Options for next phase:

1. **Diagonal print** — 350√2 ≈ 495mm available across the bed diagonal. Need to verify print orientation works for the geometry (door pocket cavity opening, overhangs).
2. **Parametric scale-down** — make `LengthX` a Param and reduce. Affects functional fit on the actual door panel — probably non-starter without door-fit redesign.
3. **Split design** — divide DoorPocket and BackingPlate at midpoint with a mating interface (dovetail, dowels, glue joint). Two halves, each ≤200mm, joined post-print.
4. **Switch printer** — Saturn 4 is smaller (218×123×260 typical for Mk2/Ultra), so probably worse. No clear option here.

**Likely next-session direction:** option 1 first (cheapest test — just orientation), fall back to option 3 (design work) if angled printing introduces unacceptable artifacts on the door pocket cavity walls.

## Open follow-ups (for next session)

**Phase: K2 Plus fit**
- [ ] Test diagonal-orientation print of DoorPocket on K2 Plus (slicer simulation first)
- [ ] If diagonal fails → design a midline split for DoorPocket + matching BackingPlate split
- [ ] Verify Accessory_1, Accessory_2 print orientation (peg overhang, surface finish on cylinder side)

**Phase: parametric refactor (deferred)**
- [ ] Run audit, work body-by-body. Start with Accessory_1 (24 issues, simplest).
- [ ] Convert peg datums (`PegPlane_L`/`PegPlane_R`, `DP2_PegL`/`DP2_PegR`) from Deactivated MapMode to attached-to-origin-plane with parametric offsets.
- [ ] Bind every dimensional constraint via `setExpression` to `<<Params>>#VarSet.<name>`.
- [ ] Decouple clearances into per-interface vars: `PegFitClearance`, `BoltHoleClearance`.
- [ ] Re-run `scripts/audit_parametric.py` between bodies, target zero issues per file.

**Phase: assembly polish (low priority)**
- [ ] Add `Joint_Fixed` between L_DoorPocket and SubAsm_BackingPlate (replace the ad-hoc grounded-only setup with explicit kinematic constraints)
- [ ] Decide screw motion: keep them as individual movable components or group via fixed joints
- [ ] Build the per-step assembly-instruction screenshots (the documentation script)
- [ ] Optional: write the frame-export Python script for exploded-view animation

## How to resume next session

1. Open `Assembly.FCStd` (it auto-loads the 4 source docs via App::Link).
2. Double-click the `Assembly` node in the tree to make it active (FreeCAD doesn't persist active state).
3. Confirm the geometry looks right — DoorPocket grounded, BackingPlate sub-asm flush, accessories in their sockets.
4. Then start the K2 Plus fit work.
