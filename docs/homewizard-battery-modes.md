# HomeWizard battery modes

The HomeWizard Plug-In Battery is controlled per *group*: all batteries behind one P1 Meter (or kWh Meter) follow the same charging strategy. In Home Assistant this is a select entity on the **P1 Meter** device, not on the battery.

## Enable the entity

The entity is disabled by default.

1. **Settings → Devices & services → HomeWizard**.
2. Open the **P1 Meter** device.
3. Under *Configuration*, click *+1 disabled entity* (the number may differ).
4. Open *Battery group charging strategy* (Dutch: *Batterijgroepmodus*), click the gear icon, enable it and save.
5. After about 30 seconds it appears, for example as `select.p1_meter_battery_group_charging_strategy` (Dutch installations: `select.p1_meter_batterijgroepmodus`).

See also the [Home Assistant HomeWizard documentation](https://www.home-assistant.io/integrations/homewizard/#plug-in-battery).

## The modes

| Option | In the app | What the battery does |
|---|---|---|
| `zero` | Net zero | charges on surplus, discharges on consumption: keeps the grid power at 0 W |
| `zero_charge_only` | Net zero (charge only) | only absorbs surplus, never discharges |
| `zero_discharge_only` | Net zero (discharge only) | only supplies consumption, never charges |
| `to_full` | One-time full charge | charges to 100 % from the grid, whatever the house does; then switches back to `zero` by itself |
| `standby` | Standby | does nothing |
| `predictive` | Smart charging | HomeWizard decides itself, based on predicted consumption, solar production, weather and prices |

`zero_charge_only`, `zero_discharge_only` and `predictive` need recent firmware (P1 Meter ≥ 6.04 or kWh Meter ≥ 5.02). `predictive` also needs the cloud connection and at least seven days of data. Without internet it falls back to net zero.

## Smart charging: two variants

In the HomeWizard Energy app you can choose:

- **Smart and grid-friendly**: charges during the solar peak and discharges when the neighbourhood uses most. Good for fixed contracts.
- **Smart with dynamic tariffs**: charges when power is cheapest, also from the grid, and discharges when it is most expensive.

With a dynamic contract, *Smart with dynamic tariffs* is what most people want. The level-3 blueprint *Battery: keep it for the house* keeps Smart charging in charge and only adds one rule: no discharging into the EV or heat pump.

## Which mode when (as used in the blueprints)

| Situation | Mode | Why |
|---|---|---|
| EV charges or heat pump runs | `zero_charge_only` | don't move energy from one store to another (and lose 25 %) |
| cheap, and grid charging pays | `to_full` | buy now, use later |
| expensive | `zero` | supply the house |
| storing solar pays, but not expensive yet | `zero_charge_only` | keep the energy for the expensive hours |
| nothing pays (e.g. under net metering in normal hours) | `standby` | avoid needless cycling |

Before the `zero_charge_only` mode existed, version 0.8 of this project simulated it by switching between `standby` and `zero` on solar export. That workaround is no longer needed.
