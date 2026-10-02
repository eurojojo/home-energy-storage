# Home Energy Storage for Home Assistant

Store your own solar power, and cheap grid power, in devices you already have. A hot water tank, a heat pump heated house and an EV can hold far more energy than a plug-in battery. This project shows how to use them with Home Assistant blueprints.

[![Release](https://img.shields.io/github/v/release/eurojojo/home-energy-storage)](https://github.com/eurojojo/home-energy-storage/releases)
[![Lint](https://github.com/eurojojo/home-energy-storage/actions/workflows/lint.yaml/badge.svg)](https://github.com/eurojojo/home-energy-storage/actions/workflows/lint.yaml)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Home Assistant](https://img.shields.io/badge/Home%20Assistant-2024.10%2B-41BDF5?logo=home-assistant)](https://www.home-assistant.io/)

🇳🇱 [Nederlandse versie](README.nl.md)

## Why bother: net metering ends

In the Netherlands net metering (*salderen*) stops on 1 January 2027. Until then the grid acts as a free battery. Every kWh you export cancels out a kWh you import.

After that date an exported kWh only earns a feed-in compensation. On a dynamic contract that is roughly the bare market price, minus a feed-in fee, and sometimes it is negative. An imported kWh still costs the full price including energy tax. The difference is usually somewhere between € 0.15 and € 0.30 per kWh, and you gain it on every kWh of your own solar power you use instead of export. Buying grid power in the cheap hours instead of the expensive ones works the same way.

Most homes with solar panels already have more storage than they think:

| Store | Typical size | Losses | When it helps |
|---|---|---|---|
| EV | 40–80 kWh | about 10 % | whenever it is at home |
| Hot water tank (heat pump or electric) | 1–2 kWh of electricity per boost, 3–6 kWh of heat | small, and a heat pump needs less power at a lower tank temperature | every day of the year |
| The house itself (heat pump) | a few kWh of heat per °C | a little extra heat loss | in the heating season |
| HomeWizard Plug-In Battery | 2.7 kWh each, you can combine several | about 25 % round trip | evening and night |

## Pick a level

### Only a HomeWizard battery? Start with HomeWizard's Smart charging

Recent HomeWizard firmware (P1 Meter 6.04 or later, kWh Meter 5.02 or later) has a mode called Smart charging (`predictive`). It predicts your consumption, your solar production and the prices, and plans charging and discharging around them. With a dynamic contract, choose *Smart with dynamic tariffs* in the HomeWizard Energy app. The battery will then also charge from the grid when power is cheap, so you don't need solar panels.

For the battery on its own, this is what I recommend to most people. You don't need a Home Assistant automation for it.
See [HomeWizard Plug-In Battery](https://www.homewizard.com/plug-in-battery/) and [how Smart charging decides](https://helpdesk.homewizard.com/en/articles/14209959-how-does-the-battery-determine-when-to-charge-and-discharge).

The levels below are for people who want to see and tune the logic themselves, or who want the battery to work together with other storage.

### Level 1: one simple blueprint for the battery

It finds cheap and expensive hours from today's lowest and highest price, checks whether charging from the grid pays, and uses the firmware modes `zero`, `zero_charge_only`, `to_full` and `standby`. Easy to read and to adjust. See [docs/level-1-simple.md](docs/level-1-simple.md).

[![Import blueprint](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2Feurojojo%2Fhome-energy-storage%2Fblob%2Fmaster%2Fblueprints%2Fautomation%2Feurojojo%2Fbattery_simple.yaml)

### Level 2: the whole price curve

This one looks at every 15-minute (or hourly) price of today and tomorrow. It counts how many slots the battery needs to fill up and charges in exactly the cheapest ones. It discharges in the most expensive slots a full battery can cover. It also knows that your own solar power is worth less in the battery than on the grid while net metering lasts, and more afterwards. See [docs/level-2-advanced.md](docs/level-2-advanced.md).

[![Import blueprint](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2Feurojojo%2Fhome-energy-storage%2Fblob%2Fmaster%2Fblueprints%2Fautomation%2Feurojojo%2Fbattery_advanced.yaml)

### Level 3: battery, hot water, heating and EV together

A set of blueprints that share one price model and one priority order. You can use each of them on its own. See [docs/level-3-home-energy-storage.md](docs/level-3-home-energy-storage.md).

| Blueprint | What it does | Import |
|---|---|---|
| Battery: keep it for the house | Smart charging stays in control, but the battery won't discharge into the EV or the heat pump | [![Import](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2Feurojojo%2Fhome-energy-storage%2Fblob%2Fmaster%2Fblueprints%2Fautomation%2Feurojojo%2Fbattery_guard.yaml) |
| Hot water tank as a heat battery | boosts on surplus or very cheap power, a comfort temperature in the cheapest block, eco in the most expensive block | [![Import](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2Feurojojo%2Fhome-energy-storage%2Fblob%2Fmaster%2Fblueprints%2Fautomation%2Feurojojo%2Fhot_water.yaml) |
| House as a heat battery | a bit warmer before the most expensive hours, a bit cooler during them | [![Import](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2Feurojojo%2Fhome-energy-storage%2Fblob%2Fmaster%2Fblueprints%2Fautomation%2Feurojojo%2Fheating.yaml) |
| EV: surplus and cheapest hours | charges on solar surplus, in the cheapest hours before you leave, or right now | [![Import](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2Feurojojo%2Fhome-energy-storage%2Fblob%2Fmaster%2Fblueprints%2Fautomation%2Feurojojo%2Fev_charging.yaml) |
| Stop exporting when it costs money | limits the inverter when the feed-in value is negative | [![Import](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2Feurojojo%2Fhome-energy-storage%2Fblob%2Fmaster%2Fblueprints%2Fautomation%2Feurojojo%2Fcurtail_export.yaml) |

## Installation

The short version. The details are in [docs/installation.md](docs/installation.md).

1. Level 1: import the blueprint and create an automation from it. That's it.
2. Level 2 and 3 also need the shared price model. Copy [`custom_templates/home_energy_storage.jinja`](custom_templates/home_energy_storage.jinja) to `/config/custom_templates/` and run the action `homeassistant.reload_custom_templates`.
3. Level 3 uses a few helpers (priority, "EV must charge", a boost timer). The package [`packages/home_energy_storage.yaml`](packages/home_energy_storage.yaml) creates them, or you make them yourself in the UI.
4. You need a price sensor with a list of prices. ENTSO-e and Nord Pool (HACS) work as they are. For other sources there is a small adapter in [docs/price-sources.md](docs/price-sources.md).
5. For the HomeWizard battery, first enable the battery group mode entity of the P1 Meter. [docs/homewizard-battery-modes.md](docs/homewizard-battery-modes.md) shows how.

## Documentation

- [After net metering: what is a kWh worth?](docs/after-net-metering.md)
- [HomeWizard battery modes](docs/homewizard-battery-modes.md)
- [Installation](docs/installation.md)
- [Price sources](docs/price-sources.md)
- [Level 1](docs/level-1-simple.md), [level 2](docs/level-2-advanced.md) and [level 3](docs/level-3-home-energy-storage.md)

## Repository layout

```
blueprints/automation/eurojojo/   the blueprints (import these)
custom_templates/                 shared price model (Jinja macros) for level 2 and 3
packages/                         helpers for level 3 and an optional signals sensor
docs/                             background, decision tables, installation
```

## Background

I started this in 2025 under the name *homewizard-dynamic-battery*. It was a pair of automations that worked around a missing "charge only" mode in the HomeWizard Plug-In Battery. That version is still available as [v0.8](https://github.com/eurojojo/home-energy-storage/tree/v0.8.0). HomeWizard has since added that mode and its own Smart charging. So the project now focuses on what matters more once net metering ends: storing your own power in every device that can hold it.

Level 3 is a generic version of what runs in my own home: a Remeha heat pump with a hot water tank, a VW ID.3 on an Alfen charger, a HomeWizard battery, solar panels on a Growatt inverter and a dynamic contract with NextEnergy. Your devices will differ. The blueprints only need the standard Home Assistant entities (`water_heater`, `climate`, `switch`, `select`, power sensors), so you can swap in your own.

## Contributing

Questions, ideas and pull requests are welcome, see [CONTRIBUTING.md](CONTRIBUTING.md). Changes are listed in [CHANGELOG.md](CHANGELOG.md).

## License

[MIT](LICENSE) © Joost Smits
