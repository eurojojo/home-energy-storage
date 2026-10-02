# Installation

Requirements: Home Assistant 2024.10 or newer. For the battery blueprints: a HomeWizard P1 Meter (or kWh Meter) with Plug-In Battery and the battery group mode entity enabled ([how](homewizard-battery-modes.md#enable-the-entity)).

## 1. Blueprints

Click an *Import blueprint* button in the [README](../README.md), or go to **Settings → Automations & scenes → Blueprints → Import blueprint** and paste the URL of a file in [`blueprints/automation/eurojojo/`](../blueprints/automation/eurojojo/).

Then click the blueprint, fill in the entities and save. Every option has a description and a sensible default.

## 2. Shared price model (level 2 and 3)

The advanced blueprints share one set of Jinja macros that turns a price list into signals: cheapest and most expensive blocks, percentiles, feed-in value and more.

1. Copy [`custom_templates/home_energy_storage.jinja`](../custom_templates/home_energy_storage.jinja) to `/config/custom_templates/home_energy_storage.jinja`. Create the folder if it doesn't exist. Use the File editor or Studio Code Server add-on, Samba, or SSH.
2. **Developer tools → Actions → `homeassistant.reload_custom_templates`** (or restart).
3. Test it in **Developer tools → Template**:

   ```jinja
   {% from 'home_energy_storage.jinja' import hes_signals %}
   {{ hes_signals('sensor.YOUR_PRICE_SENSOR', 0.145, '2027-01-01') | from_json }}
   ```

   You should see `'ok': True` with the current price, today's quartiles and the cheapest and most expensive blocks. If you see `'ok': False`, your price sensor has no price list the macro understands: see [price sources](price-sources.md).

## 3. Package with helpers (level 3)

The level-3 blueprints use a few helpers: the priority order, "EV must charge", "EV smart charging", a timer for solar boosts of the hot water tank and a number to remember the thermostat target. The package creates them all, plus an optional *Energy storage price now* sensor for dashboards.

1. In `configuration.yaml`:

   ```yaml
   homeassistant:
     packages: !include_dir_named packages
   ```

2. Copy [`packages/home_energy_storage.yaml`](../packages/home_energy_storage.yaml) to `/config/packages/`.
3. Change the price sensor, markup and date in the file (marked `← CHANGE`).
4. **Developer tools → YAML → Check configuration**, then restart.

Prefer the UI? Create the helpers yourself under **Settings → Devices & services → Helpers** with the same names (`input_select.hes_priority` with options like `Battery → Hot water → EV`, `input_boolean.hes_ev_must_charge`, `input_boolean.hes_ev_smart_charging`, `timer.hes_hot_water_boost` of 1 hour, `input_number.hes_heating_base` from −1 to 35 with initial value −1). Or pick any helpers of your own in the blueprints.

## 4. Updating

Blueprints: **Settings → Automations & scenes → Blueprints → ⋮ → Re-import blueprint**. Your automations keep their settings. Copy the newer `home_energy_storage.jinja` again and reload custom templates. See the [changelog](../CHANGELOG.md) for changes that need your attention.

## 5. Old version (0.8)

The automations of version 0.8 (YAML to paste, ENTSO-e only, hourly prices) are still available under the tag [`v0.8.0`](https://github.com/eurojojo/homewizard-dynamic-battery/tree/v0.8.0). The level-1 and level-2 blueprints replace them. Disable the old automation before you enable a new one: two automations controlling the same battery fight each other.
