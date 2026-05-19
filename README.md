# 🌡️ Shelly BLU TRV — External Sensor Control via Home Assistant

> **Shelly Smart Home Challenge 2026 — Category: Build the Logic**

## The Problem

Shelly BLU TRVs measure temperature directly at the radiator head. As soon as the radiator heats up, the built-in sensor reads temperatures far above the actual room temperature — causing the valve to close prematurely. The result: **rooms that never reach the desired temperature**, especially in small or poorly insulated spaces.

## The Solution

Replace the TRV's built-in sensor logic with an **external room temperature sensor** as the reference point, controlled entirely through Home Assistant automations. A **boost mechanism** ensures the valve opens wide when heating is needed, and closes precisely when the target temperature is reached.

```
External Sensor (Tuya) ──→ HA Automation ──→ Shelly BLU TRV (Laden)
                                        └──→ Shelly BLU TRV (WC)
```

## How It Works

### Boost Logic (Core)

The controller runs every 5 minutes and on any temperature or setpoint change:

| Condition | TRV Setpoint | Effect |
|---|---|---|
| `room_temp < target − 0.5°C` | `target + 6°C` (max 30°C) | Valve opens wide → fast heating |
| `room_temp ≥ target` | `target` | Valve closes to exact setpoint |

By setting the TRV well above the actual room temperature, the valve stays fully open until the room is warm — overcoming the self-heating sensor problem entirely.

### Schedule

| Period | Laden | WC |
|---|---|---|
| Weekdays (Mon–Fri) 22:00 – Sat 13:00 | 22°C | 20°C |
| Weekend / off-hours | 18°C | 18°C |

### Override Modes

| Mode | Laden | WC | Notes |
|---|---|---|---|
| 🏖️ Vacation | 18°C | 18°C | Frost protection |
| 🎉 Party | 22°C | 20°C | Immediate warmth |

Vacation and Party mode are **mutually exclusive** — activating one automatically disables the other.

## Components

### Hardware
- 2× **Shelly BLU TRV** (radiator thermostats)
- 1× **Tuya-compatible temperature sensor** (external room reference, placed away from the radiator)
- Home Assistant instance with Shelly and Tuya integrations

### Home Assistant Entities

| Entity | Type | Purpose |
|---|---|---|
| `sensor.laden_tuya_temperature` | Sensor | External room temperature (reference) |
| `climate.heizung_laden` | Climate | Shelly BLU TRV — main room |
| `climate.heizung_ladenwc` | Climate | Shelly BLU TRV — WC |
| `input_number.soll_temperatur_laden` | Helper | Target temperature main room |
| `input_number.soll_temperatur_wc` | Helper | Target temperature WC |
| `input_boolean.urlaubs_modus_heizung_laden` | Helper | Vacation mode toggle |
| `input_boolean.partymodus_laden` | Helper | Party mode toggle |

## Quick Start — Blueprint (recommended)

The easiest way to use this project is via the included Home Assistant Blueprint. It combines all three automations into a single, configurable import.

**Import Blueprint:**

[![Open your Home Assistant instance and show the blueprint import dialog with a specific blueprint pre-filled.](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2FYOUR_USERNAME%2Fshelly-blu-trv-external-sensor%2Fblob%2Fmain%2Fblueprint%2Fshelly_blu_trv_external_sensor.yaml)

Or manually: **Settings → Automations → Blueprints → Import Blueprint** → paste the URL to `blueprint/shelly_blu_trv_external_sensor.yaml`.

**Blueprint inputs:**

| Input | Description |
|---|---|
| External Temperature Sensor | Room sensor entity (away from radiator) |
| TRV Zone 1 / Zone 2 | Shelly BLU TRV climate entities |
| Target Temp Helpers | `input_number` entities for setpoints |
| Vacation / Party Mode | `input_boolean` toggles |
| Comfort / Setback / Party temps | Temperature values per zone and mode |
| Schedule times | Comfort start (weekdays) + setback start (weekend) |
| Hysteresis | Dead band below target before boost activates |
| Boost | °C above target while valve is open |

---

## Manual Installation

### 1. Create Helpers

Add the contents of [`helpers/helpers.yaml`](helpers/helpers.yaml) to your HA configuration, or create the helpers manually under **Settings → Devices & Services → Helpers**.

### 2. Adapt Entity Names

In `automations/heizung_regler.yaml`, replace:
- `sensor.laden_tuya_temperature` → your external temperature sensor entity
- `climate.heizung_laden` → your TRV climate entity (room)
- `climate.heizung_ladenwc` → your TRV climate entity (WC / second zone)

### 3. Import Automations

Copy the three YAML files from `automations/` into your HA automation config, or import them via the UI (**Settings → Automations → ⋮ → Import from YAML**).

### 4. Tune Parameters

In `heizung_regler.yaml`, adjust to your needs:

```yaml
hysterese: 0.5   # °C below target to activate boost (increase for less switching)
boost: 6         # °C above target for TRV setpoint while heating (increase for slower-heating rooms)
```

## Automations Overview

| File | Purpose |
|---|---|
| `heizung_regler.yaml` | Core controller — boost logic, runs every 5 min |
| `heizung_zeitplan.yaml` | Schedule — sets target temps based on weekday/time/mode |
| `heizung_mutex.yaml` | Mutex — vacation and party mode are mutually exclusive |

## Why This Works Better Than the Native TRV Control

The Shelly BLU TRV in "auto" mode uses its built-in sensor. In a small room where the radiator is the only heat source, this sensor can read 5–10°C above actual room temperature while heating. Native control therefore cuts off heating too early.

This solution **decouples the measurement point from the actuator**, a standard approach in building automation that is rarely implemented in consumer smart home systems — made possible here by the open API of Shelly devices and the flexibility of Home Assistant.

## License

MIT — free to use, adapt, and share.

---

*Built with Shelly BLU TRV + Home Assistant · Shelly Smart Home Challenge 2026*
