# Industrial Asset Tracking System

## Overview

Real-time tracking and condition monitoring of tools, machinery, and personnel inside a manufacturing facility using a hybrid BLE beacon + GPS approach. Assets indoors are located via BLE RSSI triangulation; assets that leave the site are tracked via LTE-M GPS.

## Architecture

```
BLE Beacons / GPS Tags (nRF52840)
          │  BLE 5.0
          ▼
Raspberry Pi Gateway (BLE scanner)
          │  LTE-M (SIM7080G) / Ethernet
          ▼
     MQTT Broker (Mosquitto)
          │
     Node.js Backend
     ├── InfluxDB (time-series position data)
     └── Mapbox GL JS (web map UI)
```

## Hardware Bill of Materials

| Component | Quantity | Notes |
|-----------|----------|-------|
| Nordic nRF52840 DK | 20 | Asset tags |
| u-blox NEO-M9N GPS | 20 | Outdoor positioning |
| SIM7080G LTE-M module | 20 | Cellular fallback |
| Raspberry Pi 4 (2 GB) | 4 | Site gateways (BLE scanners) |
| CR2032 coin cell | 20 | Tag power (12-month life) |

## Firmware

- **RTOS:** Zephyr RTOS (v3.x)
- **Language:** C
- BLE advertisement packet format (custom company ID `0xFF 0xAA`):

```
Byte  0–1 : Company ID
Byte  2–5 : Asset ID (uint32)
Byte  6–7 : Battery voltage (mV, uint16)
Byte  8–9 : Temperature (°C × 10, int16)
Byte 10   : Status flags (moving / stationary / alert)
```

- GPS NMEA sentences parsed on-device; only lat/lon/altitude forwarded
- Power modes: active scan every 5 s, STOP mode between intervals

## Positioning Algorithm

Indoor positioning uses a **weighted centroid** algorithm based on RSSI readings from ≥ 3 gateway Raspberry Pi nodes:

```
position = Σ(wᵢ × pᵢ) / Σwᵢ
where wᵢ = 1 / d(RSSIᵢ)²
```

Accuracy: ±2–3 m indoors, ±5 m outdoors (GPS).

## Backend & API

- **Runtime:** Node.js 20 + Express
- **Database:** InfluxDB 2.x (time-series)
- **REST endpoints:**

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/assets` | List all tracked assets |
| GET | `/assets/:id/position` | Latest position |
| GET | `/assets/:id/history?from=&to=` | Position history |
| POST | `/alerts` | Create geofence alert rule |

## Web UI

Built with Mapbox GL JS + React. Features:
- Live asset pins on a floor-plan map overlay
- Asset detail panel (battery, temperature, last seen)
- Geofence editor (alert when asset leaves/enters zone)
- Historical playback with timeline slider

## Team

| Role | Responsibilities |
|------|-----------------|
| IoT Team Lead (Aman Ali) | Hardware selection, BLE packet design, positioning algorithm, FAT oversight |
| Firmware Engineer × 2 | Zephyr firmware, power optimisation |
| Backend Engineer | Node.js API, InfluxDB schema |
| Frontend Engineer | Mapbox GL JS map UI |
