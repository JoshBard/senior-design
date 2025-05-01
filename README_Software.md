# Raspberry Pi and Meshtastic Setup Instructions

## Flashing Raspberry Pi OS
1. Download the [Raspberry Pi Imager](https://www.raspberrypi.com/software/) for your computer.
2. Get at least a 16GB microSD card and flash the Raspberry Pi OS (64-bit) to the SD card.
3. Put in your local network details and get the IP address of the Raspberry Pi from your network's static table.
4. SSH in with the username you created in the options step of the Raspberry Pi imager.

## Hardware Setup
1. Attach your [Waveshare LoRa hat](https://www.waveshare.com/sx1262-lorawan-hat.htm?sku=22002) to the Raspberry Pi.
2. Make sure that the antenna is hooked up to the hat before you power it on; otherwise, you will short the radio board immediately.

## Meshtastic Firmware Installation
1. Once the board is powered on, copy [this](https://github.com/meshtastic/firmware/releases/download/v2.5.18.89ebafc/meshtasticd_2.5.18.89ebafc_arm64.deb) Meshtastic firmware link and use `wget` to install it on your Raspberry Pi.
2. Run:
    ```bash
    sudo apt update
    sudo apt upgrade
    ```
3. Then:
    ```bash
    sudo apt install ./meshtastic_library.deb
    ```

## SPI Configuration
1. Edit the boot config:
    ```bash
    sudo nano /boot/formward/config.txt
    ```
2. Uncomment:
    ```
    dtparam=spi=on
    ```
3. Add the line right underneath:
    ```
    dtoverlay=spi0-0cs
    ```
4. Reboot:
    ```bash
    sudo reboot
    ```
5. Check that `spidev0.0` is visible:
    ```bash
    ls /dev/
    ```

## Meshtastic Daemon Configuration
1. Edit:
    ```bash
    sudo nano /etc/meshtasticd/config.yaml
    ```
2. Look for the `sx126x` entry, change that to `sx1262`, and remove the hashtags up to the line that says `Reset: 18`.  
3. Do not change anything else.
4. Enable the service:
    ```bash
    sudo systemctl enable meshtasticd --now
    ```

If you have further questions, refer to this tutorial.

## GitHub Repositories
1. Make sure the time is set on the Raspberry Pi before you clone or GitHub access will fail.
2. Copy the GitHub repositories:
   - For the onboard node, refer to this GitHub: `[https://github.com/JoshBard/onboard]`
   - For the onshore node, refer to this GitHub: `[Onshore GitHub Link]`

## Python Environment Setup
1. Python3 is already installed with Raspberry Pi OS.
2. Set up a virtual environment:
    ```bash
    python3 -m venv venv
    source venv/bin/activate
    pip install -r requirements.txt
    ```

## Utilities Setup
1. Head to the utilities folder.
2. Make the setup script executable:
    ```bash
    chmod +x meshtastic_setup.sh
    ./meshtastic_setup.sh
    ```

## Channel and Node Setup
1. Run:
    ```bash
    meshtastic --info
    ```
2. Pull the URL at the bottom from one node.
3. On the other node, run:
    ```bash
    meshtastic --seturl YOUR_URL
    ```
4. Verify both nodes:
    ```bash
    meshtastic --nodes
    ```
5. If you do not see both nodes, make sure they are on the same channel and URL.  
   Check again with:
    ```bash
    meshtastic --info
    ```

You can also check the status of the control service with:
```bash
sudo systemctl status meshtastic
```

## Control Service Setup
1. On each node, copy the `control.service` file from utilities into `/etc/systemd/`.
2. Run:
    ```bash
    sudo systemctl enable control.service
    ```

## Onshore Node: Frontend and Backend Setup
### Frontend
1. Head to `onshore/frontend`.
2. Run:
    ```bash
    npm install
    npm run build
    ```

### Backend
1. Head to `onshore/backend`.
2. Run:
    ```bash
    npm install
    ```

## Sudo Permissions Without Password
1. Go to `/etc/sudoers.d/` and create a new file.
2. Use:
    ```bash
    sudo visudo /etc/sudoers.d/11_custom
    ```
   (Pick a filename with a prefix greater than any existing file.)
3. Add this line (adjust USERNAME and path):
    ```
    USERNAME ALL=(ALL:ALL) NOPASSWD: /path/to/change_wifi.sh
    ```
4. Confirm with:
    ```bash
    sudo -l
    ```
You should see your new entry at the end of the sudoers list.

## Setup Hotspot
1. Plug in a mouse, keyboard, and monitor to the onshore Raspberry Pi.
2. Create a hotspot named `MissionPlanner` with the password `WAVES2025`.

# You are now all set!


# Project Architecture and Communication Overview

Let's dive into the overall setup of the system!

On the onshore node, the GitHub repository is split into two parts: the frontend and backend. The frontend contains the page components which are imported by the main App.js, which maintains the banner and alert messages across the entire Web UI. This is a straightforward implementation of a frontend, you can see all pages and components in `frontend/src/`. To get this setup, do:

```bash
npm install
npm run build
```

while in `frontend/` after you clone the repository. It is hosted on `onshore.local:3000`. The frontend uses Axios and Websockets to communicate with the backend. WebSockets is only used on the manual page for more responsive manual commands.

The backend server and messaging code are in `onshore/backend`. `server.js` hosts the backend and is an Express server running on the same `onshore.local:3000`. To run the server, do:

```bash
npm install
```
after you clone the repo, then:

```bash
npm start
```
anytime you want to spin up the server after that. 

The server makes Axios posts to the Flask server running on `localhost:5000` from `messaging.py`. `messaging.py` handles all LoRa transmissions using the Python Meshtastic API. `messaging.py` logs with `receive_logger.py` and `transmit_logger.py`. `messaging.py` also posts messages to the server with Axios when it receives messages. Configs for the `messaging.py` are held in `config.py`, and all Meshtastic relevant scripts are in `backend/messaging/`. The startup of all of these scripts is handled by `control.service`, so you do not need to do this manually when you are in normal use.

On the backend, there is notably a Google Maps API key that you will need to use. Place it in the `.env` file in `backend/` once you have created your own, otherwise the frontend will not display a map. Make sure you put `/backend/.env` in your `.gitignore`. It is very important that you do not publicly share your Google Maps API key!

## Communication

Communication in this project is important, especially over LoRa. There are five message types:

- **MAN** for manual commands.
- **MSSN** for mission commands like start autopilot, arm, and enable sailing.
- **WP** for waypoint uploads.  
  Latitude and longitude are sent line by line.  
  The final message is **WP_FINISHED** to inform the onboard node it can upload to the flight controller.
- **TLM** for the periodic telemetry updates from the onboard node.  
  Required every 2 minutes for the web UI to stay up.  
  Status is in `messaging/connection_status.txt`.
- **STAT** contains return codes from the onshore, so the user knows if their commands were successful.

## Onboard Node

In the onboard repository, there is one main control loop, also called `messaging.py`. There is no Axios usage in the onboard node; it just takes the messages in using the Meshtastic Python API and queues them for pymavlink, which allows communication with the flight controller. Important configuration settings for `messaging.py` are held in `config.py`. It provides logged messages with `receive_logger.py` and `transmit_logger.py`, similarly to the onshore node.

All logic is in one file (`messaging.py`) because it provides a simple approach to thread management. Originally, there were more modular structures in place, but I found that using something like ZeroMQ was too much overhead for something simple, and that managing threads in one place made more sense.

In `messaging.py`, we use a file called `.processed_msg_ids`, which allows us to avoid processing the same message twice. This is true in both onshore and onboard nodes.

All pymavlink files are in one folder, `mavlink_scripts/`. These return a status that is passed back over LoRa to the onshore node. They communicate with the flight controller, a Pixhawk 2.4.8 over `/dev/serial0`, a wired connection. This provides better reliability while out on the water. These files allow us to pull relevant telemetry data, manually control the vessel, upload waypoints, and change important mission parameters.

Essentially, we extracted the relevant Ardupilot settings and presented them for the everyday user. If a user needs to get more specific, they can download Ardupilot Mission Planner and interface over our SiK radios. This code is not part of our repository as it is an open-source project maintained [here](https://github.com/ArduPilot/MissionPlanner).

## Repository Structure

Overall, the structure of the repositories, regardless of the node, is consistent:

- Anything Meshtastic is in the `messaging/` folder.
- There are universal utilities held in the `utilities/` folder.
- Other folders will be named with their use, like `mavlink_scripts/`, `waypoints/`, `frontend/`, or `backend/`.

Setup instructions are granular and cover every step required to boot up the web UI.

Please reach out to me, Josh Bardwick, at [josh.bardwick@gmail.com](mailto:josh.bardwick@gmail.com) if you have any questions!

Refer to the diagram `software_communications` in the `reports/` folder for a better top-down view of the system.