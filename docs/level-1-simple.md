# Level 1 – simple price control for the battery

Blueprint: [`battery_simple.yaml`](../blueprints/automation/eurojojo/battery_simple.yaml)
[![Import blueprint](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2Feurojojo%2Fhomewizard-dynamic-battery%2Fblob%2Fmaster%2Fblueprints%2Fautomation%2Feurojojo%2Fbattery_simple.yaml)

> First consider HomeWizard's own **Smart charging** (`predictive`, *Smart with dynamic tariffs*). It does this and more without any automation. This blueprint is for people who want to see and tune the logic.

## What you need

- the battery group mode entity of the P1 Meter, enabled ([how](homewizard-battery-modes.md#enable-the-entity))
- the battery state of charge and the P1 power sensor
- three price sensors: current price, today's lowest and today's highest price. The ENTSO-e integration provides all three. With Nord Pool (HACS), make two template sensors from its `min` and `max` attributes:

  ```yaml
  template:
    - sensor:
        - name: Lowest price today
          unit_of_measurement: "€/kWh"
          state: "{{ state_attr('sensor.nordpool_kwh_nl_eur_3_10_021', 'min') }}"
        - name: Highest price today
          unit_of_measurement: "€/kWh"
          state: "{{ state_attr('sensor.nordpool_kwh_nl_eur_3_10_021', 'max') }}"
  ```

## How it decides

Every minute (and whenever the price changes):

```
cheap     = price ≤ lowest + margin                  (default margin € 0.03)
expensive = price ≥ highest − (highest − lowest) ÷ divisor   (default divisor 2.5)
pays      = (highest + markup) × efficiency − (lowest + markup) ≥ minimum profit
```

| # | Situation | Mode |
|---|---|---|
| 1 | import above *heavy load* (EV charging, …) | `zero_charge_only`: never discharge into it; solar may still charge |
| 2 | cheap, grid charging pays, state of charge below the limit | `to_full` |
| 3 | cheap, otherwise | `standby` under net metering, `zero_charge_only` after |
| 4 | expensive | `zero`: the battery supplies the house |
| 5 | in between | `standby` under net metering, `zero` after |

Why the difference before and after net metering? Under net metering exporting is free storage, so putting solar power in the battery only costs the 25 % round-trip loss. After net metering your own solar power is worth more in the battery than on the grid, so the battery simply runs net zero (`zero`), except in the cheap hours (keep the charge for later).

## Options

| Option | Default | Notes |
|---|---|---|
| Markup | € 0.145 | energy tax + surcharge + VAT not in your sensor; only used for "pays" |
| First day without net metering | 2027-01-01 | empty = no net metering |
| Cheap margin | € 0.03 | wider margin = more grid charging |
| Expensive divisor | 2.5 | lower = fewer, only the most expensive hours |
| Round-trip efficiency | 75 % | measured for the HomeWizard Plug-In Battery |
| Minimum profit | € 0.02 | per kWh |
| Grid-charge limit | 95 % | |
| Heavy load | 3500 W | |

## Limits of the simple approach

- Lowest and highest are single slots. With 15-minute prices the battery needs about 3.5 hours (14 slots) to fill, so "lowest + 3 ct" can be too narrow on some days and too wide on others. Level 2 counts the slots it actually needs.
- It only looks at today, not at tomorrow.

## Changes compared with version 0.8

- Uses `zero_charge_only` instead of switching between `standby` and `zero` on solar export (the "PV export hack").
- Heavy load gives `zero_charge_only` instead of `standby`, so solar power can still be stored while the EV charges.
- Knows about the end of net metering.
- Is a blueprint: no YAML editing, every value is an option.
