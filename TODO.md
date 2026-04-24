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

---

## Priority 5 — 3D Print File Organization

> **Context:** `3d-prints/` contains **295 STL files totalling ~329 MB** exported from the full
> Fusion 360 assembly. Most of the bulk is high-poly CAD reference meshes of purchased parts
> (servos, bearings, motors) that must never be sent to a slicer. The actual printable count is
> roughly 80–100 files. Goal: quarantine context-only files, then sort printable parts into
> sub-assembly packs kept under **10 MB raw** (slicer project files add overhead on top).

### Step 1 — Quarantine context-only models (DO NOT PRINT) ✅

Create `3d-prints/_context_only/` and move the following in. None of these are printed — they are
purchased hardware reference geometry used in the assembly for spatial planning.

- [x] **Servo MG996R bodies** — every file matching `*Servo MG996R v2*Body[1-9]*.stl`
  (Body1 = housing ~2–3 MB, Body2 = case cover ~2–6 MB, Body3–7 = internals/screws ~28–840 KB each)
  Affects: rover core steering servos (×8), arm servos (servo_and_horn v2 instances ×4),
  head servos (×2). **Exception: the `servo_and_horn v2 (N)_1_Body1.stl` files are the
  3D-printed servo horn — keep those in their assembly packs below.**

- [x] **Gripper servo** — `*gripper_Assy_V3*Servo v1_1_Body1.stl` and `Body2.stl` (~257 KB, ~490 KB)

- [x] **Ball Bearing 608ZZ** — all three instances (`v1_1`, `v1_2`, arm `v1_3`),
  20 bodies each. The large ones (Body8 = 9.5 MB, Body9 = 9.3 MB, Body19 = 6.5 MB, Body20 = 7.7 MB)
  account for most of the 329 MB total. This group alone is ~150 MB.

- [x] **SKF 6005 25mm Bearings** — `*25mm Bearing SKF 6005*` (12 bodies, ~715 KB + 11 × ~139 KB)

- [x] **DC Motors** — `*DC-motor v10*Body1.stl` and mirror variant (~227 KB each)

- [x] **Pololu shaft hub** — `*pololu-universal-aluminum-mounting-hub*` (~1.2 MB, purchased part)

- [x] **McMaster fasteners** — `*90592A085*` (7 instances × ~3.3 MB each = ~23 MB) and
  `*90965A130*` (~50 KB) — these are hex bolt/nut reference models

- [ ] **Review before printing** — `*Rover core v232_1_Body46.stl` (7.3 KB — likely a tiny hardware
  reference). Cross-check any Rover core `Body*.stl` under 50 KB against the CAD before including
  in a print pack.

---

### Step 2 — Organize printable parts into print packs

Create sub-folders under `3d-prints/` as below. Each pack should be under ~10 MB raw STL.
After sorting, import each folder into your slicer as a project to add print profiles/supports.

> **Naming convention:** `NN_description/` with two-digit prefix for print order.

---

#### Pack 01 — Chassis Body Panels (~3.6 MB)
`3d-prints/01_chassis_body/`

- [ ] Move all `Rover body_Rover_Full_Body*.stl` (Body1, 3, 4, 5, 6, 7, 9, 10, 13, 14, 15, 16 — 12 files, ~3.1 MB)
- [ ] Move `Rover body_Rover_Full_Lid1.stl` (~592 KB)
- **Notes:** These are the outer hull plates and electronics bay lid. Likely PETG or PLA+. Check
  for living-hinge or snap features that may need specific orientation.

---

#### Pack 02 — Head Assembly Part 1 (~3.8 MB)
`3d-prints/02_head_pack1/`

- [ ] `*head_rotator_1_Body1/2/3.stl` (yaw rotator bracket, ~475 KB total)
- [ ] `*head_rotator_1_head_1_Body1/2/3/4.stl` (head housing, ~994 KB total)
- [ ] `*espcam_holder_1_Body1/2.stl` (camera mount, ~48 KB total)
- [ ] `*head_top_connector_1_Body1/2/3/4/5/6/9/10/11/15.stl` (smaller connector parts, ~2.3 MB)
- [ ] Move both `servo_and_horn v2 (4)_1_Body1.stl` and `v2 (5)_1_Body1.stl` here
  (the **printed** servo horns for head yaw + pitch, ~109 KB each)
- **Notes:** Head houses ESP32-CAM. Plan sensor mast provisions here for LiDAR/GPS if mounting on head.

---

#### Pack 03 — Head Assembly Part 2 (~2.2 MB)
`3d-prints/03_head_pack2/`

- [ ] `*head_top_connector_1_Body7.stl` (~1.3 MB — largest head part)
- [ ] `*head_top_connector_1_Body8.stl` (~881 KB)
- [ ] `*head_top_connector_1_Body13/14.stl` (~464 KB + ~202 KB)
- **Notes:** Body7 is the largest single head part — verify print orientation for overhang.

---

#### Pack 04 — Arm Structure (~4.5 MB)
`3d-prints/04_arm_structure/`

- [ ] `*arm_1_1_Body1.stl` (first arm link, ~1.1 MB)
- [ ] `*arm_1_1_body_connector_1_Body1.stl` (~852 KB)
- [ ] `*arm_body_connector_1_Body1.stl` (base to arm connector, ~348 KB)
- [ ] `*arm2_1_Body1.stl` and `arm2_1_Body2.stl` (second link, ~329 KB + ~229 KB)
- [ ] `*arm2(Mirror)(Mirror)_1_Body1.stl` and `Body2.stl` (mirror of above for other side, ~326 KB + ~228 KB)
- [ ] `*arm3_1_Body1.stl` (third arm link, ~1.2 MB)
- **Notes:** arm2 and its mirror may require different print orientations for layer strength.
  Consider printing arm links in PETG for toughness.

---

#### Pack 05 — Arm Gripper Connector & Servo Horns (~2.6 MB)
`3d-prints/05_arm_gripper_connector/`

- [ ] `*arm_gripper_connector_1_Body1.stl` (~874 KB)
- [ ] `*arm_gripper_connector_1_Body2.stl` (~374 KB)
- [ ] `*arm_gripper_connector_1_grip_rotator_1_Body1.stl` (~987 KB)
- [ ] `*arm_1_1_servo_and_horn v2 (1)_1_Body1.stl` (printed arm servo horn, ~109 KB)
- [ ] `*arm_gripper_connector_1_servo_and_horn v2 (2)_1_Body1.stl` (~108 KB)
- [ ] `*arm_gripper_connector_1_grip_rotator_1_servo_and_horn v2 (3)_1_Body1.stl` (~108 KB)
- **Notes:** The three `Body1` servo horn files here are the **only printable parts** in those
  servo_and_horn sub-assemblies — all sibling `Servo MG996R v2_*_Body*` files go to _context_only.

---

#### Pack 06 — Gripper Finger Assembly (~4.3 MB)
`3d-prints/06_gripper_fingers/`

- [ ] `*gripper_Assy_V3*gear_horn25T_L*Body1.stl` (~799 KB)
- [ ] `*gripper_Assy_V3*gear_horn25T_R*Body1.stl` (~307 KB)
- [ ] `*gripper_Assy_V3*lower_arm_L*Body1.stl` (~151 KB)
- [ ] `*gripper_Assy_V3*lower_arm_R*Body1.stl` (~151 KB)
- [ ] `*gripper_Assy_V3*lower_plate*Body1.stl` (~590 KB)
- [ ] `*gripper_Assy_V3*middle_plate*Body1.stl` (~595 KB)
- [ ] `*gripper_Assy_V3*rear_*plate_horn25T*Body1.stl` (~207 KB)
- [ ] `*gripper_Assy_V3*rod (X4)*_1_Body1.stl` through `_4_Body1.stl` (×4, ~494 KB each = ~1.98 MB)
- [ ] `*gripper_Assy_V3*upper_arm_L*Body1.stl` (~163 KB)
- [ ] `*gripper_Assy_V3*upper_arm_R*Body1.stl` (~163 KB)
- [ ] `*gripper_Assy_V3*upper_plate*Body1.stl` (~307 KB)
- **Notes:** 4× rods are identical — print all 4 in one plate. The gear horns are mirrored;
  verify chirality before printing.

---

#### Pack 07 — Suspension Connectors (~4.9 MB)
`3d-prints/07_suspension_connectors/`

- [ ] `*back_two_wheel_connector_1_Body1.stl` (~698 KB)
- [ ] `*back_two_wheel_connector(Mirror)_1_Body1.stl` (~699 KB)
- [ ] `*main_leg_body_connector_1_Body1.stl` (~1.78 MB)
- [ ] `*main_leg_body_connector(Mirror)_1_Body1.stl` (~1.79 MB)
- **Notes:** These are the rocker-bogie pivot connectors — high stress parts. PETG or ABS
  recommended. The mirror pairs are left/right; print one of each side ×2 total (one per bogie).

---

#### Pack 08 — Wheel Hub & Axle Small Parts (~4.9 MB)
`3d-prints/08_wheel_hub_smalls/`

- [ ] `*corner_wheel2_1_Body2.stl` (~591 KB)
- [ ] `*corner_wheel2_1_Body3.stl` (~403 KB)
- [ ] `*corner_wheel2_1_Body4.stl` (~548 KB)
- [ ] `*corner_wheel2_1_Body5.stl` (~428 KB)
- [ ] `*corner_wheel2_1_Body6.stl` (~119 KB)
- [ ] `*corner_wheel2_1_Body7.stl` (~550 KB)
- [ ] `*corner_wheel2_1_Body9.stl` (~550 KB)
- [ ] `*corner_wheel2_1_Body10.stl` (~403 KB)
- [ ] `*middle_wheel_1_Body1.stl` (~403 KB)
- [ ] `*middle_wheel_1_Body2.stl` (~591 KB)
- [ ] `*middle_wheel_1_Body5.stl` (~403 KB)
- **Notes:** These are the hub faces, axle flanges, and steering knuckle small bodies.
  Note: corner_wheel2 is used ×4 (one per corner) — print multiples as needed.

---

#### Pack 09 — Corner Wheel Shell A (~4.0 MB)
`3d-prints/09_wheel_corner_A/`

- [ ] `*corner_wheel2_1_Body1.stl` (~4.0 MB — the main corner wheel outer shell)
- **Notes:** This is a large single part. Verify it fits your print bed. Needs 4 total — print 1
  to validate fit, then run the remaining 3.

---

#### Pack 10 — Corner Wheel Shell B (~4.1 MB)
`3d-prints/10_wheel_corner_B/`

- [ ] `*corner_wheel2_1_Body8.stl` (~4.1 MB — the corner wheel inner shell / TPU candidate)
- **Notes:** If printing the Mars 2020-style flexible wheel outer, use TPU here. Separate slicer
  profile required. Need 4 total.

---

#### Pack 11 — Middle Wheel Shell A (~4.0 MB)
`3d-prints/11_wheel_middle_A/`

- [ ] `*middle_wheel_1_Body3.stl` (~4.0 MB)
- **Notes:** Middle wheel outer skin. Need 2 total (one per side). TPU candidate if using flex wheels.

---

#### Pack 12 — Middle Wheel Shell B (~4.1 MB)
`3d-prints/12_wheel_middle_B/`

- [ ] `*middle_wheel_1_Body4.stl` (~4.1 MB)
- **Notes:** Middle wheel inner body. Need 2 total.

---

#### Packs 13–17 — Rover Core Structure Body Parts (~28 MB across 5 packs)
> The 54 numbered `Rover core v232_1_Body*.stl` files are generic structural parts
> (frame rails, motor mounts, axle housings, brackets). They have no named sub-assembly
> context, so these are binned by size into safe packs. **Cross-reference each body
> number against the Fusion 360 model before printing to confirm it is not a hardware
> reference** — especially any file under 50 KB.

- [ ] **Pack 13** `3d-prints/13_core_structure_A/` — Body6 (~4.0 MB) + Body33 (~4.2 MB) = ~8.2 MB
  *(The two largest single bodies; each is near the 10 MB limit solo with slicer overhead)*

- [ ] **Pack 14** `3d-prints/14_core_structure_B/` — Body3 + Body16 + Body25 + Body26 + Body28 + Body38
  (~1.1 + ~1.2 + ~0.9 + ~0.9 + ~1.1 + ~1.5 MB = ~6.7 MB)

- [ ] **Pack 15** `3d-prints/15_core_structure_C/` — Body1 + Body2 + Body4 + Body5 + Body7 + Body8 + Body9 + Body10 + Body11 + Body12 + Body13 + Body14
  (~425+118+401+651+320+584+535+231+792+138+559+183 KB = ~4.9 MB)

- [ ] **Pack 16** `3d-prints/16_core_structure_D/` — Body15 + Body17 + Body18 + Body19 + Body20 + Body21 + Body22 + Body23 + Body24 + Body27 + Body29 + Body30
  (~543+213+526+154+84+154+213+234+84+792+401+538 KB = ~3.9 MB)

- [ ] **Pack 17** `3d-prints/17_core_structure_E/` — Body31 + Body32 + Body34 + Body35 + Body36 + Body37 + Body39 + Body40 + Body41 + Body42 + Body43 + Body44 + Body45 + Body46\* + Body47 + Body48 + Body49 + Body50 + Body51 + Body52 + Body53 + Body54
  (~5.1 MB total) *\*Body46 = 7.3 KB — flag for CAD review before printing*

---

### Step 3 — Post-sort validation

- [ ] Confirm `3d-prints/` root is empty after all files are moved — no orphans left behind
- [ ] Verify total file count across all print packs matches count of printable originals
- [ ] Spot-check 3–4 files per pack by opening in slicer to confirm geometry is valid and
  not inside-out / zero-volume
- [ ] Add a `3d-prints/README.md` listing pack descriptions, recommended materials, and
  quantities per part

---

## Parking Lot / Future

- [ ] Camera feed (CSI or USB cam) streamed to GCS or web UI
- [ ] Companion computer (Raspberry Pi 5 / Jetson Orin Nano) for vision-based SLAM
- [ ] ROS2 integration for advanced mission execution
- [ ] Solar charging panel on lid for extended outdoor operation
- [ ] Weatherproofing / IP rating for electronics bay
- [ ] Custom antenna tracker using second LoRa + GPS module
- [ ] Simulation model export (Gazebo / Isaac Sim URDF from Fusion 360)
