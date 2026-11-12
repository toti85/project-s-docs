<!-- Automatikus tükör a privát Projekt S műhelyrepóból (scripts/docs_tukor.py). Személyes és hálózati adatok kitakarva. Ne itt szerkeszd. -->

# Összevetés: régi Projekt S Master Blueprint (2026-08-12) ⇄ mai műhely (toti-server, 2026-11-12)

_Feladat: `t20261112-075002-bc0419` (ügy `project-s-ujjaepites`). Készítette: Claude (psx-végrehajtó), 2026-11-12._

**Kinek szól:** olyan olvasónak, aki a repót nem látja. A **régi terv** a `PROJECT_S_MASTER_SYSTEM_BLUEPRINT_V1`
(2026-08-12, „NORMATIVE – BLUEPRINT_LOCKED”). Egy Windows-alapú, sok rétegű rendszer (Continuous Mind, Interactive Layer,
SAI) átalakítását írta le egy egyetlen igazságforrásra épülő magra: Continuity Journal (append-only, hash-láncolt napló) →
Lifecycle Engine (állapotátmenetek jogszerűsége) → Mission Contract (előre rögzített elvárás) → kétlépcsős governance
(MAY_EXECUTE / MAY_ACCEPT) → eredményátvétel, artifact-ellenőrzés, kimenetel, elfogadás. A beillesztett szöveg a 19.
szakasznál (Stage 1) **csonka**. A 2026-09-15-i független review mondta meg, mit kell megtartani, egyszerűsíteni,
lecserélni vagy elhagyni (KEEP/SIMPLIFY/REPLACE/REMOVE). Az augusztusi „canonical” kód elveszett (sem a GitHubon, sem a Data3
kötetén nincs meg, lásd az ügyiratot). A **mai műhely** ezért újraépítés: a szerveren futó `ugy` ügykezelő + `psx`
végrehajtó + elfogadási kapu + panel/MCP.

Besorolások: **megvan** · **részben** · **hiányzik, de kellene** · **nem kell** · **nem igazolt** (a fájlokból nem
bizonyítható).

---

## 1. Összefoglaló (hol tart az újjáépítés)

1. A régi terv **végrehajtási igazság-magja feladatszinten megvan**: a feladatonkénti `events.jsonl` append-only és
   hash-láncolt. Az állapot ebből determinisztikus redukcióval számolódik (`executor/psx.py` `reduce()`), a lánc
   ellenőrizhető (`psx verify-journal`). Kísérletek, idempotens beküldés, költségkeret és leállítás is van.
2. A **„végrehajtva ≠ elfogadva”** elv (blueprint 2.3–2.4, 8.1) **működik**. Előre rögzített gépi szabályok
   (`executor/acceptance.py`) és külön emberi döntés (`psx approve|reject`, `ugy elfogad|elutasit`) dönt; a végrehajtó
   önbevallása nem számít. Ez a régi terv legértékesebb elve, és a mai rendszer gerince.
3. **Részben** van meg a Mission Contract (`--elvart/--hova/--ellenorzes` + a spec hash-e a beküldéskor). Hiányzik
   belőle: a hatásosztály, a hash-alapú artifact-azonosság és a szerződés verziója.
4. **Hiányzik az ügyszintű napló.** Az `ugyek/<ügy>/allapot.json` felülírt pillanatkép, nem eseménynaplóból épül újra.
   A blueprint 2.1/10.2 elve tehát csak a feladatokra teljesül, az ügyekre nem. Ugyanígy hiányzik a reducerbe épített
   átmenet-jogszerűség-ellenőrzés: ma a CLI ellenőriz, a `reduce()` nem utasít el illegális eseményt.
5. **Hiányzik a végrehajtás előtti kockázati besorolás** (ALLOW/PAUSE/DENY). Ehelyett OS-szintű zár (Landlock) van,
   valamint a root-lépés emberi jóváhagyása (`rendszerlepes`). Toti „nincs visszafelé korlátozás” döntése miatt teljes
   kockázati kaput nem javaslok.
6. Az „intelligencia”-rétegek (SAI, StrategyCouncil, MetaController, modellrouter, parallel loop, taste model) **nem
   kellenek**, ezt a 09-15 review és toti döntése is így rögzíti. Helyettük előre megadott végrehajtó-lista és limitnél
   átadás működik.
7. A mai rendszer **túlment a terven** a bizonyítékrétegben: szó szerinti idézetek független újraletöltése, másik gyártó
   modellje mint ellenőr. A tervben ez csak elvként szerepelt (review: „Evidence Engine elve: KEEP”).

---

## 2. Tételes táblázat

Jelölések: `psx.py` = `executor/psx.py`; `acc.py` = `executor/acceptance.py`; `ugy.py` = `scripts/ugy.py`;
BP = blueprint szakasz; R = 09-15 review 4. szakasz (KEEP/SIMPLIFY/REPLACE/REMOVE táblázat).

### 2.1 Alapelvek (BP 1–2)

| Elem | Régi terv | Mai állapot | Bizonyíték | Besorolás |
|---|---|---|---|---|
| Nincs zöldmezős újraírás, strangler-migráció | BP 1 | A régi kód elveszett, ezért kényszerűségből újraírás készült; a régi kódból csak minták kerültek át | `UGY.md` „A legfontosabb tény…”, „Data3-vizsgálat” | nem kell (a régi mag nem elérhető, nincs mit fojtogatni) |
| Lifecycle Commit Doctrine: csak a naplóba írt átmenet számít | BP 2.1 | Feladatszinten igen: minden állapotváltás `Task.emit()` → `events.jsonl` (fsync), az állapot a `reduce()`-ból jön | `psx.py` 110–141, 144–213 sor | **megvan** (feladatszinten) |
| Nincs független, hiteles állapot-pillanatkép | BP 2.1, 10.2 | Feladat: a `result.json` generált nézet (`write_view`). Ügy: az `allapot.json` **felülírt, hiteles forrás**, nem naplóból épül | `psx.py` 216–227; `ugy.py` 87–98 (`allapot.json.tmp` → replace) | **részben** (ügyszinten hiányzik) |
| Tekintélyi hierarchia (napló > engine > projekció > többi) | BP 2.2 | Feladatnál napló > `reduce()` > `result.json`/panel. Ügynél `allapot.json` + `UGY.md` + git | ugyanott | részben |
| Kétlépcsős governance (MAY_EXECUTE ≠ MAY_ACCEPT) | BP 2.3, 9 | MAY_ACCEPT megvan (gépi szabály + `ember`). MAY_EXECUTE nincs külön döntésként: a beküldés = futtatás, a korlátot a zár és a keret adja | `acc.py` `futtat`/`elbiral`; `psx.py` `submit()` 844–869 | **részben** |
| DONE+ALLOW+VERIFIED ≠ elfogadás | BP 2.4 | Az `executed` esemény csak `verifying` állapotot ad; elfogadás csak `verified` (gépi) vagy `human_decision` után | `psx.py` 180–188; `CLAUDE.md` „Végrehajtva ≠ elfogadva” | **megvan** |

### 2.2 Komponensek (BP 4–5)

| Elem | Régi terv | Mai állapot | Bizonyíték | Besorolás |
|---|---|---|---|---|
| Continuity Journal (append-only, hash-lánc, egy író) | BP 10.1; R: KEEP | Feladatonként: `seq`, `prev`, `hash` = sha256(prev + kanonikus esemény), írás `flock` alatt | `psx.py` 110–120; ellenőrzés `psx verify-journal` – 3 véletlen 11-11-i feladaton és a jelenlegin „hash-lánc rendben” (2026-11-12) | **megvan** (feladatszinten) |
| Rendszerszintű/ügyszintű napló | BP 10.1; R: „bővítés célokra, döntésekre” | Nincs. Az ügy története `UGY.md` napló + git-commitok (hook) | `ugy.py` 87–98; `RENDSZER.md` 11. „ügyenkénti eseménynapló” nyitott | **hiányzik, de kellene** (kis munka: ~1 nap, a psx `emit` mintájára) |
| Lifecycle Engine (átmenet-jogszerűség) | BP 2.2, 5; R: KEEP | A `reduce()` determinisztikus, de az illegális eseményt nem utasítja el, csak bizonyos esetekben figyelmen kívül hagyja (pl. `retry_requested` csak failed/rejected-ből). A jogszerűséget a CLI ellenőrzi (`approve` csak needs_acceptance/rejected-ből, `retry` max_attempts-en belül) | `psx.py` 196–200, 938–957 | **részben** (az ellenőrzés a parancsokban van, nem egy központi átmenettáblában) |
| Mission Intake (egy belépési határ) | BP 4, 5 | Egy belépés: `psx submit` (az `ugy`, a panel, az MCP és az asszisztens mind ezt hívja). Idempotens kulcs (`O_EXCL` kulcsfájl), `submitted` esemény `spec_hash`-sel | `psx.py` 844–869; `RENDSZER.md` 3., 7. | **megvan** |
| Current state = projekció | BP 10.2 | `reduce()` + `result.json`; a panel is ebből olvas | `psx.py` 144, 216 | megvan (feladat) |
| Context Builder (korlátos, proveniencia) | BP 14 | Ügyfeladat-prompt: cél, következő lépés, legutóbbi eredmények, `KEPESSEGEK.md` (mért állapot); limitnél munkacsomag az előző kísérletek naplóvégével | `psx.py` 543–592 (`work_package`, `agent_prompt`); `RENDSZER.md` 4. | **megvan** (egyszerűsítve) |
| Executor Adapter | BP 13; R: KEEP | 6 adapter: Claude, Copilot, Codex, CodeWhale, saját ciklus (or-olcso / or-free) | `psx.py` 267–507; `KEPESSEGEK.md`: 6 végrehajtó „elérhető ok” (2026-11-12 03:17) | **megvan** |
| Executor Router (képesség-/bizonyítékalapú) | BP 5 „deferred”, 13; R: REMOVE (párhuzamos orchestratorok) | Nincs router; a spec előre megadott sorrendű listát (`providers`) tartalmaz, limitnél továbbad | `psx.py` 704–763; toti: „modellroutert nem építünk” (`CLAUDE.md`) | **nem kell** |
| Végrehajtó-megfelelőségi tesztek (identitás, timeout, cancel, artifact, replay, budget) | BP 13, 18 | Részleges: hamis limit-végrehajtók (`fakes/fake-claude-limit`, `fake-copilot-limit`), helyreállítási teszt, 2026-11-04-i próbák izolált `PSX_HOME`-ban | `executor/tests/`; `VEGREHAJTO.md` 4.; `python3 -m unittest discover -s executor/tests` → **49 teszt OK** (2026-11-12) | **részben** (adapterenkénti egységes megfelelőségi mátrix nincs) |
| Result Intake (korreláció, duplikátum, replay) | BP 8.2; R: KEEP | Feladat/kísérlet azonosító (`psx-<id>-a<n>` unit), lépés-`digest`; ugyanaz a spec → ugyanaz a feladat-azonosító | `psx.py` 543–559, 858–862 | **részben** (nincs külön eredmény-boríték hash-sel; a duplikált eredménykézbesítést nem teszteli külön teszt – nem igazolt) |
| Artifact Verification | BP 8.3; R: KEEP | `letezik`, `min-meret`, `tartalmaz`, `nem-tartalmaz`, `json`, `parancs`, `idezetek` – a végrehajtótól függetlenül futnak | `acc.py`; `RENDSZER.md` 6.; jelen feladat specje is ilyen szabályokat tartalmaz | **megvan** (kivétel: hash-azonosság szabály nincs) |
| Outcome Evaluator (SATISFIED / UNSATISFIED / INDETERMINATE) | BP 8.4 | `elbiral()`: elfogadva / hianyos / elfogadasra_var; az `idezetek` „ellenőrizhetetlen” ítélete emberhez megy | `acc.py` 112; `RENDSZER.md` 6. | **megvan** (egyszerűsítve) |
| Quality (AO8K) – alapból tanácsadó | BP 9.3 | Független ellenőr (másik gyártó modellje): külön vélemény, nem bizonyíték; a cáfolt állítás „hiányos” | `RENDSZER.md` 6.; `VEGREHAJTO.md` 11. | **megvan** |
| Governance Acceptance (MAY_ACCEPT) | BP 9.2 | Gépi szabályok + `ember` → `needs_acceptance` → `psx approve/reject` (`ugy elfogad/elutasit`); elfogadni csak toti kérésére | `psx.py` 943–949; `CLAUDE.md` | **megvan** |
| Risk Classifier + Governance Authorizer (MAY_EXECUTE) | BP 9.1 | Nincs feladatonkénti ALLOW/PAUSE/DENY. Helyette: Landlock-zár (titkok kizárva), sudo nincs, root csak `rendszerlepes` + toti jóváhagyás, CodeWhale szándékosan zár nélkül | `executor/zar.py`; `scripts/rendszerlepes.py`; `RENDSZER.md` 5. | **részben** (teljes kapu: nem kell – toti „nincs visszafelé korlátozás”) |
| Budget Manager / Kill Switch | BP 5, 9.1 | `max_cost_usd` (fizetős alap: `DEFAULT_PAID_BUDGET_USD`), `max_elofizetes_usd`, `max_credits`, `timeout_s`, `max_attempts`; `psx cancel` a teljes folyamatcsoportot leállítja (`killpg`); a CodeWhale-nél a keret nem érvényesül | `psx.py` 510–533, 704–763, 853; `RENDSZER.md` 5. táblázat | **megvan** (kivétel: CodeWhale-keret, toti döntése) |
| Human Escalation (attribúcióval) | BP 5, 6 | `waiting_human`, `human_decision` esemény a naplóban; panel/Web Push értesítés | `psx.py` 177–188; `scripts/ertesites.py` | **megvan** (a döntő személy azonosítója az eseményben – nem igazolt, nem néztem meg) |
| Recovery Policy (korlátos retry) | BP 15; R: KEEP | `interrupted` → a lépés újrafuttatható; `retry_requested` új kísérlet ugyanazon azonosítón, `max_attempts` határral; `waiting_model` magától folytatja | `psx.py` 189–213; `executor/tests/test_agent_helyreallitas.py` | **megvan** |
| RETRY = új attempt_id, a régi nem íródik át | BP 6, 15 | Új `attempt` sorszám és új systemd-unit; a régi események megmaradnak | `psx.py` 151–152, 773–797 | **megvan** |
| Projekciók (goal_queue, SQLite, self_state, status.md) | BP 10.3 | `result.json`, panel-nézet, generált `UGY.md`-fejléc, `KEPESSEGEK.md` | `psx.py` 216; `ugy.py` | megvan (más formában) |

### 2.3 Életciklus és Mission Contract (BP 6–7)

| Elem | Régi terv | Mai állapot | Bizonyíték | Besorolás |
|---|---|---|---|---|
| Kanonikus életciklus (RECEIVED → … → ARCHIVED, 15 állapot) | BP 6 | Egyszerűsített: `queued → running → verifying → accepted / needs_acceptance / rejected`, közben `waiting_model`, `waiting_human`, `failed`, `cancelled` | `psx.py` 24, 54, 144–213 | **részben** (szándékosan egyszerűbb; a PLANNING / AUTHORIZED / ARCHIVED állapot hiánya elfogadható) |
| ARCHIVED (bizonyíték megőrzése, nem törlés) | BP 6, 15 | Ügy: `ugy lezar`; a feladatnapló megmarad (`~/.local/state/project-s/psx/tasks/`, 56 feladat). Ennek mentése: napi helyi mentés | `ugy.py`; `KEPESSEGEK.md` „Mentés” | részben |
| Duplikált beküldés / replay nem léptet | BP 6, 15 | Beküldés: azonos kulcs → ugyanaz az azonosító. Döntés: `approve` megengedett `rejected`-ből is (emberi felülbírálat, új esemény). Duplikált `verified` esemény nincs külön kiszűrve | `psx.py` 858–862, 945 | **részben** |
| Mission Contract (elvárt artifact, ellenőrzések, szerződés hash-e beküldéskor) | BP 7 | `ugy inditas --elvart --hova --ellenorzes`; a `--hova` fájl létezése mindig szabály; `spec_hash` a `submitted` eseményben | `ugy.py` 7–33, 200–203; a jelen feladat `events.jsonl` 1. sora: `spec_hash: cf1492a9…` | **részben** (hiányzik: hatásosztály, kifejezett retry/eszkalációs szabály a szerződésben, artifact-hash; `elvart` csak szöveg) |
| Blokkoló minőségi szabály csak előre rögzítve | BP 9.3 | Az elfogadási szabályok a beküldött specben vannak; az „újraellenőrzés” gomb ugyanazokat futtatja | `RENDSZER.md` 7. | megvan |
| Apró hiba a szerződésben | – | A jelen feladat specjében a `letezik:` szabály kétszer szerepel (az `ugy` automatikusan hozzáadja, és a kézi `--ellenorzes` is tartalmazta) | `spec.json` → `acceptance` | kis javítás (deduplikálás) |

### 2.4 Bizonyítékok, E2E és kapuk (BP 17–19)

| Elem | Régi terv | Mai állapot | Bizonyíték | Besorolás |
|---|---|---|---|---|
| MINIMUM_EXECUTION_E2E | BP 17.1 | Futás → eredmény → korreláció → egyszeri rögzítés: élő feladatokon folyamatosan | pl. `t20261105-094605-ad7311` (Copilot, accepted) és `t20261105-094605-2940d8` (Claude, accepted) – `FOLYTATAS.md`; 56 feladat a psx-tárban | **megvan** (MAY_EXECUTE nélkül) |
| FULL_MISSION_E2E (szerződés → gépi ellenőrzés → elfogadás → archiválás → a duplikált újraküldés no-op) | BP 17.2 | Az elemek megvannak, és élesben is futnak (`proba-elfogadas` ügy). Egyetlen naplózott, végig dokumentált futás, amely a duplikátum-no-opot is mutatja: nem igazolt | `UGY.md` (korábbi feladat `t20261108-195153`); `ugyek/proba-elfogadas/` | **részben / nem igazolt** |
| Historikus negatív próbák mint regresszió | BP 17.3; R: KEEP | Új negatív esetek: DeepSeek forrás-hamis idézetei, „15 ellenőrzött idézet” = 0 – a kapu megfogta; pontossági készlet (30 eset) | `RENDSZER.md` 6.; ügy `ellenorzo-pontossag`; `executor/tests/test_pontossag.py` | **megvan** |
| Bizonyítékszintek jelölése (STATIC / FIXTURE / HISTORICAL / CURRENT_LIVE) | BP 3.3 | `KEPESSEGEK.md` mért állapot (generált, napi); a jelentésekben „Ellenőrizve / Bizonytalan” szakasz kötelező | `scripts/kepessegek.py`; feladatprompt | **részben** (nincs egységes szintcímke) |
| Autonómia-kapuk (shadow, cutover, routing ≥2 adapter stb.) | BP 18 | Nem alkalmazható: nincs régi rendszer, amelyről át kellene állni | – | **nem kell** |
| 7 szakaszos migráció | BP 19 | A szöveg csonka; a régi mag hiányában a stratégia tárgytalan | forrásfájl fejléce | **nem kell** |

### 2.5 Régi rétegek (BP 3, 11, 12, 16)

| Elem | Régi terv | Mai állapot | Bizonyíték | Besorolás |
|---|---|---|---|---|
| Continuous Mind → vékony ütemező | BP 11; R: SIMPLIFY | Nincs önálló „elme”. Ütemezés: `project-s-executor.service` felügyelő (max. 3 párhuzamos), `waiting_model` automatikus folytatás, NEAR-munkás időzítő | `RENDSZER.md` 5., 8. | megvan (egyszerűsítve) |
| Interactive Layer mint felület, saját állapotgép nélkül | BP 12; R: KEEP / REPLACE | Panel (`mcp/panel.py`), MCP (`mcp/server.py`), beszélgető asszisztens: mind a psx-en át dolgozik, nincs saját életciklusuk | `RENDSZER.md` 7. | **megvan** |
| MetaController (korlátos tervező) | BP 16; R: SIMPLIFY | Nincs; a tervezést az emberi beszélgetés és a végrehajtó maga végzi | – | nem kell (most) |
| StrategyCouncil / Arbiter | BP 16 DEFER; R: REMOVE | Nincs; eseti független ellenőr van | toti: „tanácsrendszert nem építünk” | **nem kell** |
| ExecutionValidator (generikus pass) | BP 16 STRANGLE; R: REPLACE | Típusos szabályok (`acc.py`) váltották ki | `acc.py` | megvan (a csere megtörtént) |
| self_correcting_loop → Recovery Policy | BP 16 | Lásd Recovery Policy | `psx.py` | megvan |
| parallel_loop_manager | BP 16 REMOVE; R: REMOVE | Nincs; egyszerű kapacitáskorlát (3 párhuzamos) | `RENDSZER.md` 5. | **nem kell** |
| SAI, hatutas prompt-router, taste model | R: SIMPLIFY / REPLACE / REMOVE | Nincs | `CLAUDE.md` szabályok | **nem kell** |
| Elkülönített mentés + visszaállítási próba | R 9., 17. (review) | Napi helyi mentés (9 db) – ugyanazon a gépen; a Drive-ra mentés nincs bekötve; visszaállítási próba nem volt | `KEPESSEGEK.md` „Mentés” | **hiányzik, de kellene** (a Drive-bekötés toti lépése; a próba ~1 óra) |
| Titokkezelés (kulcs nem kerül repóba) | Handoff / review, kulcsszivárgás tanulsága | pre-commit `titok_ellenor.py`, titkok `~/.config/project-s/` (600), a zár kizárja | `VEGREHAJTO.md` 8. „Kulcsszivárgás”; `scripts/titok_ellenor.py` | **megvan** |

---

## 3. Javasolt következő lépések (prioritással, kicsi és ellenőrizhető lépések)

1. **(P1) Ügyszintű eseménynapló** – `ugyek/<ügy>/esemenyek.jsonl` a psx `emit` mintájára (seq, prev, hash). Az
   `allapot.json` ebből generált nézet legyen, és legyen `ugy verify-journal <ügy>` parancs. *Ellenőrzés:* egységteszt:
   a napló visszajátszása = a mai `allapot.json`; a hash-lánc egy kézzel módosított sornál elbukik. ~1 nap.
2. **(P1) Visszaállítási próba** – a legutóbbi `~/mentes` archívum kicsomagolása üres könyvtárba; `psx verify-journal`
   néhány feladatra, `ugy lista` az új helyen. *Ellenőrzés:* napló + `KEPESSEGEK.md` „visszaállítási próba: ✅”.
   A Drive-bekötés toti lépése (`drive-belepes`).
3. **(P2) Központi átmenettábla a psx-ben** – `ENGEDETT = {(állapot, eseménytípus)}`. Az `emit()` utasítsa el (vagy
   `rejected_event` megfigyelésként naplózza) az illegális átmenetet. *Ellenőrzés:* teszt: `human_decision` `running`
   alatt → nincs állapotváltás; duplikált `verified` → no-op.
4. **(P2) Mission Contract kiegészítés** – új `hash:<fájl>:<sha256>` szabály, ill. a `--hova` fájl sha256-jának rögzítése
   a `verified` eseményben. Az `ugy` deduplikálja a szabályokat (a jelen feladatban a `letezik:` duplán szerepel).
   *Ellenőrzés:* `acc.py`-teszt.
5. **(P3) Egy dokumentált FULL_MISSION_E2E futás** – egy kicsi ügyfeladat végig: szerződés → gépi ellenőrzés → toti
   elfogadása → lezárás → ugyanaz a spec újraküldve ugyanazt az azonosítót adja. A napló-kivonat kerüljön az ügybe.

## 4. Kérdések a GPT- és a Claude-projektnek (véleményezésre)

1. **Ügyszintű napló vagy git?** Elég-e a git-előzmény (commit-lánc, GitHubon is tárolva) az ügyek tartós igazságforrásának,
   vagy kell külön hash-láncolt `esemenyek.jsonl`, amelyből az `allapot.json` újraépül? (A blueprint az utóbbit írja elő.)
2. **Mennyi MAY_EXECUTE kell?** Toti elve: „nincs visszafelé korlátozás”. Elég-e a mai OS-zár + root-jóváhagyás, vagy
   legyen egy *nem blokkoló*, csak naplózó kockázati címke (pl. „külső hatás: pénz / beküldés / fiók”), amely a NEAR-munkás
   és hasonló, kifelé ható feladatok előtt értesít?
3. **Életciklus részletessége:** maradjon-e a 9 állapotos psx-modell, vagy érdemes külön állapotként bevezetni az
   `ARCHIVED`-ot és a `PAUSED`-ot (a blueprint 15 állapotából csak ezt a kettőt)?
4. **A régi canonical mag keresése:** érdemes-e még a DEV laptopon keresni az augusztusi kódot (`lifecycle_engine_v1.py`
   stb.), vagy a review 2. ága (a szerződéseket tervként használjuk) végleges döntés legyen, és a keresés lezárható?
5. **CodeWhale keret és zár nélkül:** a blueprint szerint minden végrehajtónak kerettel és leállíthatóan kell futnia.
   Elfogadható-e tartósan a kivétel, vagy legalább a költségmérés legyen kötelező?
6. **Bizonyítékszint-címkék:** átvegyük-e a blueprint jelöléseit (STATIC / FIXTURE / HISTORICAL_LIVE / CURRENT_LIVE) a
   jelentések „Ellenőrizve” szakaszába, hogy a külső olvasó lássa, mi mért és mi csak dokumentált?

---

## Elkészült
- `ugyek/project-s-ujjaepites/eredmenyek/osszevetes-blueprint.md` (ez a fájl): összefoglaló, 5 témakörű tételes táblázat
  (kb. 45 elem) bizonyíték-hivatkozással, 5 prioritásos lépés, 6 kérdés.

## Ellenőrizve
- Forrás: `munka/forrasok/2026-08-12_PROJECT_S_MASTER_SYSTEM_BLUEPRINT_V1.md` teljes szövege (1–19. szakasz, csonka) és
  a `2026-09-15_…REVIEW.md` 1–5. szakasza (KEEP/SIMPLIFY/REPLACE/REMOVE táblázat, 84–105. sor).
- `executor/psx.py` kód: `Task.emit` (110–141: hash-lánc, flock, fsync), `reduce` (144–213), `write_view` (216–227),
  `submit` (844–869: idempotens kulcs, `spec_hash`), CLI-őrök (938–957).
- `python3 executor/psx.py verify-journal` 3 db 2026-11-11-i feladaton → „hash-lánc rendben”; a jelenlegi feladat
  `events.jsonl` első sorai (seq 1–2, prev/hash) és `spec.json` (acceptance, limits) megnézve.
- `PYTHONDONTWRITEBYTECODE=1 python3 -m unittest discover -s executor/tests -p 'test_*.py'` → `Ran 49 tests … OK`.
- `scripts/ugy.py`: `allapot.json` atomikus felülírás (87–98), `--elvart/--hova/--ellenorzes` (7–33, 200–203).
- `docs/RENDSZER.md`, `docs/KEPESSEGEK.md` (2026-11-12 03:17: 6 végrehajtó ok; mentés 9 db; Drive ❌; visszaállítási próba ❌),
  `FOLYTATAS.md` eleje, `UGY.md` korábbi összevetése.

## Bizonytalan
- A Result Intake duplikált eredménykézbesítése és a duplikált `verified` esemény kezelése: kódból következtetve, teszttel
  nem igazolt.
- A `human_decision` esemény tartalmazza-e a döntő személyt: nem néztem meg.
- A FULL_MISSION_E2E teljes láncát (a duplikátum-no-oppal együtt) egyetlen futáson nem igazoltam.
- A `scripts/tesztek.sh` teljes csomagot (ATA, NEAR, py_compile) nem futtattam (a py_compile a repóba írna); csak az
  executor-teszteket.
- A blueprint 19. szakasza utáni rész (Stage 2–7) hiányzik, arról nem lehet állítani semmit.
- `acceptance.py` szabálylistája a dokumentációból és a függvénynevekből származik, a teljes kódot nem olvastam végig.

## Következő lépés
Az ügyszintű `esemenyek.jsonl` (hash-lánc) bevezetése az `ugy.py`-ban, egységteszttel: a visszajátszás az `allapot.json`-t adja.
