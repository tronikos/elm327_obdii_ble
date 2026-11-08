# ELM327 OBD-II BLE

[![hacs](https://img.shields.io/badge/HACS-Custom-41BDF5.svg)](https://github.com/hacs/integration)

A Home Assistant custom integration that reads vehicle diagnostic data from
ELM327 Bluetooth Low Energy (BLE) OBD-II adapters.

It exposes standard SAE J1979 Mode 01 parameters (engine speed, vehicle speed,
coolant temperature, fuel level, ...) and vehicle-specific custom parameters
(battery state of charge, pack voltage, odometer, ...) for electric and hybrid
vehicles.

The integration is built on top of the standalone
[`elm327-obdii`](https://pypi.org/project/elm327-obdii/) Python library and
fetches vehicle profile data from the
[OBDb](https://obdb.community) community database and the
[WiCAN](https://github.com/meatpiHQ/wican-fw) profile repository at runtime.

---

## Table of contents

- [How you can use this integration](#how-you-can-use-this-integration)
- [Prerequisites](#prerequisites)
- [Installation](#installation)
  - [Install via HACS (recommended)](#install-via-hacs-recommended)
  - [Manual installation](#manual-installation)
- [Configuration](#configuration)
  - [Installation parameters](#installation-parameters)
  - [Configuration options](#configuration-options)
- [Sensors](#sensors)
  - [Standard OBD-II sensors](#standard-obd-ii-sensors)
  - [Custom sensors](#custom-sensors)
  - [Diagnostic sensors](#diagnostic-sensors)
- [Data updates](#data-updates)
- [Known limitations](#known-limitations)
- [Troubleshooting](#troubleshooting)
- [Examples](#examples)
- [Removing the integration](#removing-the-integration)
- [License](#license)

---

## How you can use this integration

- Monitor your vehicle's fuel level and get low-fuel alerts
- Track battery health and state of charge for electric vehicles
- Log engine temperature for maintenance scheduling
- Monitor 12V battery voltage to detect parasitic drain
- Track daily commute distance via the odometer

## Prerequisites

- A working [Bluetooth](https://www.home-assistant.io/integrations/bluetooth)
  setup on the device running Home Assistant
- An ELM327 BLE OBD-II adapter (commonly sold as "ELM327 OBD2 Bluetooth"
  adapters)
- A vehicle with an OBD-II port (standard on all vehicles sold since 1996 in
  the US, 2001 in the EU)

### Supported adapters

The integration auto-discovers BLE devices advertising with local names
starting with `OBD` or `ELM327`. Most ELM327 BLE adapters on the market use
one of these names.

**Not supported:**

- USB-only ELM327 adapters
- WiFi ELM327 adapters
- Classic Bluetooth (SPP) ELM327 adapters (this integration is BLE-only)

### Vehicle profiles

The integration supports three profile sources:

1. **None / Standard OBD-II Only** — uses standard SAE J1979 Mode 01 PIDs that
   every OBD-II-compliant vehicle supports. Preselected with fuel level as the
   default.

2. **Import from OBDb community** — fetches from
   [obdb.community](https://obdb.community), a community-maintained database
   of 160+ vehicle models with 11,000+ signal definitions. Covers EVs (VW
   e-Golf, Hyundai Ioniq 5, Kia EV6, Tesla Model 3, ...) and ICE vehicles.

3. **Import from WiCAN repository** — fetches from the
   [meatpiHQ/wican-fw](https://github.com/meatpiHQ/wican-fw) repository, which
   has 77 vehicle profiles. Mode 01 PIDs in the WiCAN profile are automatically
   promoted to standard sensors.

You can also manually add custom PIDs via the integration options after setup.

## Installation

This custom integration depends on the
[`elm327-obdii`](https://pypi.org/project/elm327-obdii/) Python package, which Home Assistant installs automatically from PyPI the
first time the integration loads. You do **not** need to install it manually.

### Install via HACS (recommended)

1. Make sure [HACS](https://hacs.xyz) is installed in your Home Assistant
   instance.
2. Add this repository as a custom repository in HACS:
   - Open HACS → **Integrations** → ⋮ → **Custom repositories**
   - Repository URL: `https://github.com/tronikos/elm327_obdii_ble`
   - Category: **Integration**
   - Click **Add**
3. Click **Install** on the "ELM327 OBD-II BLE" card.
4. Restart Home Assistant.
5. Continue with [Configuration](#configuration) below.

### Manual installation

1. Download the latest release ZIP from the
   [releases page](https://github.com/tronikos/elm327_obdii_ble/releases).
2. Unzip it and copy the `elm327_obdii_ble` folder into your Home
   Assistant `custom_components` directory, so that you end up with:

   ```
   /config/custom_components/elm327_obdii_ble/
       __init__.py
       config_flow.py
       const.py
       coordinator.py
       diagnostics.py
       entity.py
       manifest.json
       quality_scale.yaml
       sensor.py
       strings.json
       translations/
           en.json
   ```

3. Restart Home Assistant.
4. Continue with [Configuration](#configuration) below.

## Configuration

This integration is configured via the UI:

1. In Home Assistant, go to **Settings** → **Devices & services**.
2. Click **Add integration**.
3. Search for "ELM327 OBD-II BLE".
4. If your adapter was discovered via Bluetooth, confirm the setup.
5. Otherwise, select your adapter from the list of discovered Bluetooth
   devices, or enter its MAC address manually.
6. Choose a vehicle profile source (None, OBDb, or WiCAN).
7. If importing from OBDb: select make → model → year.
8. Select which standard Mode 01 PIDs to poll.
9. Review and confirm.

### Installation parameters

- **Bluetooth address**: The MAC address of your ELM327 BLE adapter.
  Auto-discovered via Bluetooth; can be entered manually if not found.
- **Read (RX) Characteristic UUID**: The GATT characteristic UUID for reading
  data from the adapter. Auto-discovered; defaults to
  `0000fff1-0000-1000-8000-00805f9b34fb`.
- **Write (TX) Characteristic UUID**: The GATT characteristic UUID for writing
  commands to the adapter. Auto-discovered; defaults to
  `0000fff2-0000-1000-8000-00805f9b34fb`.

### Configuration options

The following options can be changed after setup via **Configure**:

- **Vehicle profile**: The source of custom PID definitions (None, OBDb, WiCAN,
  or manual).
- **Standard OBD-II PIDs**: Which standard Mode 01 PIDs to poll.
- **Custom PIDs**: Add, edit, or delete custom PID definitions.
- **Battery protection**:
  - **Query battery voltage via AT RV**: Enables the ignition-on/off detection
    state machine.
  - **Ignition on threshold (V)**: Voltage above which the car is considered
    "on" (default: 13.1V).
  - **Ignition off threshold (V)**: Voltage below which the car is considered
    "off" (default: 13.0V).
  - **Post-shutdown grace period (seconds)**: How long to keep polling before
    transitioning to "car off" (default: 30s).

The on threshold must be higher than the off threshold.

If you don't want the voltage-based state machine at all — for example, on a
vehicle whose OBD-II port is only powered when the ignition is on — uncheck
**Query battery voltage via AT RV**. The integration will then:

- Poll every 5 seconds continuously (no car-off / grace-period / slow-poll
  states).
- Send the full query plan (all enabled standard + custom PIDs) on every
  cycle.
- Show the **Battery Voltage** sensor as `unknown` (no `AT RV` data is
  collected).
- Show the **Adapter State** sensor as `car_on` whenever the adapter is
  reachable, and `out_of_range` when it isn't.

Do **not** uncheck this option on vehicles where the OBD-II port stays powered
when the engine is off. Without the state machine, the integration will keep
polling 24/7 and can drain the 12V auxiliary battery over hours or days.

## Sensors

### Standard OBD-II sensors

When using the default profile, the following sensor is created by default:

- **Fuel Level** — Mode 01 PID 0x2F (%)

Additional standard PIDs can be enabled via the integration options (engine
speed, vehicle speed, engine coolant temperature, ambient air temperature,
vehicle voltage, engine load, MAF rate, engine run time, and 160+ others).

### Custom sensors

When importing a vehicle profile from OBDb or WiCAN, vehicle-specific sensors
are created. For example, the VW e-Golf profile provides:

- **Battery SOC (BMS)** — state of charge from the battery management system
  (%)
- **Battery SOC (displayed)** — state of charge as shown on the dashboard (%)
- **Battery Pack Voltage** — high-voltage battery pack voltage (V)
- **Battery Pack Current** — high-voltage battery pack current (A)
- **Odometer** — total distance traveled (km)

### Diagnostic sensors

- **Adapter State** — the polling state machine state (`car_on`, `car_off`,
  `grace_period`, `out_of_range`).
- **Battery Voltage** — the 12V auxiliary battery voltage from `AT RV` (V).

## Data updates

The integration uses a voltage-gated polling state machine to protect the
vehicle's 12V auxiliary battery:

- **Car on** (voltage ≥ on threshold): Polls every 5 seconds for real-time
  data.
- **Grace period** (voltage dropped below off threshold): Keeps polling every
  5 seconds for the configured grace period (default 30 seconds) to handle
  brief voltage dips.
- **Car off** (grace period expired): Polls every 300 seconds (5 minutes) to
  check if the car has been turned back on.
- **Out of range** (adapter not in BLE range): Polls every 3600 seconds (1
  hour) to check if the adapter has come back in range. Polling resumes
  immediately when the adapter advertises again.

Voltage is read via the `AT RV` command (adapter-reported, not a Mode 01 PID).
This works even when the ECU is asleep.

Polling intervals are fixed and not user-configurable, per the Home Assistant
integration quality scale guidelines.

## Known limitations

### BLE-only

This integration only supports Bluetooth Low Energy (BLE) ELM327 adapters.
USB and WiFi adapters are not supported.

### Formula limitations

Custom PID formulas use a structured `fmt` representation (matching OBDb's
format) that supports:

- Contiguous bit field extraction (bix + len)
- Linear scaling (mul/div/add)
- Signed values (two's complement)
- Byte-swap for Intel byte order (blsb)
- Clamping (min/max)
- Enumerations (map)

Non-contiguous bit fields and multi-field arithmetic
(e.g., `voltage × current = power`) are not supported. Approximately 1% of
WiCAN profile formulas use these patterns and are skipped on import. OBDb
profiles are fully supported.

### WiCAN profile accuracy

Some WiCAN profiles contain incorrect byte indices in their formulas (e.g., a
32-bit value specified as non-contiguous bytes when the actual CAN frame
layout is contiguous). OBDb profiles are generally more accurate because they
enforce contiguous bit fields in their schema.

### Multi-ECU responses

The integration does not set `ATCRA` (receive filter) when importing from
OBDb's matrix data alone. On vehicles with multiple ECUs responding to the
same query, the integration may pick up the wrong ECU's response. Importing
the per-vehicle repo `default.json` (done automatically in the config flow)
provides `ATCRA` filters that resolve this.

## Troubleshooting

### Adapter not found

- Ensure the adapter is plugged into the vehicle's OBD-II port and powered
  (the vehicle doesn't need to be running, but the port provides 12V).
- Check that Bluetooth is enabled and the adapter is in range.
- Try unplugging and re-plugging the adapter.
- Check the Bluetooth integration for the adapter's MAC address.

### No data / sensors show unavailable

- Ensure the vehicle's ignition is on (or at least in accessory mode).
- Some vehicles only power the OBD-II port when the engine is running.
- Check the **Adapter State** sensor — if it shows `out_of_range`, the adapter
  isn't reachable.
- Check the **Battery Voltage** sensor — if it's below the off threshold, the
  integration has entered `car_off` state.

### Wrong values

- If using a WiCAN profile, try the equivalent OBDb profile instead (OBDb
  profiles are generally more accurate).
- For custom PIDs, verify the `fmt` fields (bix, len, div) match your
  vehicle's ECU response layout.
- Check the OBDb website for your vehicle to compare expected values.
- Some inexpensive ELM327 adapters may not properly support CAN header
  switching (ATSH), which can cause custom PIDs that require addressing
  non-default ECUs to fail or return incorrect data. If custom PID sensors
  show `unknown` but standard PIDs work, try a different adapter.

### BUFFER FULL errors

The ELM327 adapter's internal buffer can overflow when polling too many PIDs
too quickly. Reduce the number of enabled PIDs in the integration options.

### Connection drops frequently

- Move the Bluetooth adapter closer to the OBD-II port.
- Check for interference from other BLE devices.
- Some cheap ELM327 clones have poor BLE antennas — try a different adapter.

## Examples

### Low fuel alert

```yaml
alias: "Low fuel alert"
trigger:
  - platform: numeric_state
    entity_id: sensor.my_car_fuel_level
    below: 25
action:
  - service: notify.mobile_app_my_phone
    data:
      message: "Fuel level is at {{ states('sensor.my_car_fuel_level') }}% — time to refuel"
```

### Battery voltage monitoring

```yaml
alias: "12V battery low voltage"
trigger:
  - platform: numeric_state
    entity_id: sensor.my_car_battery_voltage
    below: 12.0
    for:
      minutes: 10
action:
  - service: notify.mobile_app_my_phone
    data:
      message: "12V battery voltage is low ({{ states('sensor.my_car_battery_voltage') }}V) — possible parasitic drain"
```

### EV state of charge tracking

```yaml
alias: "Log daily SOC"
trigger:
  - platform: time
    at: "23:59:00"
action:
  - service: input_number.set_value
    target:
      entity_id: input_number.ev_soc_log
    data:
      value: "{{ states('sensor.my_car_battery_soc_bms') }}"
```

## Removing the integration

This integration follows standard integration removal. No extra steps are
required.

To remove a device from Home Assistant, go to **Settings** → **Devices &
services** → **ELM327 OBD-II BLE** → click the device → ⋮ → **Delete device**.

## License

This integration is released under the Apache License 2.0.

Vehicle profile data is fetched at runtime from:

- [OBDb](https://obdb.community) — licensed under
  [CC-BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/)
- [WiCAN](https://github.com/meatpiHQ/wican-fw) — licensed under
  [GPL-3.0](https://www.gnu.org/licenses/gpl-3.0.html)

Profile data is not redistributed with this integration.
