# Level 2: advanced price control for the battery

Blueprint: [`battery_advanced.yaml`](../blueprints/automation/eurojojo/battery_advanced.yaml). It needs [`home_energy_storage.jinja`](../custom_templates/home_energy_storage.jinja) ([installation](installation.md#2-shared-price-model-level-2-and-3)).

[![Import blueprint](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2Feurojojo%2Fhome-energy-storage%2Fblob%2Fmaster%2Fblueprints%2Fautomation%2Feurojojo%2Fbattery_advanced.yaml)

This is the kind of logic you can build yourself on top of the HomeWizard firmware modes. It makes the same sort of decisions as Smart charging, but you can see every number. Try Smart charging first, and compare the two.

## Counting slots

Prices come per 15 minutes, or per hour. My 2.7 kWh battery charges at 800 W, so it needs 14 slots of 15 minutes to go from 10 % to 95 %. In the evening the house uses maybe 450 W, which means a full battery covers about 22 slots. So instead of "cheap means below some threshold", the blueprint asks three questions.

Should it charge from the grid now? Only if this slot is one of the *n* cheapest of all remaining known slots, where *n* is the number of slots it needs to fill up. And only if

`price now ÷ efficiency + minimum profit ≤ value later`

where *value later* is the average of the most expensive slots a full battery can cover.

Should it discharge now? Yes, if this slot is among the *m* most expensive remaining slots, with *m* the number of slots the energy in the battery can cover. It also discharges when the price is an outlier, above Q3 + 1.5 × IQR.

Should it store solar surplus? Only if `feed-in value ÷ efficiency + minimum profit ≤ value later`. During net metering the feed-in value equals the purchase price, so this only holds when prices rise sharply later on. After net metering it holds almost always.

"Remaining slots" runs from now to the last known price. In the morning that is midnight. Once tomorrow's prices are out, around 13:00, it is the end of tomorrow.

## Decision table

It runs every minute and the first match wins:

| # | Condition | Mode |
|---|---|---|
| 1 | one of your flexible-load binary sensors is on, or the import is above the heavy-load limit | `zero_charge_only` |
| 2 | among the cheapest *n* slots, grid charging pays, and no solar is being exported | `to_full` |
| 3 | above the minimum charge, and among the most expensive *m* slots or at an outlier price | `zero` |
| 4 | storing solar pays | `zero_charge_only` |
| 5 | none of the above | `standby` |

Each mode change ends up in the logbook along with the numbers behind it.

## Quartiles and outliers

The price model also works out the five-number summary of today's prices: minimum, Q1, median, Q3 and maximum, the same idea as a box plot. The battery decision above doesn't use them, but the hot water and heating blueprints do, and they look good on a dashboard.

- Q1, the 25th percentile, marks the cheap quarter of the day.
- Q3, the 75th percentile, marks the expensive quarter.
- An outlier is a price above Q3 + 1.5 × (Q3 − Q1). That's Tukey's rule for exceptional peaks.

Version 0.8 used Q1 and Q3 directly as the cheap and expensive thresholds. That follows the shape of the day nicely, but it has no idea how many slots the battery needs. Counting slots fixes that.

## Options

| Option | Default | Notes |
|---|---|---|
| Usable capacity | 2.7 kWh | all batteries together |
| Maximum power | 800 W | all batteries together |
| Typical household load in expensive hours | 450 W | the battery can't deliver more than the house uses |
| Round-trip efficiency | 75 % | |
| Minimum and grid-charge state of charge | 10 % and 95 % | |
| Minimum profit | € 0.02 per kWh | |
| Markup | € 0.145 per kWh | see [after net metering](after-net-metering.md) |
| Net metering until | 2027-01-01 | |
| Feed-in factor and fee | 1.0 and € 0 | after net metering, feed-in value = sensor price × factor − fee |
| Flexible loads | none | binary sensors that are on while the EV charges or the heat pump runs |
| Heavy load | 3500 W | |

## What it looked like in practice

This is version 0.8, end of November 2025. The battery kept the household consumption almost flat. At night it bought cheap power from the grid while the EV charged and the dishwasher ran, without discharging into either of them. When prices went up in the morning it supplied the house, topped up for a moment when the sun came out, and then covered the evening peak.

![Flattened usage curve](img/flattened-curve.png)

<img src="img/price-curve.png" width="400" alt="Prices that day: a small morning peak, a dip at midday and a high evening peak">
