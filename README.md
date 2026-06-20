# ☀️ FusionSolar Dashboard

A single-file web dashboard for real-time monitoring of **Huawei SUN2000** solar systems via Home Assistant — with Luna2000 battery, EV wallbox, Feyree EVSE and weather forecast support.

![Dashboard Preview](screenshot.png)

---

## Features

- **Animated real-time energy flow** — PV → House → Battery → Grid → EV
- **Instant KPIs** — PV production, net house consumption (EV excluded), battery SOC, grid power, wallbox
- **Daily chart** — PV, house, battery charge/discharge, Wallbox EV as separate dataset
- **Smart EV charging**:
  - Manual mode with current selection (6–32 A)
  - Dynamic solar mode with anti-oscillation debounce
  - Scheduled mode with time window and day-of-week selection
- **Feyree EVSE panel** — status, temperature, work mode, current modulation
- **Weather & Yield** — 7-day forecast + PV production estimate
- **Dark / Light theme**

---

## ⚠️ Prerequisites — Enable Modbus TCP on the Huawei Inverter

Before setting up the Huawei Solar integration in Home Assistant, you must **enable the Modbus TCP protocol** on the SUN2000 inverter. Without this step, Home Assistant cannot communicate with the inverter.

### How to enable it

1. Open the **FusionSolar** app on your phone
2. Go to **Devices** → select your inverter
3. Navigate to **Settings** → **Communication Settings**
4. Enable **Modbus TCP** and set the port (default: **502**)
5. Save and wait for the inverter to restart

> **Alternative path** (some firmware versions):
> **Maintenance** → **Access Management** → **Advanced Settings**
> Access code: `0` or `00000`

### Verify connectivity

Once enabled, confirm the inverter is reachable from your local network:

```bash
# Linux / macOS / WSL
nc -zv 192.168.1.X 502
```

If the connection succeeds, Home Assistant will be able to communicate with the inverter via the **Huawei Solar** integration.

---

## Requirements

| Component | Notes |
|---|---|
| Home Assistant | With REST API access enabled |
| [Huawei Solar (HACS)](https://github.com/wlcrs/huawei_solar) | Required HA integration for SUN2000 |
| SUN2000 inverter | Tested on SUN2000-5KTL-L1 |
| Luna2000 battery | Optional |
| DDSU666-H meter | Recommended for accurate grid measurements |
| Tuya EV meter | For wallbox current + voltage monitoring |
| Feyree EVSE | Optional — wallbox with current modulation via Tuya Local |

---

## Installation

### 1. Copy files to Home Assistant

Place the files in the HA local web folder:

```
/config/www/
├── fusionsolar_dashboard.html
└── config.json                  ← create this from the example (see below)
```

The `/config/www/` folder is served at `http://HA_IP:8123/local/`.

### 2. Create config.json

Copy the example and fill in your details:

```bash
cp config.json.example config.json
```

Minimal `config.json`:

```json
{
  "haUrl": "http://192.168.1.100:8123",
  "haUrlLocal": "http://192.168.1.100:8123",
  "token": "eyJ0eXAiOiJKV1Qi..."
}
```

> **How to get a token**: Home Assistant → Profile → Long-Lived Access Tokens → Create Token

### 3. Open in browser

```
http://HA_IP:8123/local/fusionsolar_dashboard.html
```

---

## Entity Configuration

By default the dashboard uses the entity IDs from my Italian Huawei Solar installation. You can override any of them in `config.json` under the `entities` key — only specify what differs from the defaults.

```json
{
  "haUrl": "http://192.168.1.100:8123",
  "token": "YOUR_TOKEN",
  "entities": {
    "inverter": {
      "input_power":  "sensor.huawei_solar_power_input",
      "active_power": "sensor.huawei_solar_active_power",
      "daily_energy": "sensor.huawei_solar_daily_yield",
      "total_energy": "sensor.huawei_solar_total_yield",
      "efficiency":   "sensor.huawei_solar_efficiency",
      "internal_temp":"sensor.huawei_solar_internal_temperature",
      "pv1_voltage":  "sensor.huawei_solar_pv_voltage_1",
      "pv1_current":  "sensor.huawei_solar_pv_current_1",
      "pv2_voltage":  "sensor.huawei_solar_pv_voltage_2",
      "pv2_current":  "sensor.huawei_solar_pv_current_2"
    },
    "battery": {
      "bat_soc":             "sensor.huawei_solar_battery_state_of_capacity",
      "bat_power":           "sensor.huawei_solar_battery_charge_discharge_power",
      "bat_voltage":         "sensor.huawei_solar_battery_bus_voltage",
      "bat_current":         "sensor.huawei_solar_battery_bus_current",
      "bat_daily_charge":    "sensor.huawei_solar_battery_day_charge",
      "bat_daily_discharge": "sensor.huawei_solar_battery_day_discharge",
      "bat_total_charge":    "sensor.huawei_solar_battery_total_charge",
      "bat_total_discharge": "sensor.huawei_solar_battery_total_discharge"
    },
    "meter_grid": {
      "meter_active_power": "sensor.huawei_solar_power_meter_active_power",
      "meter_grid_v":       "sensor.huawei_solar_power_meter_voltage",
      "meter_power_factor": "sensor.huawei_solar_power_meter_power_factor",
      "meter_daily_export": "sensor.huawei_solar_daily_export",
      "meter_daily_import": "sensor.huawei_solar_daily_import",
      "meter_total_export": "sensor.huawei_solar_power_meter_exported",
      "meter_total_import": "sensor.huawei_solar_power_meter_imported"
    },
    "wallbox_ev": {
      "ev_current":      "sensor.ev_meter_current",
      "ev_voltage":      "sensor.ev_meter_voltage",
      "ev_energy_total": "sensor.ev_meter_total_energy",
      "ev_switch":       "switch.ev_circuit_breaker"
    },
    "feyree": {
      "feyree_switch":      "switch.feyree_ev_charger_charging",
      "feyree_current_set": "number.feyree_ev_charger_charging_current",
      "feyree_status":      "sensor.feyree_ev_charger_status",
      "feyree_temp":        "sensor.feyree_ev_charger_temperature",
      "feyree_workmode":    "select.feyree_work_mode"
    }
  }
}
```

### Available entity groups

| Group | Description |
|---|---|
| `inverter` | PV production, strings, temperature, yield |
| `battery` | SOC, power, voltage, daily/total statistics |
| `meter_grid` | Bidirectional grid meter (DDSU666-H) |
| `meter_casa` | Optional separate house consumption meter |
| `wallbox_ev` | Tuya EV meter + circuit breaker switch |
| `feyree` | Feyree EVSE (optional) |

See `config.json.example` for the full list of supported keys.

---

## Project Structure

```
fusionsolar-dashboard/
├── fusionsolar_dashboard.html   # Complete app — single self-contained file
├── config.json.example          # Configuration template (safe to commit)
├── .gitignore                   # Excludes config.json from version control
└── README.md
```

---

## Security Notes

- **Never commit `config.json`** — it contains your HA token and local IP
- The included `.gitignore` already excludes it
- For remote access, use HTTPS and consider HA authentication
- The long-lived token grants full API access to Home Assistant — keep it secret

---

## Tested Hardware

| Device | Status |
|---|---|
| Huawei SUN2000-5KTL-L1 | ✅ |
| Huawei Luna2000 battery | ✅ |
| Huawei DDSU666-H meter | ✅ |
| Tuya EV meter (TS011F) | ✅ |
| Feyree EVSE | ✅ |

---

## License

MIT — free for personal and commercial use.
