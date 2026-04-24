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
  Mars-Rover/        ← git submodule — fork of jakkra/Mars-Rover (ESP32 firmware + CAD)
  ardurover/
    params/          ← ArduRover .param files
    docs/            ← calibration and setup guides
  docs/
    bom.md
    print-settings.md
    assembly/
    wiring-diagram/
    gcs-setup.md
  TODO.md
  README.md
```

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
