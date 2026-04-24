# Assembly Guide

High-level chapter guide for building the RedRover. Each section corresponds to a
physical sub-assembly. Follow the order below — each stage should be testable on its own
before moving to the next.

> **Reference:** Keep the Fusion 360 model open (`Mars-Rover/CAD/Rover_Full.f3d`) throughout.
> The CAD is the source of truth for all dimensions, orientations, and fit.

---

## Chapter 1 — Tools & Prep

**Required tools:**
- M3 and M4 hex keys (ball-end preferred)
- M3 / M4 screwdriver bits
- Soldering iron with fine tip
- Heat-set insert tip for soldering iron (M3 preferred)
- Needle-nose pliers
- Bearing press tool (or M3 bolt + nut + socket as improvised press)
- Digital calipers
- Hot glue gun (cable management / light tacks only — no structural glue)

**Before starting:**
- [ ] Install all heat-set inserts into printed parts before assembly — much easier to do on the bench
- [ ] Test-fit all bearings dry before pressing — 608ZZ into wheel hubs, SKF 6005 into pivot journals
- [ ] Ream any tight bearing seats with a hobby knife or light sandpaper — do not force-press into undersized holes
- [ ] Sand/file any layer lines on servo mounting faces to ensure servo horns seat flush

---

## Chapter 2 — Rocker-Bogie Suspension

The rocker-bogie is the structural spine of the rover. Six wheels are driven by a passive
differential linkage; the front and rear corners steer independently.

**Main printed parts (from print packs 07, 13–17):**
- 2× `main_leg_body_connector` (left + right)
- 2× `back_two_wheel_connector` (left + right)
- Rover core frame bodies from packs 13–17

**Purchased hardware:**
- PVC pipe (ID 23.40 mm / OD 25 mm) — cut to lengths from CAD
- SKF 6005 25 mm bearings (pivot journals)
- 608ZZ bearings (wheel axles)
- M3 / M4 fasteners and heat-set inserts

**Assembly steps (TBD — cross-reference CAD):**
- [ ] Assemble left and right bogie links with PVC pipe inserts
- [ ] Press SKF 6005 bearings into pivot housings (main_leg_body_connector)
- [ ] Connect rocker to bogie via pivot pin — confirm free rotation, no binding
- [ ] Attach bogie sub-assemblies to core frame rails
- [ ] Install differential rocker bar between left and right bogies

**Alignment check:**
- With rover on a flat surface, all 6 wheel contact points should touch simultaneously
- Rocker pivot should float freely — it should not be fastened rigidly

---

## Chapter 3 — Drive Train

Each of the 6 wheels is independently driven by a 12V 60RPM DC motor.
Motors on the left 3 wheels share one ESC; right 3 wheels share the second ESC.

**Main printed parts (from packs 08, 09, 10, 11, 12):**
- 4× corner wheel assembly (outer + inner shell, hub small parts)
- 2× middle wheel assembly (outer + inner shell)

**Purchased hardware:**
- 6× DC motor
- 6× Pololu #1081 shaft hub
- 608ZZ bearings
- M3 fasteners

**Assembly steps (TBD):**
- [ ] Mount each DC motor into its wheel housing
- [ ] Press Pololu shaft hubs onto motor 4mm output shaft — secure M3 grub screws with Loctite
- [ ] Press 608ZZ bearings into outer wheel shells
- [ ] Assemble wheel outer shell onto hub — verify correct side (asymmetric for cornering)
- [ ] Mount wheel assemblies to bogie arms — check for wheel toe/camber

**Corner wheel note:**
Corner wheels carry a steering servo in addition to the drive motor. See Chapter 4 for
steering servo installation before finalising corner wheel assembly.

---

## Chapter 4 — Steering Servos (Corner Wheels)

Four MG946R/MG996R servos steer the front-left, front-right, back-left, and back-right wheels.
Servos are installed inside the corner wheel assembly and driven via the PCA9685 expander.

**PCA9685 servo channels (from `rover_servo.h`):**
| Channel | Servo |
|---|---|
| 6 | `SERVO_FRONT_RIGHT` |
| 7 | `SERVO_BACK_LEFT` |
| 8 | `SERVO_FRONT_LEFT` |
| 9 | `SERVO_BACK_RIGHT` |

**Trim offsets (from `rover_config.h`):**
| Servo | Offset |
|---|---|
| Front Left | +20 µs |
| Front Right | −15 µs |
| Back Left | +5 µs |
| Back Right | −25 µs |

**Assembly steps (TBD):**
- [ ] Install servo into corner wheel housing — align spline notch before inserting
- [ ] Attach 25T disc horn — set to mechanical centre first using servo tester or firmware
- [ ] Verify wheel points straight ahead at 1500 µs centre signal
- [ ] Fine-tune offsets in `rover_config.h` if required

---

## Chapter 5 — 6-DOF Arm

The arm has 6 servo-driven joints (MG946R). Currently the arm is directly RC-mapped;
inverse kinematics is planned for a future firmware update.

**Main printed parts (from packs 04, 05, 06):**
- arm link 1 (`arm_1_1_Body1`)
- arm body connector (`arm_body_connector_1_Body1`)
- arm links 2 + mirror (`arm2`, `arm2(Mirror)(Mirror)`)
- arm link 3 (`arm3_1_Body1`)
- gripper connector assembly (`arm_gripper_connector_1_Body1/2`)
- grip rotator (`grip_rotator_1_Body1`)
- gripper finger assembly (packs 06)
- 3× printed servo horns (`servo_and_horn v2 (1–3)_1_Body1`)

**PCA9685 servo channels (from `rover_servo.h`):**
| Channel | Servo |
|---|---|
| 0 | `SERVO_ARM_AXIS_1` |
| 1 | `SERVO_ARM_AXIS_2` |
| 2 | `SERVO_ARM_AXIS_3` |
| 3 | `SERVO_ARM_AXIS_4` |
| 4 | `SERVO_ARM_AXIS_5` |
| 5 | `SERVO_ARM_AXIS_6` |

**Assembly steps (TBD):**
- [ ] Install AXIS_1 servo at base rotation joint (arm_body_connector)
- [ ] Route wiring through hollow arm links before closing
- [ ] Assemble links 1 → 2 → 3 in order — test each joint range before proceeding
- [ ] Install gripper connector and grip rotator (AXIS_5)
- [ ] Install gripper servo at AXIS_6 — connect gear horns
- [ ] Balance arm weight at each joint; note if any joint stalls under static load

**Arm range of motion:**
SERVOMIN (250) to SERVOMAX (500) in PCA9685 counts (full range: 150–600).
Firmware clamps to reduced range by default to protect servo gears.

---

## Chapter 6 — Head

The head has yaw (left/right) and pitch (up/down) driven by two MG946R/MG996R servos.
An ESP32-CAM mounts in the head for the video feed.

**Main printed parts (from packs 02, 03):**
- head rotator body (`head_rotator_1_Body1/2/3`)
- head housing (`head_rotator_1_head_1_Body1/2/3/4`)
- head top connector assembly (`head_top_connector_1_Body*`)
- ESP32-CAM holder (`espcam_holder_1_Body1/2`)
- 2× printed servo horns (`servo_and_horn v2 (4)_1_Body1`, `v2 (5)_1_Body1`)

**PCA9685 servo channels (from `rover_servo.h`):**
| Channel | Servo |
|---|---|
| 11 | `SERVO_HEAD_YAW` |
| 12 | `SERVO_HEAD_PITCH` |

**RC channel mapping (from `rover_config.h`):**
- Head yaw: RC_CH3 (GPIO4)
- Head pitch: RC_CH4 (GPIO39)

**Assembly steps (TBD):**
- [ ] Install yaw servo into head_rotator bracket; attach printed horn
- [ ] Install pitch servo into head housing; attach printed horn
- [ ] Mount ESP32-CAM into espcam_holder — confirm camera faces forward
- [ ] Mount head sub-assembly onto top of rover body via head_top_connector
- [ ] Verify full yaw/pitch range without cable binding

**Sensor mast note (RedRover addition):**
If mounting RPLidar on the head/top deck rather than a separate mast, provision cable path
and mount clearance for the LiDAR sensor here.

---

## Chapter 7 — Electronics Bay

The electronics bay is inside the main chassis body, covered by the printed lid (`Lid1.stl`).

**Planned component layout (TBD — diagram needed):**
- ArduRover flight controller (centre, vibration-isolated)
- TTGO LoRa32 ESP32 board
- PCA9685 PWM servo driver (I2C to ESP32)
- Power module (battery-side of PDB)
- PDB / BEC modules
- SiK telemetry radio
- RC receiver
- Safety arming button (mounted through lid or body panel)

**Assembly steps (TBD):**
- [ ] Mount PDB to bay floor — confirm cable routing to all 6 motors and servo BEC rails
- [ ] Mount power module in-line between XT60 battery connector and PDB
- [ ] Mount ArduRover FC on vibration foam — orient arrow forward
- [ ] Mount ESP32 (TTGO LoRa32) adjacent to FC — UART link between them
- [ ] Route all servo signal wires to PCA9685
- [ ] Route RC receiver to ESP32 GPIOs 34, 35, 4, 39, 25, 36 (CH1–CH6)
- [ ] Install safety button — wire to FC safety port
- [ ] Lid test fit — confirm no cables pinched

---

## Chapter 8 — First Power-On Checklist

Before first full power-on with all components connected:

- [ ] Verify polarity on all ESC power connections before plugging in
- [ ] Confirm all servo power rails are on regulated 5V, not direct battery
- [ ] Check for any short circuits across PDB rails with multimeter (no battery connected)
- [ ] Connect battery — check for smoke, heat, or excessive current draw before proceeding
- [ ] Verify FC boots and shows solid LED (no FC-specific error indications)
- [ ] Confirm ESP32 serial output on 115200 baud — check for crash/reboot loops
- [ ] Test each servo to centre before connecting to arm/steering linkages
- [ ] Test each motor direction — verify left/right ESC mapping matches firmware
