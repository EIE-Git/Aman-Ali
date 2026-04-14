# Predictive Maintenance Platform

## Overview

An edge-AI solution that monitors vibration and temperature of CNC machine bearings and spindles in real time. An on-device TensorFlow Lite model classifies bearing health (healthy / warning / critical) and raises maintenance tickets automatically, reducing unplanned downtime.

## Architecture

```
STM32H7 Node (ADXL345)
      │  SPI (raw accelerometer data)
      │  UART → edge gateway
      ▼
NVIDIA Jetson Nano (Edge Gateway)
      ├── TFLite inference (bearing health classification)
      ├── Feature extraction (FFT, RMS, kurtosis)
      └── MQTT publish → cloud
                │
         Apache Kafka
         ├── PostgreSQL  (event history)
         └── Kibana      (dashboard & alerting)
                │
         Maintenance Ticketing System (Jira API)
```

## Hardware Bill of Materials

| Component | Quantity | Notes |
|-----------|----------|-------|
| STM32H7 Nucleo | 8 | One per machine axis |
| ADXL345 accelerometer | 8 | ±16 g, 3200 Hz max ODR |
| K-type thermocouple + MAX31855 | 8 | Spindle temperature |
| NVIDIA Jetson Nano (4 GB) | 2 | Edge inference nodes |
| DIN rail enclosure | 8 | Industrial IP65 housing |

## Firmware (STM32H7 – C / FreeRTOS)

- Sampling rate: **3200 Hz** (raw), **200 Hz** after anti-aliasing filter
- Data framing: 512-sample windows sent over UART to Jetson
- Tasks:
  - `sensorTask`: ADC/SPI read at 3200 Hz
  - `filterTask`: FIR low-pass (fc = 1000 Hz)
  - `txTask`: UART DMA burst every 256 ms

## Feature Engineering (Python – Edge)

For each 512-sample window per axis:

| Feature | Formula |
|---------|---------|
| RMS | √(Σxᵢ² / N) |
| Peak-to-peak | max(x) − min(x) |
| Kurtosis | E[(x−μ)⁴] / σ⁴ |
| FFT magnitude (top 16 bins) | |FFT(x)|[0:16] |
| Crest factor | peak / RMS |

Total feature vector: **53 features × 3 axes = 159 features**

## Machine Learning Model

| Property | Value |
|----------|-------|
| Framework | TensorFlow 2 → TFLite (INT8 quantised) |
| Model type | Dense neural network (3 hidden layers) |
| Training data | 40 h labelled vibration data (3 health states) |
| Accuracy | 96.4 % on held-out test set |
| Inference latency | ~8 ms on Jetson Nano |
| Tool | Edge Impulse (data management & model training) |

### Labels

| Class | Description |
|-------|-------------|
| 0 – Healthy | Normal operating vibration |
| 1 – Warning | Early-stage bearing wear detected |
| 2 – Critical | Immediate maintenance required |

## MQTT Message Schema (Protobuf)

```protobuf
message BearingHealth {
  uint32 machine_id    = 1;
  uint32 axis          = 2;   // 0=X, 1=Y, 2=Z
  uint64 timestamp_ms  = 3;
  float  rms           = 4;
  float  kurtosis      = 5;
  uint32 health_class  = 6;   // 0=healthy, 1=warning, 2=critical
  float  confidence    = 7;
}
```

## Alerting & Ticketing

- **Warning:** Kibana alert → email notification to maintenance team
- **Critical:** Automatic Jira ticket created via REST API with machine ID, axis, timestamp, and trend chart attachment

## Team

| Role | Responsibilities |
|------|-----------------|
| IoT Team Lead (Aman Ali) | ML pipeline, model deployment, MQTT schema, Jira integration, backlog management |
| Firmware Engineer × 2 | STM32 firmware, FreeRTOS tasks, UART framing |
| ML Engineer | Model architecture, Edge Impulse dataset, quantisation |
| DevOps Engineer | Kafka, PostgreSQL, Kibana, Docker Compose |
