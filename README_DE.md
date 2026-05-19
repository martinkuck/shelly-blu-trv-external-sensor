# 🌡️ Shelly BLU TRV — Externe Sensorsteuerung via Home Assistant

> **Shelly Smart Home Challenge 2026 — Kategorie: Build the Logic**

## Das Problem

Shelly BLU TRVs messen die Temperatur direkt am Heizkörperkopf. Sobald der Heizkörper heizt, liest der eingebaute Sensor Temperaturen weit über der tatsächlichen Raumtemperatur — das Ventil schließt zu früh. Das Ergebnis: **Räume, die die Wunschtemperatur nie erreichen**.

## Die Lösung

Der eingebaute Sensor wird durch einen **externen Raumtemperatursensor** als Referenz ersetzt, gesteuert über Home Assistant. Ein **Boost-Mechanismus** öffnet das Ventil weit, wenn geheizt werden muss, und schließt es präzise, wenn die Solltemperatur erreicht ist.

```
Externer Sensor (Tuya) ──→ HA Automation ──→ Shelly BLU TRV (Laden)
                                         └──→ Shelly BLU TRV (WC)
```

## Funktionsweise

### Boost-Logik (Kern)

Der Regler läuft alle 5 Minuten und bei jeder Temperatur- oder Sollwertänderung:

| Bedingung | TRV-Sollwert | Effekt |
|---|---|---|
| `Raumtemp < Soll − 0,5°C` | `Soll + 6°C` (max. 30°C) | Ventil weit auf → schnelles Heizen |
| `Raumtemp ≥ Soll` | `Soll` | Ventil auf exakten Sollwert |

### Zeitplan

| Zeitraum | Laden | WC |
|---|---|---|
| Werktags (Mo–Fr) 22:00 – Sa 13:00 | 22°C | 20°C |
| Wochenende / Absenkzeit | 18°C | 18°C |

### Übersteuerungsmodi

| Modus | Laden | WC |
|---|---|---|
| 🏖️ Urlaub | 18°C | 18°C |
| 🎉 Party | 22°C | 20°C |

Urlaubs- und Partymodus schließen sich **gegenseitig aus**.

## Automationen

| Datei | Zweck |
|---|---|
| `heizung_regler.yaml` | Kernregler — Boost-Logik, alle 5 Min. |
| `heizung_zeitplan.yaml` | Zeitplan — Solltemperaturen nach Wochentag/Modus |
| `heizung_mutex.yaml` | Mutex — Urlaub und Party schließen sich aus |

## Installation

1. Helfer aus `helpers/helpers.yaml` anlegen
2. Entitätsnamen in `heizung_regler.yaml` anpassen
3. Automationen in HA importieren
4. Parameter `hysterese` und `boost` nach Bedarf anpassen

Englische Volldokumentation: [README.md](README.md)

---

*Gebaut mit Shelly BLU TRV + Home Assistant · Shelly Smart Home Challenge 2026*
