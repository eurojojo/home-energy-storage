# After net metering: what is a kWh worth?

Every blueprint in this repository asks the same question. What does it cost to store a kWh now, and what will it be worth later? This page explains the prices they use for that.

## Three prices per time slot

| Symbol | Name in the blueprints | Meaning |
|---|---|---|
| S | `spot_now` | the price exactly as your price sensor reports it, usually the day-ahead market price per 15 minutes or per hour |
| C | `price_now` | what one extra kWh from the grid costs you: S plus the markup |
| R | `feed_in_now` | what one exported kWh earns you |

The markup is what your supplier adds per kWh on top of the sensor price: energy tax, a surcharge and the VAT on those. In the Netherlands in 2026 that comes to about € 0.145 per kWh including VAT, but check your own contract. If your price sensor already shows all-in prices, set it to 0.

### While net metering lasts

An exported kWh is offset against an imported kWh at the full price, so R = C. The grid is a free battery without losses. Putting your own solar power in a battery only costs you the round-trip loss of about 25 %. It pays only when the price later is clearly higher than now, which is plain price arbitrage.

### After net metering

Now R = S × feed-in factor − feed-in fee. Some typical settings:

- Dynamic contract: factor 1, or 0.826 if your sensor includes 21 % VAT and the compensation doesn't. Subtract the feed-in fee your supplier charges. R follows the market and can go below zero.
- Fixed contract: until 2030 the law guarantees at least 50 % of the bare supply price, minus possible feed-in costs. A factor and a fee get you close enough.

The gap C − R is now large, usually € 0.15 to € 0.30 per kWh. That is what you save on every kWh of your own power that you use instead of export. Storing solar power starts to pay, even with losses. The rule the blueprints use:

> Store when R ÷ efficiency + minimum profit ≤ the value later.

An example: R = € 0.08, a round-trip efficiency of 75 % and a minimum profit of € 0.02. The battery may store solar power if it can replace power later that costs at least € 0.127. That is nearly always true. Under net metering, with R = C = € 0.30, the price later would have to be € 0.42 or more.

## What counts as "later"?

For the battery, it is the average of the most expensive slots that a full battery can cover (level 2), or just today's highest price (level 1).

The hot water tank and the house need the heat anyway. Heating earlier only changes when you buy the energy, so the question becomes whether now is cheaper than the moment the device would otherwise run. That's why those blueprints work with the cheapest and the most expensive block of the day.

The EV needs its kWh before the next trip. Charge on surplus, and otherwise in the cheapest hours before you leave.

## Negative prices

On a dynamic contract the market price is sometimes negative. Until 2027 the all-in price C can drop below zero as well. Then using power earns money and exporting costs money. After 2027, R goes negative as soon as the market price does, even while importing still costs a little.

In those hours, use whatever you can: boost the hot water tank, charge the EV and the battery. If there is still power left over, the blueprint *Stop exporting when it costs money* can limit the inverter.

## Sources

- Rijksoverheid: [Salderingsregeling](https://www.rijksoverheid.nl/onderwerpen/energie-thuis/salderingsregeling)
- HomeWizard: [How the battery decides when to charge and discharge](https://helpdesk.homewizard.com/en/articles/14209959-how-does-the-battery-determine-when-to-charge-and-discharge)
