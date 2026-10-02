# Thuis energie opslaan met Home Assistant

**Sla je eigen zonnestroom, en goedkope stroom van het net, op in apparaten die je al hebt: het warmwatervat, het huis zelf, de auto en een (plug-in) thuisbatterij.**

🇬🇧 [English version](README.md) · De uitgebreide documentatie in [`docs/`](docs/) is in het Engels.

---

## Waarom: de saldering stopt

Op **1 januari 2027** stopt de salderingsregeling. Tot dan is het net een gratis batterij: elke kWh die je teruglevert, wordt verrekend met een kWh die je afneemt. Daarna krijg je voor teruglevering alleen een terugleververgoeding. Met een dynamisch contract is dat ongeveer de kale marktprijs, soms negatief en vaak met terugleverkosten. Voor afname betaal je nog steeds de volle prijs met energiebelasting.

Dat verschil is meestal **€ 0,15–0,30 per kWh**. Elke kWh zonnestroom die je zelf *gebruikt* in plaats van teruglevert, is dat verschil waard. Hetzelfde geldt voor stroom die je in de goedkope uren koopt in plaats van in de dure.

Waarschijnlijk heb je al veel opslag in huis:

| Opslag | Typische grootte | Verlies | Geschikt voor |
|---|---|---|---|
| **Elektrische auto** | 40–80 kWh | ~10 % | verreweg de grootste, als hij thuis staat |
| **Warmwatervat** (warmtepomp of elektrisch) | 1–2 kWh stroom per keer extra opwarmen (3–6 kWh warmte) | klein; een warmtepomp heeft bij een lagere vattemperatuur ook minder stroom nodig | elke dag, het hele jaar |
| **Het huis zelf** (warmtepomp) | een paar kWh warmte per °C | iets meer warmteverlies | stookseizoen |
| **Thuisbatterij** (HomeWizard Plug-In) | 2,7 kWh per stuk, meerdere te combineren | ~25 % heen en terug | avond en nacht |

Dit project laat stap voor stap zien hoe je die met Home Assistant benut.

---

## Kies je niveau

### 0. Alleen een HomeWizard-batterij? Gebruik Slim laden van HomeWizard zelf

Recente HomeWizard-firmware (P1 Meter ≥ 6.04 of kWh Meter ≥ 5.02) heeft **Slim laden** (`predictive`). Die voorspelt je verbruik, zonne-opbrengst en de prijzen, en laadt en ontlaadt op de goede momenten. Heb je een dynamisch contract, kies dan in de HomeWizard Energy-app *Slim met dynamische tarieven*. Dan laadt de batterij ook van het net als stroom goedkoop is, dus ook zonder zonnepanelen.

👉 **Voor de meeste mensen is dit de beste keuze voor alleen de batterij.** Daar is geen automatisering in Home Assistant voor nodig.
[HomeWizard Plug-In Battery](https://www.homewizard.com/nl/plug-in-battery/) · [Hoe Slim laden beslist](https://helpdesk.homewizard.com/en/articles/14209959-how-does-the-battery-determine-when-to-charge-and-discharge)

De niveaus hieronder zijn voor wie de logica zelf wil zien en bijstellen, of de batterij wil combineren met de andere opslag.

### 1. Eenvoudig: één blueprint voor de batterij

Goedkope en dure uren op basis van de laagste en hoogste prijs van vandaag, een controle of laden loont, en de nieuwe firmwaremodi. → [docs/level-1-simple.md](docs/level-1-simple.md)

[![Blueprint importeren](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2Feurojojo%2Fhomewizard-dynamic-battery%2Fblob%2Fmaster%2Fblueprints%2Fautomation%2Feurojojo%2Fbattery_simple.yaml)

### 2. Uitgebreid: de hele prijscurve

Gebruikt alle kwartier- of uurprijzen van vandaag en morgen, de laadtoestand en wat je eigen zonnestroom waard is, vóór en na de saldering. Laadt in precies de goedkoopste kwartieren die nodig zijn om de batterij te vullen, en ontlaadt in de duurste kwartieren die hij kan dekken. → [docs/level-2-advanced.md](docs/level-2-advanced.md)

[![Blueprint importeren](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2Feurojojo%2Fhomewizard-dynamic-battery%2Fblob%2Fmaster%2Fblueprints%2Fautomation%2Feurojojo%2Fbattery_advanced.yaml)

### 3. Energie opslaan in huis: batterij, warm water, verwarming, auto

Een set blueprints met één gezamenlijk prijsmodel en één prioriteitsvolgorde. Elke blueprint werkt ook los. → [docs/level-3-home-energy-storage.md](docs/level-3-home-energy-storage.md)

| Blueprint | Wat het doet |
|---|---|
| Batterij: houd hem voor het huis | Slim laden blijft de baas, maar de batterij ontlaadt nooit in de auto of de warmtepomp |
| Warmwatervat als warmtebatterij | extra opwarmen bij overschot of heel goedkope stroom, comfort in het goedkoopste blok, eco in het duurste blok |
| Huis als warmtebatterij | voorverwarmen vóór en iets lager tijdens de duurste uren |
| Auto: overschot en goedkoopste uren | laden op zonne-overschot, in de goedkoopste uren vóór vertrek, of nu meteen |
| Stop met terugleveren als het geld kost | omvormer begrenzen bij een negatieve terugleverwaarde |

De importknoppen staan in de [Engelse README](README.md#3-home-energy-storage-battery-hot-water-heating-ev).

---

## Installatie

1. **Niveau 1:** importeer de blueprint en maak er een automatisering van. Klaar.
2. **Niveau 2 en 3:** kopieer ook [`custom_templates/home_energy_storage.jinja`](custom_templates/home_energy_storage.jinja) naar `/config/custom_templates/` en voer de actie `homeassistant.reload_custom_templates` uit.
3. **Niveau 3:** voeg ook het pakket [`packages/home_energy_storage.yaml`](packages/home_energy_storage.yaml) toe (prioriteit, "auto moet laden", timer), of maak die helpers zelf aan.
4. Je hebt een prijssensor met een prijslijst nodig: ENTSO-e en Nord Pool (HACS) werken direct, andere bronnen via een kleine adapter ([docs/price-sources.md](docs/price-sources.md)).
5. Voor de HomeWizard-batterij: zet de entiteit *Batterijgroepmodus* van de P1 Meter aan ([docs/homewizard-battery-modes.md](docs/homewizard-battery-modes.md)).

Alle details: [docs/installation.md](docs/installation.md).

## Achtergrond

Dit begon in 2025 als twee automatiseringen die het ontbrekende "alleen laden" van de HomeWizard Plug-In Battery omzeilden ([v0.8](https://github.com/eurojojo/homewizard-dynamic-battery/tree/v0.8.0)). HomeWizard heeft die modus en Slim laden inmiddels zelf toegevoegd. Daarom ligt de nadruk nu op waar het na de saldering echt om gaat: je eigen stroom opslaan, in elk apparaat dat dat kan.

Niveau 3 is een algemene versie van wat er bij de maker thuis draait: warmtepomp met warmwatervat, elektrische auto, HomeWizard-batterij, zonnepanelen en een dynamisch contract.

## Licentie

[MIT](LICENSE) © Joost Smits
