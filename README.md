# Aman Ali – IoT Team Lead Projects

A portfolio of IoT projects developed and led as **IoT Team Lead**. Each project demonstrates end-to-end system design, firmware development, cloud integration, and team coordination across the full IoT stack.

---

## 📂 Projects

| # | Project | Description | Status |
|---|---------|-------------|--------|
| 1 | [Smart Environment Monitoring System](#1-smart-environment-monitoring-system) | Real-time indoor/outdoor environmental data collection and alerting | ✅ Complete |
| 2 | [Industrial Asset Tracking System](#2-industrial-asset-tracking-system) | GPS + BLE-based asset location and condition monitoring | ✅ Complete |
| 3 | [Smart Home Automation Gateway](#3-smart-home-automation-gateway) | Centralised gateway for controlling home appliances via MQTT | ✅ Complete |
| 4 | [Predictive Maintenance Platform](#4-predictive-maintenance-platform) | Vibration & temperature sensor data used to predict equipment failure | ✅ Complete |
| 5 | [Smart Energy Metering System](#5-smart-energy-metering-system) | Remote electricity consumption monitoring with anomaly detection | ✅ Complete |

---

## 1. Smart Environment Monitoring System

**Folder:** [`Smart-Environment-Monitoring/`](Smart-Environment-Monitoring/)

### Overview
Designed and led the development of a distributed sensor network that collects temperature, humidity, CO₂, PM2.5, and noise data across multiple sites. Data is streamed to a cloud dashboard with configurable threshold-based alerts.

### Tech Stack
- **Hardware:** ESP32, DHT22, MQ-135, PMS5003, Raspberry Pi 4
- **Firmware:** MicroPython / Arduino C++
- **Connectivity:** Wi-Fi (MQTT over TLS)
- **Cloud:** AWS IoT Core → AWS Lambda → DynamoDB → Grafana
- **Team Size:** 4 engineers

### Key Responsibilities
- Defined system architecture and sensor selection criteria
- Wrote firmware library for multi-sensor data aggregation
- Set up AWS IoT Core with X.509 certificate authentication
- Managed sprint planning and code reviews

---

## 2. Industrial Asset Tracking System

**Folder:** [`Industrial-Asset-Tracking/`](Industrial-Asset-Tracking/)

### Overview
End-to-end asset tracking solution for a manufacturing facility, combining GPS modules with BLE beacons to track tools, machinery, and personnel in real time.

### Tech Stack
- **Hardware:** Nordic nRF52840, u-blox NEO-M9N GPS, Raspberry Pi gateway
- **Firmware:** Zephyr RTOS (C)
- **Connectivity:** BLE 5.0, LTE-M (SIM7080G)
- **Backend:** Node.js, InfluxDB, Mapbox GL JS
- **Team Size:** 5 engineers

### Key Responsibilities
- Led hardware selection and PCB schematic review
- Designed the BLE advertisement packet format and RSSI-based positioning algorithm
- Architected LTE-M fallback logic for out-of-coverage scenarios
- Oversaw integration testing and factory acceptance testing (FAT)

---

## 3. Smart Home Automation Gateway

**Folder:** [`Smart-Home-Automation/`](Smart-Home-Automation/)

### Overview
A locally-hosted smart home gateway that bridges Zigbee, Z-Wave, and Wi-Fi devices under a unified MQTT broker, with a React-based web UI and voice-assistant integration.

### Tech Stack
- **Hardware:** Raspberry Pi 4, ConBee II (Zigbee), Z-Stick (Z-Wave)
- **Software:** Python 3, Zigbee2MQTT, Z-Wave JS, Mosquitto MQTT
- **Frontend:** React + TypeScript, REST API (FastAPI)
- **Integrations:** Google Home, Amazon Alexa
- **Team Size:** 3 engineers

### Key Responsibilities
- Defined the device-abstraction layer so new protocols can be added as plugins
- Built the REST API and WebSocket notification system
- Wrote automation rule engine (event-condition-action model)
- Coordinated with QA for regression testing on 20+ device types

---

## 4. Predictive Maintenance Platform

**Folder:** [`Predictive-Maintenance/`](Predictive-Maintenance/)

### Overview
Deployed an edge-AI predictive maintenance solution on CNC machines. Tri-axis accelerometers stream vibration data to an edge device; an on-device ML model (TFLite) classifies bearing health and triggers maintenance tickets.

### Tech Stack
- **Hardware:** STM32H7, ADXL345 accelerometer, NVIDIA Jetson Nano (edge inference)
- **Firmware:** STM32CubeIDE (C), FreeRTOS
- **ML:** Python (scikit-learn, TensorFlow Lite), Edge Impulse
- **Backend:** MQTT → Apache Kafka → PostgreSQL → Kibana
- **Team Size:** 6 engineers

### Key Responsibilities
- Scoped the ML feature-engineering pipeline (FFT, RMS, kurtosis)
- Managed model training, quantisation, and deployment to Jetson Nano
- Defined MQTT topic hierarchy and message schema (Protobuf)
- Ran retrospectives and maintained project backlog in Jira

---

## 5. Smart Energy Metering System

**Folder:** [`Smart-Energy-Metering/`](Smart-Energy-Metering/)

### Overview
Designed a smart metering system that reads energy consumption from existing meters via optical pulse counting and CT clamp sensors, transmits data over LoRaWAN to a cloud platform, and surfaces usage analytics with anomaly alerts.

### Tech Stack
- **Hardware:** STM32L4, SCT-013 CT clamp, RFM95W LoRa module
- **Firmware:** C (STM32CubeIDE), low-power modes (STOP2)
- **Connectivity:** LoRaWAN (TTN / Chirpstack)
- **Backend:** Python (FastAPI), TimescaleDB, Grafana
- **Team Size:** 4 engineers

### Key Responsibilities
- Optimised firmware for < 10 µA sleep current to achieve 5-year battery life
- Designed LoRaWAN payload encoding (Cayenne LPP format)
- Integrated anomaly detection using Z-score on rolling averages
- Coordinated field deployment and wrote installation guide

---

## 🛠 Skills & Tools Used Across Projects

| Category | Technologies |
|----------|-------------|
| Microcontrollers | ESP32, STM32, nRF52, Arduino |
| RTOS | FreeRTOS, Zephyr |
| Languages | C, C++, Python, MicroPython, TypeScript |
| Connectivity | MQTT, LoRaWAN, BLE, Zigbee, LTE-M, Wi-Fi |
| Cloud | AWS IoT Core, Azure IoT Hub, The Things Network |
| Databases | InfluxDB, TimescaleDB, DynamoDB, PostgreSQL |
| Visualisation | Grafana, Kibana, Mapbox GL JS |
| ML / AI | TensorFlow Lite, Edge Impulse, scikit-learn |
| DevOps | Git, Docker, GitHub Actions, Jira |

---

## 📄 Licence

All projects in this repository are shared for portfolio purposes.  
© Aman Ali. All rights reserved.
