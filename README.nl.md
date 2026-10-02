# Thuis energie opslaan met Home Assistant

Sla je eigen zonnestroom, en goedkope stroom van het net, op in apparaten die je al hebt. Een warmwatervat, een huis met een warmtepomp en een elektrische auto kunnen veel meer energie kwijt dan een stekkerbatterij. Dit project laat zien hoe je ze met blueprints in Home Assistant inzet.

🇬🇧 [English version](README.md). De uitgebreide documentatie in [`docs/`](docs/) is in het Engels.

## Waarom: de saldering stopt

Op 1 januari 2027 stopt de salderingsregeling. Tot die tijd werkt het net als een gratis batterij: elke kWh die je teruglevert, valt weg tegen een kWh die je afneemt.

Daarna krijg je voor teruglevering alleen nog een vergoeding. Met een dynamisch contract is dat ongeveer de kale marktprijs, vaak met terugleverkosten eraf, en soms is hij negatief. Voor stroom van het net betaal je nog steeds de volle prijs met energiebelasting. Het verschil ligt meestal tussen € 0,15 en € 0,30 per kWh. Dat verdien je op elke kWh zonnestroom die je zelf gebruikt in plaats van teruglevert. Stroom kopen in de goedkope uren in plaats van de dure werkt net zo.

De meeste huizen met zonnepanelen hebben al meer opslag dan je zou denken:

| Opslag | Typische grootte | Verlies | Wanneer het helpt |
|---|---|---|---|
| Elektrische auto | 40–80 kWh | ongeveer 10 % | als hij thuis staat |
| Warmwatervat (warmtepomp of elektrisch) | 1–2 kWh stroom per keer extra opwarmen, 3–6 kWh warmte | klein, en een warmtepomp heeft bij een lagere vattemperatuur minder stroom nodig | elke dag van het jaar |
| Het huis zelf (warmtepomp) | een paar kWh warmte per °C | iets meer warmteverlies | in het stookseizoen |
| HomeWizard Plug-In Battery | 2,7 kWh per stuk, je kunt er meer combineren | ongeveer 25 % heen en terug | 's avonds en 's nachts |

## Kies je niveau

### Alleen een HomeWizard-batterij? Begin met Slim laden van HomeWizard

Recente HomeWizard-firmware (P1 Meter 6.04 of nieuwer, kWh Meter 5.02 of nieuwer) heeft Slim laden (`predictive`). Die voorspelt je verbruik, je zonne-opbrengst en de prijzen, en plant laden en ontladen daaromheen. Heb je een dynamisch contract, kies dan in de HomeWizard Energy-app *Slim met dynamische tarieven*. Dan laadt de batterij ook van het net als stroom goedkoop is. Zonnepanelen zijn dus niet nodig.

Voor alleen de batterij raad ik de meeste mensen dit aan. Je hebt er geen automatisering in Home Assistant voor nodig.
Zie [HomeWizard Plug-In Battery](https://www.homewizard.com/nl/plug-in-battery/) en [hoe Slim laden beslist](https://helpdesk.homewizard.com/en/articles/14209959-how-does-the-battery-determine-when-to-charge-and-discharge).

De niveaus hieronder zijn voor wie de logica zelf wil zien en bijstellen, of de batterij wil laten samenwerken met andere opslag.

### Niveau 1: één eenvoudige blueprint voor de batterij

Bepaalt goedkope en dure uren aan de hand van de laagste en hoogste prijs van vandaag, kijkt of laden van het net loont en gebruikt de firmwaremodi `zero`, `zero_charge_only`, `to_full` en `standby`. Makkelijk te lezen en aan te passen. Zie [docs/level-1-simple.md](docs/level-1-simple.md).

[![Blueprint importeren](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2Feurojojo%2Fhome-energy-storage%2Fblob%2Fmaster%2Fblueprints%2Fautomation%2Feurojojo%2Fbattery_simple.yaml)

### Niveau 2: de hele prijscurve

Deze kijkt naar alle kwartier- of uurprijzen van vandaag en morgen. Hij telt hoeveel kwartieren de batterij nodig heeft om vol te raken en laadt precies in de goedkoopste daarvan. Ontladen gebeurt in de duurste kwartieren die een volle batterij kan dekken. Ook weet hij dat je zonnestroom tijdens de saldering op het net meer waard is dan in de batterij, en daarna juist minder. Zie [docs/level-2-advanced.md](docs/level-2-advanced.md).

[![Blueprint importeren](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2Feurojojo%2Fhome-energy-storage%2Fblob%2Fmaster%2Fblueprints%2Fautomation%2Feurojojo%2Fbattery_advanced.yaml)

### Niveau 3: batterij, warm water, verwarming en auto samen

Een set blueprints met één gezamenlijk prijsmodel en één prioriteitsvolgorde. Elke blueprint werkt ook los. Zie [docs/level-3-home-energy-storage.md](docs/level-3-home-energy-storage.md).

| Blueprint | Wat het doet |
|---|---|
| Batterij: houd hem voor het huis | Slim laden blijft de baas, maar de batterij ontlaadt niet in de auto of de warmtepomp |
| Warmwatervat als warmtebatterij | extra opwarmen bij overschot of heel goedkope stroom, comfort in het goedkoopste blok, eco in het duurste |
| Huis als warmtebatterij | iets warmer vóór de duurste uren, iets kouder tijdens die uren |
| Auto: overschot en goedkoopste uren | laden op zonne-overschot, in de goedkoopste uren vóór vertrek, of meteen |
| Stop met terugleveren als het geld kost | de omvormer begrenzen als terugleveren negatief uitpakt |

De importknoppen staan in de [Engelse README](README.md#level-3-battery-hot-water-heating-and-ev-together).

## Installatie

Kort samengevat. Alle details staan in [docs/installation.md](docs/installation.md).

1. Niveau 1: importeer de blueprint en maak er een automatisering van. Meer hoeft niet.
2. Niveau 2 en 3 hebben ook het gedeelde prijsmodel nodig. Kopieer [`custom_templates/home_energy_storage.jinja`](custom_templates/home_energy_storage.jinja) naar `/config/custom_templates/` en voer de actie `homeassistant.reload_custom_templates` uit.
3. Niveau 3 gebruikt een paar helpers (prioriteit, "auto moet laden", een timer). Het pakket [`packages/home_energy_storage.yaml`](packages/home_energy_storage.yaml) maakt ze aan. Je kunt ze ook zelf in de interface maken.
4. Je hebt een prijssensor met een lijst prijzen nodig. ENTSO-e en Nord Pool (HACS) werken direct, voor andere bronnen staat er een kleine adapter in [docs/price-sources.md](docs/price-sources.md).
5. Voor de HomeWizard-batterij zet je eerst de entiteit *Batterijgroepmodus* van de P1 Meter aan, zie [docs/homewizard-battery-modes.md](docs/homewizard-battery-modes.md).

## Achtergrond

Ik ben hier in 2025 mee begonnen onder de naam *homewizard-dynamic-battery*. Het waren twee automatiseringen die het ontbrekende "alleen laden" van de HomeWizard Plug-In Battery omzeilden. Die versie staat nog als [v0.8](https://github.com/eurojojo/home-energy-storage/tree/v0.8.0) op GitHub. HomeWizard heeft die modus en Slim laden inmiddels zelf toegevoegd. Daarom gaat het project nu over wat na de saldering meer oplevert: je eigen stroom opslaan, in elk apparaat dat dat kan.

Niveau 3 is een algemene versie van wat bij mij thuis draait: een Remeha-warmtepomp met warmwatervat, een VW ID.3 aan een Alfen-laadpaal, een HomeWizard-batterij, zonnepanelen op een Growatt-omvormer en een dynamisch contract bij NextEnergy. Jouw apparaten zijn anders. De blueprints gebruiken alleen standaard entiteiten van Home Assistant (`water_heater`, `climate`, `switch`, `select`, vermogenssensoren), dus je kunt je eigen apparaten invullen.

## Licentie

[MIT](LICENSE) © Joost Smits
