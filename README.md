# Home Energy Storage for Home Assistant

**Store your own solar power — and cheap grid power — in the devices you already have: the hot water tank, the house itself, the EV and a (plug-in) home battery.**

[![Release](https://img.shields.io/github/v/release/eurojojo/homewizard-dynamic-battery)](https://github.com/eurojojo/homewizard-dynamic-battery/releases)
[![Lint](https://github.com/eurojojo/homewizard-dynamic-battery/actions/workflows/lint.yaml/badge.svg)](https://github.com/eurojojo/homewizard-dynamic-battery/actions/workflows/lint.yaml)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Home Assistant](https://img.shields.io/badge/Home%20Assistant-2024.10%2B-41BDF5?logo=home-assistant)](https://www.home-assistant.io/)

🇳🇱 [Nederlandse versie](README.nl.md)

---

## Why: net metering ends

In the Netherlands net metering (*salderen*) ends on **1 January 2027**. Until then the grid is a free battery: every kWh you export is subtracted from a kWh you import. After that date an exported kWh earns only a feed-in compensation (on a dynamic contract roughly the bare market price, sometimes negative, often minus a feed-in fee), while an imported kWh still costs the full price including energy tax.

The gap is typically **€ 0.15–0.30 per kWh**. Every kWh of your own solar power that you *use* instead of export is worth that gap. The same goes for grid power you buy in the cheap hours instead of the expensive ones.

You probably already own a lot of storage:

| Store | Typical size | Losses | Good for |
|---|---|---|---|
| **EV** | 40–80 kWh | ~10 % | the biggest one by far, when it is at home |
| **Hot water tank** (heat pump or electric) | 1–2 kWh of electricity per boost (3–6 kWh of heat) | small; a heat pump also needs less power at a lower tank temperature | every day, all year |
| **The house itself** (heat pump) | a few kWh of heat per °C | a little extra heat loss | heating season |
| **Home battery** (HomeWizard Plug-In) | 2.7 kWh each, several can be combined | ~25 % round trip | evening and night |

This project shows how to use them with Home Assistant, step by step.

---

## Choose your level

### 0. Just a HomeWizard battery? Use HomeWizard's own Smart charging

Recent HomeWizard firmware (P1 Meter ≥ 6.04 or kWh Meter ≥ 5.02) has **Smart charging** (`predictive`). It predicts your consumption, solar production and the prices, and charges and discharges at the right moments. Choose *Smart with dynamic tariffs* in the HomeWizard Energy app if you have a dynamic contract, and it will also charge from the grid when power is cheap. You don't need solar panels for that.

👉 **For most people this is the best option for the battery alone.** No Home Assistant automation needed.
[HomeWizard Plug-In Battery](https://www.homewizard.com/plug-in-battery/) · [How Smart charging decides](https://helpdesk.homewizard.com/en/articles/14209959-how-does-the-battery-determine-when-to-charge-and-discharge)

The levels below are for people who want to see and tune the logic themselves, or who want to combine the battery with the other stores.

### 1. Simple: one blueprint for the battery

Cheap and expensive hours from today's lowest and highest price, a profitability check, and the new firmware modes (`zero`, `zero_charge_only`, `to_full`, `standby`). Easy to read and adjust. → [docs/level-1-simple.md](docs/level-1-simple.md)

[![Import blueprint](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2Feurojojo%2Fhomewizard-dynamic-battery%2Fblob%2Fmaster%2Fblueprints%2Fautomation%2Feurojojo%2Fbattery_simple.yaml)

### 2. Advanced: the whole price curve

Uses all 15-minute (or hourly) prices of today and tomorrow, the state of charge and the value of your own solar power before and after net metering. It charges in exactly the cheapest slots needed to fill the battery, and discharges in the most expensive slots it can cover. → [docs/level-2-advanced.md](docs/level-2-advanced.md)

[![Import blueprint](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2Feurojojo%2Fhomewizard-dynamic-battery%2Fblob%2Fmaster%2Fblueprints%2Fautomation%2Feurojojo%2Fbattery_advanced.yaml)

### 3. Home energy storage: battery, hot water, heating, EV

A set of blueprints that share one price model and one priority order. Each one can be used on its own. → [docs/level-3-home-energy-storage.md](docs/level-3-home-energy-storage.md)

| Blueprint | What it does | Import |
|---|---|---|
| Battery: keep it for the house | Smart charging stays in charge, but the battery never discharges into the EV or heat pump | [![Import](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2Feurojojo%2Fhomewizard-dynamic-battery%2Fblob%2Fmaster%2Fblueprints%2Fautomation%2Feurojojo%2Fbattery_guard.yaml) |
| Hot water tank as a heat battery | boost on surplus or very cheap power, comfort in the cheapest block, eco in the most expensive block | [![Import](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2Feurojojo%2Fhomewizard-dynamic-battery%2Fblob%2Fmaster%2Fblueprints%2Fautomation%2Feurojojo%2Fhot_water.yaml) |
| House as a heat battery | pre-heat before and save during the most expensive hours | [![Import](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2Feurojojo%2Fhomewizard-dynamic-battery%2Fblob%2Fmaster%2Fblueprints%2Fautomation%2Feurojojo%2Fheating.yaml) |
| EV: surplus and cheapest hours | charge on solar surplus, in the cheapest hours before you leave, or now | [![Import](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2Feurojojo%2Fhomewizard-dynamic-battery%2Fblob%2Fmaster%2Fblueprints%2Fautomation%2Feurojojo%2Fev_charging.yaml) |
| Stop exporting when it costs money | curtail the inverter at a negative feed-in value | [![Import](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2Feurojojo%2Fhomewizard-dynamic-battery%2Fblob%2Fmaster%2Fblueprints%2Fautomation%2Feurojojo%2Fcurtail_export.yaml) |

---

## Installation

Short version (details in [docs/installation.md](docs/installation.md)):

1. **Level 1:** import the blueprint and create an automation from it. Done.
2. **Level 2 and 3:** also copy [`custom_templates/home_energy_storage.jinja`](custom_templates/home_energy_storage.jinja) to `/config/custom_templates/` and run the action `homeassistant.reload_custom_templates`.
3. **Level 3:** also add the package [`packages/home_energy_storage.yaml`](packages/home_energy_storage.yaml) (priority, "EV must charge", boost timer), or create those helpers in the UI.
4. You need a price sensor with a price list: ENTSO-e or Nord Pool work directly, other sources via a small adapter, see [docs/price-sources.md](docs/price-sources.md).
5. For the HomeWizard battery: enable the battery group mode entity of the P1 Meter, see [docs/homewizard-battery-modes.md](docs/homewizard-battery-modes.md).

## Documentation

- [After net metering: what is a kWh worth?](docs/after-net-metering.md)
- [HomeWizard battery modes](docs/homewizard-battery-modes.md)
- [Installation](docs/installation.md)
- [Price sources](docs/price-sources.md)
- [Level 1 – simple](docs/level-1-simple.md) · [Level 2 – advanced](docs/level-2-advanced.md) · [Level 3 – home energy storage](docs/level-3-home-energy-storage.md)

## Repository layout

```
blueprints/automation/eurojojo/   the blueprints (import these)
custom_templates/                 shared price model (Jinja macros) for level 2 and 3
packages/                         helpers for level 3 and an optional signals sensor
docs/                             explanation, decision trees, installation
```

## Background

This started in 2025 as a pair of automations that worked around the missing "charge only" mode of the HomeWizard Plug-In Battery (see [v0.8](https://github.com/eurojojo/homewizard-dynamic-battery/tree/v0.8.0)). HomeWizard has since added that mode and its own Smart charging, so the focus has moved to what really matters after net metering: storing your own power, in every device that can.

Level 3 is a generic version of what runs in the author's own home (heat pump with hot water tank, EV, HomeWizard battery, solar panels, dynamic contract).

## Contributing

Questions, ideas and pull requests are welcome, see [CONTRIBUTING.md](CONTRIBUTING.md). Changes are listed in [CHANGELOG.md](CHANGELOG.md).

## License

[MIT](LICENSE) © Joost Smits
