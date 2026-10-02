# Price sources

The level-2 and level-3 blueprints need **one sensor with a list of prices** (today, and tomorrow once known) in its attributes. The macro `hes_prices()` in [`home_energy_storage.jinja`](../custom_templates/home_energy_storage.jinja) reads these formats directly:

| Source | Sensor | Attributes | Prices |
|---|---|---|---|
| [ENTSO-e](https://github.com/JaccoR/hass-entso-e) (HACS) | `sensor.entso_e_average_electricity_price` (or another ENTSO-e sensor with the `prices` attribute) | `prices`, `prices_today`, `prices_tomorrow` (`time`, `price`) | as configured in the integration (with or without VAT and markup) |
| [Nord Pool](https://github.com/custom-components/nordpool) (HACS) | `sensor.nordpool_kwh_nl_eur_…` | `raw_today`, `raw_tomorrow` (`start`, `value`) | as configured (VAT, additional costs) |
| anything else | an adapter sensor, see below | `prices` with `start`/`time`/`timestamp`/`start_time` and `price`/`value`/`total` | |

15-minute and hourly prices both work. The macro detects the slot length itself.

**Markup:** set the *markup* option of the blueprints to whatever your sensor does **not** include yet: energy tax, supplier surcharge and VAT on those. If your sensor already reports all-in prices (Tibber, or a price modifier in ENTSO-e/Nord Pool), use 0.

## Adapter sensors

The newer core integrations return prices through an *action* instead of an attribute. A trigger-based template sensor stores them in the right format. Put one of these in `configuration.yaml` or a package. Find the `config_entry` ID in Developer tools → Actions: choose the action, pick the integration, switch to YAML mode.

### Nord Pool (core integration)

Prices come in €/MWh; the adapter converts them to €/kWh.

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

The Nord Pool core integration reports market prices **without VAT**. In the Netherlands use markup = (energy tax + surcharge) incl. VAT, and if you want VAT on the market price too, multiply it: `'price': (p.price / 1000 * 1.21) | round(5)`.

### Tibber

Tibber prices are already all-in, so use **markup 0**.

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

EnergyZero returns market prices (optionally with VAT), without energy tax: set the markup accordingly.

> These adapters follow the response formats in the Home Assistant documentation, but the author uses ENTSO-e and Nord Pool (HACS) and has not tested them. Check the response in **Developer tools → Actions** if the signals stay `ok: False`, and please report it in an issue.

## Check

```jinja
{% from 'home_energy_storage.jinja' import hes_prices, hes_signals %}
{{ (hes_prices('sensor.electricity_prices') | from_json) | count }} prices
{{ hes_signals('sensor.electricity_prices', 0.145, '2027-01-01') | from_json }}
```

You should see 24–48 (hourly) or 96–192 (15-minute) prices, depending on whether tomorrow is already known.
