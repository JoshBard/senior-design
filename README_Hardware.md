# Sailboat Electronics Documentation

## Overview
This document describes the electronic components and wiring used in the senior design autonomous sailboat project. It outlines key modules, specifications, and electrical connections necessary for successful operation.  
The custom-designed power distribution PCB is included in this repository.

---

## Major Electronic Components

- **Wing Servo**: 20 kg high-torque waterproof servo
- **Rudder Servo**: 20 kg high-torque waterproof servo
- **Wind Speed Sensor**: Adafruit Anemometer (Product #1733)  
  [Adafruit Anemometer Link](https://www.adafruit.com/product/1733?gQT=1)
- **Wind Vane Sensor**: DFRobot Gravity Analog Wind Direction Sensor  
  [DFRobot Wind Vane Link](https://www.dfrobot.com/product-2340.html)
- **GPS Module**: ReadyToSky M8N GPS with built-in compass
- **Flight Controller**: Pixhawk 2.4.8 (running ArduPilot Sailboat firmware)
- **Battery**: 11.1V 3S 5000mAh LiPo battery
- **ESC**: 30A Bidirectional Electronic Speed Controller (ESC) for motor control
- **Power Management**: Custom PCB for regulated 5V/12V outputs and reverse polarity protection
- **Onboard Communication Computer**: Raspberry Pi (handles long-range communication between the sailboat and an onshore Raspberry Pi)

---

## System Architecture

- The **Pixhawk** acts as the main onboard computer, handling navigation, sail, rudder, and motor control.
- The **onboard Raspberry Pi** is used **only for radio communication** with the **onshore Raspberry Pi**.
- Commands from the onshore Raspberry Pi are relayed to the Pixhawk through the onboard Raspberry Pi over a wireless link.

---

## Wiring Connections

- **Pixhawk → Rudder Servo**: AUX 1
- **Pixhawk → Wing Servo**: AUX 2
- **Pixhawk → Motor ESC**: Main Out 1
- **Pixhawk → GPS Module**: GPS port
- **Wind Speed Sensor → Pixhawk**: ADC 3.3V input
- **Wind Vane Sensor → Pixhawk**: ADC 6.6V input
- **Battery → PM02 Power Module** → 
  - **Passthrough Power to PCB**
  - **Voltage/Current sensing to Pixhawk**
- **Power PCB**:
  - Steps down battery voltage to 5V to power the onboard Raspberry Pi
  - Provides power and drive voltage for the 30A ESC and motor

---

## Assembly Notes

- Use heat shrink tubing and silicone sealant for waterproofing all electrical joints.
- Secure battery, PM02 module, and ESC connections with proper strain relief and foam padding.
- Use corrosion-resistant connectors and marine-grade wiring wherever possible.
- Validate sensor signal conditioning (voltage levels, noise filtering) before flight controller integration.
- Ensure Raspberry Pi communication interfaces (UART, Ethernet, or Wi-Fi) are correctly configured and tested.

---

## Future Electronics Upgrades (Optional)

- Add current/voltage sensors for real-time monitoring.
- Upgrade servos to higher waterproof IP-rated models.
- Add backup GPS module for redundancy.
- Explore redundant communication links for fail-safe operation.

---
