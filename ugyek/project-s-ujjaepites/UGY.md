<!-- Automatikus tükör a privát Projekt S műhelyrepóból (scripts/docs_tukor.py). Személyes és hálózati adatok kitakarva. Ne itt szerkeszd. -->

# Ügy: Project-S újjáépítése

<!-- ALLAPOT:KEZDET (generált, ne szerkeszd kézzel) -->
| Mező | Érték |
|---|---|
| Azonosító | `project-s-ujjaepites` |
| Állapot | **elfogadasra_var** |
| Cél | Egyszerű, a szerveren futó Project-S mag: telefonról ügy + cél → tartós ügyállapot → végrehajtás → mentett eredmény → rövid beszámoló; a régi unified kódból csak a közvetlenül hasznos részek. |
| Következő lépés | A jelölt API-mentését közvetlen, rekordonként azonosítható NEAR-válaszrögzítéssel kell kiegészíteni. |
| Utolsó feladat | t20261112-131027-e452e1 – elfogadasra_var |
| Frissítve | 2026-11-12 13:15 UTC |

**Legutóbbi eredmény:** Elkészült a szerepalapú elfogadás tesztelt javításjelöltje és az átadási jelentés. Az éles és a jelölt teljes tesztcsomagja zöld, a végső célzott tesztekből 13/13 sikeres. Az éles kód változatlan maradt. A feladat részlegesen teljesített, mert az API-bizonyíték rögzítése és néhány folytatási út még kiegészítést igényel.

**Döntésre vár:** `t20261112-124102-b5adb8` (hianyos), `t20261112-131027-e452e1` (elfogadasra_var)

<!-- ALLAPOT:VEG -->

## Anyagok
- Forráskód (régi, Windows-alapú): https://github.com/toti85/project-s-unified (nyilvános),
  helyi klón: `~/src/project-s-unified` (utolsó commit 2026-07-16, `65fb73a`).
- Korábbi auditok/térképek a unified repóban (új teljes audit **nem** készült):
  `reports/MS_FA1_ORCHESTRATOR_MODULE_MAP_20260524.md`, `reports/MS_AO5_T30_DEV_COGNITIVE_MAP_20260613.md`,
  `reports/MS_AO8F_CONTINUE_CM_INTEGRATION_SINGLE_GOAL_20260618.md`, `reports/self_analysis_report.md`,
  `ARCHITECTURE_DIAGNOSIS_2026-04-26.md`, `PROJECT_S_INTERACTIVE_LAYER_ARCHITECTURE_20260517.md`.
- Az új, működő mag: `docs/UGYKEZELES.md` (ügykezelő), `mcp/` (végrehajtó).
- Szerver- és kapcsolati előzmények: `ugyek/szerver-es-kapcsolatok/UGY.md`.

## Interactive Layer ⇄ Continuous Mind ⇄ executor (célzott átnézés, 2026-11-03)

| Rész | Hol | Mit csinál | Állapot tárolása |
|---|---|---|---|
| Interactive Layer | `interactive_layer/` (FastAPI, `main.py`, `missions.py`, `sai_runner.py`) | `/api/chat` → prompt-osztályozás → `mission` → háttérben SAI (`src/core/super_intelligent_ai`) | SQLite (`missions`, `mission_events`), állapotgép: QUEUED→PLANNING→EXECUTING→…→COMPLETE/FAILED/TIMEOUT |
| Continuous Mind | `orchestrator_runtime/continuous_mind.py` (1338 sor) | szívverés-ciklus: `goal_queue.json` → legmagasabb prioritású `pending`, `auto_approve` + LOW/MEDIUM kockázatú cél → végrehajtó → `bridge_runner` kapu (ALLOW/DENY/PAUSE) → DONE/FAILED | `goal_queue.json` (cél: id, title, risk, auto_approve, priority, status, done_at), `active_loops/*.jsonl` |
| Executor | `governance/continue_executor_adapter.py` (`cn -p`), SAI direct mód, Cline inbox/outbox | prompt futtatása, kimenet `cline_outbox/last_output.md` + checksum + `output_ready.json` | fájlok az outboxban |

**Megállapítások**
1. Az Interactive Layer és a Continuous Mind **nincs összekötve**: két külön állapottár
   (SQLite-missionök vs. `goal_queue.json`), egyik sem hivatkozik a másikra.
2. A végrehajtók Windows-környezethez kötöttek (`C:\ProjectS-...` utak, npm-es `cn`,
   OpenRouter-kulcsok, Qwen-modellek); a toti-serveren egyik sem fut, és telepítésük
   „nagyobb Project-S-rendszer” lenne – ezt most nem csináljuk.
3. Ami a mostani célt közvetlenül segíti, és **megtartottuk** (átvéve, nem a kódot futtatva):
   - a CM cél-sémája (azonosító, cím/cél, állapot, időbélyegek) és állapot-logikája
     (pending → végrehajtás → done/failed/paused) → `ugyek/<id>/allapot.json` (`nyitott | fut | elakadt | lezart`);
   - az executor-minta: prompt → futtatás → kimenet fájlba → ellenőrzött lezárás →
     itt: `ps-job-<id>` systemd-unit + `claude -p` → `result.json` → `ugy _lezaras`;
   - az Interactive Layer mission-eseménynapló ötlete → `UGY.md` napló + `progress.log`.
4. Nem nyúltunk hozzá: SAI, prompt-router, bridge_runner/gateway, Taste Model, budget,
   kill switch, watchdog, Cline/Continue adapterek.

## Történeti dokumentumok (2026-06 → 2026-10) – `munka/forrasok/`
Toti 2026-11-03-án adta át; **történeti állapotot írnak le, összevetési alapnak használjuk**, nem a mai rendszer leírásának.

| Dátum | Dokumentum | Mi ez | Bizonyítékszint |
|---|---|---|---|
| 06-11 | `2026-06-11_Project_S_Strategic_Vision_v3.md` | Multi-AI konzultáció: „Sz” identitás, SAI vs CWI, Ghost Branch, elutasított irányok (SaaS, bounty-platform) | Toti kérdései + akkori állapotleírás |
| 08-12 | `2026-08-12_PROJECT_S_MASTER_SYSTEM_BLUEPRINT_V1.md` | **Normatív, lezárt architektúra** (`e44eecc`): Continuity Journal, Lifecycle Engine, kétlépcsős governance, Mission Contract, 7 szakaszos migráció | Terv; **a beillesztett szöveg a Stage 1-nél csonka** |
| 08-15 | `2026-08-15_PROJECT_S_AI_HANDOFF.docx` (+ `.txt` kivonat) | Stage 5B–7B: élő canonical mission (ACCEPTED, retry, replay, rebuild, hash-chain), CM strangler `CM_CANONICAL_STRANGLER=1` mögött | Beszámolt élő bizonyíték; a kód **nem** áll rendelkezésre |
| 09-15 | `2026-09-15_PROJECT_S_ARCHITECTURE_AND_REVENUE_REVIEW.md` | Független review: a végrehajtási igazság-magot megtartani, az „intelligencia”-rétegeket visszavágni; KEEP/SIMPLIFY/REPLACE/REMOVE; első termék: szűk beszerzési figyelő-pilot | Statikus kódvizsgálat a júliusi ZIP-en |
| 10-17 | `2026-10-17_PROJECT_S_FOLYTATASI_NAPLO_v1.6.md` | *Akkor:* DL380 fertőzött (Mimic ransomware), izolálva – **történeti adat: a DL380-at toti azóta újratelepítette (Ubuntu Server 24.04), ez a toti-server**; OVH VPS-1 + `ps_bootstrap.sh` v0.3 MCP-telepítő (helyben tesztelve, VPS-en nem futott); helyreállítási sorrend | Átadónapló |
| 10-28 | → `ugyek/bevetel-kutatas/munka/forrasok/2026-10-28_AI_LEHETOSEG_RADAR_v1.md` | 16 bevételi nyom | lásd a bevétel-ügyet |

### A legfontosabb tény: az augusztusi canonical mag most nem elérhető
- A handoff szerinti fájlok (`lifecycle_engine_v1.py`, `mission_intake_v1.py`, `result_intake_v1.py`,
  `artifact_verification_v1.py`, `outcome_evaluator_v1.py`, `canonical_mission_runtime_v1.py`,
  `cm_canonical_strangler_v1.py`, Continuity `events.jsonl`) **nincsenek** a júliusi revízióban
  (`65fb73a`, a helyi klón is ez).
- 2026-11-03-i ellenőrzés: az `e44eecc`, `2299ba2` (Stage 6), `966965b` (Stage 7A), `bc9f80e` (Stage 7B)
  commit **egyik toti85-repóban sincs meg a GitHubon**; a legkésőbbi elérhető unified-commit a `dd8c50f` (06-11) utáni `65fb73a` (07-16).
- Lehetséges lelőhely: a DEV laptop vagy a régi DL380-lemezek. A DL380 Gen9 a mostani toti-server (toti újratelepítette
  Ubuntu Serverre); a régi Windows-korszakból egy NTFS-adatkötet maradt rajta: `/dev/sda1`, `Data3`.
  Átvizsgálása: lásd lent, „Data3-vizsgálat (2026-11-04)”.
- A 09-15-i review két ága ezért élő döntés: **(1) ha előkerül** → visszaállítás + egy korlátos élő ellenőrzés,
  utána arra építünk; **(2) ha nem** → a szerződéseit tervként használjuk, és csak a következő munkához szükséges szeletet építjük újra.

### Data3-vizsgálat (2026-11-04) – LEZÁRVA, nincs találat
- **Azonosítás:** `/dev/sda1`, címke `Data3`, NTFS UUID `70742877742841E8`, PARTUUID `efd12241-d661-44d6-86c5-43c697ac154c`,
  480 068 501 504 bájt; eszköz: HP Smart Array „LOGICAL VOLUME”, WWN `600508b1001cd195e6ca64964e943c2a`, `pci-0000:03:00.0-scsi-0:1:0:0`.
- **Írásvédelem:** `blockdev --setro` a `/dev/sda`-ra és `/dev/sda1`-re, csatolás `ntfs3 ro,noexec,nosuid,nodev` → `/mnt/data3`.
  A fájlrendszert nem javítottuk, nem írtunk rá, programot/szkriptet/hookot nem futtattunk.
- **Tartalom (455 bejegyzés, 60 GB):** `ModelVault/` (GGUF-modellek, llama.cpp, RAG-index, naplók), `local_model_benchmarks/`.
  **Nincs `.git`, nincs archívum, nincs canonical fájlnév.**
- **Zsarolóvírus-nyom:** 265 kis fájl `..exe` végződéssel, mind 2026-09-01/04 dátummal, tartalmuk titkosított
  (entrópia ≈ 8 bit/bájt, fejléc nélkül) – köztük `ModelVault/rag/project_s_docs/*` (STATUS, DECISIONS, self_state.json, kódrészletek)
  és a RAG-indexek. Ezek kulcs nélkül nem olvashatók.
- **Törölt MFT-bejegyzések** (`ntfsundelete --scan`, csak olvasás; lista: `munka/data3/torolt-mind.txt`): 288 bejegyzés,
  165 visszanyerhető tartalommal – régi Project-S-gyökérfájlok 2025-05 → 2026-06 (a júliusi archívumban is megvannak).
  **2026-08-06-án egy Project-S könyvtárfa törlődött** (`API`, `archive`, `ARCHIVE_VERSIONS`, `config`, `gap030_validation_scripts_20260520`…),
  de a mappák tartalma itt már nem listázható; canonical fájlnév a törölt bejegyzések között sincs.
- **Nyers bájtkeresés** a teljes kötetre (szabad terület is): `grep -a -b -o -F -f minta.txt /dev/sda1`
  (minták: `munka/data3/minta.txt` – célfájlnevek, `CmCanonicalStrangler`, commit-előtagok, `.git`-szövegjelek).
  **Lefutott a végéig: 2026-11-04 00:30:01 UTC, kilépési kód 1 = egyetlen egyezés sincs** a 480 GB-ban.
  (A 00:37-kor kért leállításkor már nem futott, megszakítani nem kellett.) Korlát: csak ASCII/UTF-8 szöveget keresett;
  tömörített git-objektumban vagy titkosított fájlban lévő nevet így nem találhat meg.
- **Következtetés:** a Data3 a modell-/benchmarkkötet volt, nem a Project-S munkakönyvtár; az augusztusi mag rajta sem
  fájlként, sem a szabad területen olvasható szövegként nincs meg. Következő lelőhely: a DEV laptop (ha elérhető).
- A kötet csatolva maradt (csak olvasható). Leválasztás később: `sudo umount /mnt/data3`.

### Összevetés: a blueprint/review elvei ↔ a mostani Linux-műhely
| Elv (blueprint / review) | Mostani műhely | Megítélés |
|---|---|---|
| Egyetlen tartós igazság: append-only, hash-láncolt journal, egy író | `allapot.json` (felülírt pillanatkép) + `UGY.md` napló + git-előzmény (GitHubon is) | Részleges. A git-előzmény gyakorlatilag hash-láncolt napló, de az állapot nem belőle épül újra. |
| `MAY_EXECUTE ≠ MAY_ACCEPT`; a végrehajtó önbevallása nem elfogadás | `succeeded` + a Claude-válasz „Ügyállapot” szakasza frissíti az ügyet | **Hiány.** A mostani „kész” a blueprint szerint csak `EXECUTION_RECORDED`. |
| Mission Contract: előre rögzített elvárt artifact + determinisztikus ellenőrzés | Szabad szöveges feladat, nincs elvárt eredmény | **Hiány** – ez a legolcsóbb javítás. |
| mission / attempt / execution azonosítók; retry = új attempt | ügy-azonosító + `job_id`; újraindítás = új job | Elég a mostani méretben. |
| Replay/duplikátum nem léptet állapotot | Az `on_done` egyszer fut; kézi újrafuttatás duplikált naplósort írna | Kis hiány. |
| Végrehajtó cserélhető, adapter mögött | `claude -p` systemd-unitban (`mcp/jobs.py`) | Megfelel; a Continue/SAI itt nem fut. |
| Leállítás a teljes folyamatfán, időkorlát (v0.3 tanulság: cgroup-kill) | systemd-unit cgroup, `KillMode=mixed`, `RuntimeMaxSec`, kipróbálva | Megfelel. |
| Különálló hozzáféréssel védett mentés + visszaállítási próba (review §9, §17) | Napi tar.gz **ugyanarra a lemezre** + GitHub (csak az ügyek) | **Hiány**; visszaállítási próba sincs. |
| Korlátos, provenienciás kontextus | Ügycél + következő lépés + utolsó 3 összefoglaló, fájlhivatkozásokkal | Megfelel. |
| Meta-réteg, tanács, modellrouter: DEFER/REMOVE | Nem építettük | Megfelel. |
| Interactive Layer mint felület, nem saját életciklus | Felület most: Remote Control + `ugy` parancs | Egyenértékű cél, egyszerűbben. |
| Bizonyítékszintek: STATIC / TEST_FIXTURE / HISTORICAL_LIVE / CURRENT_LIVE | Nincs jelölve | Átvesszük a dokumentálásban. |

### Ebből következő javaslat (a következő lépés jelöltjei, nincs megvalósítva)
1. **Elfogadás szétválasztása:** a háttérfeladat vége csak „végrehajtva” legyen; az ügyben külön `elfogadva / elutasítva`
   (toti vagy determinisztikus ellenőrzés). Ez a blueprint `EXECUTION_RECORDED ≠ ACCEPTED` elvének kicsi, működő változata.
2. **Mini Mission Contract:** az `ugy inditas` fogadjon el opcionális elvárt fájl(oka)t és ellenőrző parancsot;
   a lezárás ezeket ellenőrzi, és csak így jelölheti a feladatot ellenőrzöttnek.
3. **Ügyenkénti `esemenyek.jsonl`** (append-only, hash-lánccal), amiből az `allapot.json` újraépíthető – csak ha az 1–2. bevált.
4. Az elveszett mag keresése (`Data3`, DEV laptop) a fenti két ág közti döntéshez.

## Napló
- 2026-11-03 – Ügyirat egységesítve; a szerver/kapcsolati előzmények átkerültek a
  `szerver-es-kapcsolatok` ügybe. Unified kód célzott átnézése kész (fent).
- 2026-11-03 – Történeti dokumentumok (06-11 vízió, 08-12 blueprint, 08-15 handoff, 09-15 review, 10-17 folytatási napló) archiválva `munka/forrasok/`-ba és összevetve a műhellyel. Az augusztusi canonical mag a GitHubon nincs meg; a `/dev/sda1` (`Data3`) kötet átnézése toti döntésére vár.
- 2026-11-03 23:53 UTC – közvetlen munka rögzítve: Az öt dokumentum a munka/forrasok/ mappába került, az összevetés az UGY.md-ben van. A blueprint elveiből a végrehajtás és leállítás, a korlátos kontextus és a meta-réteg elhagyása már teljesül a műhel
- 2026-11-04 00:19 UTC – közvetlen munka rögzítve: A Data3 (/dev/sda1, UUID 70742877742841E8) csak olvashatóan csatolva. Ez a modell- és benchmarkkötet volt: nincs rajta .git, archívum vagy canonical fájl. A 265 kis fájl tartalmát a zsarolóvírus 2026-
- 2026-11-04 00:37 UTC – következő lépés beállítva: Az augusztusi canonical mag keresése a DEV laptopon, amikor elérhető. (A Data3-kötet átvizsgálva: nincs találat.)
- 2026-11-05 17:47 UTC – közvetlen munka rögzítve (elfogadásra vár) → [`eredmenyek/2026-11-05-eros-vegrehajto-irany.md`](eredmenyek/2026-11-05-eros-vegrehajto-irany.md): Erős végrehajtó iránya: a ciklus közjószág; erőt a független végrehajtók + OS/háló-zár + végrehajtótól független bizonyítás (web-pillanatkép, kereszt-ellenőr) ad.
- 2026-11-08 19:51 UTC – feladat beküldve a végrehajtónak `t20261108-195153-318d73`: Nézd át mit javasolnak még ehhez. És ellenőrizd a mostani állapotát az ügynek és frissítsd — elvárt: (nincs megadva) – a feladat leírása szerinti jelentés → `eredmenyek/20261108-195153.md`; ellenőrzés: ember
- 2026-11-08 19:56 UTC – feladat `t20261108-195153-318d73`: végrehajtó állapota **needs_acceptance** → **elfogadasra_var** (szolgáltatók: codewhale)
- 2026-11-08 20:11 UTC – közvetlen munka rögzítve (elfogadásra vár) → [`eredmenyek/2026-11-08-ellenorzes-t20261108-195153.md`](eredmenyek/2026-11-08-ellenorzes-t20261108-195153.md): CodeWhale-jelentés ellenőrizve: böngésző és Codex állapota hibás (mindkettő telepítve, működik); többi rendben. Mért képességleltár: docs/KEPESSEGEK.md, minden végrehajtó promptjában.
- 2026-11-11 16:32 UTC – közvetlen munka rögzítve (elfogadásra vár) → [`eredmenyek/2026-11-11-jelentesellenorzo-futasok.md`](eredmenyek/2026-11-11-jelentesellenorzo-futasok.md): Jelentés-ellenőrző: 104157 elutasítva (valódi bemeneten eltört), 110148 (javítás) elfogadva; GPT-listán 49/12/12, 0 eszközhiba.
- 2026-11-12 07:50 UTC – feladat beküldve a végrehajtónak `t20261112-075002-bc0419`: Olvasd el és hajtsd végre: ugyek/project-s-ujjaepites/munka/osszevetes-feladat.md

# Összevetés: régi Projekt S terv ⇄ mai műhely (toti kérése, 2026-11-12)

Források (csak olvasás):
- Régi normatív terv: `ugyek/project-s-ujjaepites/munka/forrasok/2026-08-12_PROJECT_S_MASTER_SYSTEM_BLUEPRINT_V1.md`
- Kiegészítők: `…/forrasok/2026-09-15_PROJECT_S_ARCHITECTURE_AND_REVENUE_REVIEW.md` (KEEP/SIMPLIFY/REPLACE),
  `…/2026-06-11_Project_S_Strategic_Vision_v3.md`, `…/2026-10-17_PROJECT_S_FOLYTATASI_NAPLO_v1.6.md`, `…/2026-08-15_PROJECT_S_AI_HANDOFF.txt`
- Régi kód (csak hivatkozásra): `~/src/project-s-unified`
- Mai rendszer: `docs/RENDSZER.md`, `docs/KEPESSEGEK.md` (mért állapot), `docs/VEGREHAJTO.md`, `docs/UGYKEZELES.md`,
  `docs/ASSZISZTENS.md`, `FOLYTATAS.md`, `ugyek/project-s-ujjaepites/UGY.md`; a kódot (`executor/`, `scripts/`, `mcp/`) nézd meg,
  ha egy állítást ellenőrizni kell.

Feladat: a régi terv minden fő elemére (komponens, elv, folyamat, governance-szabály) állapítsd meg:
- **megvan** (hol: fájl/parancs, és mivel igazolt – teszt, mért futás, napló),
- **részben** (mi hiányzik),
- **hiányzik, de kellene** (miért, mekkora munka),
- **nem kell** (miért – pl. a 09-15 review visszavágta, vagy toti döntése: „nagyobb rendszert (SAI, tanácsrendszer, modellrouter)
  most nem építünk”, „nincs visszafelé korlátozás”).
Ne találgass: ha a mai állapot nem igazolható a fájlokból, írd „nem igazolt”-nak.

Átadás: `ugyek/project-s-ujjaepites/eredmenyek/osszevetes-blueprint.md`, magyarul:
1. 5–8 soros összefoglaló (hol tart az újjáépítés).
2. Táblázat: elem | régi terv (hivatkozás a szakaszra) | mai állapot | bizonyíték (fájl/parancs) | besorolás.
3. Javasolt következő 3–5 lépés prioritással (kicsi, ellenőrizhető lépések).
4. „Kérdések a GPT- és a Claude-projektnek” – 3–6 nyitott döntési kérdés véleményezésre.
A dokumentum legyen önmagában érthető (külső olvasó, aki a repót nem látja).
 — elvárt: Régi Master Blueprint ⇄ mai műhely összevetése: összefoglaló, tételes táblázat bizonyítékkal, következő lépések, kérdések a GPT/Claude-projektnek → `eredmenyek/osszevetes-blueprint.md`; ellenőrzés: letezik:eredmenyek/osszevetes-blueprint.md, min-meret:eredmenyek/osszevetes-blueprint.md:4000, tartalmaz:eredmenyek/osszevetes-blueprint.md:(?i)megvan, tartalmaz:eredmenyek/osszevetes-blueprint.md:(?i)hiányzik, tartalmaz:eredmenyek/osszevetes-blueprint.md:(?i)kérdés, ember
- 2026-11-12 07:53 UTC – feladat `t20261112-075002-bc0419`: végrehajtó állapota **needs_acceptance** → **elfogadasra_var** (szolgáltatók: claude)
- 2026-11-12 10:23 UTC – közvetlen munka rögzítve (elfogadásra vár) → [`eredmenyek/2026-11-12-ugygazda.md`](eredmenyek/2026-11-12-ugygazda.md): Ügygazda első változat élesben (figyel/folytat/kezdeményez), visszaállítási próba RENDBEN 55/55, NEAR külső-művelet védőréteg, ügynapló; 21 új teszt zöld; a rendszer saját ellenőrzése talált és javítt
- 2026-11-12 12:36 UTC – feladat `kozvetlen-20261105-174732`: elfogadasra_var → **elfogadva** (döntött: toti (rábízta: claude-code, 2026-11-12)): Az ügygazda ajánlása alapján.
- 2026-11-12 12:36 UTC – feladat `t20261108-195153-318d73`: végrehajtó állapota **rejected** → **hianyos** (szolgáltatók: codewhale)
- 2026-11-12 12:36 UTC – feladat `t20261108-195153-318d73`: elfogadasra_var → **elutasitva** (döntött: toti (rábízta: claude-code, 2026-11-12)): A gépi szabályok teljesültek, de a független utóellenőrzés (kozvetlen-20261108-201135) két téves állapotállítást talált (böngésző, Codex: valójában telepítve és működik). [KORÁBBI 2026-11-08]
- 2026-11-12 12:36 UTC – feladat `kozvetlen-20261108-201135`: elfogadasra_var → **elfogadva** (döntött: toti (rábízta: claude-code, 2026-11-12)): Az ügygazda ajánlása alapján.
- 2026-11-12 12:36 UTC – feladat `kozvetlen-20261111-163223`: elfogadasra_var → **elfogadva** (döntött: toti (rábízta: claude-code, 2026-11-12)): Az ügygazda ajánlása alapján.
- 2026-11-12 12:36 UTC – feladat `t20261112-075002-bc0419`: végrehajtó állapota **accepted** → **elfogadva** (szolgáltatók: claude)
- 2026-11-12 12:36 UTC – feladat `t20261112-075002-bc0419`: elfogadasra_var → **elfogadva** (döntött: toti (rábízta: claude-code, 2026-11-12)): Az ügygazda ajánlása alapján.
- 2026-11-12 12:36 UTC – feladat `kozvetlen-20261112-102309`: elfogadasra_var → **elfogadva** (döntött: toti (rábízta: claude-code, 2026-11-12)): Az ügygazda ajánlása alapján.
- 2026-11-12 12:41 UTC – feladat beküldve a végrehajtónak `t20261112-124102-b5adb8`: Vezesd be az ügyszintű, hash-láncolt `esemenyek.jsonl` naplót az `ugy.py`-ban. Minden ügyállapot-változás fűzzön hozzá egy eseményt (idő, típus, adat, előző hash, saját hash). Készíts visszajátszó függvényt, amely a naplóból előállítja az `allapot.json` tartalmát. Írj hozzá egységtesztet: a visszajátszott állapot egyezzen az `allapot.json`-nal, és a napló manipulálása (hash-törés) legyen észlelve. Röviden rögzítsd az eredményt az eredmenyek/esemenynaplo-ugyszint.md fájlban (mi készült, teszt eredménye). — elvárt: Az `ugy.py` ügyszinten hash-láncolt `esemenyek.jsonl`-t ír, a visszajátszás reprodukálja az `allapot.json`-t, a hash-lánc törését a teszt észleli, a tesztek zöldek, az eredményfájl létezik. → `eredmenyek/esemenynaplo-ugyszint.md`; ellenőrzés: letezik:eredmenyek/esemenynaplo-ugyszint.md, min-meret:eredmenyek/esemenynaplo-ugyszint.md:600, tartalmaz:eredmenyek/esemenynaplo-ugyszint.md:(?i)esemenyek\.jsonl, tartalmaz:eredmenyek/esemenynaplo-ugyszint.md:(?i)hash, parancs:cd /home/sagent/project-s-workshop && python3 -m pytest -q -k esemeny, parancs:python3 /home/sagent/project-s-workshop/scripts/ugygazda.py vedett-ellenor 76fb75458cb4a541728c73f7c9a3a0d202d752c3
- 2026-11-12 12:44 UTC – feladat `t20261112-124102-b5adb8`: végrehajtó állapota **rejected** → **hianyos** (szolgáltatók: codex, claude) (nem teljesült: parancs:cd /home/sagent/project-s-workshop && python3 -m pytest -q -k esemeny)
- 2026-11-12 13:10 UTC – feladat beküldve a végrehajtónak `t20261112-131027-e452e1`: Olvasd el és hajtsd végre: ugyek/project-s-ujjaepites/munka/feladat-allitas-kuszob.md

# A) Bizonyítékalapú automatikus elfogadás (toti/GPT döntése, 2026-11-12)

Kód: `scripts/ugy.py` (`ellenor_lepes` – a független ellenőr utasítása, `inditas`), `executor/acceptance.py` (szabálymotor),
`scripts/ugygazda.py` (folytatás, döntési sor, `ellenor_tablazat`, `javitas`). Ezek közül `executor/acceptance.py` és
`scripts/ugygazda.py` VÉDETT: a változtatás toti külön jóváhagyásával élesedik (a te feladatod a megvalósítás + tesztek).

## Szabály (induló, működési küszöb – nem tudományos határ)
Az ellenőr táblázata kapjon „szerep” oszlopot: | # | állítás | szerep | ítélet | bizonyíték |, szerep ∈ {dontesi, erdemi, feltaro}.
- dontesi (döntést meghatározó: ár, jogosultság, finanszírozás, vállalhatóság): automatikus lezárás csak ha MIND IGAZOLT
  és nincs feloldatlan ellentmondás (CÁFOLT vagy egymásnak ellentmondó sor).
- erdemi (egyéb érdemi): legfeljebb 20% ELLENŐRIZHETETLEN (a küszöb paraméter legyen, alap 0.20).
- feltaro (kifejezetten feltáró jelentés / állítás): lezárható „tájékoztató” eredményként; tájékoztató eredményből az
  ügygazda NEM indíthat vállalást (NEAR-ajánlat, külső művelet, pénz) – csak további kutatást/javítást.
- A régi, szerep nélküli táblázat: minden sor erdemi-nek számít (visszafelé kompatibilis).
- Bizonyítékhiány → célzott javítás (az ügygazda `javitas` útja: a hiányzó döntési/érdemi bizonyíték is kifogás), és csak a
  keret kimerülésekor akadályjelentés; nem emberi elfogadás, nem automatikus továbblépés.
- A nem nyilvános API-adat (pl. NEAR MCP) állítása akkor IGAZOLT, ha mentett válaszra hivatkozik: fájl, időpont,
  rekordazonosító. Készíts mentési segédet: `allapot/api-valaszok/<forras>/<UTC-idő>-<rekord>.json` (titok nélkül),
  és az export (`ugygazda.export`) ide is mentse a NEAR-listákat (ajánlatok, kiosztások) időbélyeggel.

## Megvalósítás
1. Új elfogadási szabály (pl. `allitasok:<ellenorzes.md>[:0.20]`) az `acceptance.py`-ban: ítélet = ok / tajekoztato / hianyos,
   részletes indoklással (hány dontesi nem igazolt, érdemi arány). Az `ugy inditas` kutatási/jelentés feladatoknál
   automatikusan tegye a meglévő `nem-tartalmaz:…CÁFOLT` mellé.
2. A tájékoztató eredmény jelölése az ügyállapotban (pl. feladat `tajekoztato: true`), a psx-ben és a panelen látható.
3. Az ügygazda: tájékoztató eredményből nem indít vállalást; a hiányzó bizonyítékot célzott javítással kéri.
4. Tesztek (`scripts/tesztek.sh`-ba kötve): dontesi ELLENŐRIZHETETLEN → nincs lezárás; erdemi 25% → nincs; 15% → ok;
   feltaro → tajekoztato; CÁFOLT → hianyos; régi formátum; tájékoztatóból nincs vállalás.

## Átadás
`ugyek/project-s-ujjaepites/eredmenyek/allitas-kuszob.md`: mi változott (fájl:sor commit-azonosítóval), tesztkimenet,
és mely védett fájl változott (toti jóváhagyásához). `scripts/tesztek.sh` ZÖLD.
 — elvárt: Állítás-szerep szerinti elfogadás (dontesi mind igazolt, erdemi ≤20% ellenőrizhetetlen, feltaro → tájékoztató), mentett API-válaszok, tesztek → `eredmenyek/allitas-kuszob.md`; ellenőrzés: letezik:eredmenyek/allitas-kuszob.md, min-meret:eredmenyek/allitas-kuszob.md:1500, tartalmaz:eredmenyek/allitas-kuszob.md:(?i)dontesi, parancs:/home/sagent/project-s-workshop/scripts/tesztek.sh, ember
- 2026-11-12 13:15 UTC – feladat `t20261112-131027-e452e1`: végrehajtó állapota **needs_acceptance** → **elfogadasra_var** (szolgáltatók: codex)
