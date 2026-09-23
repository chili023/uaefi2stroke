# uaEFI moto: PCB-Änderungen gegenüber uaEFI Rev F3

## Stand

**Schaltplan: umgesetzt und geprüft** (Branch `uaefi-moto`):
- KiCad 10 ERC: 0 Fehler (Original: 0 Fehler), nur Bibliotheks- und Footprint-Link-Warnungen wie im Original
- Netzliste gegen das Original geprüft: kein bestehendes Netz getrennt oder verbunden, alle 55 übernommenen
  Steckerpins liegen auf dem Netz des alten Steckerpins
- Neue Teile auf dem Blatt rechts (Papier A3 → A2)

**Platine (`uaefi.kicad_pcb`): vorbereitet, Verlegen der Leitungen offen.**
- Alte Teile entfernt (Mini-Fit J2/J3/J4/J5/J10, E-Gas-Treiber, F3/F4, Breakout-Stiftleisten J1/J11–J24),
  ebenso alle Leiterbahnen, die nur zu ihnen führten.
- Platine **145 × 110 mm** (vorher 100 × 100): 45 mm nach rechts für EGT/M8, 10 mm nach oben, weil die
  Superseal-Pins 15,6 mm hinter die Kante reichen. Massefläche, Eck-Sperrflächen und H1/H3/H4 mitverschoben.
- Neue Teile vorplatziert: J30/J31 oben, F11 dazwischen, M8 oben rechts, J32–J35 an der rechten Kante,
  die vier MAX31855-Schaltungen (Anordnung wie die ursprüngliche EGT1-Schaltung, Unterseite) direkt davor,
  C40/C41 unten rechts.
- KiCad-DRC: 0 Fehler außer 4× „malformed courtyard“ im Superseal-Footprint der rusEFI-Bibliothek,
  0 Abweichungen zum Schaltplan, **138 offene Verbindungen = das, was noch verlegt werden muss**.
- Danach `revision.txt` hochzählen (eigenes `BOARD_SUFFIX`/`BOARD_REVISION` und `bom_replace`-Datei **ohne**
  die Zeile, die U5/Bluetooth abwählt) und ohne `[skip ci]` pushen, dann erzeugt GitHub Gerber/BOM/CPL.

| Referenz | Teil | Footprint |
|---|---|---|
| J30 | Stecker A, TE 6437288-1 (Keying 1) | `Connectors:6437288-2` (gleiches Pinbild, am Datenblatt prüfen) |
| J31 | Stecker B, TE 6437288-2 (Keying 2) | `Connectors:6437288-2` |
| J32–J35 | EGT1–4, Omega PCC-SMP-K | `moto:Omega_PCC-SMP-K` (**vorläufig**, am echten Teil prüfen) |
| Blätter EGT2–EGT4 | MAX31855 + Filter, U2004/U3004/U4004 usw. | wie U4 |
| M8 | zweites `Module-wbo-0.6` | `hellen-one-wbo-0.6:wbo` |
| F11 | PTC 200 mA für +12V Hall (A3 und B1) | 1206 |
| C40 / C41 | 47 µF 16 V auf +5V / 470 µF 6,3 V auf +3,3VA, Einschaltfix Bluetooth (uaEFI-README) | CP_Elec 6.3x5.4 / 8x10, LCSC noch offen |
| U5 | JDY-33 Bluetooth **bestückt** (uaEFI-Stückliste wählt es ab) | |
| R39 | **4,7k bestückt** (Pull-up Radsensor 2 an B28), LCSC C17673 | 0805 |
| R36 | **nicht bestückt** (DNP) | 0805 |
| entfernt | J2, J3, J4, J5, J10 (Mini-Fit), Blätter DC Driver 1/2 (U1, U2, C1–C26, R9, R14, P15–P18), F3, F4, Breakout-Stiftleisten J1, J11–J24 | |

Das Hardware-Index-Problem von Lambda 2 ist gelöst, siehe Abschnitt 4.

---

Basis: dieses Repo (`uaefi.kicad_sch` / `uaefi.kicad_pcb`, Rev F3, KiCad 10).
Firmware-Board: `firmware/config/boards/hellen/uaefi-moto` im Fork `chili023/rusefi-clean`, Branch `uaefi-moto`.
Die Netznamen unten stammen aus `gerber/uaefi.net` (Export der aktuellen Rev).

## 1. Stecker

| Alt | Neu |
|---|---|
| J4 Mini-Fit 8 (A), J5 Mini-Fit 18 (B), J2 Mini-Fit 20 (C), J10 Mini-Fit 16 (D), J3 Mini-Fit 6 (E) | entfallen |
| – | **J30 = Stecker A: TE 6437288-1**, Superseal 1.0, 34-polig, 90°, Keying 1 |
| – | **J31 = Stecker B: TE 6437288-2**, Superseal 1.0, 34-polig, 90°, Keying 2 |
| – | **J32–J35 = EGT1–4: Omega PCC-SMP-K** (Mini-Thermoelementbuchse Typ K, liegend) |
| J7 SPOX (USB extern), J8 Mini-USB | nicht mehr nötig (USB liegt auf Stecker A), USB-C J9 für Tischbetrieb bleibt |
| U5 JDY-33 (Bluetooth) | optional, DNP empfohlen (bekanntes Einschaltproblem, siehe README) |

Symbol, Footprint und 3D-Modell für den Superseal-Header liegen schon in
`kicad6-libraries/` (`TE_6437288-2`). Keying 1 und 2 unterscheiden sich nur im Gehäuse,
das Pinraster ist gleich. **Vor der Bestellung mit der TE-Zeichnung abgleichen.**

Gegenstecker: 4-1437290-0 (A), 4-1437290-1 (B), Kontakte 3-1447221-4 / -3,
Crimpzange TE 1454509-1.

Quelle der Belegung: `moto/uaefi-moto-pinout.xlsx` (Blatt „Signale“). Tabellen unten sind daraus erzeugt.

### J30 = Stecker A, Basis (TE 6437288-1, Keying 1)

| Pin | Netz im Schaltplan | Signal |
|---|---|---|
| A1 | `+12V` | +12V Batterie / ECU-Versorgung |
| A2 | `/12V_KEY` | Zündung (Schlüssel) |
| A3 | `/12V_HALL` | +12V Hallsensoren (A) |
| A4 | `+5VP` | +5V Sensor (A1) |
| A5 | `+5VP` | +5V Sensor (A2) |
| A6 | `GNDA` | Sensormasse (A1) |
| A7 | `GNDA` | Sensormasse (A2) |
| A8 | `GND` | Leistungsmasse (A1) |
| A9 | `GND` | Leistungsmasse (A2) |
| A10 | `GND` | Leistungsmasse (A3) |
| A11 | `/OUT_INJ1` | Einspritzdüse 1 |
| A12 | `/OUT_INJ2` | Einspritzdüse 2 |
| A13 | `/OUT_IGN1` | Zündspule 1 |
| A14 | `/OUT_IGN2` | Zündspule 2 |
| A15 | `/OUT_LS1` | GPPWM1 |
| A16 | `/OUT_LS2` | GPPWM2 |
| A17 | `/IN_TPS1` | TPS1 |
| A18 | `/IN_CLT` | CLT |
| A19 | `/IN_IAT` | IAT |
| A20 | `/IN_HALL1` | Hall 1 (Kurbelwelle) |
| A21 | `+12V_RAW` | Lambda 1 Heizung + (LSU Pin 4) |
| A22 | `/3_WBO_Heater` | Lambda 1 Heizung − (LSU Pin 3) |
| A23 | `/1_WBO_Ip` | Lambda 1 Ip (LSU Pin 1) |
| A24 | `/2_WBO_Vm` | Lambda 1 Vm (LSU Pin 2) |
| A25 | `/6_WBO_Un` | Lambda 1 Un (LSU Pin 6) |
| A26 | `/5_WBO_Rtrim` | Lambda 1 Rtrim (LSU Pin 5) |
| A27 | `/VBUS` | USB VBUS |
| A28 | `/USB+` | USB D+ |
| A29 | `/USB-` | USB D− |
| A30 | `GND` | USB Masse |
| A31 | `/CAN+` | CAN High |
| A32 | `/CAN-` | CAN Low |
| A33 | `/OUT_LS_HOT1` | schwacher Low-Side 1 |
| A34 | `/IN_KNOCK_RAW` | Klopfsensor |

### J31 = Stecker B, Erweiterung (TE 6437288-2, Keying 2)

| Pin | Netz im Schaltplan | Signal |
|---|---|---|
| B1 | `/12V_HALL` | +12V Hallsensoren (B) |
| B2 | `+5VP` | +5V Sensor (B) |
| B3 | `GNDA` | Sensormasse (B) |
| B4 | `GND` | Leistungsmasse (B) |
| B5 | `/OUT_INJ3` | Einspritzdüse 3 |
| B6 | `/OUT_INJ4` | Einspritzdüse 4 |
| B7 | `/OUT_IGN3` | Zündspule 3 |
| B8 | `/OUT_IGN4` | Zündspule 4 |
| B9 | `/OUT_LS3` | GPPWM3 |
| B10 | `/OUT_LS4` | GPPWM4 |
| B11 | `/IN_AUX3` | Analog 5 |
| B12 | `/OUT_LS_HOT2` | schwacher Low-Side 2 |
| B13 | `/IN_TPS2` | TPS2 |
| B14 | `/IN_MAP` | MAP |
| B15 | `/IN_AUX1` | Analog 1 |
| B16 | `/IN_AUX2` | Analog 2 |
| B17 | `/IN_PPS1` | Analog 3 |
| B18 | `/IN_PPS2` | Analog 4 |
| B19 | `/IN_HALL2` | Hall 2 |
| B20 | `/IN_HALL3` | Hall 3 / Radsensor 1 (VSS) |
| B21 | `/IN_FLEX` | Flex |
| B22 | `/IN_BUTTON1` | Schalteingang 1 |
| B23 | `/VR_DISCRETE+` | VR1 + (diskret) |
| B24 | `/VR_DISCRETE-` | VR1 − (diskret) |
| B25 | `/VR_MAX9924+` | VR2 + (MAX9924) |
| B26 | `/VR_MAX9924-` | VR2 − (MAX9924) |
| B27 | `GND` | VR Schirm |
| B28 | `/IN_BUTTON2` | Radsensor 2 (Schalteingang 2) |
| B29 | `+12V_RAW` | Lambda 2 Heizung + (LSU Pin 4) |
| B30 | `/WBO2_Heater` | Lambda 2 Heizung − (LSU Pin 3) |
| B31 | `/WBO2_Ip` | Lambda 2 Ip (LSU Pin 1) |
| B32 | `/WBO2_Vm` | Lambda 2 Vm (LSU Pin 2) |
| B33 | `/WBO2_Un` | Lambda 2 Un (LSU Pin 6) |
| B34 | `/WBO2_Rtrim` | Lambda 2 Rtrim (LSU Pin 5) |

### J32–J35 = EGT1–4

| Buchse | + | − | Chip-Select am MAX31855 |
|---|---|---|---|
| EGT1 | `/EGT+` (vorher C10) | `/EGT-` (vorher C20) | `/EGT/SPI_CS` (PA15), bestehender U4 |
| EGT2 | NEU | NEU | `/DC1_PWM` (PC7, `MM100_OUT_PWM3`) |
| EGT3 | NEU | NEU | `/DC1_DIR` (PC8, `MM100_OUT_PWM4`) |
| EGT4 | NEU | NEU | `/DC2_PWM` (PC9, `MM100_OUT_PWM5`) |

## 2. EGT: 3 zusätzliche Kanäle

- Blatt `egt.kicad_sch` noch dreimal einfügen: MAX31855KASA + L1/L2 (470R@100MHz) +
  C28–C30 + D8 NUP2105L, identisch zu U4.
- `SCK3` (PC10) und `MISO3` (PC11) an alle vier Chips, CS wie in der Tabelle oben.
- Die vier MAX31855 **direkt hinter die Buchsen** setzen, auf dieselbe Kupferfläche und ohne
  Wärmequelle daneben. Der Chip misst die Vergleichsstelle an sich selbst.
- Firmware erwartet genau diese CS-Pins (`board_configuration.cpp`, `setEgtPins()`).

## 3. Keine E-Gas-Treiber mehr

- Blätter `DC Driver 1/2` (U1, U2 TLE9201 und Beschaltung) entfernen oder DNP.
- Die Netze `/DC1_PWM`, `/DC1_DIR`, `/DC2_PWM` werden zu EGT-Chip-Selects (siehe oben).
  Testpunkte P15/P17/P18 können bleiben.
- `/DC1_DIS` (PB14), `/DC2_DIR` (PB15) und `/DC2_DIS` (PA10) sind dann frei. Vorschlag:
  als Lötpads bzw. Testpunkte herausführen.
- Die Pull-Widerstände R9/R14 auf den DIS-Leitungen gehören zum Treiber und fallen mit weg.

## 4. Zweite Lambda-Steuerung

- Zweites `Module-wbo-0.6` (M8), angeschlossen wie M5: `V5_IN`→`+5VA`, `CANH/CANL`→`/CAN+ /CAN-`,
  GND, SWD auf eigene Stiftleiste wie J6.
- Sonden-Netze `LSU_Ip/Vm/Un/Rtrim/Htr` komplett auf Stecker B (B29–B34), Heizung + auf `+12V_RAW`. Lambda 1 liegt komplett auf Stecker A (A21–A26).
- **Hardware-Index:** Die WBO-Firmware liest SEL1/SEL2 dreiwertig (0 = low, 1 = offen, 2 = high),
  Index = 3·SEL1 + SEL2 (`wideband/firmware/boards/f0_module/port.cpp`, `shared/strap_pin.cpp`).
  M5: SEL1 low, SEL2 offen → Index 1 → rusEFI Lambda 1.
  **M8: SEL1 offen, SEL2 an PULL_UP2 → Index 5 → rusEFI Lambda 2**, ohne Einstellung in TunerStudio
  (gilt für ein frisches Modul ohne gespeicherte Konfiguration).
- Im Hellen-One-Rahmen ist das Modul ein Footprint, die Bestückung kommt aus `modules/wbo/0.6`.

## 5. +12V-Ausgang für Hallsensoren (A3 und B1)

`+12V_RAW` → PTC-Sicherung F11 (1206, 200 mA hold) → A3 und B1 (gleiches Netz).

## 6. Batterie für Stundenzähler

- BT1 (CR1220) über D1 an `/VBAT` bleibt **zwingend** bestückt.
- Empfehlung: Halter für CR2032 statt CR1220 (mehr Kapazität, leichter zu bekommen).
- Die Firmware speichert die Zähler in den RTC-Backup-Registern BKP1R–BKP6R.

## 7. Unverändert

- MCU-Modul, 12V/5V-Netzteil, Klopfsensor, beide VR-Module, CAN-Modul, WBO1, SD-Karte, Beschleunigungssensor.
- Der MAP-Sensor auf der Platine bleibt. Er dient als Barosensor für die Alpha-N-Korrektur
  (`setHellenMMbaro()`).
- Die Einspritz-, Zünd- und Low-Side-Stufen und ihre Freilaufdioden bleiben.

## 8. Layout-Hinweise

- J30 (A) und J31 (B) nebeneinander an einer Kante, zusammen etwa 95 mm. Die EGT-Buchsen an der Seitenkante.
- Leistungsmasse (A8–A10, A30, B4, B27; INJ/IGN/LS) getrennt von Sensormasse (GNDA) führen, wie bei der uaEFI.
- USB D+/D− als 90-Ω-Differenzialpaar zum Stecker, ESD-Schutz direkt am Pin.
- Gehäuse muss neu werden: 2× Superseal plus 4 Mini-Buchsen seitlich. Die Mini-Buchsen sind nicht wasserdicht.
