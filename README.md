# TE2 Swivel Lift Seat – Adaptive Gateway

[English](#english) | [Deutsch](#deutsch)

---

<a name="english"></a>
## English

An open-source hardware gateway and telemetry bridge based on the ESP32-C3 microcontroller, designed to interface with the internal CAN-bus communication of **Turny Evo II** swivel lift seats.

### 1. Background & Problem Statement

The factory installation of the swivel lift seat relies on a tethered handheld remote with an integrated LCD display. In daily practical use, several common challenges arise:

* **Cable Fatigue & Breakage:** The coiled cable is subject to constant bending and mechanical strain during transfers. It frequently gets trapped in the vehicle door, leading to damaged leads or intermittent contact.
* **Controller Freezes:** The handheld unit contains a dedicated microcontroller. Electrical noise, vehicle brownouts, or contact bounce can cause the unit to crash, locking the seat mechanism until a full power-cycle (unplugging the connector) is performed.
* **Wake-Up Confusion:** The remote enters a deep-sleep state to protect vehicle battery life. A single button press merely wakes the display without moving the seat; users often misinterpret this as a malfunction.
* **Component Wear:** The RJ-socket soldered onto the main base ECU takes mechanical abuse from repeated plugging and cable tugs.

### 2. Project Goals

1. **Passive Bus Telemetry:** Sniff, decode, and document the differential bus protocol (CAN 2.0B / TWAI) between the base ECU and the handheld unit using a non-intrusive listen-only setup.
2. **Port Protection:** Introduce an inline sacrificial splitter cable to eliminate mechanical stress on the vehicle's permanent base socket.
3. **Standalone Hardware Gateway:** Deploy an onboard ESP32-C3 under the seat frame to monitor status frames, manage failsafes, and emulate dead-man commands.
4. **Enhanced Interfaces:**
   * **Direct Console Buttons:** Weather-resistant momentary push buttons mounted flush to the seat base (eliminating the tethered remote for everyday boarding).
   * **Wireless Mobile Control:** BLE-enabled smartphone control featuring one-click auto-deploy and dead-man safety controls.

### 3. Technical Architecture & Interface

#### 3.1 Connector & Pinout (RJ11 / 6P4C)

The seat base connects to the remote unit via a 6-position, 4-contact modular jack (USOC color convention):

| Pin | USOC Color | Function | Description |
| :---: | :---: | :---: | :--- |
| **Pin 2** | Black | **GND** | 0V logic and chassis ground reference |
| **Pin 3** | Red | **VCC** | Supply voltage (+12V switched / battery level) |
| **Pin 4** | Green | **CAN_L** | Differential serial data line (Low) |
| **Pin 5** | Yellow | **CAN_H** | Differential serial data line (High) |

*Note: Outer positions Pin 1 and Pin 6 are unpopulated.*

#### 3.2 System Roles

* **Base ECU (Seat Base Assembly):** Master intelligence. Stores user trajectories, calibration values, mechanical end-stops, encoder counts, and motor current overcurrent protection (anti-pinch).
* **Handheld Remote (HMI):** Human-machine interface. Polls user buttons, renders status/diagnostic codes on screen, and transmits periodic keepalive/command frames.

### 4. Hardware Implementation

* **Phase 1: Diagnostic & Sniffer Rig**
  * 1:1 RJ11 6P4C parallel duplex splitter connected to the base socket.
  * Screw-terminal breakout adapter tapping into `GND`, `CAN_H`, and `CAN_L`.
  * 3.3V CAN transceiver module (SN65HVD230 / VP230) with the onboard 120 Ω termination resistor removed.
  * ESP32-C3 running Espressif's TWAI driver in **Listen-Only Mode** (silent mode; no ACK or error frames transmitted).
  * Embedded WebSocket server providing live stream telemetry and CSV export to a local web dashboard.
* **Phase 2: Permanent Gateway & Hardwired Controls**
  * Custom enclosure mounted securely to the seat frame.
  * Automotive-grade step-down DC/DC converter powering the ESP32-C3 directly from vehicle power.
  * Two vandal-resistant push buttons (16mm/19mm with integrated LED status rings) wired to debounced GPIO interrupts on the ESP32.
  * Active TWAI mode: Direct transmission of periodic motion frames upon button engagement.
* **Phase 3: Wireless Control & Automation**
  * BLE GATT peripheral implementation featuring a strict watchdog timer: if keepalive packets cease for >150 ms, the seat motion halts immediately.
  * Optional tie-in to the vehicle's seat occupancy sensor to inhibit automatic remote retraction while occupied.

### 5. Safety Philosophy

* **Listen-Only Prototyping:** All reverse engineering must be performed in passive silent mode to ensure the vehicle bus and medical device controllers are never disrupted.
* **ECU Failsafes Remain Untouched:** End-stop limits, velocity ramping, and anti-pinch current shutoffs are managed exclusively by the factory base ECU.
* **Watchdog Failsafe:** Custom firmware adheres strictly to dead-man operation logic; loss of signal or microcontroller resets drop all output drives instantly.

### 6. Legal Notice, Interoperability & Disclaimer

* **Interoperability & Reverse Engineering:** The documentation, protocol analysis, and hardware sniffing in this repository are performed strictly for the purpose of achieving interoperability with independently created hardware and software, in accordance with European Directive 2009/24/EC (Art. 6) and applicable national laws (e.g., § 69e German Copyright Act / UrhG). No proprietary source code, protected firmware binaries, or trade secrets are extracted or redistributed.
* **Trademarks:** All product names, trademarks, and registered trademarks (such as *Turny®*, *Turny Evo®*, and *BraunAbility®*) are property of their respective owners. Their use in this repository is strictly for identification, compatibility explanation, and descriptive purposes under fair use doctrine and § 23 German Trademark Act (MarkenG). This open-source project is independent and has not been authorized, sponsored, or endorsed by BraunAbility.
* **Safety & Regulatory Disclaimer:** This project provides experimental research and open-source documentation. It is not an officially certified medical or automotive product. Operating or modifying mobility equipment and automotive wiring carries inherent safety and regulatory risks. The authors and contributors assume no liability for personal injury, property damage, loss of vehicle homologation/approvals (TÜV, ECE, CE), or voided manufacturer warranties. Use at your own risk.

### 7. License

**English:**  
This project is licensed under the **Apache License, Version 2.0** (the "License"); you may not use this file except in compliance with the License. You may obtain a copy of the License at [http://www.apache.org/licenses/LICENSE-2.0](http://www.apache.org/licenses/LICENSE-2.0).

---

<a name="deutsch"></a>
## Deutsch

Ein Open-Source-Hardware-Gateway und Telemetrie-Bridge auf Basis des ESP32-C3-Mikrocontrollers zur Anbindung an die interne CAN-Bus-Kommunikation von **Turny Evo II** Schwenkhubsitzen.

### 1. Ausgangslage & Problemstellung

Die Werksinstallation des Schwenkhubsitzes basiert auf einem kabelgebundenen Handbedienteil mit integriertem LC-Display. Im alltäglichen Praxiseinsatz treten typische Schwachstellen auf:

* **Kabelverschleiß & Kabelbruch:** Das Spiralkabel ist beim Ein- und Aussteigen ständiger Zug- und Biegebelastung ausgesetzt. Es wird häufig in der Autotür eingeklemmt, was zu Aderbrüchen und Wackelkontakten führt.
* **Controller-Abstürze:** Im Handbedienteil arbeitet ein eigener Mikrocontroller. Störspitzen im Bordnetz, Spannungseinbrüche oder Kontaktprellen führen zu Abstürzen, die den Sitz blockieren. Eine Wiederinbetriebnahme erfordert meist einen manuellen Kaltstart (Stecker ziehen).
* **Bedienirritation durch Sleep-Modus:** Um die Fahrzeugbatterie zu schonen, schläft das Bedienteil ein. Ein erster Tastendruck weckt oft nur das Display auf, ohne den Sitz zu bewegen. Dies wird im Alltag häufig fälschlicherweise als Wackelkontakt oder Defekt interpretiert.
* **Buchsenverschleiß:** Die direkt auf die Basisplatine gelötete RJ-Buchse nimmt durch häufiges Einstecken und Hebelkräfte des Spiralkabels dauerhaft mechanischen Schaden.

### 2. Projektziele

1. **Passive Bus-Telemetrie:** Mitschneiden, Dekodieren und Dokumentieren des differenziellen Busprotokolls (CAN 2.0B / TWAI) zwischen Basis-ECU und Handteil über einen rückwirkungsfreien Listen-Only-Aufbau.
2. **Buchsenschutz:** Vorschalten eines auswechselbaren Splitter-/Verlängerungskabels („Opferkabel“), um mechanische Belastungen von der fest verbauten Sockelbuchse fernzuhalten.
3. **Autarkes Hardware-Gateway:** Feste Installation eines ESP32-C3 unter dem Sitzgestell zur Überwachung von Statusmeldungen, Handhabung von Sicherheitsroutinen und Emulation der Totmann-Steuerbefehle.
4. **Erweiterte Bedienoptionen:**
   * **Feste Konsolentaster:** Witterungsbeständige Taster direkt an der Sitzverkleidung, die das Hantieren mit dem Spiralkabel im Alltag überflüssig machen.
   * **Drahtlose Steuerung per Smartphone:** BLE-Anbindung mit Ein-Klick-Automatik zum Ausfahren und Totmann-Sicherheitssteuerung beim Einsteigen.

### 3. Technische Systemarchitektur

#### 3.1 Steckverbinder & Pinbelegung (RJ11 / 6P4C)

Die Verbindung zwischen Sitzsockel und Handbedienteil erfolgt über einen 6-poligen Modularstecker mit 4 belegten Kontakten (USOC-Farbbelegung):

| Pin | USOC-Farbe | Funktion | Beschreibung |
| :---: | :---: | :---: | :--- |
| **Pin 2** | Schwarz | **GND** | 0V Logik- und Gehäusemasse |
| **Pin 3** | Rot | **VCC** | Versorgungsspannung (+12V Bordnetz / geschaltet) |
| **Pin 4** | Grün | **CAN_L** | Differenzielle Datenleitung (Low) |
| **Pin 5** | Gelb | **CAN_H** | Differenzielle Datenleitung (High) |

*Hinweis: Die äußeren Positionen Pin 1 und Pin 6 sind nicht bestückt.*

#### 3.2 Systemrollen

* **Basis-ECU (im Sitzsockel):** Hauptrecheneinheit. Speichert fahrzeugspezifische Fahrkurven, Kalibrierdaten, Endlagen, Encoder-Werte und steuert die Motorströme (Einklemmschutz).
* **Handbedienteil (HMI):** Reine Benutzerschnittstelle. Fragt Tastendrücke ab, gibt Diagnose- und Statuscodes auf dem Display aus und sendet periodische Totmann-Fahrbefehle.

### 4. Hardware-Implementierung

* **Phase 1: Diagnose- & Sniffer-Aufbau**
  * 1:1 verdrahteter RJ11 6P4C Parallel-Splitter an der Sockelbuchse.
  * Schraubklemmen-Breakout zum Abgreifen von `GND`, `CAN_H` und `CAN_L`.
  * 3,3V-CAN-Transceiver (SN65HVD230 / VP230) mit entferntem 120-Ω-Abschlusswiderstand.
  * ESP32-C3 mit Espressif TWAI-Treiber im **Listen-Only Mode** (passiver Abhörmodus; keine ACKs, keine Error-Frames).
  * Integrierter WebSocket-Server zur Echtzeit-Darstellung und CSV-Export über ein lokales Web-Dashboard.
* **Phase 2: Festes Gateway & Rahmentaster**
  * Robustes Gehäuse, fest am Sitzrahmen montiert.
  * Kfz-zertifizierter DC/DC-Schaltregler zur Spannungsversorgung direkt aus dem Bordnetz.
  * Zwei vandalismusgeschützte Taster (16 mm / 19 mm mit LED-Ring) an entprellten GPIO-Interrupts des ESP32.
  * Aktiver TWAI-Modus: Eigenständiges Senden der zyklischen Steuerframes bei Tastendruck.
* **Phase 3: Drahtlose Steuerung & Automatisierung**
  * BLE-GATT-Profil mit schnellem Heartbeat-Watchdog: Bleibt das Signal für >150 ms aus, stoppt die Sitzbewegung sofort.
  * Optionale Anbindung an die Sitzbelegungsmatte zur hardwareseitigen Sperrung von Automatikfahrten bei besetztem Sitz.

### 5. Sicherheitskonzept

* **Rückwirkungsfreie Analyse:** Alle Sniffer-Aktivitäten erfolgen im Listen-Only-Modus, um Störungen der Steuergeräte und Fehlereinträge auszuschließen.
* **Unveränderte ECU-Sicherheitsroutinen:** Endabschaltungen, Sanftanlauf und der mechanische Einklemmschutz verbleiben vollständig unter Kontrolle der originalen Basis-ECU.
* **Fail-Safe & Totmann-Logik:** Die Firmware emuliert strikt die Totmann-Bedingungen. Bei Verbindungsabbruch oder System-Resets fällt der Ausgang sofort in den sicheren Halt.

### 6. Rechtliche Hinweise, Interoperabilität & Haftungsausschluss

* **Interoperabilität & Dekompilierung:** Die in diesem Repository dokumentierten Analysen und Bus-Mitschnitte dienen ausschließlich der Herstellung der Interoperabilität mit einer unabhängig geschaffenen Hard- und Software im Sinne der EU-Richtlinie 2009/24/EG (Art. 6) sowie § 69e UrhG (Dekompilierung zur Interoperabilität). Es werden keine proprietären Binärdateien, Firmware-Abbilder oder Betriebsgeheimnisse vervielfältigt oder veröffentlicht.
* **Markenhinweis:** Alle Produkt- und Markennamen (wie *Turny®*, *Turny Evo®*, *BraunAbility®*) sind Eigentum der jeweiligen Rechteinhaber. Die Nennung in diesem Projekt dient ausschließlich der Beschreibung von Kompatibilität und technischem Zweck im Rahmen des § 23 MarkenG (Bestimmungsgemäße Benutzung) bzw. internationaler Fair-Use-Prinzipien. Dieses Open-Source-Projekt ist unabhängig und steht in keinerlei geschäftlicher Verbindung zu BraunAbility.
* **Haftungsausschluss & Medizinprodukte-Hinweis:** Dieses Repository stellt experimentelle Entwicklungsdokumente zu Bildungs- und Forschungszwecken bereit. Es handelt sich nicht um ein zertifiziertes Medizinprodukt oder ein nach StVZO/ECE typgeprüftes Fahrzeugbauteil. Änderungen an Reha-Systemen und Fahrzeugverkabelungen erfolgen auf eigene Verantwortung und können zum Erlöschen von Betriebserlaubnis (TÜV), CE-Konformität und Garantieansprüchen führen. Die Autoren und Mitwirkenden übernehmen keinerlei Haftung für Personen-, Sach- oder Folgeschäden.

### 7. Lizenz

Dieses Projekt ist unter der **Apache-Lizenz, Version 2.0** lizenziert. Details siehe `LICENSE`-Datei oder unter [http://www.apache.org/licenses/LICENSE-2.0](http://www.apache.org/licenses/LICENSE-2.0).
