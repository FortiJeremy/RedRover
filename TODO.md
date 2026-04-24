# RedRover — Project TODO

A Curiosity/Perseverance-inspired rover, forked from [jakkra/Mars-Rover](https://github.com/jakkra/Mars-Rover),
rebuilt with ArduRover, GPS, and LiDAR-based collision avoidance.

---

## Priority 1 — Print & Assembly

- [ ] **Bill of Materials (BOM)**
  - Enumerate every 3D-printed part from the Fusion 360 model, with quantity and suggested material (PLA/PETG/TPU)
  - List all off-the-shelf hardware: M3/M4 fasteners, 608ZZ & SKF 6005 bearings, PVC pipe (ID 23.40 mm / OD 25 mm), motor shaft adapters (Pololu #1081), servo horns
  - Add all electronics (see Electrical section below) to the BOM with supplier links

- [ ] **Slicer settings document**
  - Recommended layer height, infill % and pattern per part category (structural vs. cosmetic)
  - Identify parts that require supports, brim, or special orientation
  - Flag TPU wheel outer skin print settings separately (print speed, retraction, bed adhesion)
  - Note which parts are mirrored vs. unique

- [ ] **Print sequence / dependency order**
  - Order parts by sub-assembly so printing isn't blocked by missing pieces
  - Flag long prints that should run overnight

- [ ] **Assembly guide — Rocker-Bogie suspension**
  - Step-by-step with reference screenshots pulled from the Fusion 360 model
  - Bearing press-fit tolerances and notes on reaming holes if needed
  - Rocker-bogie pivot alignment procedure

- [ ] **Assembly guide — Drive train**
  - Motor mounting and shaft-coupler installation
  - ESC bracket placement and strain-relief routing
  - Corner steering servo installation and centering procedure

- [ ] **Assembly guide — Arm (6-DOF)**
  - Joint assembly order
  - Servo horn drilling / tapping guide
  - Cable management through hollow links

- [ ] **Assembly guide — Head**
  - Yaw/pitch servo installation
  - Camera/sensor mounting provisions

- [ ] **Assembly guide — Electronics bay**
  - Panel layout diagram (flight controller, ESP32, power distribution, telemetry, GPS, LiDAR)
  - Lid fitment and access hatch notes

- [ ] **Wiring diagram**
  - Full schematic: battery → power module → PDB → ESCs, servo rail, 5 V regulators, flight controller, ESP32
  - Annotated connector callouts (XT60, JST, Dupont, RJ45 for telemetry)

---

## Priority 2 — Electrical (Hardware Gaps)

> The original design runs everything from a single ESP32. Adding ArduRover requires a dedicated
> autopilot board and supporting electronics.

- [ ] **Flight controller (ArduRover host)**
  - Select and source autopilot: Pixhawk 6C / Matek H743-Wing / CubePilot Orange+ (recommended for reliability)
  - Mount inside electronics bay with vibration isolation foam

- [ ] **GPS / compass module**
  - u-blox M8N or M9N with integrated compass (e.g., Here3, mRo GPS)
  - Design/print elevated GPS mast to reduce airframe interference
  - Wire: UART to ArduRover GPS1 port, I2C compass to I2C bus

- [ ] **LiDAR sensor for collision avoidance**
  - Select sensor: RPLidar A1M8 (360° scan, ~$100) or TFMini-Plus (single-point, cheaper)
    - RPLidar preferred for full-perimeter avoidance via ArduRover `PRX_TYPE` proximity framework
  - Power: 5 V regulated supply, ~450 mA peak
  - Interface: UART to autopilot serial port (or USB→UART to companion if RPLidar)
  - Mount: top deck centered for unobstructed 360° view (RPLidar) or front-facing (TFMini)

- [ ] **Telemetry radio**
  - SiK 915 MHz radio pair (RFD900x or HolyBro SiK v3) — one on rover, one on GCS laptop/tablet
  - Keeps the existing LoRa link for ESP32 peripheral comms (separate frequency/protocol)
  - Wire to ArduRover TELEM1 port (UART)

- [ ] **Power module / battery monitoring**
  - ArduRover-compatible power module (e.g., Matek HUBOSD or Holybro PM02) between 3S LiPo and PDB
  - Provides VBAT + current sense to autopilot ADC — needed for low-voltage failsafe and telemetry

- [ ] **Power distribution board (PDB)**
  - Route 12 V from battery to: both drive ESCs, servo power rail (via 12 V→5 V BECs), autopilot 5 V rail, RPLidar 5 V, ESP32 5 V
  - Replace the 12× individual 12 V→5 V regulators with 2–3 high-current BECs (e.g., Matek MBEC6S 5 A each) to reduce wiring complexity

- [ ] **ESC integration into autopilot**
  - Wire both brushless ESC signal wires to ArduRover PWM output channels (CH1 = left, CH3 = right, or skid-steer config)
  - Calibrate ESC endpoints via ArduRover motor test

- [ ] **Steering servo integration into autopilot**
  - Wire 4 corner steering servos to ArduRover PWM output channels
  - Determine if Ackermann or independent corner steering mode is used in ArduRover

- [ ] **ESP32 role redefinition**
  - ESP32 (TTGO LoRa32) demoted to companion/peripheral role
  - Connects to autopilot via UART (MAVLink passthrough) or acts as WiFi telemetry bridge
  - Arm, head, and auxiliary servos remain on ESP32 / PCA9685 servo driver
  - Remove direct motor/ESC control from ESP32 firmware

- [ ] **RC receiver wiring**
  - Route RC receiver SBUS/PPM output to ArduRover RC IN port (was wired directly to ESP32 GPIO)
  - ESP32 can still sniff RC channels via MAVLink RC override if needed

- [ ] **Arming switch / safety switch**
  - Add physical arming button wired to ArduRover safety switch port (required for ArduRover arming sequence)

- [ ] **Current budget and fuse sizing**
  - Document peak current draw for all loads
  - Size main fuse (recommend blade fuse holder, 30–40 A)
  - Add per-rail polyfuses or blade fuses for servo rails

---

## Priority 3 — Firmware & Software (Programmatic Gaps)

> The current codebase is a self-contained ESP32 firmware. ArduRover replaces the motion control
> layer; the ESP32 becomes a peripheral. A ground control station (GCS) replaces the custom web UI
> for autopilot functions.

### ArduRover Configuration

- [ ] **Initial ArduRover parameter file**
  - Create `ardurover/params/redrover_base.param` with baseline settings:
    - Frame type: skid-steer (`SERVO1_FUNCTION=73`, `SERVO3_FUNCTION=74`) or Ackermann
    - GPS, compass, AHRS orientation
    - Telemetry baud rates
    - Battery monitor type and scale factors
  - Document parameter rationale inline as comments

- [ ] **ESC / motor calibration procedure**
  - Document ArduRover motor test steps and PWM min/max tuning

- [ ] **Steering geometry calibration**
  - Ackermann steer turn radius calculation from CAD dimensions
  - `ATC_STR_RAT_MAX`, `TURN_MAX_G` parameter tuning notes

- [ ] **Compass/GPS calibration procedure**
  - Live calibration steps in Mission Planner / QGroundControl
  - Notes on compass interference sources on the rover frame

### LiDAR / Collision Avoidance

- [ ] **ArduRover proximity sensor setup (RPLidar)**
  - Set `PRX_TYPE=5` (RPLidar) and configure serial port
  - Configure `PRX_ORIENT`, `PRX_YAW_CORR` for physical mounting offset
  - Enable object avoidance: `AVOID_ENABLE=7`, `AVOID_MARGIN`, `AVOID_BEHAVE`

- [ ] **LiDAR driver integration (if using companion computer)**
  - ROS2 / Python node to publish `/scan` topic and forward to ArduRover via MAVLink `DISTANCE_SENSOR` messages
  - Alternatively: direct UART from RPLidar to autopilot serial using ArduRover built-in driver (no companion needed for A1)

- [ ] **TFMini-Plus fallback (single-point)**
  - Configure as `RNGFND1_TYPE=20`, set `RNGFND1_MAX_CM`, `RNGFND1_MIN_CM`
  - Wire and test before full RPLidar integration

- [ ] **Geofence setup**
  - Define a test-area polygon geofence in Mission Planner
  - Set `FENCE_ACTION=1` (RTL on breach) for initial field testing

### GPS & Navigation

- [ ] **GPS driver and port assignment**
  - Set `GPS_TYPE=1` (u-blox auto-detect), assign to correct SERIAL port
  - Verify HDOP threshold for arming (`GPS_HDOP_GOOD`)

- [ ] **Waypoint mission test**
  - Create a simple 5-waypoint square loop mission in QGroundControl
  - Test AUTO mode with and without collision avoidance enabled

- [ ] **RTL (Return to Launch) tuning**
  - Set and test `RTL_RADIUS`, `WP_RADIUS`
  - Validate GPS home point lock before arming

### ESP32 Firmware Refactor

- [ ] **MAVLink integration on ESP32**
  - Add `mavlink` library to `platformio.ini`
  - Implement MAVLink heartbeat RX/TX on Serial2 (UART link to autopilot)
  - Forward arm/disarm commands and RC override channels if needed

- [ ] **Remove direct ESC/motor control from ESP32**
  - `rover_driving.cpp` motor PWM output → disabled or redirected to MAVLink RC override
  - Retain servo control for arm and head (still managed by ESP32 / PCA9685)

- [ ] **Telemetry bridge (optional)**
  - ESP32 bridges MAVLink telemetry over WiFi websocket to a browser-based GCS (e.g., MAVLink2Rest or custom)
  - Exposes heading, GPS position, battery, mode on existing web UI

- [ ] **Arm inverse kinematics (carried over TODO)**
  - Implement IK solver for 6-DOF arm (was explicitly TODO in original codebase)
  - Consider FABRIK or analytical 6-DOF solution
  - Expose IK target coordinates via RC channels or websocket command

- [ ] **Web UI overhaul**
  - Replace joystick-only page with a dashboard:
    - MAVLink mode selector (MANUAL / AUTO / GUIDED / RTL)
    - Battery voltage / current gauge
    - GPS fix status and position
    - LiDAR proximity ring visualization
    - Arm control panel (IK target input)

### Ground Control Station

- [ ] **GCS software selection and setup**
  - Install Mission Planner (Windows) or QGroundControl (cross-platform)
  - Connect via SiK telemetry radio
  - Document connection steps in a `docs/gcs-setup.md`

- [ ] **SITL (Software-in-the-Loop) test environment**
  - Set up ArduRover SITL on WSL or Linux VM for parameter validation before hardware
  - Document launch command and how to connect QGC to SITL

---

## Priority 4 — Documentation & Repository Structure

- [ ] **Folder structure for RedRover repo**
  - Decide what lives in the fork vs. a separate RedRover repo
  - Suggested layout:
    ```
    RedRover/
      Mars-Rover/          ← upstream fork (CAD + ESP32 firmware)
      ardurover/
        params/            ← .param files
        docs/              ← calibration & setup guides
      docs/
        bom.md
        print-settings.md
        assembly/
          suspension.md
          drivetrain.md
          arm.md
          electronics-bay.md
        wiring-diagram.pdf (or .svg)
        gcs-setup.md
      TODO.md              ← this file
    ```

- [ ] **Upstream sync strategy**
  - Decide on branch strategy: maintain a `redrover` branch in the fork, periodically rebase onto upstream `main`
  - Document in `CONTRIBUTING.md` what changes stay in-fork vs. go upstream

- [ ] **CHANGELOG / build log**
  - Running log of hardware decisions, part substitutions, and lessons learned

---

## Parking Lot / Future

- [ ] Camera feed (CSI or USB cam) streamed to GCS or web UI
- [ ] Companion computer (Raspberry Pi 5 / Jetson Orin Nano) for vision-based SLAM
- [ ] ROS2 integration for advanced mission execution
- [ ] Solar charging panel on lid for extended outdoor operation
- [ ] Weatherproofing / IP rating for electronics bay
- [ ] Custom antenna tracker using second LoRa + GPS module
- [ ] Simulation model export (Gazebo / Isaac Sim URDF from Fusion 360)
