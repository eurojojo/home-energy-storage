# HomeWizard battery modes

You don't control a HomeWizard Plug-In Battery one by one. All batteries behind one P1 Meter (or kWh Meter) form a group and follow the same charging strategy. In Home Assistant that strategy is a select entity on the P1 Meter device, not on the battery.

## Enable the entity

It is disabled by default.

1. Go to Settings → Devices & services → HomeWizard.
2. Open the P1 Meter device.
3. Under *Configuration*, click *+1 disabled entity* (your number may differ).
4. Open *Battery group charging strategy* (in Dutch: *Batterijgroepmodus*), click the gear icon, enable it and save.
5. About 30 seconds later it shows up, for example as `select.p1_meter_battery_group_charging_strategy`. A Dutch installation names it `select.p1_meter_batterijgroepmodus`.

The [Home Assistant HomeWizard documentation](https://www.home-assistant.io/integrations/homewizard/#plug-in-battery) has more on this.

## The modes

| Option | Name in the app | What the battery does |
|---|---|---|
| `zero` | Net zero | charges on surplus and discharges on consumption, so the grid stays at 0 W |
| `zero_charge_only` | Net zero (charge only) | absorbs surplus, never discharges |
| `zero_discharge_only` | Net zero (discharge only) | supplies consumption, never charges |
| `to_full` | One-time full charge | charges to 100 % from the grid, whatever the house does, then switches back to `zero` by itself |
| `standby` | Standby | does nothing |
| `predictive` | Smart charging | HomeWizard decides, based on predicted consumption, solar production, weather and prices |

The modes `zero_charge_only`, `zero_discharge_only` and `predictive` need recent firmware: P1 Meter 6.04 or later, or kWh Meter 5.02 or later. Smart charging also needs the cloud connection and at least seven days of data. Without internet it falls back to net zero.

## Two flavours of Smart charging

The HomeWizard Energy app offers two variants:

- *Smart and grid-friendly* charges during the solar peak and discharges when the neighbourhood uses the most. A good fit for a fixed contract.
- *Smart with dynamic tariffs* charges when power is cheapest, from the grid too, and discharges when it is most expensive.

On a dynamic contract most people will want the second one. The level-3 blueprint *Battery: keep it for the house* leaves Smart charging in control and adds a single rule: don't discharge into the EV or the heat pump.

## Which mode when

This is how the blueprints use the modes:

| Situation | Mode | Why |
|---|---|---|
| the EV charges or the heat pump runs | `zero_charge_only` | moving energy from one store to another only loses 25 % |
| cheap, and grid charging pays | `to_full` | buy now, use later |
| expensive | `zero` | supply the house |
| storing solar pays, but prices aren't high yet | `zero_charge_only` | save the energy for the expensive hours |
| nothing pays, for example under net metering at normal prices | `standby` | no needless cycling |

Before `zero_charge_only` existed, version 0.8 of this project faked it by switching between `standby` and `zero` whenever solar power was exported. You don't need that workaround anymore.
