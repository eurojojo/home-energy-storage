# Price sources

The level-2 and level-3 blueprints need one sensor that carries a list of prices in its attributes: today, and tomorrow as soon as those prices are known. The macro `hes_prices()` in [`home_energy_storage.jinja`](../custom_templates/home_energy_storage.jinja) reads these formats as they are:

| Source | Sensor | Attributes | Prices |
|---|---|---|---|
| [ENTSO-e](https://github.com/JaccoR/hass-entso-e) (HACS) | `sensor.entso_e_average_electricity_price` (or another ENTSO-e sensor with the `prices` attribute) | `prices`, `prices_today`, `prices_tomorrow` (`time`, `price`) | as configured in the integration (with or without VAT and markup) |
| [Nord Pool](https://github.com/custom-components/nordpool) (HACS) | `sensor.nordpool_kwh_nl_eur_…` | `raw_today`, `raw_tomorrow` (`start`, `value`) | as configured (VAT, additional costs) |
| anything else | an adapter sensor, see below | `prices` with `start`/`time`/`timestamp`/`start_time` and `price`/`value`/`total` | |

15-minute and hourly prices both work, the macro figures out the slot length by itself.

Set the *markup* option of the blueprints to whatever your sensor leaves out: energy tax, the supplier surcharge and the VAT on those. Does your sensor already report all-in prices (Tibber does, and so do ENTSO-e and Nord Pool if you set a price modifier)? Then use 0.

## Adapter sensors

Newer core integrations hand out prices through an action, not an attribute. A trigger-based template sensor can store them in a format the macro understands. Put one of the examples below in `configuration.yaml` or in a package. To find the `config_entry` ID, open Developer tools → Actions, choose the action, pick your integration and switch to YAML mode.

### Nord Pool (core integration)

This integration returns €/MWh. The adapter divides by 1000 to get €/kWh.

```yaml
template:
  - triggers:
      - trigger: time_pattern
        minutes: /15
      - trigger: homeassistant
        event: start
    actions:
      - action: nordpool.get_prices_for_date
        data:
          config_entry: YOUR_CONFIG_ENTRY_ID
          date: "{{ now().date() }}"
          areas: NL
          currency: EUR
        response_variable: today
      - action: nordpool.get_prices_for_date
        continue_on_error: true
        data:
          config_entry: YOUR_CONFIG_ENTRY_ID
          date: "{{ now().date() + timedelta(days=1) }}"
          areas: NL
          currency: EUR
        response_variable: tomorrow
    sensor:
      - name: Electricity prices
        unique_id: hes_prices_nordpool
        unit_of_measurement: "€/kWh"
        state: >
          {% set p = today.NL | selectattr('start', 'le', utcnow().isoformat()) | list %}
          {{ (p[-1].price / 1000) | round(5) if p else none }}
        attributes:
          prices: >
            {% set ns = namespace(out=[]) %}
            {% for p in (today.NL if today is defined else []) + (tomorrow.NL if tomorrow is defined and tomorrow.NL is defined else []) %}
              {% set ns.out = ns.out + [{'start': p.start, 'price': (p.price / 1000) | round(5)}] %}
            {% endfor %}
            {{ ns.out }}
```

Nord Pool reports market prices without VAT. In the Netherlands, set the markup to energy tax plus surcharge including VAT. If you also want VAT on the market price itself, multiply it in the adapter: `'price': (p.price / 1000 * 1.21) | round(5)`.

### Tibber

Tibber prices already include everything, so set the markup to 0.

```yaml
template:
  - triggers:
      - trigger: time_pattern
        minutes: /15
      - trigger: homeassistant
        event: start
    actions:
      - action: tibber.get_prices
        data:
          start: "{{ today_at('00:00') }}"
          end: "{{ today_at('00:00') + timedelta(days=2) }}"
        response_variable: tibber
    sensor:
      - name: Electricity prices
        unique_id: hes_prices_tibber
        unit_of_measurement: "€/kWh"
        state: "{{ now().isoformat() }}"
        attributes:
          prices: "{{ (tibber.prices.values() | list | first) or [] }}"
```

### EnergyZero

```yaml
template:
  - triggers:
      - trigger: time_pattern
        minutes: /15
      - trigger: homeassistant
        event: start
    actions:
      - action: energyzero.get_energy_prices
        data:
          config_entry: YOUR_CONFIG_ENTRY_ID
          incl_vat: true
        response_variable: ez
    sensor:
      - name: Electricity prices
        unique_id: hes_prices_energyzero
        unit_of_measurement: "€/kWh"
        state: "{{ now().isoformat() }}"
        attributes:
          prices: "{{ ez.prices }}"
```

EnergyZero returns market prices, with VAT if you ask for it, but without energy tax. Set the markup to match.

> I wrote these adapters from the response formats in the Home Assistant documentation. I use ENTSO-e and Nord Pool (HACS) myself, so I haven't tested them. If the signals stay at `ok: False`, look at the actual response in Developer tools → Actions, and please open an issue so I can fix the example.

## Check

```jinja
{% from 'home_energy_storage.jinja' import hes_prices, hes_signals %}
{{ (hes_prices('sensor.electricity_prices') | from_json) | count }} prices
{{ hes_signals('sensor.electricity_prices', 0.145, '2027-01-01') | from_json }}
```

Expect 24 to 48 prices for hourly data, or 96 to 192 for 15-minute data, depending on whether tomorrow's prices are out yet.
