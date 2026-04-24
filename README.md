# RedRover

A Curiosity/Perseverance-inspired rover built on the [Mars-Rover](https://github.com/jakkra/Mars-Rover)
mechanical and ESP32 firmware platform, extended with:

- **ArduRover** autopilot (Pixhawk / Matek / CubePilot) for full autonomous navigation
- **GPS + compass** for waypoint missions, RTL, and geo-fencing
- **LiDAR** (RPLidar A1 / TFMini-Plus) for proximity-based collision avoidance
- **SiK telemetry** radio link to Mission Planner / QGroundControl
- Refactored ESP32 firmware bridging MAVLink for arm, head, and auxiliary servo control

## Repo structure

```
RedRover/
  Mars-Rover/           ← git submodule — fork of jakkra/Mars-Rover (ESP32 firmware + CAD)
  3d-prints/            ← STL files exported from the Fusion 360 full assembly
    _context_only/      ← reference-only meshes (bearings, servos, motors, fasteners) — DO NOT PRINT
    01_chassis_body/    ← outer hull panels + electronics bay lid
    02_head_pack1/      ← head rotator, housing, camera mount, servo horns
    03_head_pack2/      ← large head top connector parts
    04_arm_structure/   ← arm links 1–3 and body connectors
    05_arm_gripper_connector/  ← gripper connector, rotator, printed servo horns
    06_gripper_fingers/ ← gear horns, finger arms, plates, rods
    07_suspension_connectors/  ← rocker-bogie pivot connectors (left + right)
    08_wheel_hub_smalls/ ← hub faces, axle flanges, steering knuckle small bodies
    09_wheel_corner_A/  ← corner wheel outer shell (×4)
    10_wheel_corner_B/  ← corner wheel inner shell / TPU candidate (×4)
    11_wheel_middle_A/  ← middle wheel outer skin / TPU candidate (×2)
    12_wheel_middle_B/  ← middle wheel inner body (×2)
    13_core_structure_A/ ← rover core body parts — large singles
    14_core_structure_B/ ← rover core body parts — batch B
    15_core_structure_C/ ← rover core body parts — batch C
    16_core_structure_D/ ← rover core body parts — batch D
    17_core_structure_E/ ← rover core body parts — batch E (includes Body46 ⚠️ verify before printing)
  ardurover/
    params/             ← ArduRover .param files
    docs/               ← calibration and setup guides
  docs/
    10_bom.md              ← full bill of materials (original + RedRover additions)
    20_print-settings.md   ← slicer settings and material guide per pack
    30_assembly.md         ← chapter-by-chapter assembly guide
    40_wiring.md           ← pin assignments, servo channel map, power rail overview
    50_ardurover-setup.md  ← ArduRover parameter setup, calibration, LiDAR, geofence
    60_gcs-setup.md        ← Mission Planner / QGC connection and field checklist
    70_firmware-esp32.md   ← ESP32 firmware architecture and MAVLink refactor plan
  TODO.md
  README.md
```

### 3D prints overview

295 STL files were exported from the Fusion 360 assembly. **156 files** are purchased-part
reference geometry (Ball Bearing 608ZZ, SKF 6005, Servo MG996R bodies, DC motors, McMaster
fasteners, Pololu hub) — these live in `_context_only/` and should **never** be sliced.
The remaining **139 files** are printable parts, organized into 17 build packs each under 10 MB
raw (slicer project files add overhead on top of the STL sizes).

> **Note on servo horns:** the `servo_and_horn v2 (N)_1_Body1.stl` files are the 3D-printed
> servo horns — they are in the build packs, not `_context_only/`, even though their sibling
> `Servo MG996R v2_*_Body*.stl` files are context-only.

See [TODO.md — Priority 5](TODO.md) for the full step-by-step organization plan and remaining tasks.

## Getting started

Clone with submodules:

```bash
git clone --recurse-submodules git@github.com:FortiJeremy/RedRover.git
```

If you already cloned without `--recurse-submodules`:

```bash
git submodule update --init --recursive
```

## Related repos

| Repo | Purpose |
|---|---|
| [FortiJeremy/Mars-Rover](https://github.com/FortiJeremy/Mars-Rover) | ESP32 peripheral firmware + Fusion 360 CAD (submodule) |
| [jakkra/Mars-Rover](https://github.com/jakkra/Mars-Rover) | Upstream source |
| [jakkra/RoverController](https://github.com/jakkra/RoverController) | Original handheld controller |

## Build status

Early build phase — see [TODO.md](TODO.md) for full scope.
