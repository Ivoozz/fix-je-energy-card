# fix-je-energy-card

**Home Assistant Custom Energy Flow Card for GoodWe + JK BMS**
`fix-je-energy-card.js` · Unified Edition **V1.1.3**

<img width="1708" height="2520" alt="IMG_20260513_114249" src="https://github.com/user-attachments/assets/e4c54570-5b9e-43c4-9326-b44d4105a5d1" />

---

## Overview

`fix-je-energy-card` is a fully custom Home Assistant Lovelace card that renders a live, animated energy-flow diagram for a GoodWe solar inverter system with JK BMS battery storage. It is self-contained in a single JavaScript file — no dependencies, no HACS card required beyond loading the resource.

The card combines an SVG energy-flow canvas (sun arc, animated flow paths, inverter, battery, grid, home, EV nodes) with an optional bottom stats grid displaying real-time financial and state statistics. Everything updates live via the standard `hass` setter.

---

## Features

- **Animated Energy Flows**: Smooth, animated S-curve paths showing power direction (solar → inverter → battery / grid / home / EV).
- **Sun/Moon Arc Tracking**: Dynamic sun position tracking along an arc based on `sun.sun` elevation during daytime, and moon tracking during nighttime.
- **Battery SOC & Status display**: Real-time battery SOC percentage rendered on the flow line with a background mask for maximum readability. Fully color-coded battery fill with dynamic charging/discharging states.
- **Dual Battery Support**: Independent telemetry tracking for dual-stacked battery packs (SOC, voltage, current, power).
- **EV / Car Charger Node**: Renders a dedicated EV node with state machine integration (`charging`, `completed`, etc.), SOC, and charging ETA.
- **Extra PV Strings (PV3 / PV4)**: Support for additional strings with optional toggles.
- **Inverter Telemetry Panel**: A bottom-aligned panel displaying inverter daily totals (Today PV, Charge/Discharge kWh, remaining battery Ah/kWh, Today Load).
- **Custom & Financial Stats**: A customizable 2x3 statistics grid containing real-time price integration, remaining battery storage value, net grid imports, battery power, solar live power, and remaining capacity in kWh.
- **Full Visual Editor**: Configure everything directly in the Lovelace card editor UI.

---

## Custom & Financial Stats Integration

When **Custom & Financial Stats** is enabled in the editor, a new grid is rendered below the energy flow diagram with the following Dutch labels and live values:

| Dutch Title | English Meaning | Source / Calculation |
|---|---|---|
| **Batterij Vermogen** | Battery Power | Displays the current charge/discharge rate in Watts. |
| **Zon Live** | Live Solar Power | Real-time PV generation in Watts (sum of PV1, PV2, and extra PV3/PV4 strings). |
| **Netto Import** | Net Grid Import | Net power import from the grid in Watts. Green when exporting, red when importing. |
| **Huidige Prijs** | Current Price | The live electricity price from your selected price entity, formatted as `€x.xxx`. |
| **Opslag Waarde** | Storage Value | The current monetary value of the energy stored in your battery. Calculated as: `Remaining Battery Energy (kWh) × Current Price (€/kWh)` and formatted as `€x.xxx`. |
| **Resterende Accu** | Remaining Battery | Remaining capacity in kWh, calculated dynamically based on SOC and battery capacity. |

### Configuration of Price Entity
To enable financial stats (Huidige Prijs & Opslag Waarde):
1. In the Lovelace visual editor, scroll to the **Custom & Financial Stats** section.
2. Toggle the section **+ Enable** chip.
3. Select your dynamic electricity price sensor in the **Current Price Sensor** entity picker (e.g., a Nordpool sensor or a dynamic energy contract utility sensor that returns a numerical price per kWh).

---

## Installation

### Method 1 — HACS (Recommended)

1. In HACS, go to **Frontend → ⋮ → Custom repositories**
2. Paste: `https://github.com/Ivoozz/fix-je-energy-card` (without `.git`)
3. Category: **Lovelace**
4. Install the card – the resource is added automatically.
5. Click the **+ Explore & Download Repositories** button.
6. Search for **fix-je-energy-card**.
7. Click **Download** – the resource is added automatically.
8. **Hard refresh** your browser (`Ctrl + Shift + R` / `Cmd + Shift + R`).
9. Open the visual editor to configure entities.

> HACS handles resource registration automatically. No manual resource entry needed.

---

### Method 2 — Manual

1. Copy `fix-je-energy-card.js` to your HA config folder:
   ```
   /config/www/fix-je-energy-card.js
   ```

2. Register as a Lovelace resource:
   *(Settings → Dashboards → Resources → Add)*
   ```yaml
   url: /local/fix-je-energy-card.js
   type: module
   ```

3. Add to a dashboard view:
   ```yaml
   type: custom:fix-je-energy-card
   ```

4. Open the visual editor to configure entities.

---

## Configuration Reference

All keys are configured through the visual editor. The YAML equivalents are listed below for reference or manual setup.

### Core / Solar

| Key | Default | Description |
|---|---|---|
| `inverter_name` | `''` | Label shown in the inverter node |
| `pv1_power` | `sensor.goodwe_pv1_power` | PV string 1 power (W) |
| `pv2_power` | `sensor.goodwe_pv2_power` | PV string 2 power (W) |
| `pv3_power` | `''` | PV string 3 — optional, enable via Extra PV toggle |
| `pv4_power` | `''` | PV string 4 — optional, enable via Extra PV toggle |
| `pv_total_power` | `sensor.goodwe_pv_power` | Total PV power (W) |
| `pv_max_power` | `7500` | Max PV power for bar scaling (W) |
| `today_pv` | `sensor.goodwe_today_s_pv_generation` | Today's PV generation (kWh) |
| `total_pv_gen_entity` | `sensor.goodwe_total_pv_generation` | Lifetime PV generation (kWh) |
| `inv_temp` | `sensor.goodwe_inverter_temperature_module` | Inverter temperature |
| `today_batt_chg` | `sensor.goodwe_today_battery_charge` | Today battery charge (kWh) |
| `today_load` | `sensor.goodwe_today_load` | Today load (kWh) |
| `sun` | `sun.sun` | Sun entity for arc position |

### Grid

| Key | Default | Description |
|---|---|---|
| `grid_active_power` | `sensor.goodwe_active_power` | Grid active power (W) |
| `grid_import_energy` | `sensor.goodwe_today_energy_import` | Today grid import (kWh) |
| `grid_export_energy` | `''` | Today grid export (kWh) — optional |
| `grid_power_alt` | `sensor.grid_phase_a_power` | Alternate grid power sensor |
| `invert_grid_power` | `false` | Invert sign — enable if positive = exporting |
| `consump` | `sensor.goodwe_house_consumption` | House consumption (W) |

### Primary Battery

| Key | Default | Description |
|---|---|---|
| `_show_battery` | `true` | Show primary battery section |
| `battery_soc` | `sensor.jk_soc` | Battery state of charge (%) |
| `battery_power` | `sensor.jk_power` | Battery power (W) |
| `battery_current` | `sensor.jk_current` | Battery current (A) |
| `battery_voltage` | `sensor.jk_voltage` | Battery voltage (V) |
| `battery_temp1` | `sensor.jk_temp1` | Cell temp probe 1 |
| `battery_temp2` | `sensor.jk_temp2` | Cell temp probe 2 |
| `battery_mos` | `sensor.jk_mos` | BMS MOS temperature |
| `battery_min_cell` | `sensor.jk_cellmin` | Min cell voltage |
| `battery_max_cell` | `sensor.jk_cellmax` | Max cell voltage |
| `batt_dis` | `sensor.goodwe_today_battery_discharge` | Today discharge (kWh) |
| `battery_full_ah` | `314` | Battery capacity (Ah) |
| `battery_full_wh` | `16076` | Battery capacity (Wh) |
| `goodwe_battery_soc` | `sensor.goodwe_battery_state_of_charge` | Fallback SOC |
| `goodwe_battery_curr` | `sensor.goodwe_battery_current` | Fallback current |
| `invert_battery_power` | `false` | Invert sign — enable if positive = discharging |

### Secondary Battery

| Key | Default | Description |
|---|---|---|
| `_show_battery2` | `false` | Enable secondary battery (chip toggle) |
| `battery2_soc` | `''` | Secondary SOC (%) |
| `battery2_power` | `''` | Secondary power (W) |
| `battery2_current` | `''` | Secondary current (A) |
| `battery2_voltage` | `''` | Secondary voltage (V) |
| `battery2_mos` | `''` | Secondary BMS temperature |

### EV / Car Charger

| Key | Default | Description |
|---|---|---|
| `_show_ev` | `false` | Enable EV section (chip toggle) |
| `charger_state` | `''` | Charger state entity (string: `charging`, `completed`, etc.) |
| `charger_power` | `''` | Charger power (W) |
| `charger_current` | `''` | Charger current (A) |
| `charger_soc` | `''` | Car battery SOC (%) |
| `charger_eta` | `''` | Charge ETA in minutes — optional |
| `charger_battery_capacity_wh` | `''` | EV battery capacity (Wh) |

### Custom & Financial Stats

| Key | Default | Description |
|---|---|---|
| `_show_custom_stats` | `false` | Show Custom & Financial stats section (chip toggle) |
| `current_price_entity` | `''` | Current electricity price sensor entity ID (e.g. in €/kWh) |

---

## Visual Editor Sections

| Section | Toggle | Description |
|---|---|---|
| General | — | Inverter name, battery capacity unit selection |
| Solar | — | PV1, PV2 entities |
| Extra PV Strings | `+ Enable` chip | PV3, PV4 |
| Solar Extras | — | Totals, temperatures, today stats |
| Grid | — | Grid power, import/export, consumption |
| Primary Battery | `+ Enable` chip | Full BMS telemetry |
| Secondary Battery | `+ Enable` chip | Second pack configuration |
| EV / Car Charger | `+ Enable` chip | Charger state, SOC, ETA |
| Custom & Financial Stats | `+ Enable` chip | Electricity price entity selection and display toggle |

---

## Colour Logic

| Metric | Thresholds |
|---|---|
| **SOC** | ≤25% red · ≤50% orange · ≤75% blue · >75% green |
| **Inverter / Env Temp** | ≤25°C green · ≤45°C orange · >45°C red |

---

## File Structure

```
fix-je-energy-card.js
│
├── class FixJeEnergyCardEditor   (visual editor — HTMLElement, shadow DOM)
│   ├── _render()           builds editor sections
│   ├── makeSection()       collapsible section with optional chip toggle
│   ├── picker()            ha-selector entity picker
│   ├── textField()         ha-textfield input
│   ├── numberField()       ha-textfield numeric input
│   ├── switchRow()         pill toggle (used for invert flags)
│   └── pickerMaybeDisabled() picker with override veil overlay
│
└── class FixJeEnergyCard         (main card — HTMLElement, shadow DOM)
    ├── setConfig()         merges config with stub defaults, triggers static build
    ├── _buildStaticSVG()   renders full SVG + HTML stat panel (called once per config)
    ├── _updateDynamic()    updates all live values, colours, animations (called on every hass update)
    ├── _val()              safe numeric entity reader
    ├── _strVal()           safe string entity reader
    ├── _socColor()         SOC → hex colour
    ├── _cellTempColor()    temperature → hex colour
    ├── _cellVoltColor()    cell voltage → hex colour
    ├── _tempColor()        general temperature → hex colour
    └── _remCapColor()      remaining capacity → hex colour
```

---

## Troubleshooting

### Card does not appear / shows "Custom element doesn't exist"

- Confirm the resource is registered: **Settings → Dashboards → Resources**. You should see `/hacsfiles/fix-je-energy-card/fix-je-energy-card.js` (HACS) or `/local/fix-je-energy-card.js` (manual) with type `JavaScript Module`.
- Hard refresh the browser: `Ctrl + Shift + R` (Windows/Linux) or `Cmd + Shift + R` (Mac).
- If using the mobile app, clear app cache or force-close and reopen.
- If installed manually, confirm the file is at `/config/www/fix-je-energy-card.js` — not inside a subfolder.

### Visual editor is blank or fails to load

- Open browser DevTools (`F12`) → Console tab. Look for any red errors referencing `fix-je-energy-card`.
- Ensure no other version of `fix-je-energy-card.js` is registered as a duplicate resource. Go to Resources and remove any stale entries.
- Try clearing the HA frontend cache: **Developer Tools → Template** → reload page.

### Entities show `--` or do not update

- Open **Developer Tools → States** and confirm the entity ID exists and has a valid numeric state (not `unavailable` or `unknown`).
- Entity IDs are case-sensitive. Check for typos in the editor.
- If using GoodWe integration, ensure the inverter is online and the integration is polling correctly.
- The card skips `unavailable` and `unknown` states by design — the tile will show `--` until the entity returns a valid value.

### Flow animations not showing

- The card requires `sun.sun` to be present for the sun arc. If it is missing, the sun node will not animate but the rest of the card will function normally.
- Flow path animations are driven by power values. A path will only animate when its corresponding power reading is above zero.
- If all flows are static, check that your inverter entities are returning live values and not stale/unavailable states.

### Battery section is missing

- The Primary Battery section requires `_show_battery: true`. In the visual editor, click the **+ Enable** chip next to **Primary Battery**.
- Secondary Battery, EV, Extra PV Strings, and Custom & Financial Stats sections each have their own **+ Enable** chip — they are hidden by default.

### After update via HACS, card looks wrong or broken

1. Hard refresh the browser (`Ctrl + Shift + R`).
2. If the issue persists, go to **Settings → Dashboards → Resources**, delete the fix-je-energy-card entry, then re-add it (HACS will re-register it on the next HA restart).
3. Restart Home Assistant and hard refresh again.

---

*Last updated: v1.1.3*
