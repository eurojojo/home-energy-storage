# Level 2 – advanced price control for the battery

Blueprint: [`battery_advanced.yaml`](../blueprints/automation/eurojojo/battery_advanced.yaml) · needs [`home_energy_storage.jinja`](../custom_templates/home_energy_storage.jinja) ([installation](installation.md#2-shared-price-model-level-2-and-3))
[![Import blueprint](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2Feurojojo%2Fhomewizard-dynamic-battery%2Fblob%2Fmaster%2Fblueprints%2Fautomation%2Feurojojo%2Fbattery_advanced.yaml)

This is what you could build yourself on top of the HomeWizard firmware modes: the same kind of decisions as Smart charging, but transparent and with your own numbers. Try Smart charging first and compare.

## The idea: count slots

Prices come per 15 minutes (or per hour). A battery of 2.7 kWh at 800 W needs **14 slots of 15 minutes** to fill from 10 % to 95 %. In the evening the house uses perhaps 450 W, so a full battery covers about **22 slots**. So instead of "cheap = below some threshold", the blueprint asks:

- **Charge from the grid now?** Only if this slot is one of the *n* cheapest of all remaining known slots, where *n* is the number of slots needed to fill the battery, **and**

  `price now ÷ efficiency + minimum profit ≤ value later`

  where *value later* is the average of the most expensive slots a full battery can cover.

- **Discharge now?** If this slot is one of the *m* most expensive remaining slots, where *m* is the number of slots the energy in the battery can cover (or if the price is an outlier, above Q3 + 1.5 × IQR).

- **Store solar surplus?** If `feed-in value ÷ efficiency + minimum profit ≤ value later`. Under net metering the feed-in value equals the purchase price, so this is only true when prices rise sharply later. After net metering it is almost always true.

"Remaining slots" means now until the end of the known prices: until midnight in the morning, until the end of tomorrow once tomorrow's prices are published (around 13:00).

## Decision tree

Evaluated every minute, first match wins:

| # | Condition | Mode |
|---|---|---|
| 1 | a flexible load runs (binary sensors you choose) or import > heavy load | `zero_charge_only` |
| 2 | one of the cheapest *n* slots, grid charging pays, not exporting | `to_full` |
| 3 | above the minimum state of charge and one of the most expensive *m* slots, or an outlier price | `zero` |
| 4 | storing solar pays | `zero_charge_only` |
| 5 | otherwise | `standby` |

Every change of mode is written to the logbook with the numbers behind it.

## Quartiles and outliers

The price model also provides the five-number summary of today's prices (minimum, Q1, median, Q3, maximum). Box plots use the same idea. They are not needed for the decision above, but the hot water and heating blueprints use them, and they are nice on a dashboard:

- **Q1** (25th percentile): the cheap quarter of the day
- **Q3** (75th percentile): the expensive quarter
- **Outlier**: above Q3 + 1.5 × (Q3 − Q1), Tukey's rule for exceptional peaks

Version 0.8 used Q1 and Q3 directly as cheap and expensive thresholds. That adapts to the shape of the day, but doesn't know how many slots the battery needs. Counting slots does.

## Options

| Option | Default | Notes |
|---|---|---|
| Usable capacity | 2.7 kWh | all batteries together |
| Maximum power | 800 W | all batteries together |
| Typical household load in expensive hours | 450 W | the battery can't supply more than the house uses |
| Round-trip efficiency | 75 % | |
| Minimum / grid-charge state of charge | 10 % / 95 % | |
| Minimum profit | € 0.02/kWh | |
| Markup | € 0.145/kWh | see [after net metering](after-net-metering.md) |
| Net metering until | 2027-01-01 | |
| Feed-in factor / fee | 1.0 / € 0 | after net metering: feed-in value = sensor price × factor − fee |
| Flexible loads | – | binary sensors for EV charging and heat pump running |
| Heavy load | 3500 W | |

## In practice (version 0.8, November 2025)

The battery kept the household consumption almost flat: it bought cheaply from the grid while the EV charged at night and the dishwasher ran, without discharging into them. It supplied the house when prices rose in the morning, topped up briefly in the sun, and covered the evening peak.

![Flattened usage curve](img/flattened-curve.png)

<img src="img/price-curve.png" width="400" alt="Prices of that day: a morning peak, a midday dip and a high evening peak">
