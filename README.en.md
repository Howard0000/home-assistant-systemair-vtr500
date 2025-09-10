# Home Assistant Integration for Systemair SAVE VTR-500

> [Read this guide in Norwegian (Les denne guiden på Norsk)](README.no.md)

This repository provides a complete configuration for integrating and controlling a Systemair SAVE VTR-500 ventilation unit with Home Assistant via Modbus TCP.

![Lovelace Dashboard](image/Ventilasjon%20kort.png)

## Features

*   **Full Mode Control:** Manage all modes like Auto, Manual (Low, Normal, High), Party, Boost, Away, Holiday, and Stop.
*   **Detailed Sensors:** Read data for temperatures, humidity, fan speeds, heat recovery, and alarms.
*   **Thermostat Control:** Functions as a thermostat to adjust the desired supply air temperature.
*   **Advanced Automation:** Uses Node-RED to automatically adjust fan speed based on humidity and CO2 levels, including a night setback feature.
*   **Custom UI:** A functional Lovelace dashboard built with `custom:button-card` and Mushroom Cards.
*   **Alarm Monitoring:** Displays the status of A, B, C, and filter alarms.

## Disclaimer
> This is an unofficial community project and is not developed, supported, or maintained by Systemair. All configuration and use are at your own risk. For official documentation and support, please refer to [Systemair's official websites](https://www.systemair.com/).

### Installation via HACS (Recommended Method)

While this is not a standard integration, you can add this repository to HACS to easily track updates.

1.  In Home Assistant, navigate to **HACS** -> **Integrations**.
2.  Click the three dots in the top right corner and select **"Custom repositories"**.
3.  In the "Repository" field, paste this URL:
    `https://github.com/Howard0000/ha-systemair-vtr500`
4.  For "Category", select **"Integration"**.
5.  Click **"ADD"**.
6.  The repository will now appear in your HACS integrations list. Click on it and then **"DOWNLOAD"**.
7.  This only downloads the project files. You must still follow the manual configuration steps below to get it working.
---

## 1. Requirements

### Hardware
*   **Systemair SAVE VTR-500** ventilation unit (or another model with Modbus RS485 support).
*   **Modbus RTU to TCP/IP Converter:** This guide and configuration use an **Elfin EW11**.

### Software
*   A working **Home Assistant** installation.
*   **HACS (Home Assistant Community Store)** installed.
*   **Node-RED Add-on** installed and configured in Home Assistant.

### HACS Frontend Integrations
Ensure the following are installed via HACS:
*   [Mushroom Cards](https://github.com/piitaya/lovelace-mushroom)
*   [button-card](https://github.com/custom-cards/button-card)
*   [Number Box Card](https://github.com/htmlchinchilla/numberbox-card)

---

## 2. Installation and Configuration

A step-by-step guide to take you from physical installation to a fully automated system.

### Step 2.1: Physical Installation of the Elfin EW11

> **WARNING:** Always disconnect the power to the ventilation unit before opening it. If you are uncertain, consult an electrician.

1.  **Locate the Modbus and Power Port:** On the main board of the VTR-500, find the terminal for external communication, marked `A(+)`, `B(-)`, `24V`, and `GND`.
    ![Wiring Diagram VTR-500](image/koblingsskjemaVTR-500.png)
2.  **Connect the Elfin EW11:** Wire the connections as shown in the diagram below.
    ![Wiring Diagram EW11](image/koblings%20skjema%20EW11.png)
3.  **Restore Power:** Once everything is securely connected, turn the power back on.

### Step 2.2: Configure the Elfin EW11

1.  **Connect to the EW11's Network:** Connect to its Wi-Fi network `EW1x_...` (no password required).
2.  **Open the Web UI:** Go to `http://10.10.100.254`. Log in with `admin` / `admin`.
3.  **Connect to Your Wi-Fi:** Under "System Settings" -> "WiFi Settings", set "Wifi Mode" to "STA", find your home network, enter the password, and save.
    ![System Settings EW11](image/system%20settings%20EW11.png)
4.  **Restart and Find New IP:** The device will restart. Find its new IP address (check your router's DHCP list) and set a static IP for it.
5.  **Configure Serial Port:** Log in to the new IP address. Go to "Serial Port Settings" and apply the settings shown below.
    ![Serial Port Settings EW11](image/serial%20port%20settings%20EW11.png)
6.  **Configure Communication:** Go to "Communication Settings" and add a new profile as shown below.
    ![Communication Settings EW11](image/communication%20settings%20EW11.png)
7.  **Verify:** Go to the "Status" page. The data packet counters should now be increasing, confirming that communication is working.
    ![Communication EW11](image/kommunikasjon%20EW11.png)

### Step 2.3: Home Assistant Configuration

1.  **Enable Packages:** Ensure your `configuration.yaml` contains:
    ```yaml
    homeassistant:
      packages: !include_dir_named packages
    ```
2.  **Add the Configuration:** Copy the file `packages/systemair.yaml` from this repository into your `/config/packages/` directory.
3.  **Update IP Address:** Open the `packages/systemair.yaml` file and change the `host` to the static IP address of your Elfin EW11.
4.  **Restart Home Assistant.**

### Step 2.4: Set up the Lovelace Dashboard

This project provides the dashboard configuration in both English and Norwegian.

1.  Choose your preferred language by navigating into either the `lovelace/en/` or `lovelace/no/` folder.
2.  Inside your chosen folder, you will find three YAML files.
3.  On your Home Assistant dashboard, add **three separate "Manual" cards**.
4.  Copy the content from each of the three YAML files into its own "Manual" card.

### Step 2.5: Import the Node-RED Flow

1.  Choose your preferred language. Open the file `node-red/en/flows.json` or `node-red/no/flows.json`. Copy the entire content of the file.
2.  Open Node-RED, go to Menu -> Import, and paste the content.
3.  **IMPORTANT:** Review the new nodes and update the `entity_id`s to match your own humidity and CO2 sensors.
4.  Click "Deploy".
    ![Node-RED Flow](image/Node-Red%20VTR500.png)

### Bonus: How the Night Setback Works

The Node-RED flow includes a night setback logic to save energy and reduce noise. When activated, it lowers the target temperature by ~3°C and sets the fan speed to "Low".

**Important:** This feature does not activate on its own. It is controlled by a `switch` entity created by the Node-RED flow.
*   If you used the English `flows.json`, the entity is `switch.night_setback_ventilation_on`.
*   If you used the Norwegian `flows.json`, the entity is `switch.nattsenking_ventilasjon_pa`.

To use it, you must create your own automation or script to turn this switch on.

---

## File Descriptions

*   **`packages/systemair.yaml`**: The core configuration for all sensors, switches, and scripts. This file is in English.
*   **`lovelace/en/` & `lovelace/no/`**: Contains the three YAML files for the Lovelace dashboard, provided in both English and Norwegian.
*   **`node-red/en/` & `node-red/no/`**: Contains the `flows.json` file for advanced automation, provided in both English and Norwegian.
*   **`images/`**: Screenshots and diagrams used in this guide.
*   **`README.md`**: This guide in English.
*   **`README.no.md`**: This guide in Norwegian.

## Acknowledgements and Credits
*   The core configuration (`systemair.yaml`) is based on the fantastic work by **@Ztaeyn** in his [HomeAssistant-VTR-Modbus](https://github.com/Ztaeyn/HomeAssistant-VTR-Modbus) repository.
*   The detailed guide for installation is published on [domotics.no](https://www.domotics.no/post/home-assistant-automasjon-av-ventilasjonsanlegg-via-modbus) and was written by Mads Nedrehagen.
*   The project has been further developed by @Howard0000. An AI assistant helped clean up the `README.md`.

## 📝 License

MIT — see `LICENSE`.
