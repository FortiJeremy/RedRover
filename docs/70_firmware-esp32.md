# ESP32 Firmware

The ESP32 firmware lives in `Mars-Rover/src/` and is built with PlatformIO
targeting the **TTGO LoRa32 v1** (Espressif ESP32, Arduino framework).

---

## Current Architecture (upstream Mars-Rover)

The firmware runs on a single ESP32 and is the sole controller for the entire rover.
It handles RC input, WiFi/LoRa control, motor driving, servo control, IMU, and telemetry.

### Module overview

| File | Responsibility |
|---|---|
| `main.cpp` | Setup, main loop, mode switching, controller arbitration |
| `rover_config.h` | All pin assignments, RC ranges, servo trim offsets, constants |
| `rover_driving.cpp/.h` | ESC PWM output (left/right), steering servo, drive modes |
| `rover_servo.cpp/.h` | PCA9685 I2C servo driver — all 13 servo channels |
| `arm.cpp/.h` | 6-DOF arm axis movement with speed ramping (uses `Ramp` library) |
| `rover_head.cpp/.h` | Head yaw/pitch servo control |
| `rc_receiver_rmt.cpp/.h` | 6-channel RC PWM input via ESP32 RMT peripheral |
| `wifi_controller.cpp/.h` | Async WebSocket server + UDP; AP or STA mode |
| `lora_controller.cpp/.h` | LoRa radio TX/RX via arduino-LoRa library |
| `gyro_accel_sensor.cpp/.h` | MPU-6050 I2C IMU — reads accel/gyro/angle, runs in FreeRTOS task |
| `rover_settings_switch.cpp/.h` | Reads 2-position GPIO switches at boot to select startup mode |
| `switch_checker.cpp/.h` | RoverMode / ArmMode enum and debounce logic |
| `data/index.html` | Web UI (virtual joystick) — uploaded to SPIFFS via `pio run -t uploadfs` |
| `data/virtualjoystick.js` | Joystick library for web UI |

### Control arbitration

Three input sources compete for control. Priority order (highest wins):
1. **WiFi WebSocket** — overrides RC and LoRa when a client connects
2. **LoRa** — overrides RC when LoRa packets received and WiFi not connected
3. **RC receiver** — default fallback

### Drive modes (RoverMode)

Controlled by RC CH5 switch position:
- `DRIVE_TURN_NORMAL` — both ESCs same signal; corner servos steer normally
- `DRIVE_TURN_SPIN` — left/right ESCs run opposite directions for pivot turn

### Arm modes (ArmMode)

Controlled by RC CH6 switch position:
- `ARM_MODE_MOVE` — joysticks drive arm axes 1–4 (CH1–CH4)
- `ARM_MODE_MOVE_GRIPPER` — joysticks drive axes 5–6 (gripper)
- Direct axis control — no IK. **IK is listed as TODO in the original codebase.**

### Key constants (rover_config.h)

```c
RC_LOW    = 1000   // µs
RC_CENTER = 1500   // µs
RC_HIGH   = 2000   // µs
RC_FILTER_SAMPLES = 2          // sliding average filter window

SERVOMIN      = 250            // PCA9685 counts (normal range)
SERVOMAX      = 500
SERVOMIN_FULL_RANGE = 150      // extended range
SERVOMAX_FULL_RANGE = 600

SERVO_FRONT_LEFT_OFFSET  = +20
SERVO_FRONT_RIGHT_OFFSET = -15
SERVO_BACK_LEFT_OFFSET   = +5
SERVO_BACK_RIGHT_OFFSET  = -25
```

### Libraries (platformio.ini)

```ini
lib_deps =
    ESP32Servo           ; ESC and direct servo PWM
    Ramp                 ; Smooth servo speed ramping in arm.cpp
    MPU6050_tockn        ; MPU-6050 IMU driver
    Adafruit PWM Servo Driver Library  ; PCA9685 I2C servo expander
    AsyncTCP             ; Required by ESP Async WebServer
    ESP Async WebServer  ; WebSocket + HTTP server for web UI
    arduino-LoRa (sandeepmistry)  ; LoRa radio
```

---

## RedRover Refactor Plan

The ESP32 is demoted from sole controller to peripheral/companion.
ArduRover FC handles all motor/ESC control and navigation; ESP32 manages
servos (arm, head, steering) and acts as a MAVLink bridge.

### Changes required

#### 1. Remove direct ESC control
- `rover_driving.cpp`: disable/stub out `motors_left` and `motors_right` ESC PWM output
- GPIO 17 and 13 no longer used for ESC signal
- `rover_driving_move()` to be replaced with MAVLink RC override or removed entirely

#### 2. Remove direct RC receiver input for motion
- RC receiver SBUS/PPM physically rewired to FC RC IN port
- The 6 RC GPIO inputs (34, 35, 4, 39, 25, 36) may be repurposed or left as monitoring-only
- Arm and head RC channels can still be read locally on ESP32, or forwarded via MAVLink

#### 3. Add MAVLink serial link to FC
- New UART on ESP32 (e.g. `Serial2`) connected to FC TELEM2 port
- Add `mavlink` library to `platformio.ini`
- Implement heartbeat TX (Type: `MAV_TYPE_ONBOARD_CONTROLLER`)
- Parse incoming heartbeat + `RC_CHANNELS_OVERRIDE` from FC

#### 4. Retain servo control
- PCA9685 I2C bus and all 13 servo channels remain on ESP32
- `rover_servo.cpp` unchanged — still drives arm (axes 1–6), steering (4 corners), head (yaw + pitch)
- Arm mode / head control still driven by WiFi WebSocket or LoRa

#### 5. Telemetry bridge (optional)
- ESP32 reads MAVLink telemetry from FC (battery, GPS, mode, heading)
- Forwards over WebSocket to browser or custom GCS web UI
- Existing `wifi_controller.cpp` WebSocket infrastructure can be reused

#### 6. Arm IK (future)
- `arm.cpp` currently maps RC channels 1:1 to axis µs positions
- Plan: implement FABRIK or analytical 6-DOF IK
- IK target (x,y,z + euler) accepted via WebSocket JSON message or MAVLink custom message
- `arm_move()` already has speed parameter — reuse for IK interpolation

---

## Build & Flash

```bash
# Build
pio run

# Flash
pio run -t upload

# Monitor serial
pio device monitor

# Upload web UI to SPIFFS
pio run -t uploadfs

# Decode crash stack trace
xtensa-esp32-elf/bin/xtensa-esp32-elf-addr2line.exe -pfiaC -e .pio/build/esp-wrover-kit/firmware.elf 0x...
```

Default port: `COM8` at 921600 baud upload / 115200 baud monitor (set in `platformio.ini`).
Update `upload_port` / `monitor_port` in `platformio.ini` if your port differs.
