# Smart Home Automation Gateway

## Overview

A locally-hosted smart home gateway that bridges **Zigbee**, **Z-Wave**, and **Wi-Fi** devices under a single unified MQTT broker. A React/TypeScript web UI provides device control and automation rule management, with optional cloud voice-assistant (Google Home / Amazon Alexa) integration.

## Architecture

```
Zigbee devices ──► ConBee II ──┐
Z-Wave devices ──► Z-Stick  ──┤   Raspberry Pi 4
Wi-Fi devices  ──► LAN       ──┤   ├── Mosquitto MQTT broker
                                └──►├── Zigbee2MQTT
                                    ├── Z-Wave JS
                                    └── FastAPI backend
                                             │
                                    React Web UI (port 3000)
                                             │
                                   ┌─────────┴─────────┐
                              Google Home         Amazon Alexa
```

## Supported Protocols

| Protocol | Adapter | Max Devices |
|----------|---------|-------------|
| Zigbee 3.0 | Dresden Elektronik ConBee II | 50 |
| Z-Wave Plus | Aeotec Z-Stick Gen5 | 232 |
| Wi-Fi (REST / LAN) | Native TCP | Unlimited |

## Software Components

| Component | Version | Purpose |
|-----------|---------|---------|
| Mosquitto | 2.x | MQTT broker |
| Zigbee2MQTT | 1.36+ | Zigbee ↔ MQTT bridge |
| Z-Wave JS UI | 9.x | Z-Wave ↔ MQTT bridge |
| FastAPI | 0.110+ | REST API & WebSocket server |
| React + TypeScript | 18 | Web front-end |

## Device Abstraction Layer

All devices, regardless of protocol, are represented with a common schema:

```json
{
  "id": "living_room_light",
  "name": "Living Room Light",
  "protocol": "zigbee",
  "capabilities": ["on_off", "brightness", "color_temp"],
  "state": {
    "on": true,
    "brightness": 80,
    "color_temp": 4000
  }
}
```

New protocols are added as Python plugins implementing the `DevicePlugin` interface.

## Automation Rule Engine

Rules follow an **Event → Condition → Action** model:

```yaml
- name: "Evening lights on"
  trigger:
    type: time
    at: "19:00"
  condition:
    - entity: sun.elevation
      below: 0
  actions:
    - entity: living_room_light
      service: turn_on
      brightness: 60
```

Rules are stored as YAML files in `rules/` and hot-reloaded at runtime.

## API Reference

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/devices` | List all devices |
| GET | `/devices/{id}` | Get device state |
| POST | `/devices/{id}/command` | Send command to device |
| GET | `/automations` | List automation rules |
| POST | `/automations` | Create new rule |
| DELETE | `/automations/{id}` | Delete rule |

## Voice Assistant Integration

- **Google Home:** Actions on Google + Smart Home trait mapping
- **Amazon Alexa:** Alexa Smart Home Skill + Lambda handler

Both integrations use OAuth 2.0 via an optional cloud relay service.

## Team

| Role | Responsibilities |
|------|-----------------|
| IoT Team Lead (Aman Ali) | Device-abstraction layer, rule engine, API design, team coordination |
| Backend Engineer | FastAPI server, WebSocket, voice integrations |
| Frontend Engineer | React UI, real-time device state via WebSocket |
