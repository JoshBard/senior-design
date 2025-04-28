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
   - For the onshore node, refer to this GitHub: `[https://github.com/JoshBard/onshore]`

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
