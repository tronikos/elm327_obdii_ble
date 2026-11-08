# ELM327 OBD-II BLE

Read vehicle diagnostic data from ELM327 Bluetooth Low Energy (BLE) OBD-II
adapters in Home Assistant.

- Standard SAE J1979 Mode 01 PIDs (engine speed, vehicle speed, coolant
  temperature, fuel level, ...)
- Vehicle-specific custom PIDs imported from the
  [OBDb](https://obdb.community) community database (160+ vehicle models) or
  the [WiCAN](https://github.com/meatpiHQ/wican-fw) profile repository
  (77 vehicle profiles)
- Voltage-gated polling state machine to protect the vehicle's 12V auxiliary
  battery
- Auto-discovery of BLE adapters with `OBD*` / `ELM327*` local names

Depends on the [`elm327-obdii`](https://pypi.org/project/elm327-obdii/)
Python library (installed automatically from PyPI).
