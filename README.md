# Engineering Addendum

Welcome to the WAVES (Wind-Powered Autonomous Vessel) project. This document is intended for any future team taking over development, deployment, or maintenance. It contains lessonswe learned over the last year, the current system state, and pointers to save you weeks of work that has already been done. Please check the two repositories at the bottom of the page for more code. This repo is an overview and documentation repository.

---

## 1. Project Overview

- **Hardware:** 1.22 m hull with 0.85 m wing sail, adjustable keel, Altium-designed PCB, Pixhawk 2.4.8 (Rover firmware), two 20 kg servos (rudder & wing), two Waveshare LoRa radio hats, and two Raspberry Pi 3Bs.  
- **Software:** ArduPilot for autopilot; onshore/offshore Raspberry Pi running Python scripts (pymavlink & Meshtastic API); Express.js backend with Socket.io; React frontend.  
- **Current Capabilities:** One hour, 1 km autonomous missions under motor power; offline manual control via web UI and Ardupilot Mission Control; offline telemetry logging & CSV download.

---

## 2. Quick Start & Deployment

1. **Hardware Assembly:**  
   - Attach keel bulb (4×10-32 screws), then keel to hull, waterproof boxes and servos on mounting plates, control wing + counterweight.
   - Verify waterproof seals (XTC-3D resin & RV tape), cable-gland tightness, and fuse box connections.  
2. **Power-On Sequence:**  
   - **Continuity Check:** Ohm-test battery connector in the electronics box.
   - Connect LiPo (XT60). LEDs on PCB, Pixhawk, and LoRa hat should illuminate.  
3. **Network & UI:**  
   - Power on onshore Pi hotspot (SSID: `MissionPlanner`, PW: `WAVES2025`).  
   - Navigate to `http://onshore.local:3000`.
   - Change network on WiFi page if network access is desired.  
4. **Mission Setup:**  
   - Ensure GPS lock (visible in UI & Mission Planner; LED blinks every 10 s).  
   - Upload waypoints via web UI or CSV.  
   - Select `Sail` or `Motor` autonomous mode; click **Start Mission**.

---

## 3. Lessons Learned

- **LoRa Hat Reliability:** Hat style is less plug and play than USB radios, expect driver quirks and antenna-short circuits if not installed correctly. (VERY IMPORTANT) Always secure the SMA connector before powering on! 
- **Meshtastic Threading:** Original multi-file threading led to deadlocks; consolidated into a single control loop using Python’s `threading` module. 
- **Pixhawk Calibration:** Compass calibration has been unreliable. We pivoted to a new GPS module with built in compass, but a newer module may fix the issue.
- **Power System:** Surges in power from the motor can cause issues in the box. Fuses are there to protect the system so check if the fuse has blown if the motor or servos are not responding to input.
- **Mechanical Wear:** PLA wing joints show fatigue after ~10 missions—consider periodic replacement or metal dowels if high usage expected.  

---

## 4. Extension Points

- **UI Modifications:** Things could look a better in the Web UI and a some reorganizing might improve the user experience. React components live in `frontend/src/pages/` if edits are to be made.
- **Automations Modifitcations:** Consider integrating watchdog timers for thread health; failures currently only log and exit the worker thread after a 10s deadlock.  
- **Hull Modifications:** Constructing the hull with PLA then fiberglassing or using acrylic may help waterproofing and long term reliability. We did see some issues with the keel tension points
- **Wing Modifications:** The wing and keel are a complicated system, further testing must be done to get a vessel that can sail very well. Our boat does repond as expected to wind input but weight and shape changes must be made for better dynamics.

---

## 5. Contacts & Resources

- **Team Members:** Joshua Bardwick (jbardwic@bu.edu), Nyomi Inda (nyomi146@bu.edu), Mateo Balzola (mbalzola@bu.edu), George Cicero (gacicero@bu.edu), Philip Nied (pnied@bu.edu)
- **ArduPilot Docs:** https://ardupilot.org  
- **Meshtastic API:** https://meshtastic.org/docs  
- **Onshore Repository:** [onshore](https://github.com/JoshBard/onshore)
- **Onboard Repository:** [onboard](https://github.com/JoshBard/onboard)
