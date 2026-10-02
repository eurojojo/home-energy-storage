# Changelog

All notable changes to this project. The format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/), versions follow [semantic versioning](https://semver.org/).

## [1.0.0] – 2026-10

The project moves from "a smarter HomeWizard battery" to **home energy storage after net metering**: storing your own power in the hot water tank, the house, the EV and the battery.

### Added
- Level 1 blueprint `battery_simple.yaml`: the simple min/max approach of 0.8 as a configurable blueprint, using the new firmware modes.
- Level 2 blueprint `battery_advanced.yaml`: counts the cheapest and most expensive slots the battery actually needs, works with 15-minute prices and knows the value of solar power before and after net metering.
- Level 3 blueprints: `battery_guard.yaml`, `hot_water.yaml`, `heating.yaml`, `ev_charging.yaml`, `curtail_export.yaml`.
- Shared price model `custom_templates/home_energy_storage.jinja` (ENTSO-e, Nord Pool and adapter sensors; 15-minute and hourly prices).
- Package `packages/home_energy_storage.yaml` with helpers and a signals sensor.
- Documentation: after net metering, HomeWizard modes, installation, price sources, a page per level.
- Recommendation to use HomeWizard's own Smart charging (`predictive`) for the battery alone.
- Dutch README, contributing guide, issue template, yamllint workflow.

### Changed
- "Charge only" is now the firmware mode `zero_charge_only` instead of switching between `standby` and `zero` on solar export.
- Heavy loads (EV) give `zero_charge_only` instead of `standby`, so solar power can still be stored.
- The markup is configurable (was a hard-coded € 0.14).

### Removed
- The YAML automations and template sensors in `automations/` (still available under tag `v0.8.0`). Disable them before you use the new blueprints.

## [0.8.0] – 2025-11-29

- Simple and advanced (Q1/Q3 percentile) automations for the HomeWizard Plug-In Battery with ENTSO-e prices, including the "PV export hack" for charge-only behaviour.

[1.0.0]: https://github.com/eurojojo/homewizard-dynamic-battery/compare/v0.8.0...v1.0.0
[0.8.0]: https://github.com/eurojojo/homewizard-dynamic-battery/releases/tag/v0.8.0
