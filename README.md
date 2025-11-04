# Smart-Hamster-Wheel

# Smartes Hamsterrad mit Home Assistant Reisetagebuch

Willkommen beim Projekt "Smartes Hamsterrad"! Dieses Repository enthält alle notwendigen Dateien, Codes und Anleitungen, um ein Standard-Hamsterrad (z.B. von Getzoo) in einen vollwertigen Fitness-Tracker zu verwandeln.

Das System erfasst die Aktivität des Hamsters über einen induktiven Sensor, verarbeitet die Daten auf einem **ESP32-C6** und sendet sie an **Home Assistant**. Dort wird ein detailliertes Dashboard inklusive eines "Reisetagebuchs" generiert.

<img src="media/Home%20Assistant%20Dashboard.png" alt="Home Assistant Dashboard" width="600" />

---

## Features

* **Live-Geschwindigkeit** in km/h.
* **Tagesdistanz** (wird täglich um 20:00 Uhr zurückgesetzt).
* **Gesamtdistanz** (persistent gespeichert auf dem ESP).
* **Rekord-Tracking** für die höchste Geschwindigkeit und die längste Tagesstrecke (über `input_number` Helfer in HA).
* **"Hamster-Reisetagebuch"** als Lovelace Markdown-Karte, die die Gesamtdistanz in eine virtuelle Reise umrechnet.
* **Push-Benachrichtigungen** am Morgen mit den "Stats der Nacht" (via Home Assistant Automation).

---

## Technische Funktionsweise & Schaltung

Das Herzstück des Projekts ist eine custom Platine, die die Stromversorgung und Signalverarbeitung übernimmt.

1.  **Stromversorgung:** Das System wird über den USB-C-Anschluss des **Waveshare ESP32-C6-Zero** mit 5V versorgt.
2.  **Sensor-Spannung (Step-Up):** Der induktive Sensor (H1) benötigt eine höhere Spannung (12V oder 24V). Diese wird durch einen **Pololu Step-Up Volt Regulator (U2)** erzeugt, der seine Eingangsspannung (5V) direkt vom ESP32-C6 (Pin 1) erhält.
3.  **Signalverarbeitung (Voltage Divider):** Der Sensor (H1) gibt ein 12V/24V-Signal aus. Dies wäre zu hoch für den ESP. Daher wird das Signal über einen **Spannungsteiler (R1/R2)** auf ein für den ESP (GPIO5, Pin 8) sicheres 3.3V-Level reduziert.
4.  **Widerstands-Wahl (R2):**
    * Verwende **R2 = 10kΩ** bei einem **12V**-Sensor.
    * Verwende **R2 = 22kΩ** bei einem **24V**-Sensor.
    * R1 ist immer **3.3kΩ**.
5.  **Impuls-Erfassung:** 4 Reißzwecken wurden in das Laufrad eingedrückt und verklebt. Der induktive Sensor erfasst diese Metallstifte und erzeugt **4 Impulse pro Umdrehung**.

![Schaltplan](Platine : PCB/Platine bestückt.jpg)

---

## Benötigte Hardware (BOM)

* **MCU:** Waveshare ESP32-C6-Zero
* **Sensor:** 3-Draht Induktiver Näherungssensor (z.B. LJ12A3-4-Z/BX, NPN, 12-24V)
* **Spannungsregler:** Pololu Step-Up Volt Regulator (z.B. 12V oder 24V je nach Sensor)
* **Widerstände:**
    * 1x 3.3kΩ (R1)
    * 1x 10kΩ (R2 für 12V) ODER 22kΩ (R2 für 24V)
* **Sonstiges:**
    * Schraubterminal (3-Pin) für Sensoranschluss (H1)
    * Stiftleisten für ESP und Regler
    * 4x Reißzwecken
* **Platine:** Custom PCB (Gerber/Design-Dateien in `/Hardware/PCB/`)
* **Gehäuse:** Custom 3D-Druck (STL-Dateien in `/Hardware/Case/`)

---

## Software-Installation

### 1. ESPHome

Der Code für den ESP32-C6 befindet sich in `Software/ESPHome/hamsterrad-esphome.yaml`.

1.  **Secrets:** Diese Konfiguration verwendet Secrets! Stelle sicher, dass du deine `secrets.yaml` (im ESPHome Hauptverzeichnis) mit den WLAN-Daten, einem API-Key und einem OTA-Passwort füllst, die in der `hamsterrad-esphome.yaml` referenziert werden.
    ```yaml
    # Beispiel für deine secrets.yaml
    wifi_ssid: "DEIN_WLAN_NAME"
    wifi_password: "DEIN_WLAN_PASSWORT"
    hamsterrad_api_key: "ERSTELLE_EINEN_SICHEREN_API_KEY"
    hamsterrad_ota_password: "ERSTELLE_EIN_OTA_PASSWORT"
    ```
2.  **Anpassungen:** Überprüfe die `substitutions` oben in der YAML-Datei. Passe ggf. den `rad_durchmesser_cm` (bei dir 31.4) und die `impulse_pro_umdrehung` (bei dir 4.0) an.
3.  **Flashen:** Kompiliere und flashe den Code auf deinen ESP32-C6.

### 2. Home Assistant

Der Code für die Lovelace-Karte befindet sich in `software/home-assistant/Karte Home Assistant Code.rtf`.

1.  **Helfer-Entitäten:** Die Karte ist auf diverse Helfer und Sensoren angewiesen, die du in Home Assistant anlegen musst (z.B. unter "Helfer" in den Einstellungen):
    * `sensor.hamsterrad_geschwindigkeit_max_12h` (z.B. über einen `max` Hilfssensor)
    * `sensor.hamsterrad_geschwindigkeit_durchschnitt_wenn_aktiv` (z.B. über einen `average` Hilfssensor)
    * `input_number.hamsterrad_distanz_taglich_rekord`
    * `input_number.hamsterrad_geschwindigkeit_rekord`
    * *...sowie die Sensoren, die von ESPHome bereitgestellt werden (`sensor.hamsterrad_distanz_taglich`, `sensor.hamsterrad_geschwindigkeit` usw.)*
2.  **Lovelace-Karte:** Erstelle eine neue "Manuelle Karte" in deinem Dashboard und kopiere den Inhalt der `Karte Home Assistant Code.rtf` hinein.
3.  **Anpassung:** Passe die Kilometer-Marken und Orte im "locations"-Dictionary in der Markdown-Karte an deinen Wohnort an.

---

## Projekt-Entstehung & Geschichte

Wenn du mehr über die Entstehungsgeschichte, die Motivation und die Herausforderungen (z.B. die "Verhandlungen" mit der Verlobten) erfahren möchtest, lies den originalen Magazin-Artikel, den wir für dieses Projekt geschrieben haben.

<details>
<summary><b>Klick hier, um den Magazin-Artikel zu lesen...</b></summary>

### Kilometer-Zähler für kleine Pfoten: Mein Hamsterrad wird smart und erzählt Geschichten

Wessen Traum ist es nicht, morgens aufzuwachen und genau zu wissen, wie viele Kilometer der eigene Hamster in der Nacht gelaufen ist? Welche Geschwindigkeit hatte er? Und wie weit ist er schon gekommen?

Schon als Kind war ich absolut fasziniert, wenn ich in Büchern und Berichten las, welche unglaublichen Strecken diese winzigen Tiere in einer einzigen Nacht zurücklegen können. Ich fragte mich schon damals: „Kann das wirklich sein? Das will ich selbst erforschen.“ Leider hatte ich als Kind und Jugendlicher Pech: Keiner meiner damaligen Hamster mochte sein Laufrad. Sie ignorierten es konsequent. Und so blieb dieser kleine Traum unerfüllt.

Viele Jahre später, jetzt als Erwachsener, habe ich durch meine Verlobte wieder einen Hamster. Und dieser ist das genaue Gegenteil. Er liebt sein Rad und verbringt mit großer Freude jede Nacht darin. Man kann sich vorstellen, was passierte: Der alte Kindheitstraum war sofort wieder da! Dieses Mal hatte ich bereits durch andere Projekte auch die nötigen Fertigkeiten, um ihn wahr werden zu lassen.

#### „Bauen ja – stören nein!“

Die Überzeugungsarbeit bei meiner Verlobten dauerte zum Glück nicht lange, aber sie steckte mir sofort ganz klare Grenzen für das Projekt: Der Hamster durfte auf keinen Fall gestört, niemals durch Kabel oder Sensoren gefährdet werden, zudem musste das wertvolle Getzoo-Korklaufrad absolut intakt bleiben.

Das war eine echte Herausforderung, denn meine ersten Ideen waren damit sofort vom Tisch. Sensoren, die einen Schall aussenden, könnte er hören und sich gestört fühlen. Sensoren, die Licht aussenden, wie eine Lichtschranke, könnten ihn nachts stören, besonders da das Licht im gesamten Nagarium reflektiert würde. Mechanische Sensoren wiederum würden das Rad zu sehr abbremsen und den Laufwiderstand erhöhen.

Die Wahl fiel daher auf einen sogenannten "induktiven Sensor". Dieser reagiert einfach, wenn ein Stück Metall an seiner Sensorfläche vorbeigeführt wird – komplett kontaktlos, ohne Geräusche und ohne störendes Licht. Die perfekte hamstersichere Lösung.

Als "Gehirn" des Ganzen kam für mich nur ein winziger Mini-Computer infrage (ein ESP32-C6), da ich mit diesem Chip bereits mehrere andere Smart-Home-Projekte umgesetzt habe.

#### „Vom Küchentisch zum 3D-Druck"

(Details siehe technischer Abschnitt oben)

Der allererste Versuchsaufbau fand auf einem kleinen Test-Board statt. Mit ein paar Kabeln, dem Laptop zum Programmieren und dem Laufrad - und ja, das Ganze fand in der Küche statt. Hier war dann auch bereits der Sensor final am Laufrad befestigt. Als Metall, das den Sensor betätigen soll, habe ich vier Reißbrettstifte ins Rad eingedrückt und geklebt. Eine Veränderung am Rad, die meine Verlobte akzeptiert hat und die den Hamster nicht stört oder gefährdet.

Dieser provisorische Aufbau wirkte mir aber auf Dauer viel zu unprofessionell. Ich habe daher eine richtige, professionellere Platine entworfen. Als die Platine da war und bestückt, konnte es ans Entwerfen eines Gehäuses gehen. Dieses habe ich am Laptop in 3D konstruiert und mit Hilfe meines 3D-Druckers gefertigt.

#### Der Lohn der Mühe: Tägliche Berichte und eine Weltreise

Der wahre Zauber dieses Projekts zeigt sich jeden Morgen. Pünktlich beim Aufstehen bekomme ich eine Push-Nachricht, die mir verrät, wie viele Kilometer der Hamster gelaufen ist und ob er einen neuen Rekord aufgestellt hat.

Als Krönung der ganzen Datensammelei habe ich das "Hamster-Reisetagebuch" ins Leben gerufen. Das Beeindruckendste daran: Die Strecke von über 250 km ist er in nicht einmal einem Monat gelaufen (Start der Reise war der 5. Oktober 2025)!

Die Liste der Ziele habe ich mir übrigens nicht mühsam von Hand rausgesucht. Statt einer einzelnen Route habe ich eine KI gebeten, mir ganz viele interessante Orte zu nennen, die in verschiedener Luftlinien-Entfernung von unserem Zuhause liegen – bis hin zum ultimativen Ziel: Vancouver, 9.650 km entfernt!

Und so ist unser Hamster jetzt ein Weltenbummler. Jeden Morgen schauen wir, wo er steckt: Trier (35 km) und Luxemburg (85 km) hat er locker passiert, Paris (195 km) war ein kurzer Stopp, und Stuttgart (215 km) liegt auch schon hinter ihm.

Ich bin gespannt, ob er es eines Tages wirklich bis nach Kanada schafft!
</details>

---

## Galerie & Mediendateien

Hier sind weitere Bilder vom Aufbau und den Komponenten.

| Testaufbau in der Küche | Bestückte Platine (V2.0) | Platine im 3D-Gehäuse |
| :---: | :---: | :---: |
| ![Testaufbau](media/Test%20Aufbau.jpg) | ![Bestückte Platine](media/Platine%20bestückt.jpg) | ![Platine im 3D-Gehäuse](media/Platine%20mit%203D%20Gehäuse.jpg) |

---

## Lizenz

Dieses Projekt steht unter der **MIT-Lizenz**. Du darfst die Designs und Codes gerne verwenden, anpassen und teilen. Siehe die `LICENSE`-Datei für Details.
