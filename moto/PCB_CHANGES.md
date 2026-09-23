# uaEFI moto: PCB-Änderungen gegenüber uaEFI Rev F3

## Stand

**Schaltplan: umgesetzt und geprüft** (Branch `uaefi-moto`):
- KiCad 10 ERC: 0 Fehler (Original: 0 Fehler), nur Bibliotheks- und Footprint-Link-Warnungen wie im Original
- Netzliste gegen das Original geprüft: kein bestehendes Netz getrennt oder verbunden, alle 55 übernommenen
  Steckerpins liegen auf dem Netz des alten Steckerpins
- Neue Teile auf dem Blatt rechts (Papier A3 → A2)

**Platine (`uaefi.kicad_pcb`): noch offen.** In KiCad: *Tools → Update PCB from Schematic* (F8), die alten
Molex-Footprints verschwinden, die neuen Teile platzieren und routen. Danach `revision.txt` hochzählen
(z. B. eigenes `BOARD_SUFFIX`/`BOARD_REVISION`) und ohne `[skip ci]` pushen, dann erzeugt GitHub Gerber/BOM/CPL.

| Referenz | Teil | Footprint |
|---|---|---|
| J30 | Stecker A, TE 6437288-1 (Keying 1) | `Connectors:6437288-2` (gleiches Pinbild, am Datenblatt prüfen) |
| J31 | Stecker B, TE 6437288-2 (Keying 2) | `Connectors:6437288-2` |
| J32–J35 | EGT1–4, Omega PCC-SMP-K | `moto:Omega_PCC-SMP-K` (**vorläufig**, am echten Teil prüfen) |
| Blätter EGT2–EGT4 | MAX31855 + Filter, U2004/U3004/U4004 usw. | wie U4 |
| M8 | zweites `Module-wbo-0.6` | `hellen-one-wbo-0.6:wbo` |
| F11 | PTC 200 mA für +12V Hall (A6) | 1206 |
| entfernt | J2, J3, J4, J5, J10 (Mini-Fit), Blätter DC Driver 1/2 (U1, U2, C1–C26, R9, R14, P15–P18), F3, F4 | |

Das Hardware-Index-Problem von Lambda 2 ist gelöst, siehe Abschnitt 4.

---

Basis: dieses Repo (`uaefi.kicad_sch` / `uaefi.kicad_pcb`, Rev F3, KiCad 10).
Firmware-Board: `firmware/config/boards/hellen/uaefi-moto` im Fork `chili023/rusefi-clean`, Branch `uaefi-moto`.
Die Netznamen unten stammen aus `gerber/uaefi.net` (Export der aktuellen Rev).

## 1. Stecker

| Alt | Neu |
|---|---|
| J4 Mini-Fit 8 (A), J5 Mini-Fit 18 (B), J2 Mini-Fit 20 (C), J10 Mini-Fit 16 (D), J3 Mini-Fit 6 (E) | entfallen |
| – | **J_A: TE 6437288-1**, Superseal 1.0, 34-polig, 90°, Keying 1 |
| – | **J_B: TE 6437288-2**, Superseal 1.0, 34-polig, 90°, Keying 2 |
| – | **J_EGT1..4: Omega PCC-SMP-K** (Mini-Thermoelementbuchse Typ K, liegend) |
| J7 SPOX (USB extern), J8 Mini-USB | nicht mehr nötig (USB liegt auf J_A), USB-C J9 für Tischbetrieb bleibt |
| U5 JDY-33 (Bluetooth) | optional, DNP empfohlen (bekanntes Einschaltproblem, siehe README) |

Symbol, Footprint und 3D-Modell für den Superseal-Header liegen schon in
`kicad6-libraries/` (`TE_6437288-2`). Keying 1 und 2 unterscheiden sich nur im Gehäuse,
das Pinraster ist gleich. **Vor der Bestellung mit der TE-Zeichnung abgleichen.**

Gegenstecker: 4-1437290-0 (A), 4-1437290-1 (B), Kontakte 3-1447221-4 / -3,
Crimpzange TE 1454509-1.

### J_A: Leistung und Ausgänge (Keying 1)

| Pin | Netz im uaEFI-Schaltplan | Signal |
|---|---|---|
| A1 | `+12V` | +12V (vorher A8) |
| A2 | `/12V_KEY` | Zündung (vorher A7) |
| A3 | `GND` | Masse Leistung |
| A4 | `GND` | Masse Leistung |
| A5 | `GND` | Masse Leistung |
| A6 | `NEU /12V_HALL` | +12V für Hallsensoren: +12V_RAW → PTC 200 mA → TVS |
| A7 | `/OUT_INJ1` | INJ1 |
| A8 | `/OUT_INJ2` | INJ2 |
| A9 | `/OUT_INJ3` | INJ3 |
| A10 | `/OUT_INJ4` | INJ4 |
| A11 | `/OUT_INJ5` | INJ5 |
| A12 | `/OUT_INJ6` | INJ6 |
| A13 | `/OUT_IGN1` | IGN1 |
| A14 | `/OUT_IGN2` | IGN2 |
| A15 | `/OUT_IGN3` | IGN3 |
| A16 | `/OUT_IGN4` | IGN4 |
| A17 | `/OUT_IGN5` | IGN5 |
| A18 | `/OUT_IGN6` | IGN6 |
| A19 | `/OUT_LS1` | GPPWM1 (mit Freilaufdiode) |
| A20 | `/OUT_LS2` | GPPWM2 (mit Freilaufdiode) |
| A21 | `/OUT_LS3` | GPPWM3 (mit Freilaufdiode) |
| A22 | `/OUT_LS4` | GPPWM4 (mit Freilaufdiode) |
| A23 | `/OUT_LS_HOT1` | schwacher Low-Side 1 (Relais) |
| A24 | `/OUT_LS_HOT2` | schwacher Low-Side 2 (Relais) |
| A25 | `+12V_RAW` | Lambda 1 Heizung + |
| A26 | `/3_WBO_Heater` | Lambda 1 Heizung − |
| A27 | `+12V_RAW` | Lambda 2 Heizung + |
| A28 | `NEU /WBO2_Heater` | Lambda 2 Heizung − (M8 LSU_Htr) |
| A29 | `/VBUS` | USB VBUS |
| A30 | `/USB+` | USB D+ |
| A31 | `/USB-` | USB D− |
| A32 | `GND` | USB-Masse |
| A33 | `/CAN+` | CAN H |
| A34 | `/CAN-` | CAN L |

### J_B: Sensorik (Keying 2)

| Pin | Netz im uaEFI-Schaltplan | Signal |
|---|---|---|
| B1 | `+5VP` | +5V |
| B2 | `+5VP` | +5V |
| B3 | `GNDA` | Sensormasse |
| B4 | `GNDA` | Sensormasse |
| B5 | `GNDA` | Sensormasse |
| B6 | `/IN_TPS1` | TPS1 |
| B7 | `/IN_TPS2` | TPS2 |
| B8 | `/IN_CLT` | CLT |
| B9 | `/IN_IAT` | IAT |
| B10 | `/IN_MAP` | MAP |
| B11 | `/IN_AUX1` | Analog 1 (PA0) |
| B12 | `/IN_AUX2` | Analog 2 (PA1) |
| B13 | `/IN_PPS1` | Analog 3 (PA3) |
| B14 | `/IN_PPS2` | Analog 4 (PC4) |
| B15 | `/IN_AUX3` | Analog 5 (PA7) |
| B16 | `/IN_HALL1` | Hall 1 |
| B17 | `/IN_HALL2` | Hall 2 |
| B18 | `/IN_HALL3` | Hall 3 / VSS |
| B19 | `/IN_FLEX` | Flex |
| B20 | `/IN_BUTTON1` | Schalteingang 1 |
| B21 | `/VR_DISCRETE+` | VR1 + |
| B22 | `/VR_DISCRETE-` | VR1 − |
| B23 | `/VR_MAX9924+` | VR2 + |
| B24 | `/VR_MAX9924-` | VR2 − |
| B25 | `GND` | VR-Schirm |
| B26 | `/IN_KNOCK_RAW` | Klopfsensor |
| B27 | `/1_WBO_Ip` | Lambda 1 Ip |
| B28 | `/2_WBO_Vm` | Lambda 1 Vm |
| B29 | `/6_WBO_Un` | Lambda 1 Un |
| B30 | `/5_WBO_Rtrim` | Lambda 1 Rtrim |
| B31 | `NEU /WBO2_Ip` | Lambda 2 Ip (M8) |
| B32 | `NEU /WBO2_Vm` | Lambda 2 Vm (M8) |
| B33 | `NEU /WBO2_Un` | Lambda 2 Un (M8) |
| B34 | `NEU /WBO2_Rtrim` | Lambda 2 Rtrim (M8) |

### J_EGT1..4

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
- Sonden-Netze `LSU_Ip/Vm/Un/Rtrim/Htr` auf J_B31..34 und J_A28, Heizung + auf `+12V_RAW`.
- **Hardware-Index:** Die WBO-Firmware liest SEL1/SEL2 dreiwertig (0 = low, 1 = offen, 2 = high),
  Index = 3·SEL1 + SEL2 (`wideband/firmware/boards/f0_module/port.cpp`, `shared/strap_pin.cpp`).
  M5: SEL1 low, SEL2 offen → Index 1 → rusEFI Lambda 1.
  **M8: SEL1 offen, SEL2 an PULL_UP2 → Index 5 → rusEFI Lambda 2**, ohne Einstellung in TunerStudio
  (gilt für ein frisches Modul ohne gespeicherte Konfiguration).
- Im Hellen-One-Rahmen ist das Modul ein Footprint, die Bestückung kommt aus `modules/wbo/0.6`.

## 5. +12V-Ausgang für Hallsensoren (J_A6)

`+12V_RAW` → PTC-Sicherung (z. B. 1206, 200 mA hold) → J_A6, dazu TVS gegen GND am Pin.

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

- J_A und J_B nebeneinander an einer Kante, zusammen etwa 95 mm. Die EGT-Buchsen an der Seitenkante.
- Leistungsmasse (J_A3–5, INJ/IGN/LS) getrennt von Sensormasse (GNDA) führen, wie bei der uaEFI.
- USB D+/D− als 90-Ω-Differenzialpaar zum Stecker, ESD-Schutz direkt am Pin.
- Gehäuse muss neu werden: 2× Superseal plus 4 Mini-Buchsen seitlich. Die Mini-Buchsen sind nicht wasserdicht.
