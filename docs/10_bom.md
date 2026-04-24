# Bill of Materials

---

## Original Mars-Rover Hardware

These are the parts from the upstream jakkra/Mars-Rover design.
Quantities are for the full rover with arm and head.

### Drive System

| Part | Qty | Notes |
|---|---|---|
| 12V 60RPM DC motor | 6 | One per wheel — see [issue #6](https://github.com/jakkra/Mars-Rover/issues/6) for specific model |
| Brushless ESC | 2 | One per side (each drives 3 motors in parallel) |
| Pololu 4mm shaft hub (M3 holes) | 6 | [Pololu #1081](https://www.pololu.com/product/1081) — motor-to-wheel adapter |

### Steering & Servos

| Part | Qty | Notes |
|---|---|---|
| MG946R or MG996R servo | 4 | Corner wheel steering |
| MG946R servo | 6 | 6-DOF arm axes |
| MG946R or MG996R servo | 2 | Head yaw + pitch |
| Round Metal Servo Horn 25T Disc | 12 | One per servo — check spline compatibility |

### Power

| Part | Qty | Notes |
|---|---|---|
| 3S LiPo battery | 1 | Capacity TBD based on runtime target — suggest 5000–6000 mAh |
| 12V → 5V switching voltage regulator | 12 | One per servo — can be replaced with high-current BECs (see RedRover additions) |

### Compute & Control

| Part | Qty | Notes |
|---|---|---|
| TTGO LoRa32 v1 (ESP32) | 1 | Main MCU — board used in platformio.ini |
| Adafruit PCA9685 PWM servo driver | 1 | I2C servo expander — drives all 13 servos on channels 0–12 |
| MPU-6050 IMU (gyro/accel) | 1 | On I2C bus with PCA9685 |
| 6-channel RC receiver | 1 | Generic PPM/PWM receiver |
| LoRa radio module | 1 | Built into TTGO LoRa32 (SX1276 or SX1278) |

### Structural

| Part | Qty | Notes |
|---|---|---|
| PVC pipe, ID 23.40 mm / OD 25 mm | TBD | Rocker-bogie arms — measure from CAD |
| Ball bearing 608ZZ | 5+ | Check CAD for exact count per side |
| Bearing SKF 6005 (25 mm) | 5+ | Larger wheel pivot bearings — check CAD |
| M3 screws assorted | ~100 | Primary fastener throughout |
| M4 screws assorted | ~20 | Larger structural joints |
| M3 heat-set inserts | ~50 | For 3D-printed holes — strongly recommended over direct screw-in |

---

## RedRover Additions (ArduRover / GPS / LiDAR build)

### Autopilot

| Part | Qty | Notes |
|---|---|---|
| ArduRover-compatible flight controller | 1 | **Recommended:** Pixhawk 6C, Matek H743-Wing, or CubePilot Orange+ |
| Vibration isolation foam / mounting kit | 1 | Required under FC — standard FC mounting foam |

### Navigation

| Part | Qty | Notes |
|---|---|---|
| u-blox M8N or M9N GPS + compass module | 1 | **Recommended:** Here3 or mRo GPS — integrated compass |
| GPS mast (printed or purchased) | 1 | Elevates GPS above chassis electronics noise |

### Collision Avoidance

| Part | Qty | Notes |
|---|---|---|
| RPLidar A1M8 (360° 2D LiDAR) | 1 | **Recommended** — full perimeter coverage, ~$100, supported natively by ArduRover `PRX_TYPE=5` |
| *OR* TFMini-Plus (single-point ToF) | 1 | Budget alternative — forward-only, `RNGFND1_TYPE=20` |

### Telemetry

| Part | Qty | Notes |
|---|---|---|
| SiK 915 MHz radio pair | 1 pair | e.g. RFD900x or HolyBro SiK v3 — one on rover, one to GCS |

### Power

| Part | Qty | Notes |
|---|---|---|
| ArduRover power module | 1 | e.g. **Matek HUBOSD** or Holybro PM02 — provides VBAT + current sense to FC ADC |
| 5A BEC (12V → 5V) | 2–3 | e.g. Matek MBEC6S — replaces 12× individual regulators. One per servo rail zone |
| Power distribution board (PDB) | 1 | Routes 12V from battery to ESCs, BECs, FC, LiDAR |
| 30–40A blade fuse + holder | 1 | Main fuse between battery and PDB |
| XT60 connector pair | 2–3 | Battery and PDB connections |

### Safety

| Part | Qty | Notes |
|---|---|---|
| ArduRover safety arming button | 1 | Wired to FC safety switch port — required for arming |

---

## Connectors & Wiring Hardware

| Part | Notes |
|---|---|
| JST connectors assorted | ESC signal, servo extensions |
| Dupont 2.54mm header kit | ESP32 and FC GPIO breakouts |
| Heat-shrink tubing assorted | |
| Cable ties / wire loom | |
| XT30 or XT60 pigtails | ESC power inputs |

---

## Still TBD

- [ ] Exact DC motor model/SKU — see Mars-Rover issue #6
- [ ] Cable loom lengths — measure from CAD / physical assembly
- [ ] ESC model selection — confirm PWM range compatible with ArduRover output
- [ ] 3S LiPo C-rating and capacity — size to current budget once calculated
- [ ] Heat-set insert sizes per part — note during assembly
