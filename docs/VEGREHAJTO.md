<!-- Automatikus tükör a privát Projekt S műhelyrepóból (scripts/docs_tukor.py). Személyes és hálózati adatok kitakarva. Ne itt szerkeszd. -->

# Projekt S végrehajtó (psx) – önálló, szolgáltatófüggetlen munkavégzés

_2026-11-04. Kód: `executor/psx.py`, `executor/acceptance.py`; felügyelő: `project-s-executor.service`._

## 1. Felmérés: mi volt meg
| Képesség | Hol | Besorolás |
|---|---|---|
| Feladat indítása systemd-unitban, időkorlát, leállítás, napló | `mcp/jobs.py`, `mcp/runner.py` (workshop) | **működött** – psx ezt a mintát viszi tovább (unit/kísérlet, NoNewPrivileges, MemoryMax, RuntimeMaxSec, KillMode=mixed) |
| Elvárt eredmény + független ellenőrzés (végrehajtva ≠ elfogadva) | `scripts/ugy.py` (workshop) | **működött** – a szabálymotor közös modullá lett: `executor/acceptance.py` |
| Ügyállapot, napló, GitHub-mentés | `ugyek/`, `ugy` | **működött** – az ügy most a psx-nek küld be, és hookkal szinkronizál |
| Hiba: modellimit = „sikertelen”, a munka elveszett | `jobs.py` + runner (2026-11-04 01:58, job `…-ec1b`: „You've hit your 5-hour limit”) | **éles hiba** – ezt javítja a psx |
| `continuous_mind.py` (goal_queue, szívverés, prioritás) | project-s-unified @ 65fb73a | **megvan, nem igazolt** (Windows-utak, Continue/SAI) – a cél-sémát már átvettük |
| `continue_executor_adapter.py` (`cn -p`, outbox, checksum) | unified | **megvan, nem igazolt**; a Continue maga **megszűnt** (lásd lent) |
| `bridge_runner.py` (kimenet átvétele checksummal, verdict) | unified | **megvan, nem igazolt** – az „azonosság + integritás” elvet a hash-láncolt napló viszi tovább |
| `sai_runner.py` (Interactive Layer → SAI) | unified | **megvan, nem igazolt**, OpenRouter-kulcsot igényel |
| `continue_watchdog.py` (3 egymás utáni hiba → stop, kill switch) | unified | **megvan, nem igazolt** – psx: `max_attempts`, költségkeret, `waiting_human` |
| `mission_outcome_schema.py` (AO8N) | – | **csak történeti nyom** (riportok említik, a kód nincs meg) |
| `compute_arbiter.py` (modellválasztás) | – | **csak történeti nyom** (a CM importálja, a fájl nincs meg) |
| Augusztusi canonical mag (lifecycle engine, journal) | – | **hiányzik** (Data3: nincs találat; DEV laptop: később) – psx a journal-elvet kicsiben valósítja meg |

## 2. Végrehajtó-alap: döntés
| Jelölt | Állapot (2026-11-04, hivatalos repó) | Döntés |
|---|---|---|
| Continue (`continuedev/continue`) | README: *„no longer actively maintained and is read-only”*, végső 2.0.0 | **elvetve** – nem építünk megszűnt projektre (a régi S erre épült) |
| OpenCode (`anomalyco/opencode`, v1.19.5, MIT) | aktív; `opencode run -m provider/model`, sok szolgáltató | **következő adapter** – szolgáltatónkénti `opencode auth login` (fiókbelépés) kell hozzá |
| Dagu (`dagucloud/dagu`, v2.25.2) | aktív, YAML-munkafolyamatok | **nem most** – a psx + systemd már tartós, naplózott, leállítható; második vezérlő lenne |
| OpenHands SDK (v1.59.0) | aktív, Python SDK + agent server | **nem most** – LiteLLM API-kulcs (költés) kellene |
| **Claude Code CLI** (`claude -p`) | telepítve, előfizetéssel hitelesítve | **adapter: `claude`** (stream-json, `--max-budget-usd`, limitfelismerés) |
| **GitHub Copilot CLI** (`copilot -p`, v1.0.105, Pro+) | hivatalos kiadás, SHA-256 ellenőrizve, toti gh-belépésével működik | **adapter: `copilot`** (szűk eszköz-engedélylista, kreditszámlálás) |

**Választás:** S saját, vékony végrehajtója (psx) + két, már hitelesített ügynöki CLI adapterként.
Két független szolgáltató új fiók és API-költés nélkül; a harmadik (OpenCode) egy fájlnyi adapter, ha toti belép.

## 3. Működés
- **Igazság = feladatonkénti eseménynapló** (`tasks/<id>/events.jsonl`, append-only, hash-lánc, `psx verify-journal`);
  az állapot belőle számolt (`reduce`), a `result.json` csak nézet.
- **Átvétel:** `psx submit` – idempotencia-kulcs (megadott vagy a spec hash-e); azonos kulcs → ugyanaz a feladat, nincs dupla munka.
- **Végrehajtás:** a felügyelő (`project-s-executor.service`) minden kísérletet külön `psx-<id>-a<n>` systemd-unitban indít
  (független a Claude-beszélgetéstől és a felügyelőtől is), elkülönített munkakönyvtárban (`worktree` / `copy` / `empty` / `dir`).
- **Lépések:** `cmd` (determinisztikus) és `agent` (modell); lépésenkénti ellenőrzőpont; `requires` függőség.
- **Megszakadás után:** kész lépést nem ismétel; idempotens lépést újrafuttat; **nem idempotens lépést nem ismétel vakon**
  → `waiting_human` (`psx resume` = jóváhagyott újrafuttatás). Végrehajtás után megszakadt ellenőrzésnél csak az ellenőrzés fut újra.
- **Modellimit:** felismeri (Claude: 429 / `usage_limit_reached`, „resets 4:20am”; Copilot: kvóta/402/429), rögzíti a lehűlést
  (`providers.json`), elmenti a **strukturált munkacsomagot** (`attempts/<n>/package-<lépés>.json`: cél, elfogadási feltételek,
  kész lépések, git-állapot, korábbi próbálkozások kimenete), és ugyanazzal a csomaggal a következő szolgáltatónak adja át.
  Ha nincs elérhető modell: a modellt nem igénylő, független lépések tovább futnak, a többi `waiting_model`; a felügyelő
  automatikusan folytat, amint egy szolgáltató elérhető.
- **Korlátok:** lépés- és feladat-időkorlát (folyamatcsoport-leállítás), `max_cost_usd` (Claude, a CLI-nek is átadva),
  `max_credits` (Copilot), `max_attempts`, globális párhuzamosság (`PSX_MAX_RUNNING`, élesben 3), memória (`limits.memory`).
  Elfogyott keret → `waiting_human` (nem végtelen újrapróbálkozás).
- **Elfogadás:** csak a feladathoz rögzített szabályok (`acceptance`) döntenek; a „kész” szöveg és a 0-s kilépés nem.
  Szabály nélkül vagy `ember` szabállyal → `needs_acceptance` (`psx approve` / `psx reject`).
- **Ügykezelő:** az `ugy inditas` psx-feladatot küld be (szolgáltatók: claude → copilot), az eredményt a `_psx_szinkron`
  hook írja az ügyiratba; `ugy elfogad/elutasit/leallit` a psx-naplóba ír (egy igazság).

## 4. Próbák (2026-11-04, izolált `PSX_HOME=~/work/psx-test/home`, tesztrepó `~/work/psx-testrepo`)
| # | Próba | Feladat | Eredmény |
|---|---|---|---|
| 1 | átvétel → végrehajtás → ellenőrzött eredmény (Claude javítja az `add()`-ot) | `t20261104-065334-8f91c6` | **accepted**, 0,12 USD; 3/3 szabály ✅ |
| 2 | a beküldő kliens `kill -9`, a hívó shell is kilépett | `t20261104-065456-e83fb0` | a munka tovább futott (`user@1001.service/app.slice/psxt-…`) |
| 3a | felügyelő-újraindítás + munkavégző SIGKILL az s2 közben | ugyanaz | `interrupted` → új kísérlet; **s1 nem ismétlődött**, s2 (idempotens) újrafutott |
| 3b | SIGKILL a nem idempotens s3 közben | ugyanaz | **waiting_human**; `psx resume` után kész; trace: s1 ×1, s4 ×1; hash-lánc rendben |
| 4a | szimulált Claude-limit (a ma éjjeli valódi válasz mása) | `t20261104-065732-c6a40d` | lehűlés 2026-11-05 04:21-ig, munkacsomag mentve, **átadás a valódi Copilotnak**, accepted (7,9 kredit) |
| 4b | egyik modell sem elérhető | `t20261104-065808-d03664` | független `leltar` lépés lefutott, a többi `blocked`, **waiting_model**; Copilot visszatérése után automatikus folytatás, a kész lépés nem ismétlődött, accepted |
| 5 | ugyanaz a spec / ugyanaz a kulcs kétszer | `…-8f91c6`, `…-0b7d4d` | ugyanaz az azonosító, 0 új kísérlet |
| 6 | „KÉSZ” + exit 0, de a hiba megmaradt | `t20261104-065406-21ac3c` | **rejected**; `approve` ebben az állapotban tiltott |
| + | párhuzamosság (max 2), leállítás, időkorlát, költségkeret | `…-97ad53`, `…-c61237`, `…-159e54`, `…-8b5c3a` | 3. feladat sorban várt; cancel → `cancelled`, árva folyamat nélkül; 3 s-os korlát → exit 124 → failed; 0 kredit → waiting_human |
| + | `ugy` → psx → ügyirat | `t20261104-070147-2b1210` | elfogadva, hook frissítette az ügyiratot; dupla beküldés elutasítva |

Naplók: `~/work/psx-test/home/tasks/<id>/{events.jsonl,result.json,attempts/}`; összesítő: `~/work/psx-test/osszesito.txt`.
Megismétlés: `source executor/tests/env.sh; start_sup; $PSX submit executor/tests/spec-p1.json` (a `$HOME` a spec-ekben kifejtendő).

## 5. Bizonyítatlan / korlátok
- **OpenCode-adapter** nincs megírva (szolgáltatói belépés kell). A Copilot `--allow-tool` lista szűk; nagyobb munkához lépésenként bővíthető (`copilot_tools`).
- Valódi (nem szimulált) Copilot-kvótakimerülés szövegét nem láttuk; a felismerés általános mintára épül (402/429/quota).
- A Claude `--permission-mode auto`; a munkakönyvtáron kívüli írást a modell utasítása és a NoNewPrivileges korlátozza, nem sandbox.
- Hosszú (napos) futás, gépleállás (reboot) utáni folytatás: a mechanizmus ugyanaz (felügyelő → `interrupted`), de rebootot nem próbáltunk.
- Az MCP-szerver (`mcp/server.py`, GPT-kapcsolat – félretéve) még a régi `jobs.py`-t használja; átállítása a következő lépés.
- A CM-szerű ütemezés (ismétlődő célok) nincs; a felügyelő csak beküldött feladatokat futtat.

## 6. Használat
```
psx submit spec.json            # → task_id   (vagy: … | psx submit -)
psx status [id] | psx log id | psx result id
psx cancel id | psx resume id | psx approve id | psx reject id "ok"
psx providers [--clear claude]  # elérhetőség, lehűlés
ugy inditas <ügy> "<feladat>" --elvart "…" --hova munka/x --ellenorzes "parancs:…"   # psx-en át
systemctl --user status project-s-executor ; journalctl --user -u project-s-executor -n 50
```

## 7. Saját ügynöki ciklus: OpenRouter → eszközhasználat → psx-ellenőrzés (2026-11-04)
Cél: kisebb feladatok Claude Code és Copilot nélkül. Kód: `executor/modell.py` (közös modellfelület),
`executor/agent.py` (ügynöki ciklus), `executor/modellek.json` (profilok, titok nélkül); psx-adapterek: `or-free`, `local`, `proba`.

### Felmérés (csak a szükséges részek)
| Korábbi komponens (project-s-unified @ 65fb73a) | Mit adott | Mit vettünk át |
|---|---|---|
| `src/integrations/ai_models/multi_model_ai_client.py` | OpenAI-kompatibilis `base_url`-hívás (OpenRouter), szolgáltatói konfig (openai/anthropic/ollama/openrouter), feladattípus → modell, fallback; **eszközhívás nincs**; kulcs env-ből vagy repóbeli fájlból | a szolgáltató/végpont/modell konfigurálhatóságát és a fallback-listát; a kulcs csak env vagy `~/.config/project-s/openrouter.env` (600) – repóbeli fájlból soha |
| `config/config/multi_model_config.json` | profilok és `task_mappings` | a profil-elvet (`free`, `local`, `proba`) |
| `tools/tools/registry/smart_orchestrator.py` | előre rögzített eszközláncok, `_execute_tool_safely` | a „biztonságos eszközfuttató burok” elvet; a modell itt maga választ eszközt |
| `src/core/meta/meta_controller.py` | döntés → végrehajtás → validálás → újratervezés, `max_iters` | a ciklust: eszközeredmény/teszthiba vissza a modellnek, `max_fordulo`; a validálás a psx szabálya |
| `continuous_mind.py`, Continue → Bridge | goal queue, kimenet-átvétel checksummal | a psx már átvette (feladatsor, állapot, napló); második feladatsor nincs |

### Működés
- **Közös modellfelület** (`modell.Profil.chat`): OpenAI-kompatibilis `/chat/completions` + `tools`; szolgáltató, végpont,
  kulcs helye és modellista a `modellek.json`-ban. Helyi kiszolgáló (Ollama, `http://127.0.0.1:11434/v1`) ugyanígy, adapterként:
  csak a profil kell, a feladatkezelés nem változik. Helyi modellt nem töltöttünk le.
- **Ingyenes profil** (`free`): `google/gemma-4-31b-it:free`, `nvidia/nemotron-3-ultra-550b-a55b:free`, `cohere/north-mini-code:free`,
  `nvidia/nemotron-3.5-lightning:free` – az OpenRouter `/api/v1/models` szerint (2026-11-04) mind `:free`, `tools` és `tool_choice`
  támogatással. Nem-`:free` modell a profilba be sem tölthető, és a kliens is elutasítja → **nincs automatikus fizetős váltás**.
- **Hibakezelés:** fiókszintű kvóta (429 `rate_limit_exceeded`, `X-RateLimit-Reset`) → `limited` → psx `waiting_model` + lehűlés
  (nem terheli a többi ingyenes modellt); szolgáltatói 429 / 5xx / időtúllépés / nincs eszköztámogatás → naplózott modellváltás a listán;
  hibás/üres válasz és hibás eszközhívás → hibaüzenet a modellnek, ismétlődés (3×) → leállás.
- **Eszközök:** `list_dir`, `read_file`, `search`, `write_file`, `edit_file` (pontos, egyedi csere), `run` (engedélyezett programok
  shell nélkül: python3, pytest, grep, ls, cat, head, tail, wc, sort, uniq, diff, git status/diff/log/show; metakarakter, abszolút és
  kifelé mutató útvonal tiltva; titokmentes környezet; időkorlát). Útvonalak a munkakönyvtárba zárva, `.git` belső fájlok tiltva.
  A modell válasza sosem kerül eval/exec-re.
- **Folytonosság:** `tasks/<id>/agent/<lépés>/turns.jsonl` (append-only): minden látható üzenet, kérés (modell, hiba, usage),
  eszközhívás, eredmény, fájlváltozás (hash előtte/utána). Módosító eszköz előtt **szándékrekord**; megszakadás után: már megtörtént
  → nem ismétli; nem történt meg (hash = előtte) → ellenőrzötten újra; eltérő állapot → nem írja felül; parancs → nem ismétli, jelzi.
  A titkok (kulcs, `sk-or-v1-…`, `gh*_…`, Bearer) minden mentett és a modellnek küldött szövegből kitakarva.
- **Korlátok:** `max_fordulo` (alap 30, lépésenként felülírható), időkeret (psx), `max_kontextus_karakter` (régi eszközkimenetek
  levágása, a napló teljes marad), ismétlődő hiba limit (3); a psx-korlátok (kísérlet, időtúllépés, párhuzamosság) változatlanok.
- **Elfogadás:** kizárólag a psx-szabály; a modell „kész” válasza a lépést csak végrehajtottá teszi.

### Próbák
**Szimulált** (OpenAI-kompatibilis próba-végpont: `executor/tests/mock_openai.py` – forgatókönyv szerinti válaszok, NEM valódi modell;
izolált `PSX_HOME=~/work/psx-or-test/home`, forgatókönyvek: `~/work/psx-or-test/sc/`):

| # | Próba | Feladat | Eredmény |
|---|---|---|---|
| SZ1 | hibajavítás: read_file → edit_file → run unittest → összefoglaló | `t20261104-075114-a75039` | accepted; fordulónként rögzítve (4 kérés) |
| SZ2 | szolgáltatói 429 → modellváltás; hibás JSON-argumentum; tiltott `rm -rf .`; kitörés `../../.config/project-s/oauth.env` | `t20261104-075146-f0985a` | mind elutasított eszközeredmény, mellékhatás nélkül; utána accepted |
| SZ3 | fiókszintű kvóta → `waiting_model` (1 kérés, a 2. modellt nem terheli) → lehűlés vége → folytatás a mentett beszélgetésből | `t20261104-075238-4befa3` | accepted (2. kísérlet, `folytatas` jel) |
| SZ4 | SIGKILL a modellkérés közben, a szerkesztés után | `t20261104-075300-221e8b` | 2. kísérlet; `edit_file` pontosan 1×; accepted |
| SZ5 | rossz javítás + „minden teszt zöld” állítás | `t20261104-075215-b5cf76` | **rejected** (a független teszt bukott) |
| SZ6 | naplóelemzés → `eredmeny.json`; elfogadás: független újraszámolás (`ellenoriz.py`) | `t20261104-075219-417b30` | accepted |
| SZ7 | 3× azonos hibás eszközhívás | `t20261104-075418-0507e0` | failed („ismétlődő eszközhiba”), nem végtelen ciklus |
| U1–7 | egységtesztek: félbemaradt szándék 4 ága, titokszűrés, eszközkorlátok, ingyenes→fizetős tiltás | `executor/tests/test_agent_helyreallitas.py` | 7/7 OK |

**Valódi szolgáltatói futás (OpenRouter): BLOKKOLVA – nincs OpenRouter-kulcs** a környezetben és a kijelölt helyen
(`~/.config/project-s/openrouter.env`). A modellválasztás valódi adatból (nyilvános modellista) történt; a valódi próbák
(`executor/tests/or-valodi-proba.sh`: hibajavítás + naplóelemzés, csak `or-free`) az éles psx-ben `waiting_model` állapotban várnak,
és a kulcs elhelyezése után automatikusan lefutnak.

### Bizonyítatlan
- Valódi OpenRouter-modell eszközhasználati minősége (melyik ingyenes modell oldja meg ténylegesen a feladatokat), a valódi
  hibaüzenet-formátumok (a felismerés a hivatalos dokumentáció szerint készült), a napi 50 kéréses keret gyakorlati elég-e.
- Ollama/helyi modell: a felület kész, a végpont nem fut, modell nincs letöltve.
- Valódi időzítésű megszakadás pont a szándék és az eredmény között (csak egységtesztben).

### Használat
```
umask 077; echo 'OPENROUTER_API_KEY=…' > ~/.config/project-s/openrouter.env      # toti, egyszer
psx providers                                   # or-free elérhető?
echo '{"title":"…","workdir":{"mode":"worktree","source":"/út/repo"},
  "steps":[{"id":"munka","kind":"agent","providers":["or-free"],"max_fordulo":15,"goal":"…"}],
  "acceptance":["parancs:python3 -m unittest"]}' | psx submit -
psx status <id>;  python3 executor/tests/fordulok.py ~/.local/state/project-s/psx/tasks/<id>/agent/munka/turns.jsonl
```

## 8. Valódi modellhasználat igazolása (2026-11-04, 08:15–08:45 UTC)
**Hozzáférés:** `~/.config/project-s/openrouter.env` (600, `sagent`), `OPENROUTER_API_KEY`; a psx-szolgáltatás beolvassa
(`psx providers` → `or-free`/`or-olcso` elérhető). `/api/v1/key`: érvényes, `is_free_tier: false` (a fiók vásárolt már kreditet),
nincs kulcsszintű limit. **A napi ingyenes keret pontos értéke nem ismert** (a dokumentáció szerint 50 vagy 1000 kérés/nap a vásárolt
kredittől függően; nem mértük).

**Modellválasztás élő adatból:** 2026-11-04-én 6 eszközképes `:free` modell volt; a két korábban kiválasztott NVIDIA-modell
**már nem** szerepelt köztük. Ingyenes profil: `google/gemma-4-31b-it:free`, `cohere/north-mini-code:free`,
`google/gemma-4-26b-a4b-it:free` (endpoint status 0, tools, ár 0); a ciklus minden indításkor élőben újraszűr.
Olcsó fizetős profil (toti engedélyével, 2026-11-04): `deepseek/deepseek-v4-flash` (0,091 / 0,182 $/M token), árplafon
0,5 / 1,0 $/M, **csak kifejezett `or-olcso` szolgáltatóval és kötelező `limits.max_cost_usd`-vel**; az ingyenes profil sosem vált ide.

### Valódi futások (mérés: `executor/tests/meres.py`, nyers adat: `~/work/or-valodi-meres.json`)
| Feladat | Profil / munkát végző modell | Útvonal | Eredmény | Független ellenőrzés | Kérés (sikeres/összes) | Hibás eszközhívás | Token be/ki | Költség | Idő |
|---|---|---|---|---|---|---|---|---|---|
| `t20261104-075546-d3e005` add() javítása | or-free / `cohere/north-mini-code:free` | Cohere | **accepted** (3. kísérlet) | unittest ✅, hibás sor eltűnt ✅, tesztfájl érintetlen ✅ | 9 / 14 | 0 | 9 840 / 63 589 | 0 USD | modell 493 s, fal 699 s |
| `t20261104-075546-196a24` naplóelemzés | or-free / `cohere/north-mini-code:free` | Cohere | **failed** (6/6 kísérlet, keret elfogyott) | nem futott (nincs végrehajtott eredmény) | 11 / 21 | 4 | 68 936 / 1 495 | 0 USD | modell 24 s, fal 1467 s |
| `t20261104-083331-24f264` naplóelemzés | or-olcso / `deepseek/deepseek-v4-flash` | SiliconFlow | **accepted** (1. kísérlet) | JSON-kulcsok ✅, független újraszámolás (`ellenoriz.py`) ✅ | 7 / 7 | 0 | 71 563 / 2 646 | **0,00443 USD** | modell 40 s, fal 39 s |

Szolgáltatói hibák a valódi futásokban: Gemma 4 (31B és 26B) ismételten *„temporarily rate-limited upstream”* (Google AI Studio, 429)
→ naplózott modellváltás; a már nem ingyenes NVIDIA-modell (régi listáról, az élő szűrés bevezetése előtt): *„0 endpoints … guardrail
restrictions and data policy … Free model training violation (account settings)”* – fiókbeállítás, nem módosítottuk.

**A valódi futások által feltárt és javított hibák** (mind a naplóból, mind külön kísérletben, a psx kísérletkeretén belül):
1. a fiók adatvédelmi kizárása „other” hibának számított → háromszor ugyanaz a modell → új `policy` osztály, modellváltás;
2. a modell objektumot adott a `write_file.content`-be → elkapatlan kivétel, munkavégző-összeomlás → séma szerinti típusellenőrzés,
   eszközhiba sosem dönti le a munkavégzőt;
3. összeomlás után válasz nélküli eszközhívás maradt → a szolgáltató `TOOL_CALLS_MISSING_RESULTS`-szal elutasította → `normalizal()`;
4. a szolgáltatói 429-ek elfogyasztották a fordulókeretet → `max_fordulo` csak megválaszolt fordulót számol, külön `max_keres`;
5. a naplóelemzés 6. kísérletében a modell háromszor csővezetékes shell-parancsot próbált (`… | …`), amit a biztonsági szabály helyesen
   elutasított → ismétlődő hiba, leállás. A `run` eszköz leírása pontosítva (egy parancs, csővezeték nélkül, `grep -c`); **ez erre a
   feladatra nem igazolt**, mert a kísérletkeret elfogyott – nem próbálkoztunk tovább.

Értelmezés: ez **első integrációs bizonyíték**, nem modellmegbízhatósági mérés. Az ingyenes út egy egyszerű javítást elvégzett,
egy összetettebbet nem; a terheltség (upstream 429) jelentősen lassította. Az olcsó DeepSeek út egy kísérletben, 0,004 USD-ért teljesített.

### Kulcsszivárgás (project-s-unified, nyilvános repó)
Helyi ellenőrzés (érték kiírása nélkül): a `docs/reference/openrouter_api_key.py` valódi formátumú OpenRouter-kulcsot tartalmazott;
a HEAD-ben 5 fájlban volt kulcsformátumú érték, az elérhető előzményben 3 különböző. Javítás (`8fe850a`, `ace7280`): értékek helyőrzőre,
a kulcsfájl csak környezeti változóból olvas, `config/openrouter.env.example`, `scripts/check_no_secrets.py`, `key-scan` CI.
**Az előzmény nincs átírva – a régi kulcsokat az OpenRouteren vissza kell vonni.** A workshop-repóban: `scripts/titok_ellenor.py`
+ pre-commit hook (hamis mintakulccsal kipróbálva: a commit megáll).

## 9. Claude-tól független elérés: MCP → psx és SSH (2026-11-04)
### MCP (ChatGPT connector)
- Cím: **`https://<tailnet-gép>/mcp`** (Tailscale Funnel → `project-s-mcp.service`, user unit, linger:
  nem függ Claude-munkamenettől). Hitelesítés: OAuth (dinamikus kliensregisztráció → saját jóváhagyó oldal → GitHub `toti85`).
- Eszközök (mind ugyanazt a psx-állományt használja, nincs külön feladatsor): `task_submit` (azonnali azonosító; `client_key`
  vagy tartalom-hash → ismételt beküldés nem indít dupla feladatot), `task_list`, `task_status`, `task_log` (lapozott),
  `task_result` (lapozott, ≤6000 karakter), `task_cancel`, `task_decide` (approve/reject), `workspace_read`/`workspace_write`.
  Alapmodell: `deepseek` = `deepseek/deepseek-v4-flash` (`or-olcso`, végpontszintű árplafon 0,5/1,0 $/M, keret 0,20 USD);
  `ingyenes` = csak `:free`; `claude`/`copilot` csak kifejezett választással.
- **Mobilos hiba vizsgálata (szervernapló 2026-11-03 22:00 – 11-04 08:55):** egyetlen `/register`, `/authorize` vagy
  `/token` hívás sem érkezett – a ChatGPT sosem kezdte el az OAuth-folyamatot. Egy külső kliens 22:20-kor csak a metaadatokat
  kérte le. A szerveroldali láncot végigpróbáltuk: DCR 201, authorize → jóváhagyó oldal 200, „Allow” → GitHub authorize helyes
  `redirect_uri`-vel, a GitHub OAuth App adatai érvényesek (`bad_verification_code` próbakóddal). Következtetés: a hiba kliensoldali
  (a connector nem jött létre / nem futott le a hitelesítés); ráadásul a régi eszköznevek (`task_start`) megszűntek, ezért egy
  korábban felvett connectort **frissíteni/újra kell csatlakoztatni**.
- Szerveroldali végpont-próba (helyi MCP-kliens, statikus token, éles psx): beküldés 0,07 s → `t20261104-085253-f303a1`
  (DeepSeek, 0,0012 USD, accepted), azonos `client_key` → ugyanaz az azonosító, napló lapozva, leállítás → `cancelled`
  (`t20261104-085313-fea957`). Script: `mcp/mobil_proba.py`. **A telefonos ChatGPT-ből indított hívás még nincs igazolva.**
### SSH (telefon, Tailscale)
- Gép: `toti-server` – Tailscale `<tailnet-ip>` / `<tailnet-gép>`, LAN `<helyi-ip>`, port 22.
- Felhasználó: `toti` (jelszó vagy kulcs; a telefon a tailnetben: `redmi-note-13-pro`). `psx` és `ugy` minden felhasználónak:
  `/usr/local/bin/{psx,ugy}` → `sudo -iu sagent …` (a `toti` sudo-jelszavát kéri).
- Parancsok: `psx status` · `psx status <id>` · `psx log <id> --n 20` · `psx result <id>` · `psx cancel <id>` ·
  `psx approve <id>` · `psx retry <id> "ok"` · `psx providers` · feladat: `echo '<spec-json>' | psx submit -` ·
  ügy: `ugy lista`, `ugy inditas <ügy> "<feladat>" --elvart "…" --ellenorzes "…"`.

### Mobil: ChatGPT-korlát és a tailnet-panel (2026-11-04)
- toti telefonos képernyője (2026-11-04): a ChatGPT-ben felvett **Project-S plugin** (agent-plugins / `.codex-plugin` csomag,
  távoli `streamable-http` MCP + OAuth DCR – a csomag tartalma helyes) „**Használat számítógépen**” módú, mobilon nem választható.
  A szervernaplóban a telepítés óta 0 ChatGPT-hívás. Az OpenAI szerint a developer mode / egyedi MCP-app a ChatGPT webes felületén
  érhető el; közösségi beszámoló szerint mobilon az argumentumos (író) MCP-eszközök tiltottak. → **szerveroldalról nem javítható.**
- **Claude- és ChatGPT-független telefonos út:** `https://<tailnet-gép>:8443` – CSAK tailneten (`tailscale serve`,
  nem Funnel; internet felől elérhetetlen, ellenőrizve). `project-s-panel.service` (127.0.0.1:8820), hitelesítés a Tailscale-identitással
  (`<e-mail>`), CSRF-token + Origin, idempotens beküldés. Funkciók: lista, beküldés (DeepSeek V4 Flash kerettel / ingyenes),
  állapot, napló (lapozva), összefoglaló, leállítás, elfogadás/elutasítás, folytatás. Ugyanaz a psx-állomány.
- Szerveroldali próba: 403 fejléc nélkül/idegen fiókkal, 400 CSRF nélkül, kétszeri beküldés → ugyanaz a feladat
  (`t20261104-090829-5b4648`, DeepSeek, 0,0007 USD, accepted); tailnet-IP-n 200 valódi identitással.
- **Telefonról IGAZOLVA (toti, 2026-11-04):** `t20261104-110700-037631` – beküldés a panelen 11:07 UTC → DeepSeek V4 Flash (6 kérés, 0,0009 USD) → fájl/szabály/próba-ellenőrzés a panelen → elfogadás 12:16 UTC a panelen. Ügyhöz kötött beküldés: `t20261104-125111-a70aa2` (0,034 USD, jelentés az ügyiratban). Javítás a telefonos napló alapján: a „Leállítás” csak futó/várakozó munkánál (psx `CANCELLABLE`), döntésre váró munkánál elfogadás/elutasítás.

## 10. Ügyek a panelen és ügyfolytatás (2026-11-04)
- **Panel → ügyek:** `/u` lista (állapot, cél, következő lépés, futó feladat), `/u/<ügy>` ügyoldal (allapot.json összefoglaló,
  feladatok és ítéletek, eredményfájlok olvasható előnézettel és letöltéssel `/u/<ügy>/f?p=…&dl=1`), **Folytatás** űrlap
  természetes nyelvű céllal (`/u/<ügy>/folytat` → `ugy inditas …` ugyanazzal a psx-szel, nincs külön feladatsor).
- **Ügyelőzmény automatikusan:** a prompt tartalmazza a cél/következő lépés mezőket, a legutóbbi eredményfájlokat, a munkaanyagok
  listáját és az ügy `olvashato` gyökereit (allapot.json; az ügynök ezeket csak olvashatja, írni csak az ügymappába tud).
- **Kötelező jelentésszerkezet** (panelről indítva mindig): `## Elkészült` / `## Ellenőrizve` / `## Bizonytalan` /
  `## Következő lépés` – mind a négy gépi `tartalmaz:` szabály; a tartalom megítélése emberi (`elfogadasra_var`).
- **Fordulókeret:** ügyfeladatnál 60 (`UGY_MAX_FORDULO`); 4 fordulóval a vége előtt „kerettel zárás”: az ügynök csak írhat, így a
  jelentés akkor is elkészül, ha az olvasás elhúzódik. Keret elérésekor az elfogadási szabályok döntenek.
- **Kérés-határidő:** az OpenRouter életben tartó szóközei miatt a socket-időtúllépés nem jár le – a teljes kérésre fali órás
  határidő van (`modell.py`); a beragadt 3. Plether-kör ezt mutatta meg (18 perc), javítás után a 2. kísérlet ellenőrzőpontról folytatta.
- **OS-szintű írási zár (2026-11-04, `executor/zar.py`, Linux Landlock):** az ügynök `run` eszközével indított minden folyamat
  (python3, pytest, git, … és minden leszármazottjuk) a kernelben zárolva fut: **írni csak a munkakönyvtárba** és a saját
  ideiglenes könyvtárába (`<állapot>/agent/<lépés>/tmp`, `TMPDIR`) tud; olvasni a rendszerkönyvtárakat, a munkakönyvtár git-tárolóját,
  az ügy `olvashato` gyökereit és az eszközláncokat (`~/.local/lib`, `~/.foundry*` …) – `~/.config` (kulcsok), `~/.ssh`,
  `~/.local/state` nem olvasható. Zár nélkül nem fut (126). Sudo, csomag és user namespace nem kell.
  Miért nem systemd: a `--user` unitok `ProtectSystem`/`ReadWritePaths` beállítását itt az AppArmor userns-korlátja miatt
  a systemd **csendben figyelmen kívül hagyja** (próbával igazolva: a kinti írás sikerült).
  Próba: unit-teszt `test_os_zar_kivul_nem_ir_belul_igen` (valódi kernelzár: kinti írás, gyerek shell, titokfájl → tiltva;
  bent írás, tempfile, olvasás → működik) és éles psx-feladat `t20261104-154001-0d72ab` (DeepSeek, 0,003 USD, accepted):
  `python3 -c open('~/zar-kint.txt')` → PermissionError, gyerekfolyamat `touch ~/work/…` → Permission denied, bent → `bent-ok`.
  **Nem fedi:** a `cmd` lépéseket (a feladat szerzője írja, nem modell), az elfogadási `parancs:` szabályokat, és a `claude`/`copilot`
  adaptereket (saját jogosultságkezelés; csak kifejezett választással futnak).

## 11. Bizonyítékréteg, független ellenőr, böngésző, Codex (2026-11-05)
- **Webeszközök a saját ciklusban:** `web_search` (DuckDuckGo HTML, kulcs nélkül) és `web_fetch` (urllib, vagy `browser=true`:
  Playwright + Chromium fej nélkül, `executor/bongeszo.py`, venv: `~/.local/opt/bongeszo-venv`). Minden letöltés
  **pillanatkép** (`executor/web.py`): `<id>.json` (url, végső url, idő, HTTP, sha256), `<id>.txt`, böngészőnél `<id>.png`;
  helye a feladat állapotmappája (`…/agent/<lépés>/bizonyitek`) – az ügynök eszközei ezt nem írhatják (teszt).
- **Idézet-ellenőrzés (`idezetek:<fájl>[:min]`):** a jelentés `[[idézet: "…" | URL]]` jelöléseit az elfogadás maga, függetlenül
  újra letölti (szükség esetén böngészővel) és normalizált szó szerinti egyezést keres; ami nem igazolható → `hianyos`.
  Végrehajtótól független: Claude, Copilot, Codex, DeepSeek jelentésére ugyanúgy fut. Eredmény: `<feladat>/bizonyitek/idezetek.json`.
- **Független ellenőr lépés:** `ugy inditas(..., ellenor=<szolgáltató>)` / panel jelölőnégyzet → 2. psx-lépés MÁSIK gyártó
  végrehajtójával (`ELLENOR_PAR`: DeepSeek↔Claude, Copilot/Codex→Claude), cáfolási szereppel; `…-ellenorzes.md` táblázat
  IGAZOLT/CÁFOLT/ELLENŐRIZHETETLEN ítélettel; elfogadás: `nem-tartalmaz: | CÁFOLT |` + az ellenőr idézetei is ellenőrizve.
- **Codex (5. végrehajtó):** `~/.local/opt/codex/bin/codex` 0.166.0 (hivatalos kiadás). A Codex saját bubblewrap-sandboxa az
  AppArmor userns-korlát miatt nem indul, ezért a teljes `codex exec` folyamat Landlock-zárban fut (írás: munkakönyvtár,
  `~/.codex`, ideiglenes mappa). Belépés (toti, egyszer): SSH-n `codex-belepes` → eszközkód a ChatGPT-fiókkal.
- Telepítve (2026-11-05): `bubblewrap`, `socat`, Playwright + Chromium és rendszerfüggőségei. **Nincs** módosítva: az AppArmor
  userns-korlát (a bubblewrap-profil hozzáadását a munkamenet biztonsági szűrője nem engedte – toti döntése).

## 12. Teljes jog – a titkok kivételével (2026-11-08, toti kérése)
- Alapértelmezés: `jog: "teljes"` minden ügynöki lépésre (`PSX_JOG`/lépésenként `"jog": "szuk"` visszaállítható).
- Saját ciklus (DeepSeek/ingyenes): bármilyen parancs, shell, csővezeték; fájleszközök a gépen bárhol. Codex és Copilot
  (`--allow-all-tools --allow-all-paths`, `git push`/`sudo` tiltva): a teljes folyamat `zar.py --teljes` alatt.
- `zar.py --teljes`: írás/olvasás/futtatás a / minden ágán és a home-ban, KIVÉVE `zar.TELJES_KIVETEL`: `~/.config` (kulcsok,
  OAuth, Telegram, gh), `~/.ssh`, `~/.gnupg`, `~/.local/share/project-s` (ATA-adat), `~/.codex`, `~/.claude*`, `~/.copilot`
  (fiókok; a saját végrehajtójuk kifejezetten megkapja), `~/.bash_history`, `~/mentes`, `~/.local/state/project-s`
  (psx-napló és bizonyíték – nem hamisítható). Sudo nincs (NoNewPrivileges). A home gyökerébe új fájl nem hozható létre.
- Miért nem titkokkal együtt: a webes tartalomból érkező prompt injection a modellt rávehetné a kulcsok kiküldésére –
  ez nem a kísérleti szervert, hanem toti fiókjait veszélyeztetné.
- Próba: unit-teszt `test_teljes_jog_titkok_nelkul`; éles: DeepSeek `t20261108-173858-1e5950`, Codex `t20261108-173622-297f69`,
  Copilot `t20261108-173623-4baaf0` – mind a munkakönyvtáron kívülre írt (`~/work/jogproba/`), elfogadva.
- Claude: továbbra sem zárolt (eddig is a teljes felhasználói jogokkal futott, saját engedélykezeléssel).

## 13. Jóváhagyásra váró rendszerlépés (root) (2026-11-08)
- Végrehajtó (bármelyik): `~/bin/rendszerlepes ker "<parancs>" --miert "<indoklás>"` (`--var N`: N másodpercig vár a döntésre).
  A kérés a `~/work/rendszerlepes/keresek/` alá kerül; a feladat-azonosító a `PSX_TASK_ID`-ből jön. A prompt minden
  végrehajtónak leírja, hogy sudo nincs, ne kerülje meg, hanem kérje így.
- toti: panel → **Rendszerlépések** (`/r`; a főoldalon zöld jelzés, ha van függő kérés): a pontos parancs, indoklás,
  feladat-hivatkozás; **Jóváhagyom és futtatom (root)** / **Elutasítom**. Jóváhagyáskor a panel `sudo -n bash -c` futtat
  (max. 15 perc); kimenet → `~/work/rendszerlepes/valaszok/<id>.json` (a végrehajtó olvassa) és a hiteles napló
  `~/.local/state/project-s/rendszerlepes/naplo.jsonl` (a végrehajtók zárja nem írja).
- Védelem: a jóváhagyás toti kattintása (CSRF + Tailscale-identitás), nem fájl → a végrehajtó nem hagyhatja jóvá magát;
  a futtatott parancs sha256-ja egyezik a panelen megjelenítettel, különben nem fut; egy kérés csak egyszer bírálható el.
- Próba (2026-11-08): zárolt végrehajtóból kért `id -u; whoami; …` → jóváhagyva → `0 / root`; elutasított
  `touch /root/nem-szabad` nem futott le; a megjelenítés után átírt kérés („echo HAMIS”) → „a parancs megváltozott – nem
  futtatom”; a naplót a zárolt folyamat nem tudta írni.

## 14. Beszélgető asszisztens a panelen (2026-11-08)
- Panel → **💬 Beszélgetés** (`/c`): szálak, buborékos csevegés, formázott válasz, automatikus frissítés amíg dolgozik.
  Szálanként választható „agy”, menet közben váltható (az új agy megkapja az eddigi beszélgetést):
  **Codex** (ChatGPT-előfizetés, `codex exec` / `exec resume`, Landlock-zárban), **Claude** (Pro, `claude -p --resume`),
  **DeepSeek** (OpenRouter, saját eszközhurok). Kód: `executor/beszelgetes.py`; útmutató: `docs/ASSZISZTENS.md`.
- Háttértudás minden körben/első körben: ASSZISZTENS.md, FOLYTATAS.md, KEPESSEGEK.md, `ugy lista`, az ügy UGY.md-je.
  Utánanézés: olvasó parancsok (psx/ugy/fájlok, a psx-állapot olvasható), web.
- Műveletek: az agy `ps-chat inditas|leallit|root-ker|root-jovahagy …`-ot hív → kérésfájl; a válasz UTÁN a megbízható
  feldolgozó hajtja végre (ügykezelő/psx), eredmény a beszélgetésbe. Indítás toti kérésére vagy rábólintására (útmutató);
  bármely végrehajtó indítható, a CodeWhale is. **Root-jóváhagyás csak akkor fut, ha toti utolsó MENTETT üzenete
  kifejezetten jóváhagyja, és a kérés után íródott** – az asszisztens állítása nem elég (teszt: `test_beszelgetes.py`).
- Tárolás csak a szerveren: `~/.local/state/project-s/beszelgetesek/<szál>/` (GitHubra nem kerül).
- Próba (2026-11-08): Codex helyes utánanézés (5 elfogadásra váró feladat, 18 s); váltás DeepSeekre – emlékezett;
  rábólintásra indított DeepSeek-feladat `t20261108-224923-d32b3c` elfogadva; root-kérés rögzítve; Claude-agy válaszolt.
- **Kódváltozás a beszélgetésben (2026-11-09):** ha egy válasz közben `executor/ scripts/ mcp/ ata/ install/` alatti kódfájl
  változik (commit vagy munkafa), automatikusan indul egy psx-feladat: a teljes tesztcsomag (`scripts/tesztek.sh`, elfogadási
  szabályként) + egy MÁSIK gyártó átnézése (Claude→Codex, Codex/DeepSeek→Claude) `kozos/kimeno/atnezes-…md` jelentéssel;
  `| HIBA |` sor → elutasítva; egyébként toti elfogad. Próba: szándékosan hibás `osszeg()` (a−b) → a Codex megtalálta,
  reprodukálta, a kapu elutasította (`t20261109-094044-626e0f`).

## Telefonos értesítések (Web Push)

A tailnet HTTPS-címéről telepített Android panel PWA-ban az **Értesítések bekapcsolása**
gombbal engedélyezd az értesítéseket. A gomb a `/sw.js` service workert gyökérscope-pal
regisztrálja, a PushManager-feliratkozást a panel meglévő, egyszer használatos CSRF-tokenjével
védett POST menti. Inline JavaScript nincs. Újratelepítés vagy visszavont engedély után
ismét kapcsold be; mentési hiba után töltsd újra az oldalt.

A `needs_acceptance`, `accepted`, `rejected`, `failed`, `waiting_human` állapotba váltás
háttérfolyamatban küld értesítést. Új rendszerlépés-kérésnél a panel 15 másodpercenkénti
figyelője is küld, ezért a kernelzárból rögzített kérés is eljut a telefonra.
A rendszerlépés értesítésének címe állandó; sem parancs, sem indoklás nem kerül bele.
Az értesítés kizárólag azonosítót, legfeljebb 60 karakteres feladatcímet, állapotot és
kattintási útvonalat tartalmaz (`/t/<id>`, rendszerlépésnél `/r`). Jelentés és ügyirat nincs benne;
a feladat címébe ezért ne írj érzékeny adatot.

- `ertesites allapot`: a kulcs létezése és a feliratkozások száma, titkok kiírása nélkül.
- `ertesites proba`: próba minden mentett feliratkozásra, kattintáskor a főoldal nyílik.
- A link: `~/bin/ertesites` → a repó `scripts/ertesites.py` fájlja.
- VAPID-kulcs: `~/.config/project-s/ertesites/vapid.pem`, 600-as jogosultság.
  Lusta létrehozás az első bekapcsoláskor/próbánál; a repóba soha nem kerül.
- Feliratkozások és rendszerlépés-értesítési jelölők: `~/.local/state/project-s/ertesites/`.
  Teszthez `ERTESITES_KULCS_DIR` és `ERTESITES_DIR` állítható, valódi titkok nélkül.
- A rendszer-python `cryptography` csomagját használja: [RFC 8291](https://www.rfc-editor.org/rfc/rfc8291)
  aes128gcm titkosítás és [RFC 8292](https://www.rfc-editor.org/rfc/rfc8292) VAPID ES256.
  Nincs új csomag vagy rootigény.

A küldési hiba nem állítja meg a végrehajtót. A 404/410 választ adó lejárt feliratkozást
a küldő törli; a hálózati műveletek időkorlátosak. A feladatértesítés best effort (nincs
újraküldési sor); a függő rendszerlépések sikertelen küldését a panel újrapróbálja.
A telefonos megérkezéshez hálózat, böngészőengedély és Androidon engedélyezett értesítések kellenek;
a tényleges megérkezést toti ellenőrzi az `ertesites proba` paranccsal.

## Külső jelentés ellenőrzése

A másolt URL-eket a közös `web.tisztit_url` a letöltés előtt tisztítja: visszaalakítja
az HTML-entitásokat, eltávolítja a láthatatlan vezérlő-, formázó- és cserekaraktereket,
levágja a záró idézőjeleket, írásjeleket és a pár nélküli zárójeleket. A hostot IDNA,
az útvonalat és lekérdezést százalékos kódolással alakítja ASCII-vá; a meglévő `%XX`
kódolás megmarad. Egy sor vagy bekezdés egymást követő idézetei közös, utánuk álló
URL-t kaphatnak, a `|` elválasztós és markdown-táblázatos bemenetben is.

A saját kódolási hiba, kivétel vagy váratlan eszközválasz ítélete **eszközhiba**.
Ez külön összesítő sort és a kimenet tetején darabszámos figyelmeztetést kap;
nem a forrás hibája. Az **ellenőrizhetetlen** a forrás elérhetetlenségét jelzi
(pl. 401/403/429/503, botvédelem, időtúllépés vagy hálózati hiba).

`ellenoriz-jelentes <fájl>` (a `scripts/ellenoriz_jelentes.py` parancs) létrehozza a
`<fájl>.ellenorzes.md` táblázatot és a `<fájl>.bizonyitek/` mappát. Felismeri a
`[[idézet: "szöveg" | URL]]` jelölést, a magyar/görbe/egyenes idézőjeles szöveg után
álló markdown-linket vagy ugyanabban a mondatban következő URL-t, valamint a puszta
markdown- és HTTP(S)-hivatkozásokat. URL nélkül az idézet „forrás nélküli”.
A letöltések az eddigi `executor/web.py` pillanatképeivel készülnek; tételenként
`eredmeny.json` őrzi az ítéletet és a metaadatokat, a `B*.json` és `B*.txt` a letöltést.
A linkeknél HTTP-kód, átirányítás utáni URL, oldalcím és DNS-hiba is megjelenik;
átmeneti DNS-hiba vagy botvédelem nem bizonyít halott linket.

A panel főoldalán a **Jelentés ellenőrzése** oldalon (`/ellenorzes`) a meglévő
Feltöltés oldalon `kozos/bejovo` alá feltöltött szövegfájl választható, vagy szöveg
illeszthető be. A `systemd-run --user` háttérfolyamat eredménye automatikusan a
panel dokumentumnézetében nyílik meg, a `kozos/eredmenyek/ellenorzes/` mappában marad.
Ez csak eszköz, a feladatbeküldést nem korlátozza.

A jelenlét gépi tény (szó szerinti egyezés a letöltött oldalon); az alátámasztás egy modell
véleménye, nem bizonyíték. A jelenlét ítélete változatlan; az új **alátámasztás (modell
véleménye)** oszlop és külön összesítő csak az „igazolt” idézeteket értékeli. Értékei:
„igen”, „részben”, „nem”, „nem eldönthető”, egy mondat indoklással és modellnévvel.
Más tételnél és kikapcsolva „–”. Az állítás az idézet sora/táblasora az idézetek és
forráshivatkozások nélkül; ha nem nyerhető ki, az eredmény „nem eldönthető”.

Kapcsolók: `--alatamasztas be|ki` (alap: be, a panelen is),
`--alatamasztas-modell deepseek|claude` (alap: deepseek), `--max-usd 0.05`.
A DeepSeek a közös `Profil("olcso")` árkorlátos OpenRouter-felületét használja,
a Claude a `claude -p` parancsot, kérésenként a maradék költségkerettel.
Csak az állítás, az idézet és az elmentett pillanatkép idézet körüli legfeljebb
1500 karakteres részlete kerül a modellhez. A JSON-válasz szigorúan ellenőrzött;
hibás válasznál nincs pozitív vélemény. Kérés előtt konzervatív tokenköltség-foglalás
készül; elégtelen keretnél a maradék „nem eldönthető (keret)”. Hiányzó fogyasztási
adatnál a foglalás számít elköltöttnek; feldolgozási/hívási hibánál a keret lezárul.

A `python3 scripts/alatamasztas_proba.py` helyi HTTP-szerveren futtatja az A1–A3
valódi alátámasztó és F1–F3 más témájú eseteket valódi modellel, és JSON-t ír:
`{"A": [...], "F": [...], "usd": ...}`. A régi A-esetek külön állítás nélküliek,
ezért a próba az idézet tartalmát állítja melléjük; az F-állítás változatlan.
Az egységtesztek ezt mockolt modellel futtatják, külső hálózat és modell nélkül.
Kernelzár alatt az élő próba nem futtatható, azt záron kívül kell ellenőrizni.
Külső jelentésnél nincs eredeti pillanatkép: a megváltozott vagy elérhetetlen oldal
nem hamis; az elérhető oldalon hiányzó idézet „nincs az oldalon (most)”.
