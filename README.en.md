# Home Assistant Integration for Systemair SAVE VTR-500

> [Les denne guiden på norsk](README.md)
>
⚠️ This is not a HACS integration.
Please follow the manual installation instructions below (YAML files, Lovelace dashboard, and Node-RED flow).

This repository contains a complete configuration to integrate and control a Systemair SAVE VTR-500 ventilation unit with Home Assistant via Modbus TCP.

![Lovelace Dashboard](image/Ventilasjon%20kort.png)

## Features

* **Full mode control:** Control all modes such as Auto, Manual (Low, Normal, High), Party, Boost, Away, Holiday, and Stop.
* **Detailed sensors:** Read temperatures, humidity, fan speeds, heat recovery, and alarms.
* **Temperature control:** Works as a thermostat to adjust the desired intake temperature.
* **Advanced automation:** Uses Node-RED to automatically adjust fan speed based on humidity and CO₂ levels, including night setback.
* **Custom UI:** A functional Lovelace dashboard built with `custom:button-card` and Mushroom Cards.
* **Alarm monitoring:** Displays status for A, B, C, and filter alarms.

## Disclaimer
> This is an unofficial community project and is not developed, supported, or maintained by Systemair. Use at your own risk. For official documentation and support, please see [Systemair’s official website](https://www.systemair.com/).

---

## 1. Requirements

### Hardware
* **Systemair SAVE VTR-500** ventilation unit (or another model with Modbus RS485 support).
* **Modbus RTU to TCP/IP converter:** This guide and configuration use an **Elfin EW11**.

### Software
* A working **Home Assistant** installation.
* **HACS (Home Assistant Community Store)** installed.
* **Node-RED Add-on** installed and configured in Home Assistant.

### HACS Frontend Integrations
Make sure the following are installed via HACS:
* [Mushroom Cards](https://github.com/piitaya/lovelace-mushroom)
* [button-card](https://github.com/custom-cards/button-card)
* [Number Box Card](https://github.com/htmlchinchilla/numberbox-card)

---

## 2. Installation & Configuration

This is a step-by-step guide from the physical setup to finished automation.

### Step 2.1: Physical Installation of the Elfin EW11

> **WARNING:** Always disconnect power to the ventilation unit before opening it. If unsure, consult an electrician.

1. **Locate the Modbus and power terminals:** On the VTR-500 main board, find the external communication terminal labeled `A(+)`, `B(-)`, `24V`, and `GND`.  
   ![VTR-500 wiring diagram](image/koblingsskjemaVTR-500.png)
2. **Wire the Elfin EW11:** Connect the wires as shown in the diagram below.  
   ![EW11 wiring diagram](image/koblings%20skjema%20EW11.png)
3. **Restore power:** When everything is safely connected, power the unit back on.

### Step 2.2: Configure the Elfin EW11

1. **Connect to the EW11’s Wi-Fi:** Connect to the Wi-Fi network `EW1x_...` (no password).
2. **Open the web interface:** Go to `http://10.10.100.254`. Log in with `admin` / `admin`.
3. **Join your home Wi-Fi:** Under “System Settings” → “WiFi Settings”, set “WiFi Mode” to **STA**, find your home network, enter the password, and save.  
   ![System Settings EW11](image/system%20settings%20EW11.png)
4. **Restart and find the new IP:** The device will reboot. Find the new IP address (check your router) and assign it a static IP.
5. **Configure the serial port:** Log in at the new IP. Go to “Serial Port Settings” and apply the values shown below.  
   ![Serial Port Settings EW11](image/serial%20port%20settings%20EW11.png)
6. **Configure communication:** Go to “Communication Settings” and add a new profile as shown below.  
   ![Communication Settings EW11](image/communication%20settings%20EW11.png)
7. **Verify:** Open the “Status” page. The packet counters should start increasing.  
   ![EW11 communication status](image/kommunikasjon%20EW11.png)

### Step 2.3: Home Assistant Configuration

1. **Enable “packages”:** Ensure your `configuration.yaml` contains:
   ```yaml
   homeassistant:
     packages: !include_dir_named packages
   ```
2. **Add the configuration:** Copy `packages/systemair.yaml` from this repo into your `/config/packages/` directory.
3. **Update the IP address:** Open `packages/systemair.yaml` and set `host` to the static IP address of your Elfin EW11.
4. **Restart Home Assistant.**

### Step 2.4: Set Up the Lovelace Dashboard

This project provides the dashboard configuration in both Norwegian and English.

1. Choose your preferred language by opening either `lovelace/no/` or `lovelace/en/`.
2. Inside your chosen folder you’ll find three YAML files.
3. In your Home Assistant dashboard, add **three separate “Manual” cards**.
4. Copy the contents of each YAML file into its own Manual card.

### Step 2.5: Import the Node-RED Flow

1. Choose your preferred language. Open `node-red/no/flows.json` or `node-red/en/flows.json`. Copy the entire file content.
2. In Node-RED, go to Menu → Import, and paste the content.
3. **IMPORTANT:** Review the new nodes and update `entity_id` to your own humidity and CO₂ sensors.
4. Click “Deploy”.  
   ![Node-RED Flow](image/Node-Red%20VTR500.png)

### Bonus: How Night Setback Works

The Node-RED flow includes built-in logic for night setback. When activated, it lowers the temperature by approx. 3 °C and sets the fan speed to “Low”.

**Important:** This function does not activate by itself. It is controlled by a `switch` entity created by the Node-RED flow.
* If you used the Norwegian `flows.json`, the entity is `switch.nattsenking_ventilasjon_pa`.
* If you used the English `flows.json`, the entity is `switch.night_setback_ventilation_on`.

To use it, create an automation or script that turns this switch on.

---

## File Overview

* **`packages/systemair.yaml`** – Main configuration for all sensors, switches, and scripts (English).
* **`lovelace/no/` & `lovelace/en/`** – The three YAML files for the Lovelace dashboard (Norwegian/English).
* **`node-red/no/` & `node-red/en/`** – `flows.json` for advanced automation (Norwegian/English).
* **`image/`** – Screenshots and diagrams used in this guide.
* **`README.md`** – This guide in Norwegian.
* **`README.en.md`** – This guide in English.

## Acknowledgements & Credits
* The core configuration (`systemair.yaml`) is based on the work by **@Ztaeyn** in the [HomeAssistant-VTR-Modbus](https://github.com/Ztaeyn/HomeAssistant-VTR-Modbus) repository.
* The installation guide is published on [domotics.no](https://www.domotics.no/post/home-assistant-automasjon-av-ventilasjonsanlegg-via-modbus) and written by Mads Nedrehagen.
* The project is further developed by @Howard0000. An AI assistant helped edit the `README.md`.

## 📝 License
MIT — see `LICENSE`.

