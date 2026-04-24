# Ground Control Station Setup

How to connect a GCS to the RedRover for configuration, monitoring, and autonomous mission control.

---

## GCS Software Options

| Application | Platform | Best for |
|---|---|---|
| **Mission Planner** | Windows | Full ArduRover parameter editing, initial setup, calibration |
| **QGroundControl** | Windows / macOS / Linux / Android | Mission planning, field monitoring, clean UI |

Recommendation: use **Mission Planner** for initial setup and calibration, then
**QGroundControl** (QGC) for day-to-day field use. Both support MAVLink 2.

---

## Connection via SiK Telemetry Radio

The primary GCS link uses a SiK 915 MHz radio pair (e.g. RFD900x or HolyBro SiK v3):
- One radio connected to ArduRover FC TELEM1 port (on the rover)
- One radio connected to GCS laptop via USB

### Mission Planner
1. Plug GCS radio into USB port — note COM port assigned (Device Manager)
2. Open Mission Planner → top-right dropdown: select COM port, set baud to **57600**
3. Click **Connect** — wait for parameter download (~30 sec on first connect)

### QGroundControl
1. QGC auto-detects SiK radio on USB — connection banner should appear
2. If not auto-detected: Application Settings → Comm Links → Add → Serial → select port → 57600

---

## Connection via WiFi (ESP32 hosted web UI)

The ESP32 can host a WiFi AP for browser-based control (no GCS software needed).

1. Set rover boot mode switch to `ROVER_SWITCH_STATE_AP` (see [70_firmware-esp32.md](70_firmware-esp32.md))
2. Connect laptop/phone to WiFi SSID **"Rover"**
3. Navigate to `http://192.168.4.1` in browser
4. Virtual joystick UI loads — WebSocket connects automatically

> This path bypasses ArduRover — it controls the ESP32 directly for arm/head servos
> and (in the original firmware) direct motor drive. Post-refactor it will be a
> MAVLink telemetry dashboard rather than a direct motion controller.

---

## MAVLink Telemetry via ESP32 WiFi Bridge (RedRover planned)

After the ESP32 MAVLink refactor ([70_firmware-esp32.md](70_firmware-esp32.md)):

1. ESP32 bridges MAVLink from FC TELEM2 over WiFi WebSocket
2. QGC can connect via UDP: Application Settings → Comm Links → Add → UDP → port 14550
3. Or use MAVLink2Rest running on ESP32 companion

---

## First Field Session Checklist

### Before leaving the bench
- [ ] SiK radios paired and link LED solid on both units
- [ ] Mission Planner connects and downloads parameters cleanly
- [ ] GPS fix acquired indoors or near window (HDOP < 1.4)
- [ ] Battery monitor reading correct voltage in MP HUD
- [ ] All RC channels reading correct values in MP Radio Calibration screen

### At the field
- [ ] Set home point (Mission Planner → right-click map → Set Home Here) or confirm GPS auto-home
- [ ] Draw polygon geofence appropriate to test area
- [ ] Arm rover (safety button → hold RC arm switch or MP arm button)
- [ ] Test MANUAL mode first — verify steering direction and motor direction
- [ ] Test HOLD mode — rover should stop and hold position
- [ ] Test simple 3-waypoint AUTO mission before longer routes
- [ ] Keep hand on RC transmitter to switch back to MANUAL at any time

---

## Driving Modes Reference

| Mode | ArduRover name | Behaviour |
|---|---|---|
| MANUAL | `MANUAL` | Direct RC passthrough to ESCs — no autopilot intervention |
| HOLD | `HOLD` | Stop and hold current position |
| STEERING | `STEERING` | Autopilot stabilises heading; throttle from RC |
| AUTO | `AUTO` | Follows uploaded waypoint mission |
| GUIDED | `GUIDED` | Fly-to-point commands from GCS map click |
| RTL | `RTL` | Return to GPS home point and stop |

---

## Useful MAVLink Shell Commands (MAVProxy)

```bash
mavproxy.py --master=/dev/ttyUSB0 --baud=57600

# Check modes
mode MANUAL
mode AUTO

# Arm / disarm
arm throttle
disarm

# Request parameter
param fetch
param show WP_RADIUS

# Upload mission from file
wp load my_mission.txt
wp set 1       # jump to waypoint 1

# Stream position
set streamrate 4
```
