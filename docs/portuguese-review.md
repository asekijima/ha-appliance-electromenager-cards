# Portuguese review and testing

Reviewed on 2026-11-13. Both files cover all 2,384 English reference keys across 56 sections. Configuration keys, entity IDs and interpolation placeholders retain their original spelling.

## Terminology decisions

The choices below follow regional manufacturer and product usage. This is a terminology review, not a statistical ranking of word frequency.

| Concept | Portugal | Brazil | Evidence |
|---|---|---|---|
| Washing machine / dryer | Máquina de lavar roupa / Máquina de secar roupa | Lavadora de roupas / Secadora de roupas | [Bosch Portugal](https://www.bosch-home.pt/pt/category-page/maquinas-de-lavar-e-secar-roupa), [Electrolux Brazil](https://loja.electrolux.com.br/eletrodomesticos/lavanderia-e-limpeza), [Consul Brazil](https://www.consul.com.br/) |
| Kitchen range hood | Exaustor | Coifa | [Consul: coifas, exaustores and depuradores](https://www.consul.com.br/facilita-consul/bem-pensado-para-voce/coifa-exaustor-depurador-como-escolher) |
| Air fryer | Fritadeira a ar | Air fryer | [Philips Portugal](https://www.philips.pt/c-m-ho/cozinhar/airfryer/all), [Mondial Brazil](https://www.mondial.com.br/eletroportateis/eletroportateis-para-cozinha/air-fryer) |
| Storage water heater | Termoacumulador | Aquecedor de água | The card depicts a storage tank; [Bosch Portugal](https://www.bosch-homecomfort.com/pt/pt/ocs/termoacumuladores-eletricos-1099286-c/) uses termoacumulador for that appliance. |
| UPS | UPS | Nobreak | [Intelbras](https://www.intelbras.com/pt-br/energia/nobreaks) |
| Flow rate | Caudal | Vazão | [Bosch Portugal glossary](https://www.bosch-homecomfort.com/pt/pt/conhecimentos/glossario/), [Lorenzetti Brazil](https://www.lorenzetti.com.br/noticias/releases/inverno-come%C3%A7a-com-queda-das-temperaturas-e-aumenta-a-busca-por-solu%C3%A7%C3%B5es-de-aquecimento-dentro-de-casa) |
| Pet fountain | Fonte para animais | Bebedouro para animais | [Brazilian product listings](https://www.petz.com.br/gato/acessorios-de-alimentacao/bebedouros) use bebedouro and fonte. |

Home Assistant's [language metadata](https://github.com/home-assistant/frontend/blob/dev/src/translations/translationMetadata.json) identifies Portugal as `pt` and Brazil as `pt-BR`. The resolver maps `pt` to our `pt-PT` file and keeps Brazil separate. Explicit card overrides also accept `pt-PT`, `pt-BR`, case variations and underscores.

## Review fixes

- Centralized language selectors remain shared across all 53 editor definitions; the schema inspection covers 76 exposed card variants.
- Fixed the blank first-use message in the single Appliance card. The translated prompt now appears when no appliance is selected, reappears after clearing a selection, and follows language changes.
- Corrected UPS runtime to **Autonomia**, including its editor fields. Pump runtime remains **Tempo de funcionamento**.
- Corrected vacuum time to **Duração** and the pet feeder's **Last feed** to **Última alimentação**.
- Corrected Brazilian kitchen hood, air fryer, UPS and pet fountain names, smoke-alert grammar, and related labels.
- Corrected gender agreement for doors, windows, valves, locks, blinds and spa covers, plus inconsistent humidity targets and supply-temperature labels.

## Automated checks

From the repository root:

```sh
node tests/portuguese.cjs
node build.js
node build.js --check
find src -name '*.js' -print0 | xargs -0 -n1 node --check
node --check dist/ha-appliance-electromenager-cards.js
```

The translation checks cover key completeness, nonempty values, placeholders, configuration identifiers, regional matching, overrides, language-selector values and selected context-sensitive terms. CI runs them.

Serve the repository locally:

```sh
python3 -m http.server 8765 --bind 127.0.0.1
```

Open [the browser suite](http://127.0.0.1:8765/docs/tests.html) and [the demo](http://127.0.0.1:8765/docs/demo.html). Select **PT-PT**, then **PT-BR** in the demo. Manufacturer/program names and raw sensor text are user data and can remain in their original language.

After synchronizing upstream `67000df` (version 1.131.0), the browser suite has 1,510 checks: 1,503 pass, including all 76 added for this contribution. Seven failures are also present in the untouched upstream baseline:

- `customCards = 117` (the repository currently registers 130 cards).
- `single card list in French`.
- `room climate comfort`.
- `room climate advice: air the room (drier outside)`.
- `ceiling spot: brightness slider by default`.
- `slider tile kept (greyed) when off`.
- `slider can be turned on for other lights`.

Existing build warnings concern missing keys in other languages; neither Portuguese translation produces missing-key warnings.

## Fork release workflow

PR [#1](https://github.com/asekijima/ha-appliance-electromenager-cards/pull/1) includes the latest upstream changes and prerelease version **1.131.1-pt.1**. Review it before merging, then publish tag `v1.131.1-pt.1` from the merged fork commit and install that release from the fork through HACS. The local ZIP is an optional test snapshot, not the release itself.

## Home Assistant smoke test

1. After the fork PR is reviewed, merged and released, install the fork release through HACS and test on a dashboard you can edit. Back up the current bundle and resource configuration first. Avoid loading both the fork and upstream copies simultaneously; reload the browser without its cached resource.
2. In the Home Assistant profile, select **Português**, then **Português (BR)**. Check card names, editor labels and the appliance picker in each.
3. Try a washing machine, fridge, UPS, water meter, pet feeder and a thermostat. Inspect running, idle and alert states at desktop and phone widths.
4. On a standalone card, set `language: pt-BR` while Home Assistant is Portuguese (Portugal), then reverse the two. Confirm the card honors the override. Shared editor labels follow the Home Assistant language; card-specific editor labels may follow the explicit override.
5. Also try the camera/NVR, PoE switch, water filter, reef aquarium and Raspberry Pi cards. Check an interpolated alert and number formatting. Currency and temperature units should follow the existing settings.
6. Clear the appliance selection in the single Appliance card and confirm that its translated selection prompt appears.
