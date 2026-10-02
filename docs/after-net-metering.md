# After net metering: what is a kWh worth?

All blueprints in this repository make the same comparison: **what does it cost to store a kWh now, and what is it worth later?** This page explains the three prices they use.

## Three prices per time slot

| Symbol | Name in the blueprints | Meaning |
|---|---|---|
| S | `spot_now` | the price exactly as your price sensor reports it (often the day-ahead market price, per 15 minutes or per hour) |
| C | `price_now` | what one extra kWh **from the grid** costs you: S + *markup* |
| R | `feed_in_now` | what one kWh **exported** earns you |

The *markup* is everything your supplier adds per kWh: energy tax, supplier surcharge and VAT on those. In the Netherlands in 2026 this is about € 0.145 per kWh including VAT (check your own contract). If your price sensor already shows all-in prices, use 0.

### While net metering lasts

Every exported kWh is offset against an imported kWh at the full price, so **R = C**. The grid works as a free, lossless battery. Storing your own solar power in a battery only *loses* energy (about 25 % round trip). It only pays when the price later is clearly higher than now, which is ordinary price arbitrage.

### After net metering

**R = S × feed-in factor − feed-in fee.** Typical values:

- **Dynamic contract:** feed-in factor 1 (or 0.826 if your sensor includes 21 % VAT and the compensation does not), plus whatever feed-in fee your supplier charges. R follows the market price and can be negative.
- **Fixed contract:** the law guarantees at least 50 % of the bare supply price until 2030, minus possible feed-in costs. Model it with a factor and a fee, or use a fixed R.

Now **C − R** is large, typically € 0.15–0.30 per kWh. Every kWh of your own power that you use instead of export saves that amount. Storing solar power becomes worthwhile, also with losses:

> Store when **R ÷ efficiency + minimum profit ≤ the value later**.

With R = € 0.08, 75 % round-trip efficiency and € 0.02 minimum profit, the battery may store solar power if it can later replace power that costs at least € 0.127. That is almost always the case. Under net metering, with R = C = € 0.30, the price later would have to be at least € 0.42.

## What counts as "later"?

- **Battery:** the average of the most expensive slots it can cover with a full charge (level 2), or simply today's highest price (level 1).
- **Hot water tank and house:** you will need the heat anyway. Heating it earlier only shifts *when* you buy the energy. So the question is simply: is now cheaper than when the device would otherwise run? That is why these blueprints use the cheapest and most expensive *blocks* of the day.
- **EV:** you need the kWh before your next trip. Charge on surplus, otherwise in the cheapest hours before you leave.

## Negative prices

With a dynamic contract the market price is sometimes negative. Then:

- until 2027: C can become negative too. Using power earns money, and exporting costs money if R = C < 0.
- after 2027: R < 0 as soon as the market price is negative. Exporting costs money even if importing still costs a little.

Use as much as you can (boost the hot water tank, charge the EV and the battery). Whatever is left can be curtailed, see the blueprint *Stop exporting when it costs money*.

## Sources

- Rijksoverheid: [Salderingsregeling](https://www.rijksoverheid.nl/onderwerpen/energie-thuis/salderingsregeling)
- HomeWizard: [How the battery decides when to charge and discharge](https://helpdesk.homewizard.com/en/articles/14209959-how-does-the-battery-determine-when-to-charge-and-discharge)
