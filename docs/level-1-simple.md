# Level 1: simple price control for the battery

Blueprint: [`battery_simple.yaml`](../blueprints/automation/eurojojo/battery_simple.yaml)

[![Import blueprint](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2Feurojojo%2Fhome-energy-storage%2Fblob%2Fmaster%2Fblueprints%2Fautomation%2Feurojojo%2Fbattery_simple.yaml)

> Have a look at HomeWizard's own Smart charging first (`predictive`, *Smart with dynamic tariffs*). It does all of this and more, without an automation. This blueprint is for people who want to see the logic and tune it.

## What you need

- the battery group mode entity of the P1 Meter, enabled ([how](homewizard-battery-modes.md#enable-the-entity))
- the battery's state of charge and the P1 power sensor
- three price sensors: the current price, today's lowest and today's highest

The ENTSO-e integration gives you all three prices. With Nord Pool (HACS) you can make the last two from its `min` and `max` attributes:

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

It runs every minute, and whenever the price changes. First it works out three things:

```
cheap     = price ≤ lowest + margin                           (margin € 0.03)
expensive = price ≥ highest − (highest − lowest) ÷ divisor    (divisor 2.5)
pays      = (highest + markup) × efficiency − (lowest + markup) ≥ minimum profit
```

Then the first matching row wins:

| # | Situation | Mode |
|---|---|---|
| 1 | import above the heavy-load limit, e.g. while the EV charges | `zero_charge_only`: no discharging into the car, solar can still charge |
| 2 | cheap, grid charging pays and the battery is below the limit | `to_full` |
| 3 | cheap, but one of the above doesn't hold | `standby` under net metering, `zero_charge_only` after |
| 4 | expensive | `zero`, the battery supplies the house |
| 5 | anything in between | `standby` under net metering, `zero` after |

Why does net metering matter here? While it lasts, exporting is free storage. Putting solar power in the battery then only costs you the 25 % round-trip loss. Once it ends, your own solar power is worth more in the battery than on the grid. So the battery simply runs net zero (`zero`), and in the cheap hours it holds on to its charge for later.

## Options

| Option | Default | Notes |
|---|---|---|
| Markup | € 0.145 | energy tax, surcharge and VAT that your sensor doesn't include. Only used for "pays". |
| First day without net metering | 2027-01-01 | leave empty if you have no net metering |
| Cheap margin | € 0.03 | a wider margin means more grid charging |
| Expensive divisor | 2.5 | lower it to discharge only in the very top hours |
| Round-trip efficiency | 75 % | what I measured on my HomeWizard Plug-In Battery |
| Minimum profit | € 0.02 | per kWh |
| Grid-charge limit | 95 % | |
| Heavy load | 3500 W | |

## Where the simple approach falls short

The lowest and highest price are single slots. With 15-minute prices the battery needs about 3.5 hours, or 14 slots, to fill up. "Lowest plus 3 cents" is too narrow on some days and too wide on others. Level 2 counts the slots it actually needs.

It also only looks at today, never at tomorrow.

## What changed since version 0.8

- `zero_charge_only` replaces the old trick of switching between `standby` and `zero` on solar export.
- A heavy load now gives `zero_charge_only` instead of `standby`, so solar power still goes into the battery while the EV charges.
- It knows when net metering ends.
- It is a blueprint now. You don't have to edit YAML, every value is an option.
