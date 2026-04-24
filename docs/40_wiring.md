# Wiring & Electrical

Wiring reference for the RedRover. The original Mars-Rover was a single-board ESP32 system;
the RedRover adds an ArduRover flight controller between the battery/ESCs and the ESP32,
which becomes a peripheral/companion board.

---

## System Architecture Overview

```
Battery (3S LiPo)
    │
    ├─ XT60 ──► Power Module ──► (VBAT + current sense to FC ADC)
    │                │
    │           PDB / Bus
    │           │          │          │          │
    │         ESC L       ESC R     BEC 1      BEC 2/3
    │       (Motors       (Motors   (5V servo  (5V for FC,
    │        L×3)          R×3)      rail)      ESP32, LiDAR)
    │
    └─ Fuse (30–40A blade)
```

---

## ESP32 (TTGO LoRa32 v1) Pin Assignment

From `src/rover_config.h`:

### RC Receiver Inputs (still wired direct to ESP32 in current firmware; to be rerouted to FC in RedRover)

| ESP32 GPIO | Signal | RC Channel |
|---|---|---|
| 34 | RC_CH1_INPUT | Steering |
| 35 | RC_CH2_INPUT | Throttle |
| 4 | RC_CH3_INPUT | Head Yaw |
| 39 | RC_CH4_INPUT | Head Pitch |
| 25 | RC_CH5_INPUT | Rover mode switch |
| 36 | RC_CH6_INPUT | Arm mode switch |

> **RedRover change:** RC receiver SBUS/PPM output will move to ArduRover FC RC IN port.
> ESP32 will receive RC channel values via MAVLink RC override messages from FC.

### Motor ESC Outputs (to be removed in RedRover — FC takes over)

| ESP32 GPIO | Signal |
|---|---|
| 17 | ESC left (3× motors) — PWM 1000–2000 µs |
| 13 | ESC right (3× motors) — PWM 1000–2000 µs |

> **RedRover change:** These pins will be disabled. ArduRover FC outputs drive both ESCs
> directly on its PWM output channels.

### I2C Bus

| ESP32 GPIO | Signal |
|---|---|
| 21 | SDA |
| 22 | SCL |

I2C devices on this bus:
- **PCA9685** PWM servo driver (13 servo channels — see servo channel table below)
- **MPU-6050** IMU (gyro/accel)

### Settings Switches (boot mode selection)

| ESP32 GPIO | Signal |
|---|---|
| 2 | ROVER_SETTINGS_SWITCH_1 |
| 23 | ROVER_SETTINGS_SWITCH_2 |

3-position switch combination controls startup mode:
- `ROVER_SWITCH_STATE_STATION_LORA` → WiFi station + LoRa enabled
- `ROVER_SWITCH_STATE_AP` → WiFi AP mode (hosted web UI)
- `ROVER_SWITCH_STATE_LORA_ONLY` → LoRa only, no WiFi

---

## PCA9685 Servo Driver Channel Map

All servos are driven through a single PCA9685 I2C PWM expander.
From `src/rover_servo.h`:

| PCA9685 Channel | RoverServoId | Connected servo |
|---|---|---|
| 0 | `SERVO_ARM_AXIS_1` | Arm base rotation |
| 1 | `SERVO_ARM_AXIS_2` | Arm shoulder |
| 2 | `SERVO_ARM_AXIS_3` | Arm elbow |
| 3 | `SERVO_ARM_AXIS_4` | Arm wrist pitch |
| 4 | `SERVO_ARM_AXIS_5` | Arm wrist roll |
| 5 | `SERVO_ARM_AXIS_6` | Gripper open/close |
| 6 | `SERVO_FRONT_RIGHT` | Front-right corner steering |
| 7 | `SERVO_BACK_LEFT` | Back-left corner steering |
| 8 | `SERVO_FRONT_LEFT` | Front-left corner steering |
| 9 | `SERVO_BACK_RIGHT` | Back-right corner steering |
| 10 | `SERVO_UNUSED` | Spare |
| 11 | `SERVO_HEAD_YAW` | Head left/right rotation |
| 12 | `SERVO_HEAD_PITCH` | Head up/down tilt |

**PWM range (from `rover_config.h`):**
- Default: SERVOMIN = 250, SERVOMAX = 500 (out of 4096)
- Full range: SERVOMIN_FULL_RANGE = 150, SERVOMAX_FULL_RANGE = 600
- Standard RC centre = 1500 µs

---

## ArduRover Flight Controller Wiring (RedRover additions)

### Serial / UART Connections

| FC Port | Connected to | Protocol | Baud |
|---|---|---|---|
| GPS1 (UART) | GPS/compass module | u-blox UBX + NMEA | 38400 or 115200 |
| TELEM1 (UART) | SiK telemetry radio | MAVLink 2 | 57600 |
| TELEM2 (UART) | ESP32 UART (TX/RX) | MAVLink 2 | 115200 |
| SERIAL5 (or spare) | RPLidar A1 UART | LIDAR serial | 115200 |
| RC IN | RC receiver SBUS/PPM | SBUS / PPM | — |
| Safety SW | Arming button | GPIO | — |
| ADC/Power | Power module | Analog | — |

### FC PWM Output Channels

| FC Output | Connected to | ArduRover function |
|---|---|---|
| CH1 | ESC Left (3× motors) | Throttle left (skid-steer) |
| CH3 | ESC Right (3× motors) | Throttle right (skid-steer) |
| CH2 | Front steering servo pair | Steering output |
| CH4 | Rear steering servo pair | Steering output |

> Exact channel assignments will depend on FC model and servo output configuration.
> Configure via `SERVO1_FUNCTION`, `SERVO3_FUNCTION` etc. in ArduRover params.

### I2C Compass

- GPS module compass connected to FC I2C compass port (separate from ESP32 I2C bus)

---

## Power Rail Summary

| Rail | Source | Consumers |
|---|---|---|
| 12V main | Battery → PDB | Both ESCs, BEC inputs |
| 5V servo rail | BEC 1 (5A min) | All 13 servos via PCA9685 |
| 5V logic rail | BEC 2 | FC, ESP32, RPLidar, RC receiver |
| 5V (FC internal) | FC BEC or USB | FC processor only |

**Fusing:**
- Main: 30–40A blade fuse between battery and PDB
- Servo rail: polyfuse or 5A blade fuse per BEC output recommended
- Motor circuits: fused through ESC internal protection

---

## Wiring Diagram

> ⚠️ Full annotated schematic TBD — to be drawn in KiCad or draw.io and exported as SVG/PDF.
> Target: `docs/wiring-diagram/redrover-full-schematic.svg`
