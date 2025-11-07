# Home Assistant-integrasjon for Systemair SAVE VTR-500

> [Read this guide in English](README.en.md)
>
⚠️ Dette er ikke en HACS-integrasjon.
Følg veiledningen nedenfor for manuell installasjon av YAML-filer, Lovelace-kort og Node-RED-flyt.

Dette repositoriet inneholder en komplett konfigurasjon for å integrere og styre en Systemair SAVE VTR-500 ventilasjonsenhet med Home Assistant via Modbus TCP.

![Lovelace Dashboard](image/Ventilasjon%20kort.png)

## Funksjoner

*   **Full modus-styring:** Kontroller alle moduser som Auto, Manuell (Lav, Normal, Høy), Party, Boost, Borte, Ferie og Stopp.
*   **Detaljerte sensorer:** Leser av temperaturer, fuktighet, viftehastigheter, varmegjenvinning og alarmer.
*   **Temperaturkontroll:** Fungerer som en termostat for å justere ønsket inntakstemperatur.
*   **Avansert automasjon:** Bruker Node-RED til å justere viftehastigheten automatisk basert på fuktighets- og CO2-nivåer, inkludert nattsenking.
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
2.  **Koble til Elfin EW11:** Koble ledningene som vist i diagrammet under.
    ![Koblingsskjema EW11](image/koblings%20skjema%20EW11.png)
3.  **Gjenopprett strømmen:** Når alt er trygt koblet, slå på strømmen til anlegget.

### Trinn 2.2: Konfigurere Elfin EW11

1.  **Koble til EW11s nettverk:** Koble til Wi-Fi-nettverket `EW1x_...` (ikke passord).
2.  **Åpne web-grensesnitt:** Gå til `http://10.10.100.254`. Logg inn med `admin` / `admin`.
3.  **Koble til ditt Wi-Fi:** Under "System Settings" -> "WiFi Settings", sett "Wifi Mode" til "STA", finn ditt hjemmenettverk, skriv inn passord og lagre.
    ![System Settings EW11](image/system%20settings%20EW11.png)
4.  **Restart og finn ny IP:** Enheten vil restarte. Finn den nye IP-adressen den har fått (sjekk i ruteren din) og sett en statisk IP for den.
5.  **Konfigurer serieport:** Logg inn på den nye IP-adressen. Gå til "Serial Port Settings" og sett verdiene som vist under.
    ![Serial Port Settings EW11](image/serial%20port%20settings%20EW11.png)
6.  **Konfigurer kommunikasjon:** Gå til "Communication Settings" og legg til en ny profil som vist under.
    ![Communication Settings EW11](image/communication%20settings%20EW11.png)
7.  **Verifiser:** Gå til "Status"-siden. Telleverk for datapakker skal nå øke.
    ![Kommunikasjon EW11](image/kommunikasjon%20EW11.png)

### Trinn 2.3: Konfigurasjon i Home Assistant

1.  **Aktiver "Packages":** Sørg for at `configuration.yaml` inneholder:
    ```yaml
    homeassistant:
      packages: !include_dir_named packages
    ```
2.  **Legg til konfigurasjonen:** Kopier filen `packages/systemair.yaml` fra dette repoet inn i din `/config/packages/`-mappe.
3.  **Oppdater IP-adressen:** Åpne `packages/systemair.yaml`-filen og endre `host` til den statiske IP-adressen til din Elfin EW11.
4.  **Start Home Assistant på nytt.**

### Trinn 2.4: Sett opp Lovelace Dashboard

Dette prosjektet tilbyr dashboard-konfigurasjonen på både norsk og engelsk.

1.  Velg ditt foretrukne språk ved å navigere inn i enten mappen `lovelace/no/` eller `lovelace/en/`.
2.  Inne i din valgte mappe vil du finne tre YAML-filer.
3.  På ditt Home Assistant-dashboard, legg til **tre separate "Manuell"-kort**.
4.  Kopier innholdet fra hver av de tre YAML-filene inn i hvert sitt "Manuell"-kort.

### Trinn 2.5: Importer Node-RED Flow

1.  Velg ditt foretrukne språk. Åpne filen `node-red/no/flows.json` eller `node-red/en/flows.json`. Kopier hele innholdet i filen.
2.  Åpne Node-RED, gå til Meny -> Import, og lim inn innholdet.
3.  **VIKTIG:** Gå gjennom de nye nodene og oppdater `entity_id` til dine egne fukt- og CO2-sensorer.
4.  Klikk "Deploy".
    ![Node-RED Flow](image/Node-Red%20VTR500.png)

### Bonus: Hvordan Nattsenking Fungerer

Node-RED-flyten inneholder en innebygd logikk for nattsenking. Når den aktiveres, senker den temperaturen med ca. 3 grader og setter viftehastigheten til "Lav".

**Viktig:** Funksjonen aktiveres ikke av seg selv. Den styres av en `switch`-entitet som lages av Node-RED-flyten.
*   Hvis du brukte den norske `flows.json`, heter entiteten `switch.nattsenking_ventilasjon_pa`.
*   Hvis du brukte den engelske `flows.json`, heter den `switch.night_setback_ventilation_on`.

For å bruke den må du selv lage en automasjon eller et skript som slår på denne bryteren.

---

## Filforklaring

*   **`packages/systemair.yaml`**: Hovedkonfigurasjonen for alle sensorer, brytere og skript. Denne filen er på engelsk.
*   **`lovelace/no/` & `lovelace/en/`**: Inneholder de tre YAML-filene for Lovelace-dashboardet, levert på både norsk og engelsk.
*   **`node-red/no/` & `node-red/en/`**: Inneholder `flows.json`-filen for avansert automasjon, levert på både norsk og engelsk.
*   **`image/`**: Skjermbilder og diagrammer brukt i denne guiden.
*   **`README.md`**: Denne guiden på norsk.
*   **`README.en.md`**: Denne guiden på engelsk.

## Anerkjennelser og Credits
*   Kjernekonfigurasjonen (`systemair.yaml`) er basert på arbeidet til **@Ztaeyn** i hans [HomeAssistant-VTR-Modbus](https://github.com/Ztaeyn/HomeAssistant-VTR-Modbus) repositorium.
*   Guiden for installasjon er publisert på [domotics.no](https://www.domotics.no/post/home-assistant-automasjon-av-ventilasjonsanlegg-via-modbus) og skrevet av Mads Nedrehagen.
*   Prosjektet er videreutviklet av @Howard0000. En KI-assistent har hjulpet til med å rydde i `README.md`.

## 📝 Lisens
MIT — se `LICENSE`.




