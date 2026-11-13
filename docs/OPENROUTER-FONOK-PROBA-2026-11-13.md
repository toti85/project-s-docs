<!-- Automatikus tükör a privát Projekt S műhelyrepóból (scripts/docs_tukor.py). Személyes és hálózati adatok kitakarva. Ne itt szerkeszd. -->

# OpenRouter főnök-modellek próbája – akadályjelentés

Állapot: ELAKADT; valódi modellpróba nem indult. Fizetős hívás: 0; e kör költsége: 0 USD.

## Ellenőrzött akadály

A meglévő `openrouter_katalogus.kliens().elerheto()` hívás a fejlesztési munkamenetből `PermissionError` hibával állt le. A kliens a hitelesítési adatot nem éri el. Ez nem bizonyítja, hogy a szerverről hiányzik a kulcs; az itteni jogosultság nem elegendő. Titok kiolvasása, másolása vagy a zár módosítása nem történt.

A `proba()` jelenleg két `chat()` hívásra képes, de nem ad meg `max_tokens` értéket, és nincs összesített költségkorlátja. A visszakapott költség utólagos; a hiányzó `usage.cost` értéket 0-ként kezeli. Az 1 USD betartását ezért önmagában a próbák utáni költségösszegzés nem garantálja.

## Előkészített jelöltek

A meglévő `lista()` 322 eszközképes katalógustételt adott vissza. Katalógusfájl időbélyege (UTC): 2026-11-12T18:50:56+00:00. Ez gyorsítótárból olvasott katalógus, a modellek mai működését nem igazolja. Öt családból a négy legalacsonyabb bemeneti+kimeneti árú modellt választottam.

| Modell | Bemenet USD/M token | Kimenet USD/M token |
|---|---:|---:|
| `openai/gpt-5-nano` | 0.05 | 0.4 |
| `openai/gpt-5.6-luna` | 0.2 | 1.2 |
| `openai/gpt-5.6-luna-pro` | 0.2 | 1.2 |
| `openai/gpt-5.4-nano` | 0.2 | 1.25 |
| `moonshotai/kimi-k2` | 0.57 | 2.3 |
| `moonshotai/kimi-k2.5` | 0.5 | 2.5 |
| `moonshotai/kimi-k2-0905` | 0.6 | 2.5 |
| `moonshotai/kimi-k2-thinking` | 0.6 | 2.5 |
| `qwen/qwen3.7-flash` | 0.03 | 0.13 |
| `qwen/qwen3.5-flash-02-23` | 0.065 | 0.26 |
| `qwen/qwen3-14b` | 0.12 | 0.24 |
| `qwen/qwen3-32b` | 0.08 | 0.28 |
| `deepseek/deepseek-v4-flash` | 0.09 | 0.18 |
| `deepseek/deepseek-v4-flash-0731` | 0.0434 | 0.3 |
| `deepseek/deepseek-v3.2` | 0.26 | 0.38 |
| `deepseek/deepseek-v3.2-exp` | 0.27 | 0.41 |
| `google/gemini-2.5-flash-lite` | 0.1 | 0.4 |
| `google/gemini-3.1-flash-lite` | 0.25 | 1.5 |
| `google/gemini-3.1-flash-lite-preview` | 0.25 | 1.5 |
| `google/gemini-2.5-flash` | 0.3 | 2.5 |

## Folytatás feltétele

A próbát olyan meglévő, engedélyezett végrehajtási környezetből kell futtatni, amely a kliens hitelesítését és a panel által olvasott `openrouter-probak.json` írását jogosultan eléri. Előbb kérésenkénti token-/árkorlát és összesített, hibás vagy ismeretlen költségnél is konzervatív keretkezelés szükséges. A jogosultságot nem kerüljük meg.

Ebben a körben nincs új igazolt modell; a panel próbaeredményeit nem módosítottam. A 3. tétel akadályozott, a következő tétel a dokumentáció.
