# Changelog

Notable changes per version. The format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/) and the version numbers follow [semantic versioning](https://semver.org/).

## [1.0.0] - 2026-10

The project was called *homewizard-dynamic-battery* and is now *home-energy-storage*. It used to be about a smarter HomeWizard battery. Now it is about storing your own power once net metering ends, in the hot water tank, the house, the EV and the battery. Old links to the repository keep working, GitHub redirects them.

### Added
- Level 1 blueprint `battery_simple.yaml`. The simple min/max approach of 0.8 as a blueprint you configure in the UI, using the new firmware modes.
- Level 2 blueprint `battery_advanced.yaml`. It counts the cheapest and most expensive slots the battery really needs, works with 15-minute prices, and knows what solar power is worth before and after net metering.
- Level 3 blueprints `battery_guard.yaml`, `hot_water.yaml`, `heating.yaml`, `ev_charging.yaml` and `curtail_export.yaml`.
- A shared price model, `custom_templates/home_energy_storage.jinja`. It reads ENTSO-e, Nord Pool and adapter sensors, with 15-minute or hourly prices.
- The package `packages/home_energy_storage.yaml` with helpers and a signals sensor.
- Documentation on net metering, the HomeWizard modes, installation and price sources, plus a page per level.
- Advice to use HomeWizard's own Smart charging (`predictive`) if you only have the battery.
- A Dutch README, a contributing guide, an issue template and a yamllint workflow.

### Changed
- "Charge only" now uses the firmware mode `zero_charge_only`. The old trick of switching between `standby` and `zero` on solar export is gone.
- A heavy load such as the EV gives `zero_charge_only` instead of `standby`, so solar power can still go into the battery.
- The markup is an option now. It used to be a fixed € 0.14.

### Removed
- The YAML automations and template sensors in `automations/`. You can still find them under the tag `v0.8.0`. Disable them before you start using the new blueprints.

## [0.8.0] - 2025-11-29

- Simple and advanced (Q1/Q3 percentile) automations for the HomeWizard Plug-In Battery with ENTSO-e prices, including the "PV export hack" that made the battery charge only.

[1.0.0]: https://github.com/eurojojo/home-energy-storage/compare/v0.8.0...v1.0.0
[0.8.0]: https://github.com/eurojojo/home-energy-storage/releases/tag/v0.8.0
