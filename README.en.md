# Home Assistant Integration for Systemair SAVE VTR-500

> [Les denne guiden på Norsk](README.no.md)

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
2.  **Add the Configuration:** Place the `systemair.yaml` file from this repository into your `/config/packages/` directory.
3.  **Update IP Address:** Open `systemair.yaml` and change the `host` to the static IP address of your Elfin EW11.
4.  **Restart Home Assistant.**

### Step 2.4: Set up the Lovelace Dashboard

The configuration is split into three files. You need to add **three separate "Manual" cards** to your dashboard.

**How-to:**

1.  Navigate to your dashboard and select **"Edit Dashboard"**.
2.  Click **"+ ADD CARD"**, and choose the **"Manual"** card type.
3.  Repeat the process for each of the files below, copying the content from each file into a new "Manual" card.
    *   **Card 1:** `Custom button-card.yaml` (Main control panel)
    *   **Card 2:** `thermostat.yaml` (Thermostat control)
    *   **Card 3:** `type entities.yaml` (Sensor entities)
4.  Save each card and arrange them as you like.

### Step 2.5: Import the Node-RED Flow

1.  Open Node-RED, go to Menu -> Import, and paste the content from `flows.json`.
2.  **IMPORTANT:** Review the new nodes and update the `entity_id`s to match your own humidity and CO2 sensors.
3.  Click "Deploy".
    ![Node-RED Flow](image/Node-Red%20VTR500.png)

### Bonus: How the Night Setback Works

The Node-RED flow includes a night setback logic to save energy and reduce noise. When activated, it lowers the target temperature by ~3°C and sets the fan speed to "Low".

**Important:** This feature does not activate on its own. It is controlled by a `switch` entity in Home Assistant named `switch.nattsenking_ventilasjon_pa`.

To use it, you must create your own automation or script to turn this switch on. Examples:
*   **Via voice assistant:** "Hey Google, activate night mode".
*   **Via a Home Assistant automation:** Turn the switch on at a fixed time.
*   **Via a button on your dashboard.**

The automation will automatically restore normal operation at **04:00 on weekdays** and **06:00 on weekends**.

---

## File Descriptions

*   **`systemair.yaml`**: The main Home Assistant configuration ("package").
*   **`flows.json`**: The Node-RED flow for automation.
*   **`Custom button-card.yaml`**: Code for the main control panel.
*   **`thermostat.yaml`**: Code for the thermostat card.
*   **`type entities.yaml`**: Code for the sensor list.
*   **`/image`**: Screenshots and diagrams used in this guide.

## Acknowledgements and Credits
*   The core configuration (`systemair.yaml`) is based on the fantastic work by **@Ztaeyn** in his [HomeAssistant-VTR-Modbus](https://github.com/Ztaeyn/HomeAssistant-VTR-Modbus) repository.
*   The detailed guide for installation is published on [domotics.no](https://www.domotics.no/post/home-assistant-automasjon-av-ventilasjonsanlegg-via-modbus) and was written by Mads Nedrehagen.
*   The project has been further developed by @Howard0000. An AI assistant helped clean up the `README.md`.

## 📝 License

MIT — see `LICENSE`.
