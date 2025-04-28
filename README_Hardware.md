# Sailboat Electronics System Documentation

## Overview

This document outlines the electronics hardware architecture, wiring, power management, and control systems used in the Senior Design Autonomous Sailboat Project.  
The project goal is to create a fully autonomous sailboat capable of navigating wind and motor propulsion, remotely monitored and commanded via an onshore communication system.  
All electronics systems were designed, integrated, and tested by George Cicero.

---

## 1. System Architecture Overview

The autonomous sailboat electronics system is structured around three main subsystems:
- **Core Control System**: Pixhawk 2.4.8 flight controller managing servos and motor.
- **Communication Bridge**: Onboard Raspberry Pi connected to Pixhawk via UART, using a LoRa radio HAT to interface with the onshore control system.
- **Power Distribution**: A custom-designed PCB that manages power regulation, fusing, voltage sensing, and noise suppression.

A modular approach was implemented for waterproofing, maintenance, and safe power delivery.

---

## 2. Major Hardware Components

- **Pixhawk 2.4.8 Flight Controller**:  
  Central control unit running ArduPilot sailboat firmware. Manages sensor readings, servo actuation, motor control, and system state.
  
- **Servos**:  
  - **Wing Servo**: 20 kg waterproof servo connected to AUX 2.
  - **Rudder Servo**: 20 kg waterproof servo connected to AUX 1.

- **Motor and ESC**:  
  - **Motor**: Brushed DC motor driven by a 30A Bidirectional Electronic Speed Controller (ESC).
  - **ESC**: Receives PWM signal from Pixhawk Main Out 1. Supports both forward and reverse operation for dynamic maneuvering.

- **Sensors**:
  - **GPS**: ReadyToSky M8N GPS with built-in compass for navigation.
  - **Wind Speed Sensor**: Adafruit Anemometer (Product #1733), interfaced through ADC 3.3V input.
  - **Wind Vane Sensor**: DFRobot Gravity Analog Wind Direction Sensor, interfaced through ADC 6.6V input.

- **Onboard Raspberry Pi**:
  - Handles communication over LoRa to the onshore command station.
  - Connected to Pixhawk via UART, running MAVLink protocol.

- **Power Management System**:
  - **Battery**: 11.1V 3S 5000mAh 15C LiPo battery.
  - **PM02 Power Module**: Provides voltage/current telemetry to Pixhawk and passthrough power to the PCB.
  - **Custom PCB**: Distributes regulated 5V and 12V outputs, protects circuits with fuses, and filters voltage noise.

- **Telemetry Link**:
  - LoRa HAT radio communication between the onboard Raspberry Pi and the onshore system.
  - Onshore website interfaces with real-time telemetry data.

---

## 3. Power Distribution and Protection

- **Battery Connection**:
  - The 11.1V LiPo battery feeds into the PM02 module.
  - The PM02 provides direct passthrough power to the custom PCB and feeds voltage and current telemetry to Pixhawk.

- **Custom PCB Protection**:
  - **10A fuse** protects the motor ESC branch (including buck converters).
  - **1A fuse** protects sensitive electronics (sensors, Raspberry Pi).
  - Voltage is stepped down to 5V and 12V for different components.
  - Capacitor filtering is used to reduce motor-induced back-EMF noise.

- **Buck Converter**:
  - Converts battery voltage down to clean 5V for Raspberry Pi operation.
  - Supplies stable power to sensors.

---

## 4. Wiring Connections

| Connection | Description |
|:-----------|:------------|
| Pixhawk AUX 1 → Rudder Servo | PWM control |
| Pixhawk AUX 2 → Wing Servo | PWM control |
| Pixhawk Main Out 1 → ESC → Motor | PWM motor speed control |
| Pixhawk GPS port → ReadyToSky M8N GPS Module | GPS data input |
| ADC 3.3V input → Wind Speed Sensor | Analog signal |
| ADC 6.6V input → Wind Vane Sensor | Analog signal |
| PM02 Module → Pixhawk | Battery voltage and current sensing |
| PM02 Passthrough → PCB | Raw battery voltage for regulation and distribution |
| PCB → Raspberry Pi (5V supply) | 5V regulated power |
| LoRa HAT → Onboard Raspberry Pi | Radio communication link |

---

## 5. Control System Initialization and Testing

- **Pixhawk Setup**:
  - Fully initialized and configured using Mission Planner.
  - Calibrated accelerometers, gyroscopes, GPS, compass, and ADC sensors.
  - Manual RC transmitter control verified for rudder, wing, and motor.

- **PWM Range Settings**:
  - **Servos (rudder/wing)**:
    - PWM range: 900–1500 μs
    - Center value: 1200 μs
  - Servo endpoints tuned manually to ensure proper mechanical range.

- **Mission Planner Testing**:
  - Sensor readings (GPS, battery voltage, wind sensors) monitored live.
  - Manual servo/motor output testing conducted via Mission Planner interface.

- **Radio Link Testing**:
  - Confirmed MAVLink heartbeat between onboard Raspberry Pi and Pixhawk.
  - Live telemetry successfully received onshore through LoRa radio.

- **Onshore Website Monitoring**:
  - Onshore website interfaces with LoRa telemetry feed to display real-time sensor and system status data.

---

## 6. Waterproofing and Housing

- **Electronics Housing**:
  - All electronics are mounted inside a waterproof electronics box.
  - Cable entry points are sealed using professional-grade cable glands.

- **Battery Housing**:
  - The LiPo battery is isolated in a dedicated battery box to reduce fire/electrical risks.
  - Quick-disconnect connectors are used for rapid removal and maintenance.

- **Environmental Protections**:
  - Conformal coating applied where necessary.
  - Heat shrink tubing and silicone seals used extensively.

---

## 7. Safety and Fail-Safes

- **Power System**:
  - Fusing implemented on both high-current and low-current branches.
  - Capacitors used for voltage noise suppression and back-EMF protection.

- **Control System**:
  - Manual transmitter override available for emergency handover control.
  - GPS lock and voltage warning monitoring configured in Pixhawk for autonomous failsafes.

- **Telemetry Monitoring**:
  - Real-time battery status, GPS position, and control status continuously transmitted to onshore system.

---

## 8. Future Upgrades

- Add secondary GPS receiver for redundancy.
- Integrate current sensors at ESC level for more accurate power monitoring.
- Improve waterproofing using IP68 connectors for mission-critical joints.
- Add secondary LoRa radio channel for telemetry backup.
- Integrate autonomous sailing algorithms beyond simple heading control.

---
