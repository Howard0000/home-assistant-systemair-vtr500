# Home Assistant-integrasjon for Systemair SAVE VTR-500

Dette repositoriet inneholder en komplett konfigurasjon for å integrere og styre en Systemair SAVE VTR-500 ventilasjonsenhet med Home Assistant via Modbus TCP. Konfigurasjonen inkluderer detaljert sensoravlesning, full modus-styring, avanserte automasjoner i Node-RED, og et polert Lovelace-dashboard.

Prosjektet er basert på en detaljert guide som kan finnes her:
*   [Home Assistant Automasjon av Ventilasjonsanlegg via Modbus (domotics.no)](https://www.domotics.no/post/home-assistant-automasjon-av-ventilasjonsanlegg-via-modbus)

![Lovelace Dashboard](screenshot.png)
<!-- Du bør erstatte screenshot.png med et faktisk bilde av ditt dashboard -->

## Funksjoner

*   **Full modus-styring:** Kontroller alle moduser som Auto, Manuell (Lav, Normal, Høy), Party, Boost, Borte, Ferie og Stopp.
*   **Detaljerte sensorer:** Leser av temperaturer, fuktighet, viftehastigheter, varmegjenvinning og alarmer.
*   **Temperaturkontroll:** Fungerer som en termostat for å justere ønsket inntakstemperatur.
*   **Avansert automasjon:** Bruker Node-RED til å justere viftehastigheten automatisk basert på fuktighets- og CO2-nivåer.
*   **Tilpasset brukergrensesnitt:** Et funksjonelt Lovelace-dashboard bygget med `custom:button-card` og Mushroom Cards.
*   **Alarm-overvåking:** Viser status på A-, B-, C- og filter-alarmer.

---

## 1. Krav

### Maskinvare
*   **Systemair SAVE VTR-500** ventilasjonsenhet (eller en annen modell med Modbus RS485-støtte).
*   **Modbus RTU til TCP/IP konverter:** Guiden og denne konfigurasjonen bruker en **Elfin EW11**. Dette er en kritisk komponent som fungerer som en bro mellom anleggets RS485-port og ditt trådløse nettverk.

### Programvare
*   En fungerende **Home Assistant**-installasjon.
*   **HACS (Home Assistant Community Store)** installert.
*   **Node-RED Add-on** installert og konfigurert i Home Assistant.

### HACS Frontend-integrasjoner
Sørg for at følgende er installert via HACS:
*   [Mushroom Cards](https://github.com/piitaya/lovelace-mushroom)
*   [button-card](https://github.com/custom-cards/button-card)
*   [Number Box Card](https://github.com/htmlchinchilla/numberbox-card)

---

## 2. Installasjon og Konfigurasjon

Dette er en trinnvis guide som tar deg fra fysisk installasjon til ferdig automasjon.

### Trinn 2.1: Fysisk Installasjon av Elfin EW11

> **ADVARSEL:** Alltid koble fra strømmen til ventilasjonsanlegget før du åpner det og jobber med koblinger. Hvis du er usikker på strøm og koblinger, spesielt "svakstrøm", bør du konsultere en elektriker.

1.  **Finn Modbus-porten:** På hovedkortet til VTR-500, finn Modbus-terminalen. Den er vanligvis merket med `A(+)` og `B(-)`.
2.  **Finn Strømkilde:** Elfin EW11 trenger 5-36V strøm. VTR-500 hovedkortet har ofte en 24V-utgang som kan brukes. Finn en terminal merket `24V` og `GND` (jord).
3.  **Koble til Elfin EW11:**
    *   Koble `A+` fra anlegget til `A` på EW11.
    *   Koble `B-` fra anlegget til `B` på EW11.
    *   Koble `24V` fra anlegget til `+` (Power Supply) på EW11.
    *   Koble `GND` fra anlegget til `-` (Power Supply) på EW11.

<!-- Foreslå å legge til bilde av koblingsskjema her, likt det du har i guiden -->
![Koblingsskjema](wiring_diagram.png)

4.  **Gjenopprett strømmen:** Når alt er trygt koblet, kan du slå på strømmen til ventilasjonsanlegget. Du skal nå se at lysene på Elfin EW11-enheten slår seg på.

### Trinn 2.2: Konfigurere Elfin EW11

EW11 må konfigureres til å koble seg på ditt nettverk og oversette Modbus-signalene korrekt.

1.  **Koble til EW11s nettverk:** EW11 vil opprette sitt eget Wi-Fi-nettverk (f.eks. `EW1x_...`). Koble deg til dette fra en PC eller mobil. Det krever ikke passord.
2.  **Åpne web-grensesnitt:** Åpne en nettleser og gå til `http://10.10.100.254`. Logg inn med standard brukernavn `admin` og passord `admin`.
3.  **Koble til ditt Wi-Fi:** Under "System Settings", finn "WiFi Settings". Sett "Wifi Mode" til "STA", finn ditt hjemmenettverk (SSID), skriv inn passordet og lagre.
4.  **Restart og finn ny IP:** Enheten vil restarte og koble seg til ditt nettverk. Finn den nye IP-adressen den har fått (sjekk i DHCP-listen på ruteren din). **Det anbefales sterkt å sette en statisk IP-adresse for enheten.**
5.  **Konfigurer serieport (Serial Port Settings):** Logg inn på den nye IP-adressen. Gå til "Serial Port Settings" og sett verdiene til å matche anlegget ditt. For Systemair VTR-500 er disse vanligvis:
    *   **Baud Rate:** `115200`
    *   **Data Bit:** `8`
    *   **Parity:** `Even`
    *   **Stop Bit:** `1`
    *   **Protocol:** `Modbus`
6.  **Konfigurer kommunikasjon (Communication Settings):**
    *   Gå til "Communication Settings" og slett eventuelle eksisterende profiler.
    *   Legg til en ny med "+Add".
    *   **Protocol:** `Tcp Server`
    *   **Local Port:** `502` (standard for Modbus TCP)
7.  **Verifiser:** Gå til "Status"-siden. Hvis alt er riktig, vil du se at tellerne for "Received Bytes/Frames" og "Sent Bytes/Frames" begynner å øke. Dette bekrefter at kommunikasjonen mellom EW11 og ventilasjonsanlegget fungerer!

### Trinn 2.3: Konfigurasjon i Home Assistant

1.  **Aktiver "Packages":** Sørg for at Home Assistant er satt opp til å laste inn pakker. Legg til følgende i din `configuration.yaml`-fil:
    ```yaml
    homeassistant:
      packages: !include_dir_named packages
    ```
2.  **Opprett mappe:** Lag en mappe som heter `packages` i din `/config`-mappe.
3.  **Legg til konfigurasjonen:** Lagre filen `systemair.txt` fra dette repoet som **`systemair.yaml`** inne i `/config/packages/`-mappen.
4.  **Oppdater IP-adressen:** Åpne `config/packages/systemair.yaml` og endre `host` til den statiske IP-adressen du satte for din Elfin EW11.
5.  **Start Home Assistant på nytt.**

### Trinn 2.4: Sett opp Lovelace Dashboard

1.  Åpne et dashboard, gå i redigeringsmodus, velg "Manuell"-kort og lim inn innholdet fra `Custom button-card.txt`.

### Trinn 2.5: Importer Node-RED Flow

1.  Åpne Node-RED, gå til Meny -> Import, og lim inn innholdet fra `flows.json`.
2.  **VIKTIG:** Gå gjennom de nye nodene og oppdater `entity_id` for dine fukt- og CO2-sensorer.
3.  Klikk "Deploy".

Du er nå ferdig! Du har et fullt fungerende, intelligent ventilasjonssystem styrt av Home Assistant.