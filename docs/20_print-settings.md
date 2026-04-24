# 3D Print Settings

Reference slicer settings and material recommendations for all RedRover printed parts.
Print packs are in `3d-prints/` — see the [repo README](../README.md) for the full pack list.

---

## Material Guide by Sub-Assembly

| Pack | Sub-assembly | Recommended material | Rationale |
|---|---|---|---|
| 01 | Chassis body panels | PETG or PLA+ | Light structural load, cosmetic accuracy matters |
| 02–03 | Head assembly | PLA+ | Low stress, dimensional accuracy for servo fits |
| 04–05 | Arm structure + gripper connector | PETG | Repeated torsional loads on arm links |
| 06 | Gripper fingers / gear horns | PETG | Wear surface on gear teeth |
| 07 | Suspension connectors (rocker-bogie) | PETG or ASA | High stress pivot joints — do NOT use PLA |
| 08 | Wheel hub small parts | PETG | Axle loads |
| 09, 11 | Wheel outer skins | **TPU 95A** (flex) or PETG (rigid) | TPU gives Mars 2020-style grip on outdoor terrain |
| 10, 12 | Wheel inner shells | PETG | Structural backbone of wheel |
| 13–17 | Rover core structure | PETG | Frame members carry most structural load |

---

## General Slicer Settings (starting point — tune per printer)

| Parameter | Value | Notes |
|---|---|---|
| Layer height | 0.2 mm | 0.15 mm for servo horn mating surfaces |
| Line width | 0.4 mm (nozzle) | |
| Infill — structural parts | 40% gyroid | Rocker-bogie pivots, arm links, wheel hubs |
| Infill — cosmetic/light parts | 20% gyroid | Panels, head housing |
| Perimeters / walls | 4 | Minimum for all structural parts |
| Top/bottom layers | 5 | |
| Print speed | 50 mm/s | Slow to 30 mm/s for first layer and bridges |
| Supports | See per-pack notes | Use tree/organic supports where possible |
| Bed adhesion | Brim 5 mm | For tall narrow parts; raft for TPU |

---

## TPU-Specific Settings (wheel outer skins — packs 09 & 11)

| Parameter | Value |
|---|---|
| Print speed | 20–25 mm/s |
| Retraction | Disabled or ≤0.5 mm (direct drive) / Off (Bowden) |
| Temperature | ~230°C nozzle, ~45°C bed |
| Cooling fan | 100% after layer 3 |
| Infill | 15% gyroid — gives flex behaviour |
| Walls | 3 |
| Bed adhesion | PEI sheet or glue stick on glass |

---

## Pack-Specific Notes

### Pack 07 — Suspension Connectors
- `main_leg_body_connector` and its mirror are the single highest-stress printed parts.
  Orient so layer lines run along the length of the arm (perpendicular to the load).
- If using PETG, anneal in oven at 65°C for 2 hrs after printing for improved creep resistance.

### Pack 09 / 10 — Corner Wheels (×4 each)
- Body1 (~4 MB) is a large print — validate fit with a single copy before queuing all 4.
- Print corner wheel outer and inner as separate jobs; glue or snap together after.
- Wheel outer needs 4 copies; mirror is handled in hardware (symmetric wheel), not by mirroring the STL.

### Pack 11 / 12 — Middle Wheels (×2 each)
- Same approach as corner wheels — validate one before printing the second.

### Pack 13 — Core Structure Large Bodies
- Body6 and Body33 are each ~4 MB — likely main chassis frame rails. Verify print bed fits.
- Best orientation: flat face down, no supports on inner channels if hollowed.

### Pack 17 — Body46 ⚠️
- Body46 is 7.3 KB — very small mesh. Open in CAD before printing; may be a hardware reference
  that was not caught in the context-only pass.

---

## Print Order Recommendation

Print in this dependency order so you can begin dry-fitting before all parts are done:

1. Packs 13–17 (core structure) — longest print time, needed first for chassis assembly
2. Pack 07 (suspension connectors)
3. Pack 08 + 09 + 10 (wheel hubs + corner wheel shells) × 4
4. Pack 11 + 12 (middle wheel shells) × 2
5. Pack 01 (chassis body panels)
6. Pack 04 + 05 + 06 (arm structure + gripper)
7. Packs 02 + 03 (head)
