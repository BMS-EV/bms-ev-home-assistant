# BMS-EV — Home Assistant

BMS-EV controllers publish **native MQTT auto-discovery**. Home Assistant finds the
battery on its own and builds the entities — there is no custom component to install
and nothing to configure by hand.

**Full documentation:** [docs.bms-ev.com](https://docs.bms-ev.com/)

---

## Setup

1. In the controller's web interface, set your MQTT broker address, username and password.
2. Make sure Home Assistant uses the same broker ([MQTT integration](https://www.home-assistant.io/integrations/mqtt/)).
3. That's it. The device appears under **Settings → Devices & Services → MQTT** within a few seconds.

The controller publishes to `homeassistant/sensor/…/config`, the discovery prefix Home
Assistant listens on by default. It registers itself as a device (manufacturer *BMS-EV*),
so every entity lands under one device page rather than scattered across your entity list.

---

## Entities

Discovered automatically:

### Battery

| Entity | Unit | Notes |
|---|---|---|
| State Of Health | % | reported by the vehicle's own BMS |
| Battery Voltage | V | pack terminal voltage |
| Battery Current | A | charge positive, discharge negative |
| Stat Batt Power | W | instantaneous power |
| Battery Total Capacity | Wh | |
| Battery Remaining Capacity (real) | Wh | as reported by the pack |
| Battery Remaining Capacity (scaled) | Wh | after the configured SoC window |
| Battery Charged Energy | Wh | cumulative |
| Battery Discharged Energy | Wh | cumulative |

### Cells

| Entity | Unit | Notes |
|---|---|---|
| Cell Max Voltage | V | highest cell in the pack |
| Cell Min Voltage | V | lowest cell |
| Cell Voltage Delta | V | spread — the number to watch for pack health |
| Balancing Active Cells | count | how many cells are balancing right now |
| Balancing Status | text | |
| *Individual cell voltages* | V | one entity per cell, disabled by default |

Per-cell entities are registered but **disabled by default** — a 96-cell pack would
otherwise add 96 entities to your recorder. Enable the ones you want in the device page.

### Temperature

| Entity | Unit |
|---|---|
| Temperature Min | °C |
| Temperature Max | °C |
| CPU Temperature | °C |

### Limits and state

| Entity | Unit | Notes |
|---|---|---|
| Battery Max Charge Power | W | limit the BMS currently allows |
| Battery Max Discharge Power | W | |
| BMS Status | text | |
| Pause Status | text | |
| Emulator Status | text | controller state |
| Event Level | text | active fault severity |

### Diagnostics

Free Heap, Largest Heap Block — useful when reporting an issue.

---

## Dashboard

A starting point, using entities exactly as discovered. Replace `bms_ev` if your
controller uses a different MQTT topic name.

```yaml
type: vertical-stack
cards:
  - type: gauge
    entity: sensor.bms_ev_state_of_health
    name: State of Health
    min: 0
    max: 100
    severity:
      green: 80
      yellow: 70
      red: 0

  - type: glance
    entities:
      - entity: sensor.bms_ev_battery_voltage
        name: Voltage
      - entity: sensor.bms_ev_battery_current
        name: Current
      - entity: sensor.bms_ev_stat_batt_power
        name: Power

  - type: history-graph
    hours_to_show: 24
    entities:
      - sensor.bms_ev_cell_min_voltage
      - sensor.bms_ev_cell_max_voltage

  - type: entities
    title: Pack health
    entities:
      - entity: sensor.bms_ev_cell_voltage_delta
        name: Cell delta
      - entity: sensor.bms_ev_temperature_min
        name: Coldest cell
      - entity: sensor.bms_ev_temperature_max
        name: Hottest cell
      - entity: sensor.bms_ev_balancing_active_cells
        name: Cells balancing
```

### Energy dashboard

`Battery Charged Energy` and `Battery Discharged Energy` are cumulative, so they can
feed Home Assistant's Energy dashboard directly:

**Settings → Dashboards → Energy → Add battery system**, then pick those two entities
for energy going in and out of the battery.

---

## Alerts worth setting up

Cell delta is the single most useful number for spotting a pack going out of balance:

```yaml
automation:
  - alias: "Battery cell imbalance"
    trigger:
      - platform: numeric_state
        entity_id: sensor.bms_ev_cell_voltage_delta
        above: 0.1
        for: "00:10:00"
    action:
      - service: notify.persistent_notification
        data:
          title: "Battery cell imbalance"
          message: >
            Cell voltage spread is
            {{ states('sensor.bms_ev_cell_voltage_delta') }} V.
            A widening spread usually means one cell is ageing faster than the rest.
```

---

## If the device doesn't appear

| Check | How |
|---|---|
| Controller reaches the broker | Its web interface shows the MQTT connection state |
| Same broker on both sides | Home Assistant's MQTT integration must point at the same host |
| Discovery enabled in HA | On by default; if you turned it off, the device will not appear |
| Messages arriving | `mosquitto_sub -h <broker> -t 'homeassistant/#' -v` should show config messages |

The controller republishes its discovery messages periodically, so a Home Assistant
restart is enough to pick it up again — you do not need to re-flash anything.

---

## Manual configuration

Auto-discovery covers everything above, so hand-written sensors are rarely needed.
[`examples/mqtt_sensors.yaml`](examples/mqtt_sensors.yaml) is kept for unusual setups —
a separate broker, a filtered bridge, or a controller on firmware older than 16.5.0.

---

## Related

- [bms-ev-mqtt-examples](https://github.com/BMS-EV/bms-ev-mqtt-examples) — MQTT topics, Node-RED, Grafana, InfluxDB
- [bms-ev-docs](https://github.com/BMS-EV/bms-ev-docs) — compatibility dataset: 70 battery profiles × 56 inverters
- [docs.bms-ev.com](https://docs.bms-ev.com/) — full technical documentation

Built on [dalathegreat/Battery-Emulator](https://github.com/dalathegreat/Battery-Emulator).
