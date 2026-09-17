# BMS-EV Home Assistant Integration

Home Assistant custom integration for [BMS-EV controllers](https://bms-ev.com/) via MQTT auto-discovery.

**Full documentation:** [docs.bms-ev.com](https://docs.bms-ev.com/)

## What this provides

Home Assistant entities auto-discovered when a BMS-EV controller is connected to the same MQTT broker:

| Entity | Type | Unit | Description |
|--------|------|------|-------------|
| Battery State of Charge | sensor | % | SOC reported by original vehicle BMS |
| Battery State of Health | sensor | % | SOH from vehicle BMS (if available) |
| Battery Voltage | sensor | V | Pack terminal voltage |
| Battery Current | sensor | A | Charge (+) / discharge (-) current |
| Battery Power | sensor | W | Instantaneous power |
| Min Cell Voltage | sensor | V | Lowest cell in pack |
| Max Cell Voltage | sensor | V | Highest cell in pack |
| Cell Delta | sensor | mV | Max-Min voltage difference (balance indicator) |
| Min Cell Temperature | sensor | °C | Lowest cell temperature |
| Max Cell Temperature | sensor | °C | Highest cell temperature |
| Isolation Resistance | sensor | kΩ | HV insulation health |
| Charge Current Limit | sensor | A | Max charging current allowed by BMS |
| Discharge Current Limit | sensor | A | Max discharging current allowed by BMS |
| Contactor State | binary_sensor | on/off | Main contactors closed/open |
| BMS State | sensor | text | idle/charging/discharging/balancing/fault |
| Error State | sensor | text | Active fault codes (if any) |

## Installation

### Requirements

- Home Assistant 2024.1 or newer
- MQTT broker accessible from BMS-EV controller and Home Assistant
- BMS-EV controller with firmware 3.5+ (MQTT auto-discovery support)

### Via HACS (recommended)

_This integration is being submitted to the HACS default repositories. Until then:_

1. HACS → Integrations → menu (⋮) → Custom repositories
2. Add repository URL: `https://github.com/BMS-EV/bms-ev-home-assistant`
3. Category: Integration
4. Click Install

### Manual installation

1. Copy `custom_components/bms_ev/` directory into your Home Assistant `config/custom_components/` directory
2. Restart Home Assistant
3. Go to Settings → Devices & Services → Add Integration → search "BMS-EV"

## Configuration

The integration reads from MQTT topics under `bms/{device_uid}/#` where `device_uid` is printed on the BMS-EV controller label.

### MQTT topics published by the BMS-EV controller

- `bms/{device_uid}/info` — device info (firmware version, hardware revision, battery model, inverter model)
- `bms/{device_uid}/spec_data` — battery specifications (nominal V, capacity, chemistry)
- `bms/{device_uid}/balancing_data` — per-cell voltages and temperatures
- `bms/{device_uid}/status` — real-time state (SOC, voltage, current, contactors, faults)

_Note: firmware currently publishes `/info`, `/spec_data`, `/balancing_data`, `/status` topics. There is no `/telemetry` topic._

## Example dashboard

See [examples/](examples/) for Lovelace dashboard configurations and automation examples.

## Related

- [BMS-EV MQTT examples](https://github.com/BMS-EV/bms-ev-mqtt-examples) — Node-RED flows, Grafana panels, InfluxDB scripts
- [BMS-EV documentation](https://docs.bms-ev.com/) — full technical reference
- [Compatibility Matrix](https://docs.bms-ev.com/compatibility/) — supported battery + inverter combinations

## Issues

Report integration bugs or missing entities: https://github.com/BMS-EV/bms-ev-home-assistant/issues

## License

[MIT License](LICENSE)

## Contact

- Shop: https://bms-ev.com/
- Documentation: https://docs.bms-ev.com/
- Email: office@bms-ev.com
