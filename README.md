# হাওয়াই জাহাজ — Hawai Jahaj

> An autonomous GPS waypoint quadcopter built from scratch by two university students.  
> Point A → Point B. Hover. Wait. Land on command.

---

## Overview

Hawai Jahaj is a 250mm quadcopter that flies a pre-programmed GPS route autonomously. Upload coordinates before flight, arm the drone, and it handles takeoff, navigation, and hovering at the destination on its own. Landing is triggered manually over Wi-Fi.

Built on an ESP32 with a custom PID stabilization loop, GPS-guided navigation, and a browser-based mission uploader — no proprietary flight stack, no black boxes.

---

## Team

| Role | Responsibility |
|---|---|
| CSE | Firmware, PID algorithm, navigation logic, web UI |
| EEE | Power system, ESC wiring, circuit design, frame assembly |

Previously built a Line Following Robot together. This is the next step.

---

## Features

- **Autonomous A → B flight** via pre-uploaded GPS coordinates
- **Dual-loop control architecture** — inner PID at ~100Hz, outer GPS loop at ~10Hz on separate ESP32 cores
- **Browser-based mission uploader** served directly from the ESP32 (no app needed)
- **Live telemetry panel** — current GPS position, flight phase, battery voltage
- **Geofenced safety** — holds position on GPS anomaly, won't arm without satellite fix
- **Modular payload bay** — standardized JST connector for swappable sensors and cameras
- **3D printed frame** — crash-repairable arms, reprints in hours not days

---

## Hardware

| Component | Part |
|---|---|
| Flight controller | ESP32 (dual-core, 240MHz) |
| IMU | MPU-6050 (I2C) |
| GPS | Neo-6M / Neo-8M (UART) |
| Compass | HMC5883L or QMC5883 (I2C) |
| Motors | 4× Brushless 1000–1400KV |
| ESCs | 4× 10–20A (or 4-in-1 board) |
| Battery | 2S/3S LiPo 1500–2200mAh |
| Frame | 250mm X-config, PETG printed |
| Connector | XT60 |

---

## Firmware Architecture

```
Core 1 (100Hz)   →   IMU read → Complementary filter → PID → PWM to ESCs
Core 0 (10Hz)    →   GPS parse → Haversine bearing → Heading + throttle targets
```

### Flight State Machine

```
IDLE → TAKEOFF → NAVIGATE → HOVER_B → LANDING → DISARMED
```

| State | Description |
|---|---|
| `IDLE` | Waiting for mission upload and arm command |
| `TAKEOFF` | Climb to target altitude, confirmed by GPS |
| `NAVIGATE` | Haversine bearing to point B, yaw correction + forward throttle |
| `HOVER_B` | GPS within 2m of point B — hold position |
| `LANDING` | Manual trigger over WebSocket — gradual throttle reduction |
| `DISARMED` | Motors off |

### Web Interface

ESP32 runs in Access Point (AP) mode. Connect any phone or laptop to its network, open the browser, and you get:

- Mission upload form (GPS coords, altitude, speed)
- Live status panel (position, phase, voltage)
- Manual land button

Communication over **WebSocket** for low latency.

---

## Repository Structure

```
hawai-jahaj/
├── firmware/
│   ├── main.cpp              # Entry point, FreeRTOS task setup
│   ├── pid.cpp / pid.h       # PID controller (roll, pitch, yaw)
│   ├── imu.cpp / imu.h       # MPU-6050 read + complementary filter
│   ├── gps.cpp / gps.h       # Neo-6M NMEA parsing
│   ├── nav.cpp / nav.h       # Haversine navigation, state machine
│   ├── esc.cpp / esc.h       # PWM output via LEDC peripheral
│   └── web.cpp / web.h       # WebSocket server + mission uploader UI
├── frame/
│   ├── center_plate.stl
│   ├── arm.stl               # Print ×4
│   └── payload_bay.stl
├── schematics/
│   └── wiring_diagram.pdf
├── docs/
│   └── DEVLOG.md             # Full Obsidian-style development log
└── README.md
```

---

## Build Log

Full development log with phase-by-phase notes, decisions, and lessons is in [`docs/DEVLOG.md`](docs/DEVLOG.md).

---

## Pre-Flight Checklist

- [ ] GPS fix confirmed (≥ 6 satellites)
- [ ] Battery voltage above threshold
- [ ] Mission coordinates uploaded and verified on web UI
- [ ] All 4 motors spin correct direction (prop-off test first)
- [ ] PID values from last session loaded
- [ ] Area clear — no people within 10 meters
- [ ] Geofence bounds set for test location
- [ ] Device connected to drone Wi-Fi AP

---

## Safety

**LiPo batteries are the highest physical risk in this build.**

- Never charge unattended
- Never discharge below 3.5V per cell
- Store at storage voltage (~3.8V per cell)
- Keep a fire-safe bag nearby during charging

The drone will not arm without a confirmed GPS fix (≥ 6 satellites). The geofence logic holds position if GPS reads an implausible location jump.

---

## Roadmap

- [x] Project scoping and architecture design
- [ ] Parts sourced
- [ ] Frame printed and assembled
- [ ] ESC calibration and motor direction confirmed
- [ ] IMU reading stable with complementary filter
- [ ] PID tuned (prop-off, then tethered, then free hover)
- [ ] GPS parsing and fix quality gating
- [ ] Haversine navigation logic
- [ ] State machine integration
- [ ] Web mission uploader live
- [ ] First autonomous A→B flight

**Version 2.0 (future)**
- Dedicated RC transmitter (replace Wi-Fi)
- Return-to-home after reaching point B
- Multi-waypoint mission support
- Pi Zero 2W companion computer for onboard vision
- Barometer for altitude hold
- Carbon fiber frame

---

## Budget

Total build cost: ~$56–92 USD

| Part | Est. Cost |
|---|---|
| 4× Brushless motors | $12–18 |
| 4× ESCs / 4-in-1 board | $10–20 |
| PETG filament (frame) | $2–4 |
| LiPo battery | $10–15 |
| LiPo balance charger | $8–12 |
| Propellers | $3–5 |
| MPU-6050 | $1–2 |
| Neo-6M GPS + compass | $5–8 |
| Wires, connectors, XT60 | $5–8 |
| ESP32 | On hand |

---

## License

MIT

---

*হাওয়াই জাহাজ — Bengali for "flying machine" (literally "air ship")*
