# ArduRover Setup

Configuration guide for the ArduRover autopilot on the RedRover.
ArduRover replaces the ESP32 as the motion controller for drive and navigation;
see [70_firmware-esp32.md](70_firmware-esp32.md) for how the ESP32 is refactored to work alongside it.

Parameter files live in `ardurover/params/`.

---

## Hardware Prerequisites

Before configuring ArduRover, the following must be physically connected:

- [ ] Flight controller mounted on vibration isolation foam, arrow pointing forward
- [ ] GPS/compass module connected to FC GPS1 UART + I2C compass port
- [ ] RC receiver connected to FC RC IN (SBUS or PPM)
- [ ] Both ESCs signal wires routed to FC PWM output channels
- [ ] SiK telemetry radio connected to FC TELEM1
- [ ] ESP32 UART connected to FC TELEM2
- [ ] RPLidar UART connected to FC SERIAL5 (or appropriate spare serial port)
- [ ] Power module connected between battery and FC power input

---

## Initial Connection

1. Connect SiK radio to GCS computer (see [60_gcs-setup.md](60_gcs-setup.md))
2. Open Mission Planner → Connect → select SiK COM port → 57600 baud
3. Flash latest ArduRover stable firmware via Mission Planner **Initial Setup → Install Firmware**
   - Select "Rover" vehicle type
4. Perform mandatory hardware calibration (see below) before first parameter load

---

## Mandatory Calibration Sequence

Complete all of these before loading the baseline param file.

### 1. Accelerometer calibration
Mission Planner → Initial Setup → Mandatory Hardware → Accel Calibration
- Follow on-screen 6-position calibration (flat, left, right, nose-up, nose-down, back)

### 2. Compass calibration
Mission Planner → Initial Setup → Mandatory Hardware → Compass
- Select external compass (GPS module compass), disable internal FC compass if present
- Run onsite calibration — rotate rover through all axes until bars fill

### 3. RC calibration
Mission Planner → Initial Setup → Mandatory Hardware → Radio Calibration
- Move all sticks and switches to extremes
- Verify channel assignments match expected (Throttle = CH2, Steering = CH1)

### 4. ESC calibration
Mission Planner → Optional Hardware → Motor Test
- Set `MOT_PWM_MIN` and `MOT_PWM_MAX` to match ESC endpoints
- Run motor test at 5–10% to verify correct rotation direction
- Swap ESC signal wires if motor direction is wrong

---

## Baseline Parameter File

`ardurover/params/redrover_base.param` — TBD, load after calibrations are saved.

Key parameter groups to configure:

### Frame type (skid-steer)
```
SERVO1_FUNCTION = 73    ; throttle left
SERVO3_FUNCTION = 74    ; throttle right
```
> Adjust channel numbers to match your physical wiring. For Ackermann (independent
> corner steering), use `SERVO1_FUNCTION=26` (steering) + `SERVO3_FUNCTION=70` (throttle).

### GPS
```
GPS_TYPE  = 1           ; u-blox auto-detect
GPS_NAVFILTER = 8       ; automotive (good default for ground vehicle)
GPS_HDOP_GOOD = 140     ; HDOP threshold for arming (1.4 = tight, loosen if needed outdoors)
SERIAL3_PROTOCOL = 5    ; GPS on SERIAL3 (adjust port number per FC)
SERIAL3_BAUD = 38        ; 38400 baud
```

### Compass
```
COMPASS_USE    = 1
COMPASS_USE2   = 0       ; disable internal compass if external present
COMPASS_ORIENT = 0       ; adjust if GPS module is not mounted arrow-forward
```

### Battery monitor
```
BATT_MONITOR   = 4       ; voltage + current via power module
BATT_VOLT_PIN  = varies  ; check FC pinout
BATT_CURR_PIN  = varies
BATT_VOLT_MULT = varies  ; calibrate against known voltage
BATT_AMP_PERVLT = varies ; calibrate against known current
BATT_LOW_VOLT  = 10.5    ; 3S low voltage warning (3.5V/cell)
BATT_CRT_VOLT  = 10.2    ; 3S critical — trigger failsafe
```

### Telemetry
```
SERIAL1_PROTOCOL = 2    ; MAVLink 2 on TELEM1 (SiK radio)
SERIAL1_BAUD     = 57   ; 57600
SERIAL2_PROTOCOL = 2    ; MAVLink 2 on TELEM2 (ESP32)
SERIAL2_BAUD     = 115  ; 115200
```

### Failsafes
```
FS_ACTION    = 2         ; RTL on RC failsafe
FS_TIMEOUT   = 1.5       ; seconds before failsafe triggers
BATT_FS_VOLTAGE_ACTION = 2  ; RTL on low battery
```

---

## LiDAR / Proximity Sensor Setup (RPLidar A1)

```
PRX_TYPE     = 5         ; RPLidar
PRX_ORIENT   = 0         ; 0 = forward, adjust if rotated
PRX_YAW_CORR = 0         ; yaw offset correction in degrees
SERIAL5_PROTOCOL = 11   ; proximity sensor on SERIAL5 (adjust port)
SERIAL5_BAUD = 115       ; 115200
```

Collision avoidance:
```
AVOID_ENABLE  = 7        ; enable all avoidance
AVOID_MARGIN  = 2.0      ; stop/steer when obstacle within 2 m
AVOID_BEHAVE  = 1        ; 0=slide, 1=stop
```

### TFMini-Plus (alternative single-point)
```
RNGFND1_TYPE     = 20    ; TFMini
RNGFND1_MIN_CM   = 10
RNGFND1_MAX_CM   = 600
RNGFND1_ORIENT   = 0     ; forward-facing
SERIAL5_PROTOCOL = 9     ; rangefinder
SERIAL5_BAUD     = 115
```

---

## Navigation & Waypoint Tuning

### Waypoint following
```
WP_RADIUS    = 0.5       ; waypoint achieved within 0.5 m
WP_SPEED     = 1.0       ; m/s default cruise speed
```

### RTL (Return to Launch)
```
RTL_RADIUS   = 1.0       ; orbit radius at home point
TURN_MAX_G   = 0.5       ; max lateral G in turns
```

### Steering rate controller (tune after basic driving works)
```
ATC_STR_RAT_MAX  = 120   ; degrees/sec max turn rate
ATC_STR_RAT_P    = 0.2   ; starting proportional gain
ATC_STR_RAT_I    = 0.1
ATC_STR_RAT_D    = 0.005
```

---

## SITL (Software In The Loop) Testing

Before using on hardware, validate parameter changes in ArduRover SITL.

### Setup (WSL or Linux VM)
```bash
# Install ArduPilot deps (Ubuntu/Debian)
sudo apt install git python3 python3-pip python3-dev
pip3 install mavproxy

# Clone ArduPilot
git clone --recurse-submodules https://github.com/ArduPilot/ardupilot.git
cd ardupilot
Tools/environment_install/install-prereqs-ubuntu.sh -y
. ~/.profile

# Run ArduRover SITL
sim_vehicle.py -v Rover --console --map
```

### Connect QGroundControl to SITL
- QGC → Application Settings → Comm Links → Add → UDP → port 14550
- SITL broadcasts MAVLink on UDP 14550 by default

---

## Geofence

```
FENCE_ENABLE  = 1
FENCE_TYPE    = 2        ; polygon
FENCE_ACTION  = 1        ; RTL on breach
FENCE_RADIUS  = 50       ; fallback circle fence radius in metres
```

Draw polygon fence in Mission Planner → Flight Plan → Fence before first autonomous test.
