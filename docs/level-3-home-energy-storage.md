# Level 3: home energy storage

Use every device in the house that can hold energy. The hot water tank, the building itself, the EV and the home battery each get their own blueprint, and all of them share one price model and one priority order.

After net metering, this is where the money is. A HomeWizard battery stores 2.7 kWh and loses a quarter of it on the way. A hot water tank takes the heat of 1 to 2 kWh of electricity every day and loses very little. A house with floor heating can shift several kWh of heating by a few hours, and an EV soaks up 10 to 20 kWh on one sunny afternoon. The numbers behind this are in [after net metering](after-net-metering.md).

## My setup, and how yours may differ

I built this for my own home, so that is the example throughout this page:

| Role | What I have | Integration | Other common options |
|---|---|---|---|
| P1 meter and battery | HomeWizard P1 Meter, Plug-In Battery 2.7 kWh | HomeWizard | Zendure, Marstek, a big hybrid battery (control differs, the battery guard needs a HomeWizard) |
| Solar | Growatt MIN 4200TL-XE inverter | growatt_local | SolarEdge, Enphase, SMA, Huawei: anything with a power sensor, plus an on/off switch or export limit for curtailment |
| Hot water and heating | Remeha heat pump with hot water tank, eTwist thermostat | remeha_home | Daikin, Mitsubishi, Panasonic, Vaillant, an electric boiler: any `water_heater` and `climate` entity |
| EV | VW ID.3 (58 kWh) on an Alfen Eve charger | 50five, Alfen | Zaptec, Easee, Wallbox, go-e, a smart plug for a granny charger: anything with an on/off switch |
| Prices | NextEnergy dynamic contract, Nord Pool 15-minute prices and ENTSO-e | nordpool, entsoe | Tibber, Frank Energie, Zonneplan, ANWB: see [price sources](price-sources.md) |

My settings: priority Battery → Hot water → EV, hot water at 50, 55, 65 and 45 °C, heating ±0.5 °C, and the battery guard watching the heat pump and the charger phase.

## Overview

| Blueprint | Device | Stores | Needs |
|---|---|---|---|
| [Battery: keep it for the house](../blueprints/automation/eurojojo/battery_guard.yaml) | HomeWizard Plug-In Battery | electricity | battery mode entity, binary sensors for "flexible load running" |
| [Hot water tank as a heat battery](../blueprints/automation/eurojojo/hot_water.yaml) | `water_heater`, heat pump or electric | heat | price sensor, P1 power, boost timer |
| [House as a heat battery](../blueprints/automation/eurojojo/heating.yaml) | `climate`, a heat pump thermostat | heat in the building | price sensor, number helper |
| [EV: surplus and cheapest hours](../blueprints/automation/eurojojo/ev_charging.yaml) | a charger with an on/off `switch` | electricity | price sensor, P1 power, two helpers |
| [Stop exporting when it costs money](../blueprints/automation/eurojojo/curtail_export.yaml) | inverter `switch` or limit `number` | nothing, it curtails | price sensor |

Install the [shared price model and the package](installation.md) first. Then add one blueprint at a time and keep an eye on the logbook for a few days before you add the next. I'd go in this order: battery guard, hot water, heating, EV, curtailment.

## Priority: who gets the surplus?

Solar surplus physically flows to whatever device takes it first. The HomeWizard battery reacts within seconds, so it usually wins. The priority helper `input_select.hes_priority` from the package tells the other blueprints how to count.

Say hot water comes before the battery. Then the power the battery is charging with counts as surplus for the tank too. The tank starts its boost, the heat pump takes the power and the battery gets less.

If the battery comes first, only real export counts. The tank also skips its "very cheap" boost while the battery is still filling from the sun. The EV follows the same rules.

"EV must charge" (`input_boolean.hes_ev_must_charge`) always puts the EV first. It charges, and the hot water tank leaves the surplus alone.

Which order is best depends on your home:

| Store | Strong point | Weak point |
|---|---|---|
| Battery | covers exactly the expensive evening hours | small, 25 % loss |
| Hot water | cheap heat, every day | full after one or two boosts |
| EV | huge | only helps when it is home and needs charge |

I use Battery → Hot water → EV. On a sunny day the battery is full by early afternoon, the tank gets its boost after that, and the car gets what is left. When I need the car full, I switch on "EV must charge".

## Battery: keep it for the house

Smart charging (`predictive`) stays in control, with one extra rule. While the EV charges or the heat pump runs, the battery doesn't discharge (`zero_charge_only`). Emptying it into another store only wastes energy.

There's one exception. If the rest of the house also draws a lot at the same time, say the oven or the induction hob at more than 1500 W besides the flexible loads, the battery may supply. Its 800 W then goes to that load and not into the car.

| Condition | Mode |
|---|---|
| a flexible load runs and the rest of the house stays at or below the exception | `zero_charge_only` |
| otherwise | the normal mode, `predictive` or `zero` |

It only switches between those two modes. If you set `to_full` or `standby` by hand, it leaves the battery alone.

For the binary sensors: many heat pump integrations have a "compressor running" or "heating" state. My Remeha shows it in the hot water status and the zone status. For the EV you can use the charger status, or a threshold helper on the charger power (Settings → Helpers → Threshold, above 1000 W for example). My charger is the only load on phase 3, so I use the P1 power of that phase.

## Hot water tank as a heat battery

| # | Condition | Target |
|---|---|---|
| 1 | the solar boost timer is running | boost, 65 °C |
| 2 | very cheap: a negative price, or among the cheapest 10 % of today and at most 75 % of the median. Skipped while the battery has priority and is still filling. | boost |
| 3 | in today's most expensive block of 3 hours, if that block is at least 15 % above the median | eco, 45 °C |
| 4 | in today's cheapest block of 2 hours | comfort, 55 °C |
| 5 | otherwise | normal, 50 °C |

The solar boost starts after net metering ends. A surplus of at least 1000 W for 10 minutes starts the boost timer, 1 hour by default. Before 2027 it doesn't, because exporting is still free storage.

Why those temperatures? A heat pump heats water more efficiently at a low temperature, so the normal setpoint stays low. Extra heat goes in only when it is cheap or free, and during the expensive hours the tank may cool a little. Pick temperatures that suit your tank and your household. Many heat pumps reach 55 °C with the compressor and need an electric element above that, which has a COP of 1. In that case a boost to 65 °C only pays with solar surplus or a very low price.

About legionella: keep the disinfection program of your device switched on, unless you really know what you are doing. The Dutch guideline ISSO 55.1 asks for 60 °C for at least 20 minutes, for the whole tank, on a regular basis. A boost to 65 °C often counts, so you could schedule the device's own program right after your usual boost time. In my own home Home Assistant plans the weekly disinfection in the cheapest block and raises an alarm if it gets missed. I left that out of these blueprints on purpose, because a mistake there is a health risk.

Some devices only accept a new temperature in a particular operation mode. My Remeha needs `performance`. Fill that in under *Operation mode to set first*.

## House as a heat battery

| Phase | When | Target |
|---|---|---|
| pre-heat | the 2 hours before today's most expensive block, if those hours cost at most 85 % of the block | original + 0.5 °C |
| save | during the most expensive block of 3 hours, if it is at least 15 % above the median | original − 0.5 °C |
| normal | the rest of the time | the original target again |

This only happens in the heating season (below 14 °C outside) and in the HVAC modes you pick. The original target goes into `input_number.hes_heating_base`. If you, or the thermostat's own schedule, change the target in the middle of a phase, the blueprint backs off until the next phase and leaves your new target alone.

It works best with floor heating and a well-insulated house. Start with 0.5 °C and see if anyone in the house notices. My Remeha eTwist accepts the change as a temporary override of its clock program, so the thermostat falls back to its own schedule even if Home Assistant goes down.

## EV: surplus and cheapest hours

It only acts while `input_boolean.hes_ev_smart_charging` is on.

| # | Condition | Charger |
|---|---|---|
| 1 | EV must charge | on |
| 2 | in the cheapest *N* hours of the 24 hours before "ready by", by default 4 hours before 07:00 | on |
| 3 | a surplus of at least 1500 W for 5 minutes | on |
| 4 | an import of at least 500 W for 10 minutes | off |
| 5 | an import of at least 1500 W outside rows 1 and 2, so clearly charging from the grid | off |

⚠️ Test pausing and resuming with your own charger and car first. Do it during the day, while you are at home. Some combinations really don't like it. The car shows a charging error, or the session won't continue until you plug the cable in again. That happened to me with the ID.3 and the Alfen, which is why my own setup no longer pauses sessions remotely. If yours behaves the same, use only "EV must charge" and the charger's own schedule.

One phase at 6 A is about 1400 W, three phases about 4100 W. Below that, the car can't run on surplus alone. Chargers that let you set the current (as a `number` entity) can follow the surplus more closely. This blueprint doesn't do that yet, because every charger does it differently.

## Stop exporting when it costs money

When the feed-in value drops below the threshold (€ 0 by default), the inverter is switched off or its limit goes to the "curtailed" value. When the value rises again, it is restored. During net metering the feed-in value is the all-in price. After that it is the market price × factor − fee, so from 2027 on it reacts to every negative market price.

If your inverter has a power or export limit as a `number`, use that rather than switching it off. The house can then still use the solar power, and only the export stops. My Growatt has both, an on/off switch and an output limit.

## What is not included

Solar and consumption forecasts. In my own home a small Python planner uses Solcast forecasts and a household profile, so in the morning it already knows the battery will fill up from the sun anyway. These blueprints react to the *measured* surplus and the price curve. That is simpler, and for these loads it gets you most of the way.

Grid charging of the home battery. Smart charging takes care of that. If you'd rather use your own logic, take the [level-2 blueprint](level-2-advanced.md) instead of the battery guard. It has the same "flexible loads" option. Don't run both at once.

Washing machines and dishwashers. Their own timer works fine, or build a small automation on `cheapest_block` from the signals sensor.
