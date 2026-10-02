# Level 3 – home energy storage

**Use every device that can store energy: the hot water tank, the house, the EV and the home battery.** Each one gets its own blueprint, and they share one price model and one priority order.

After net metering this is where the money is. A HomeWizard battery stores 2.7 kWh with 25 % loss. A hot water tank stores the heat of 1–2 kWh of electricity every day with hardly any loss. A floor-heated house can shift several kWh of heating by a few hours. And an EV takes 10–20 kWh in one sunny afternoon. See [after net metering](after-net-metering.md) for the economics.

## Overview

| Blueprint | Device | Stores | Needs |
|---|---|---|---|
| [Battery: keep it for the house](../blueprints/automation/eurojojo/battery_guard.yaml) | HomeWizard Plug-In Battery | electricity | battery mode entity, "flexible load running" binary sensors |
| [Hot water tank as a heat battery](../blueprints/automation/eurojojo/hot_water.yaml) | `water_heater` (heat pump or electric) | heat | price sensor, P1 power, boost timer |
| [House as a heat battery](../blueprints/automation/eurojojo/heating.yaml) | `climate` (heat pump thermostat) | heat in the building | price sensor, number helper |
| [EV: surplus and cheapest hours](../blueprints/automation/eurojojo/ev_charging.yaml) | charger with an on/off `switch` | electricity | price sensor, P1 power, two helpers |
| [Stop exporting when it costs money](../blueprints/automation/eurojojo/curtail_export.yaml) | inverter `switch` or limit `number` | – | price sensor |

Install the [shared price model and the package](installation.md) first, then add the blueprints one at a time and watch the logbook for a few days before adding the next. A good order: battery guard → hot water → heating → EV → curtailment.

## Priority: who gets the surplus?

Physically, solar surplus flows to whatever consumes first. The HomeWizard battery reacts within seconds, so it usually wins. The priority helper `input_select.hes_priority` (from the package) tells the other blueprints how to count:

- If **hot water comes before the battery**, the power the battery is charging counts as available surplus for the hot water tank. The tank starts its boost, the heat pump takes the power, and the battery gets less.
- If **the battery comes first**, only real export counts, and the hot water tank doesn't start a "very cheap" boost while the battery is still filling from the sun.
- The same goes for the EV.
- **EV must charge** (`input_boolean.hes_ev_must_charge`) always puts the EV first: it charges, and the hot water tank doesn't take the surplus.

What order is best? It depends on your home:

| Store | Strong point | Weak point |
|---|---|---|
| Battery | supplies the expensive evening hours exactly | small, 25 % loss |
| Hot water | cheap heat, every day | full after one or two boosts |
| EV | huge | only useful when it is at home and needs charge |

The author uses **Battery → Hot water → EV**: the battery is full by early afternoon on a sunny day, after that the tank gets a boost, and the EV gets what is left (or everything, with "EV must charge").

## Battery: keep it for the house

Smart charging (`predictive`) stays in charge. One rule is added: **while the EV charges or the heat pump runs, the battery doesn't discharge** (`zero_charge_only`), because emptying it into another store only loses energy. Exception: if the rest of the house also uses a lot (oven, induction hob, more than 1500 W besides the flexible loads), the battery may supply, because its 800 W then goes to that load.

| Condition | Mode |
|---|---|
| a flexible load runs and the rest of the house uses ≤ the exception | `zero_charge_only` |
| otherwise | normal mode (`predictive` or `zero`) |

It only switches between those two modes, so a manual `to_full` or `standby` is left alone.

*Binary sensors:* many heat pump integrations have a "compressor running" or "heating" state; for the EV use the charger status or a threshold helper on the charger power (Settings → Helpers → Threshold, e.g. above 1000 W). For a charger on its own phase, the P1 power of that phase works as well.

## Hot water tank as a heat battery

| # | Condition | Target |
|---|---|---|
| 1 | solar boost timer running | boost (65 °C) |
| 2 | very cheap (negative, or among the cheapest 10 % of today and ≤ 75 % of the median), unless the battery has priority and is still filling | boost |
| 3 | in today's most expensive block (3 h) and that block is ≥ 15 % above the median | eco (45 °C) |
| 4 | in today's cheapest block (2 h) | comfort (55 °C) |
| 5 | otherwise | normal (50 °C) |

*Solar boost:* after net metering, a surplus of at least 1000 W for 10 minutes starts the boost timer (1 hour by default). Under net metering it doesn't, because exporting is free storage then.

*Why these temperatures?* A heat pump heats water more efficiently at a low temperature, so the normal setpoint stays low (50 °C). Extra heat is stored only when it is cheap or free. Eco lets the tank cool a bit during the expensive hours. Choose temperatures that suit your tank and your household. Many heat pumps reach 55 °C with the compressor and only go above that with an electric element (COP 1): then a boost to 65 °C only pays with solar surplus or very low prices.

*Legionella:* keep your device's own disinfection program unless you know what you are doing. The Dutch guideline (ISSO 55.1) asks for 60 °C for at least 20 minutes for the whole tank, regularly. Boosts to 65 °C often count as disinfection, so you could plan the device's own program right after the usual solar boost time. The author lets Home Assistant plan the weekly disinfection in the cheapest block and alarm if it is missed. That is not in these blueprints, because a mistake there is a health risk.

*Operation mode:* some devices only accept a new temperature in a particular mode (for example `performance` on Remeha heat pumps). Fill in that mode in the blueprint.

## House as a heat battery

| Phase | When | Target |
|---|---|---|
| pre-heat | the 2 hours before today's most expensive block, if those hours cost ≤ 85 % of the block | original + 0.5 °C |
| save | during the most expensive block (3 h, ≥ 15 % above the median) | original − 0.5 °C |
| normal | otherwise | original (restored) |

Only in the heating season (outdoor temperature below 14 °C) and in the HVAC modes you choose. The original target is kept in `input_number.hes_heating_base`. If you or the thermostat's schedule change the target during a phase, the blueprint backs off until the next phase and leaves the new target alone.

Works best with floor heating and a well-insulated house. Start with 0.5 °C and see whether you notice it.

## EV: surplus and cheapest hours

Only acts while `input_boolean.hes_ev_smart_charging` is on.

| # | Condition | Charger |
|---|---|---|
| 1 | EV must charge | on |
| 2 | in the cheapest *N* hours of the 24 hours before "ready by" (default 4 h before 07:00) | on |
| 3 | surplus ≥ 1500 W for 5 minutes | on |
| 4 | import ≥ 500 W for 10 minutes | off |
| 5 | import ≥ 1500 W (charging from the grid outside 1 and 2) | off |

⚠️ **Test pausing and resuming with your own charger and car, during the day, while you are at home.** Some combinations don't like it: the car reports a charging error, or the session doesn't resume until you reconnect the cable. The author's charger got confused this way, which is why his own setup does not pause sessions remotely any more. If that happens to you, use only "EV must charge" and the charger's own schedule.

*Surplus:* one phase at 6 A is about 1400 W, three phases about 4100 W. Below that the car can't charge on surplus alone. Chargers that can set the current (a `number` entity) can follow the surplus more closely. That is not in this blueprint (yet), because every charger does it differently.

## Stop exporting when it costs money

When the feed-in value is below the threshold (default € 0), the inverter is switched off or its limit is set to the "curtailed" value; when it is above, it is restored. Under net metering the feed-in value is the all-in price, after net metering the market price × factor − fee. So after 2027 it reacts to every negative market price.

Use a power limit (`number`) rather than switching off if your inverter has one: then the house can still use the solar power, only the export stops.

## What is not included

- **Solar and consumption forecasts.** The author's own setup uses a small Python planner with Solcast forecasts and a household profile. For example, it knows in the morning that the battery will fill from the sun anyway. These blueprints react to *measured* surplus and the price curve. That is simpler and, for these loads, almost as good.
- **Home battery grid charging in level 3.** Smart charging does that. If you want your own logic, use the [level-2 blueprint](level-2-advanced.md) instead of the battery guard (it has the same "flexible loads" option). Never run both.
- **Washing machines and dishwashers.** Start them with their own timer, or use a separate automation on `cheapest_block` from the signals sensor.

## The author's home, as an example

| Role | Device | Integration |
|---|---|---|
| P1 Meter + battery | HomeWizard P1 Meter, Plug-In Battery 2.7 kWh | HomeWizard |
| Solar | Growatt MIN 4200TL-XE inverter | growatt_local |
| Hot water + heating | Remeha Confida heat pump with hot water tank, eTwist thermostat | remeha_home |
| EV | VW ID.3 (58 kWh) on an Alfen Eve charger | 50five / Alfen |
| Prices | Nord Pool 15 min + ENTSO-e, NextEnergy dynamic | nordpool, entsoe |

Priority Battery → Hot water → EV, hot water 50 / 55 / 65 / 45 °C, heating ±0.5 °C, battery guard with the heat pump and the charger phase as flexible loads.
