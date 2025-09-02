# Home Assistant-integrasjon for Systemair SAVE VTR-500

Dette repositoriet inneholder en komplett konfigurasjon for å integrere og styre en Systemair SAVE VTR-500 ventilasjonsenhet med Home Assistant via Modbus TCP.

![Lovelace Dashboard](image/Ventilasjon%20kort.png)

## Funksjoner

*   **Full modus-styring:** Kontroller alle moduser som Auto, Manuell (Lav, Normal, Høy), Party, Boost, Borte, Ferie og Stopp.
*   **Detaljerte sensorer:** Leser av temperaturer, fuktighet, viftehastigheter, varmegjenvinning og alarmer.
*   **Temperaturkontroll:** Fungerer som en termostat for å justere ønsket inntakstemperatur.
*   **Avansert automasjon:** Bruker Node-RED til å justere viftehastigheten automatisk basert på fuktighets- og CO2-nivåer.
*   **Tilpasset brukergrensesnitt:** Et funksjonelt Lovelace-dashboard bygget med `custom:button-card` og Mushroom Cards.
*   **Alarm-overvåking:** Viser status på A-, B-, C- og filter-alarmer.

## Ansvarsfraskrivelse (Disclaimer)
> Dette er et uoffisielt community-prosjekt og er ikke utviklet, støttet eller vedlikeholdt av Systemair. All konfigurasjon og bruk skjer på eget ansvar. For offisiell dokumentasjon og support, vennligst se [Systemairs offisielle nettsider](https://www.systemair.com/).

---

## 1. Krav

### Maskinvare
*   **Systemair SAVE VTR-500** ventilasjonsenhet (eller en annen modell med Modbus RS485-støtte).
*   **Modbus RTU til TCP/IP konverter:** Guiden og denne konfigurasjonen bruker en **Elfin EW11**.

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

> **ADVARSEL:** Alltid koble fra strømmen til ventilasjonsanlegget før du åpner det. Hvis du er usikker, bør du konsultere en elektriker.

1.  **Finn Modbus- og strøm-porten:** På hovedkortet til VTR-500, finn terminalen for ekstern kommunikasjon, merket med `A(+)`, `B(-)`, `24V` og `GND`.
    ![Koblingsskjema VTR-500](image/koblingsskjemaVTR-500.png)
2.  **Koble til Elfin EW11:**
    *   Koble `A+` fra anlegget til `A` på EW11.
    *   Koble `B-` fra anlegget til `B` på EW11.
    *   Koble `24V` fra anlegget til `+` (Power Supply) på EW11.
    *   Koble `GND` fra anlegget til `-` (Power Supply) på EW11.
    ![Koblingsskjema EW11](image/koblings%20skjema%20EW11.png)
3.  **Gjenopprett strømmen:** Når alt er trygt koblet, slå på strømmen til anlegget.

### Trinn 2.2: Konfigurere Elfin EW11

1.  **Koble til EW11s nettverk:** Koble til Wi-Fi-nettverket `EW1x_...` (ikke passord).
2.  **Åpne web-grensesnitt:** Gå til `http://10.10.100.254`. Logg inn med `admin` / `admin`.
3.  **Koble til ditt Wi-Fi:** Under "System Settings" -> "WiFi Settings", sett "Wifi Mode" til "STA", finn ditt hjemmenettverk, skriv inn passord og lagre.
    ![System Settings EW11](image/system%20settings%20EW11.png)
4.  **Restart og finn ny IP:** Enheten vil restarte. Finn den nye IP-adressen den har fått (sjekk i ruteren din) og sett en statisk IP for den.
5.  **Konfigurer serieport:** Logg inn på den nye IP-adressen. Gå til "Serial Port Settings" og sett:
    *   **Baud Rate:** `115200`
    *   **Data Bit:** `8`
    *   **Parity:** `Even`
    *   **Protocol:** `Modbus`
    ![Serial Port Settings EW11](image/serial%20port%20settings%20EW11.png)
6.  **Konfigurer kommunikasjon:** Gå til "Communication Settings" og legg til en ny profil med:
    *   **Protocol:** `Tcp Server`
    *   **Local Port:** `502`
    ![Communication Settings EW11](image/communication%20settings%20EW11.png)
7.  **Verifiser:** Gå til "Status"-siden. Telleverk for datapakker skal nå øke, som bekrefter at kommunikasjonen fungerer.
    ![Kommunikasjon EW11](image/kommunikasjon%20EW11.png)

### Trinn 2.3: Konfigurasjon i Home Assistant

1.  **Aktiver "Packages":** Sørg for at `configuration.yaml` inneholder:
    ```yaml
    homeassistant:
      packages: !include_dir_named packages
    ```
2.  **Legg til konfigurasjonen:** Plasser `systemair.yaml` fra dette repoet i mappen `/config/packages/`.
3.  **Oppdater IP-adressen:** Åpne `systemair.yaml` og endre `host` til den statiske IP-adressen til din Elfin EW11.
4.  **Start Home Assistant på nytt.**

### Trinn 2.4: Sett opp Lovelace Dashboard

1.  Åpne et dashboard, gå i redigeringsmodus, velg "Manuell"-kort og lim inn innholdet fra `Custom button-card.yaml`.

### Trinn 2.5: Importer Node-RED Flow

1.  Åpne Node-RED, gå til Meny -> Import, og lim inn innholdet fra `flows.json`.
2.  **VIKTIG:** Gå gjennom de nye nodene og oppdater `entity_id` til dine egne fukt- og CO2-sensorer.
3.  Klikk "Deploy".
    ![Node-RED Flow](image/Node-Red%20VTR500.png)

---

## Filforklaring

*   **`systemair.yaml`**: Hovedkonfigurasjonen for Home Assistant ("package").
*   **`flows.json`**: Node-RED-flyt for automasjon.
*   **`Custom button-card.yaml`**: Hovedfilen for Lovelace-dashboardet.
*   **`thermostat.yaml` / `type entities.yaml`**: Støttefiler for dashboardet.
*   **`/image`**: Skjermbilder og diagrammer brukt i denne guiden.

## Anerkjennelser og Credits
Dette prosjektet hadde ikke vært mulig uten arbeidet til andre i Home Assistant-miljøet.
*   Kjernekonfigurasjonen (`systemair.yaml`) er basert på det fantastiske arbeidet gjort av **@Ztaeyn**. Hans repositorium [HomeAssistant-VTR-Modbus](https://github.com/Ztaeyn/HomeAssistant-VTR-Modbus) var det avgjørende startpunktet.
*   Den detaljerte guiden for fysisk installasjon og konfigurasjon av Elfin EW11 er publisert på [domotics.no](https://www.domotics.no/post/home-assistant-automasjon-av-ventilasjonsanlegg-via-modbus), og skrevet av Mads Nedrehagen.
*   Prosjektet er videreutviklet og vedlikeholdt av @Howard0000. En KI-assistent har hjulpet til med å forenkle forklaringer og rydde i `README.md`.

## 📝 Lisens
MIT — se `LICENSE`.



