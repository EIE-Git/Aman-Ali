# Smart Energy Metering System

## Overview

A low-power smart metering solution that reads electricity consumption data from existing meters via optical pulse counting and SCT-013 CT clamp sensors. Data is transmitted over **LoRaWAN** to a cloud backend where usage analytics and anomaly detection are presented on a Grafana dashboard.

## Architecture

```
STM32L4 Meter Node
  ├── SCT-013 CT clamp (current)
  ├── Optical pulse counter
  └── RFM95W LoRa module
          │  LoRaWAN (OTAA)
          ▼
  LoRaWAN Network Server
  (The Things Network / Chirpstack)
          │  HTTP webhook / MQTT
          ▼
   FastAPI Ingest Service
          ├── TimescaleDB (hypertable)
          └── Grafana dashboard
                    │
             Email / SMS Alerts
             (anomaly detection)
```

## Hardware Bill of Materials

| Component | Quantity | Notes |
|-----------|----------|-------|
| STM32L4 Nucleo-L476RG | 10 | Ultra-low-power MCU |
| SCT-013-030 CT clamp | 10 | 30 A non-invasive current sensor |
| RFM95W LoRa module | 10 | 868 MHz (EU868) |
| 18650 Li-ion cell + TP4056 | 10 | Primary power |
| Optical pulse sensor (TCRT5000) | 10 | Pulse-output meter interface |

## Firmware (STM32L4 – C)

### Power Optimisation

| Mode | Current | Duration |
|------|---------|---------|
| STOP2 (deep sleep) | < 10 µA | Between measurement windows |
| Active (ADC + LoRa TX) | ~12 mA | ~500 ms per cycle |
| Average (15-min interval) | ~0.2 µA equiv | — |

**Estimated battery life: > 5 years** on a single 18650 cell.

### Measurement Cycle (every 15 minutes)

1. Wake from STOP2 on RTC alarm
2. Sample CT clamp at 1 kHz for 1 s → compute IRMS
3. Count optical pulses accumulated since last wake
4. Encode payload (Cayenne LPP format)
5. LoRaWAN OTAA join (if not joined) → transmit
6. Return to STOP2

## LoRaWAN Payload (Cayenne LPP)

| Channel | Type | Value |
|---------|------|-------|
| 0 | Analog Input | IRMS (A × 100) |
| 1 | Analog Input | Active power (W) |
| 2 | Analog Input | Pulse count (Wh × 10) |
| 3 | Temperature | Board temperature (°C) |
| 4 | Analog Input | Battery voltage (V × 100) |

Total payload: **15 bytes** — well within DR0 (51-byte limit).

## Cloud Backend

### TimescaleDB Schema

```sql
CREATE TABLE readings (
    time        TIMESTAMPTZ NOT NULL,
    meter_id    TEXT        NOT NULL,
    irms_a      REAL,
    power_w     REAL,
    energy_wh   REAL,
    temperature REAL,
    battery_v   REAL
);
SELECT create_hypertable('readings', 'time');
```

### FastAPI Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/ingest` | Receive decoded LoRaWAN uplink |
| GET | `/meters` | List all meters |
| GET | `/meters/{id}/readings` | Paginated readings |
| GET | `/meters/{id}/daily` | Daily energy summary |

## Anomaly Detection

Uses a **Z-score** on a rolling 7-day window to detect unusual consumption:

```python
z = (current_reading - rolling_mean) / rolling_std
if abs(z) > threshold:    # default threshold = 3
    trigger_alert(meter_id, current_reading, z)
```

Alerts are sent via email/SMS using AWS SNS.

## Grafana Dashboard Panels

1. Real-time power consumption (gauge)
2. Hourly energy bar chart (last 24 h)
3. Daily kWh heatmap (last 30 days)
4. Monthly cost estimate
5. Anomaly event log

## Team

| Role | Responsibilities |
|------|-----------------|
| IoT Team Lead (Aman Ali) | Firmware power optimisation, LoRaWAN payload design, anomaly detection, field deployment |
| Firmware Engineer | CT clamp ADC driver, RTC alarm, TP4056 charge management |
| Backend Engineer | FastAPI ingest service, TimescaleDB schema |
| Data Engineer | Grafana dashboards, anomaly alert pipeline |
