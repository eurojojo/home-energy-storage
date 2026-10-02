# Installation

You need Home Assistant 2024.10 or newer. For the battery blueprints you also need a HomeWizard P1 Meter (or kWh Meter) with a Plug-In Battery, and the battery group mode entity has to be enabled ([how](homewizard-battery-modes.md#enable-the-entity)).

## 1. Blueprints

Click one of the *Import blueprint* buttons in the [README](../README.md). You can also go to Settings → Automations & scenes → Blueprints → Import blueprint and paste the URL of a file in [`blueprints/automation/eurojojo/`](../blueprints/automation/eurojojo/).

Then open the blueprint, fill in your entities and save. Each option has a short description and a default that should work for most homes.

## 2. Shared price model (level 2 and 3)

The advanced blueprints share a set of Jinja macros. They turn a price list into the signals the blueprints work with, such as the cheapest and most expensive block of the day, quartiles and the feed-in value.

1. Copy [`custom_templates/home_energy_storage.jinja`](../custom_templates/home_energy_storage.jinja) to `/config/custom_templates/home_energy_storage.jinja`. Create the folder if it isn't there yet. The File editor or Studio Code Server add-on works, and so do Samba and SSH.
2. In Developer tools → Actions, run `homeassistant.reload_custom_templates`, or restart Home Assistant.
3. Test it in Developer tools → Template:

   ```jinja
   {% from 'home_energy_storage.jinja' import hes_signals %}
   {{ hes_signals('sensor.YOUR_PRICE_SENSOR', 0.145, '2027-01-01') | from_json }}
   ```

   With `'ok': True` you'll see the current price, today's quartiles and the cheapest and most expensive blocks. `'ok': False` means the macro can't read a price list from your sensor. [Price sources](price-sources.md) explains what to do then.

## 3. Package with helpers (level 3)

The level-3 blueprints use a handful of helpers: the priority order, "EV must charge", "EV smart charging", a timer for solar boosts of the hot water tank, and a number that remembers the thermostat target. The package creates them. It also adds an optional *Energy storage price now* sensor for dashboards.

1. Add this to `configuration.yaml`:

   ```yaml
   homeassistant:
     packages: !include_dir_named packages
   ```

2. Copy [`packages/home_energy_storage.yaml`](../packages/home_energy_storage.yaml) to `/config/packages/`.
3. In that file, change the price sensor, the markup and the date. They are marked with `← CHANGE`.
4. Check the configuration in Developer tools → YAML, then restart.

If you'd rather use the UI, create the helpers under Settings → Devices & services → Helpers:

- `input_select.hes_priority`, with options such as `Battery → Hot water → EV`
- `input_boolean.hes_ev_must_charge` and `input_boolean.hes_ev_smart_charging`
- `timer.hes_hot_water_boost`, 1 hour
- `input_number.hes_heating_base`, from −1 to 35, initial value −1

You can also pick helpers of your own in the blueprints.

## 4. Updating

For a blueprint, go to Settings → Automations & scenes → Blueprints, open the ⋮ menu and choose *Re-import blueprint*. Your automations keep their settings. Copy the newer `home_energy_storage.jinja` as well and reload custom templates. The [changelog](../CHANGELOG.md) mentions any change that needs your attention.

## 5. Coming from version 0.8

The automations of version 0.8 (YAML to paste, ENTSO-e only, hourly prices) are kept under the tag [`v0.8.0`](https://github.com/eurojojo/home-energy-storage/tree/v0.8.0). The level-1 and level-2 blueprints replace them. Disable the old automation before you enable a new one, or the two will fight over the same battery.
