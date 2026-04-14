# Smart Environment Monitoring System

## Overview

A distributed IoT sensor network that continuously monitors indoor and outdoor environmental conditions across multiple sites. Data is aggregated in the cloud and presented on a real-time Grafana dashboard with configurable threshold alerts.

## Architecture

```
Sensors (ESP32)
     │  MQTT over TLS (Wi-Fi)
     ▼
AWS IoT Core
     │
     ├── AWS Lambda  ──► DynamoDB (storage)
     │
     └── Amazon SNS  ──► Email / SMS Alerts
                              │
                         Grafana Dashboard
```

## Sensors & Parameters

| Sensor | Parameter | Unit |
|--------|-----------|------|
| DHT22 | Temperature | °C |
| DHT22 | Relative Humidity | % RH |
| MQ-135 | CO₂ equivalent | ppm |
| PMS5003 | PM2.5 / PM10 | µg/m³ |
| MAX4466 | Noise Level | dB |

## Hardware Bill of Materials

| Component | Quantity | Notes |
|-----------|----------|-------|
| ESP32 DevKit v1 | 6 | One per monitoring node |
| DHT22 | 6 | Temperature & humidity |
| MQ-135 | 6 | Air quality |
| PMS5003 | 6 | Particulate matter |
| Raspberry Pi 4 (4 GB) | 1 | Central gateway / dashboard host |
| 5 V / 2 A USB-C PSU | 6 | Node power |

## Firmware

- Language: **Arduino C++ / MicroPython**
- Sampling interval: configurable (default 60 s)
- Deep-sleep between readings to reduce power consumption
- X.509 mutual TLS authentication with AWS IoT Core

### Directory Structure

```
firmware/
├── main/
│   ├── main.ino          # Entry point
│   ├── sensors.h/.cpp    # Sensor abstraction layer
│   ├── mqtt_client.h/.cpp# MQTT wrapper (TLS)
│   └── config.h          # Wi-Fi & AWS endpoint config
└── lib/
    ├── DHT/
    ├── PubSubClient/
    └── PMS/
```

## Cloud Setup

1. Create an AWS IoT Core Thing for each node
2. Generate and flash X.509 certificates onto each ESP32
3. Deploy the Lambda function (`cloud/lambda/ingest.py`) as the IoT Rule action
4. Provision the DynamoDB table with the schema in `cloud/dynamodb_schema.json`
5. Import `cloud/grafana_dashboard.json` into Grafana

## Alert Thresholds (defaults)

| Parameter | Warning | Critical |
|-----------|---------|----------|
| Temperature | > 28 °C | > 35 °C |
| CO₂ | > 1000 ppm | > 2000 ppm |
| PM2.5 | > 35 µg/m³ | > 75 µg/m³ |
| Noise | > 70 dB | > 85 dB |

## Team

| Role | Responsibilities |
|------|-----------------|
| IoT Team Lead (Aman Ali) | Architecture, firmware library, AWS setup, sprint planning |
| Firmware Engineer | Node firmware development & unit tests |
| Cloud Engineer | Lambda, DynamoDB, SNS integration |
| QA Engineer | Integration tests, field validation |
