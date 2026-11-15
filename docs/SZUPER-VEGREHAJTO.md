<!-- Automatikus tükör a privát Projekt S műhelyrepóból (scripts/docs_tukor.py). Személyes és hálózati adatok kitakarva. Ne itt szerkeszd. -->

# Szuper-végrehajtó – terv (toti döntése, 2026-11-14)

**Cél:** a főnök a psx-re építve egy egységes, önállóan fejlődő végrehajtót kapjon, TELJES hozzáféréssel (mint a Claude Code-munkamenet, vagy több).
A projekt és a szerver kísérleti: haladni kell, nem mindent túlellenőrizni.

## Döntések (toti, 2026-11-14)
1. A főnök zár nélkül, a szerver titkaival és sudo-val fut (`executor/beszelgetes.py`: `TELJES_HOZZAFERES`; visszakapcsolás: `FONOK_ZAR=1`).
2. Nincs napi hívásplafon az ügygazdán (`ugygazda/szabalyzat.json`: `tervezes_max_hivas_naponta` = 100000).
3. A psx-et fejlesztjük (nem írjuk újra); a többi végrehajtó jó részeit beolvasztjuk, bevált külső mintákkal.

## Maradó szabályok (kevés, szándékosan)
- Hitelesítő adat soha nem kerül a repóba / jelentésbe.
- Toti dönt: pénzköltés/regisztráció, külső beküldés, OVH, tűzfal/SSH szűkítése.
- A HP-laptop leállítása/újraindítása/grafikus munkamenet megszakítása előtt egyeztetés (Krisztián is használja).
- Kód- és élesítésváltozásnál teszt + commit + független átnézés (ez fogta a hamis objektumot és a DoS-t).
- Napló és visszaállíthatóság (git, napi mentés) – ez a biztonsági háló, nem a zár.

## Építési sorrend
1. **Teljes hozzáférés a főnöknek** – KÉSZ (zár ki, teljes környezet, kontextus frissítve, laptop híd, ügykezelő, élesítés közvetlenül).
2. **Közös ügynöki hurok** a külön adapterek (codex/claude/copilot/or-olcso/codewhale) helyett: terv → eszközhívás → ellenőrzés → javítás, egységes állapot- és
   kontextusformátum, megszakítható és folytatható. Az adapterek a modellréteg (`executor/modell.py`) alatt maradnak cserélhetők; a hurok az eszközöket és az állapotot birtokolja.
3. **Eszközök a főnöknek közvetlenül:** shell, fájl, böngésző (`executor/web.py`, playwright), keresés, Gmail, laptop-híd, `ugy`/`psx` – egy eszközjegyzékben, egységes naplózással.
4. **Alügynökök:** a főnök psx-feladatokat indít párhuzamosan (munkafa-izolációval), összegyűjti az eredményeket; limitnél a meglévő átadási lánc (tartaléklánc) dolgozik.
5. **Beolvasztás:** a végrehajtók eltérő erősségei (Codex kód, Claude kutatás/hosszú kontextus, DeepSeek olcsó tömegmunka) modellválasztási szabályként a hurokban, nem külön rendszerként.
6. **Önfejlesztés:** a főnök a saját kódját fejleszti a fejlesztési példányban → teszt → független átnézés más gyártóval → ellenőrzött élesítés (már működik).

## Hurok-terv (2. lépés, első változat – 2026-11-14)
**Felmérés.** Ma négy külön ciklus van: a psx `agent` lépése szolgáltatónként egy adapterhívás (`executor/psx.py` `ADAPTERS`:
claude/copilot/codex/codewhale a saját CLI-jük belső ciklusával, `or-olcso`/`or-free` a saját `executor/agent.py` `Ciklus`-szal,
a modellréteg `executor/modell.py` `Profil`); a beszélgető főnök (`executor/beszelgetes.py` `agy_*`) külön, eszközök nélküli
válaszhurok. Közös bennük a munkacsomag (`work_package`) és a tartaléklánc; hiányzik a **külső, ellenőrző visszacsatolás**:
az adapter „kész”-e után az eredményt csak a végső elfogadás nézi, javítási kör nélkül.

**Minimális közös hurok = `kind: "loop"` psx-lépés** (a meglévő adapterek fölött, azokat nem írja újra):
```json
{"id": "l1", "kind": "loop", "goal": "…", "check": "python3 -m unittest -q", "max_rounds": 3,
 "providers": ["codex", "claude", "or-olcso"], "check_timeout_s": 600}
```
1. kör: ügynöki hívás (tartalékláncon) → `check` parancs a munkakönyvtárban → exit 0: `step_done`.
2. Bukásnál a következő kör célja kiegészül az ellenőrzés exit-kódjával és kimenetének végével (a munkakönyvtár marad).
3. `max_rounds` után `step_failed` + `failed`. Minden kör esemény (`loop_round`, `loop_check`) → megszakítás után
   a naplóból folytat; limitnél a meglévő átadás/várakozás működik (a kör újraindul).

**Következő lépések:** eszközjegyzék a hurok alá (3.), a `beszelgetes.py` főnöke hurokként indít (`ps-chat inditas --hurok`),
modellválasztási szabály körönként (pl. 2. bukás után másik gyártó).

## Eszközjegyzék (3. lépés, első változat – 2026-11-14)
`executor/eszkozjegyzek.py`: shell, fajl_olvas, fajl_ir, csere, web_letolt, web_keres, psx, ugy, laptop (hp-dev híd) egy
jegyzékben; `hiv(nev, args)` sosem dob (ismeretlen eszköz / hibás paraméter / kivétel → `ok: false`), minden hívás egy JSONL-sor
(`~/.local/state/project-s/eszkozok/naplo.jsonl`, titokszerű részek kitakarva), `openai_eszkozok()` OpenAI tools-formátum.
CLI: `python3 executor/eszkozjegyzek.py lista|openai|hiv <nev> '<json>'`.
**Bekötve (4. kör):** a saját ügynöki ciklus (`executor/agent.py` `Ciklus`, `or-olcso`/`or-free`) teljes jognál a jegyzék `psx` és `ugy`
eszközét is megkapja (`JEGYZEK_ESZKOZOK`); a hívás előtt szándékrekord, megszakadás után nem ismétli vakon. A laptop-híd
szándékosan nincs a végrehajtók listájában. 

## Alügynökök (4. lépés, első változat – 2026-11-14)
`executor/alugynok.py` (és az eszközjegyzék `alugynok` eszköze – csak a főnöknek, a végrehajtók ezt nem kapják meg):
`indit <feladatok.json> [--csoport N]` → feladatonként psx-feladat (`check` esetén `loop` lépés, `forras` esetén saját
git worktree, keret ≤ 2 USD/feladat); `allapot N`; `begyujt N` → Markdown összesítő (állapot, ellenőrzések, worktree-változások,
utolsó válasz; `~/.local/state/project-s/alugynokok/N.md`). Párhuzamosság, tartaléklánc és korlátok a psx felügyelőjéé (max 3 egyszerre).
Nyitott: a worktree-k takarítása (`git worktree prune`), az eredmények visszafésülése a főnök döntése.

## Beolvasztás (5. lépés, első változat – 2026-11-14)
A végrehajtók erősségei modellválasztási szabályként a psx-ben (`TIPUS_LANC`, `providers_for`): a lépés (vagy alügynök-feladat)
`tipus` mezője – `kod` → codex, claude, copilot, or-olcso; `kutatas` → claude, codex (DeepSeek nem, toti 2026-11-11);
`tomeg` → or-olcso, or-free. Kifejezett `providers` felülírja. A `loop` lépés a 3. körtől (2 bukás után) az utoljára használt
gyártó modelljeit a lánc végére teszi (`gyartovaltas`, esemény `loop_gyartovaltas`; kikapcsolás: `"gyartovaltas": false`).
Költségkeret: automatikus `max_cost_usd` csak az OpenRouter token alapú fizetős végrehajtójára (`or-olcso`, `FIZETOS_OPENROUTER`);
az előfizetésesek (claude, codex, copilot) névleges keretet kapnak (`max_elofizetes_usd`, alap 5 / `max_credits`), a CodeWhale
toti döntése szerint keret nélkül fut. Limitnél a meglévő átadás működik (próba: `t20261114-191218-6e7ee0` – codex limitben → claude, 1. kör, `accepted`).

## Bevált külső minták, amiket átveszünk
- *Ügynöki hurok és eszközmodell* (Claude Code, OpenHands, Aider): rövid, ellenőrző visszacsatolású ciklus; fájlszerkesztés célzott cserével; a teszt futtatása a hurok része.
- *Munkafa-izoláció* (git worktree minden feladatra): olcsó kísérletezés, könnyű visszavonás.
- *Kontextustömörítés + munkacsomag*: hosszú feladat megszakítás után folytatható (a psx munkacsomagja már ilyen).
- *Alügynök-delegálás* párhuzamos, szűk feladatokra; az eredményt a főnök ellenőrzi.
- *Függetlenül ellenőrzött eredmény*: gépi szabály ahol lehet, más gyártó modellje ahol nem.

## Mérés
Siker = a főnök egyedül, ember nélkül elvégez egy többlépéses feladatot (kód + teszt + élesítés vagy külső rendszer), és a napló mutatja. A `docs/KEPESSEGEK.md` mért képességei mellé kerül egy „főnök önállóan" szakasz.
**Mérve (2026-11-14 19:50 UTC, éles felügyelő, ember nélkül):** alügynök-csoport `fonok-onallo-20261114` – #1 `tipus: kod`
(`t20261114-195037-585fd4`: a codex lehűlésben → claude, kód + unittest, ellenőrző hurok 1. kör exit 0, `accepted`), #2 `tipus: tomeg`
(`t20261114-195037-ce1aac`: or-olcso, 1. kör exit 0, `accepted`, 0,0017 USD). A 11. tétel építőelemei (hurok, eszközjegyzék,
alügynökök, modellválasztás, önfejlesztés átnézéssel és élesítéssel) mind élesben futnak; a `KEPESSEGEK.md` „Főnök önállóan”
szakasza ezt naponta méri. Nyitott továbbfejlesztés: ügyhöz kötött hurok (`ugy inditas --hurok`), worktree-takarítás,
az alügynök-eredmények visszafésülése, böngésző/Gmail az eszközjegyzékben.

## Nyitott pontok
- A felhasználói systemd-busz elérhetősége (zárkikerülés, `docs/HP-DEV-HOZZAFERES.md`) teljes hozzáférésnél már nem hiba, de a cgroup-alapú hp-dev azonosítás gyengesége marad (a főnök tokent használ).
- Költségkeret: nincs hívásplafon; az OpenRouter/előfizetéses limitek természetes korlátok (a tartaléklánc kezeli).

## Kapacitásgazda (FONOK-BACKLOG 14., 1. kör, 2026-11-15)
`executor/kapacitas.py` – központi szolgáltató-nyilvántartás a meglévő adatokból (psx adapterek, `providers.json`, feladatnaplók):
gyártó, költségtípus (`penz` / `elofizetes` / `ingyenes` – külön, mint a két költségkönyv), erősség (`TIPUS_LANC`), elérhetőség,
lehűlés vége, 7 napos használat/limit/kész/költség, és a `waiting_model` miatti várakozási óra.
- `python3 executor/kapacitas.py tabla [--nap 7] [--json]`, `… valaszt <kod|kutatas|tomeg|-> [--kizar GYARTO,…]`
- **Tartaléklánc a psx-ben** (`psx.lanc`): ha a lépés minden saját szolgáltatója limites/lehűlésben van, a típus lánca
  (ismeretlen típusnál `claude`, `codex`) következik várakozás helyett. Kifejezett `providers` esetén csak `"tartalek": true`-val;
  `"tartalek": false` tiltja. A feladat más lépéseit elvégző gyártó kimarad (a független ellenőr nem kapja a szerző gyártóját).
  Az `ugy inditas` munkalépése kifejezett végrehajtó nélkül tartalékkal fut; toti által választott végrehajtónál nincs csere.
- Fizetős OpenRouter (pénz) tartalékként nem kerül be automatikusan. Mérőszám: várakozási óra/nap (indulás: 12,66 óra/7 nap).

## Helyi intelligencia (FONOK-BACKLOG 14., 2. rész, 1. kör, 2026-11-15)
- Felhasználói szintű llama.cpp (CPU, b12398 kiadás, `~/.local/opt/llamacpp/`), modellek `~/.local/share/project-s/modellek/`;
  szolgáltatás: `project-s-llama.service` (user, `install/systemd-user/`), csak `127.0.0.1:11434` – ez a psx `local` végpontja
  (a `modellek.json` `local` profilja, név: `qwen3-4b-instruct`). Sudo, GPU-meghajtó, hálózati változás nem kellett.
- Mérés (Xeon E5-2699 v3, 72 szál, `llama-bench`, pp512/tg64):
  | modell | szál | előfeldolgozás tok/s | generálás tok/s |
  |---|---|---|---|
  | Qwen3-4B-Instruct-2507 Q4_K_M (2,3 GiB) | 16 | 99 | 13,1 |
  | Qwen3-4B-Instruct-2507 Q4_K_M | 36 | 163 | 12,4 |
  | Qwen3-8B Q4_K_M (4,7 GiB) | 36 | 101 | 6,9 |
  Eszközhívás (`--jinja`) működik. Éles psx-próba: `t20261115-063442-5544ee` accepted, 1 kísérlet, 0 USD (egyszerű fájlművelet).
- A díjmentes szolgáltatókat (`kapacitas.KOLTSEG == "ingyenes"`: local, or-free) a pénzkeret nem tiltja és nem állítja le.
- Korlát: CPU-n lassú (kb. 12 tok/s) – rövid rutinlépésre, tartaléknak való; webkutatásra/nehéz kódra nem mértük.

## Tapasztalati memória (FONOK-BACKLOG 14/3, 1. kör, 2026-11-15)
`scripts/tapasztalat.py` – aktív memória a meglévő forrásokból (új alrendszer helyett az `ugy` és a kezdeményezés-nyilvántartás bővítése):
- **Kinyerés** (`kinyer`, 6 óránál régebbi tárnál automatikus): `ugyek/*/allapot.json` lezárt kimenetű feladatai (elfogadva → `bevalt`,
  elutasítva/hiányos/sikertelen → `sikertelen`) és a lezárt kezdeményezések tanulsága (`tanulsag`). Tétel: forrás, dátum, típus,
  környezet (`linux-toti-server`), állapot (`ervenyes`). Tár: `~/.local/state/project-s/tapasztalat/` (0700, repón kívül).
- **Titokszűrés helyben, modell nélkül:** `scripts/titok_ellenor.kitakar` (API-kulcs-minták + Bearer/JWT, `jelszó=` / `token=`,
  e-mail, IPv4, hosszú hex); ha kitakarás után bármi gyanús marad, a tétel nem kerül be.
- **Használat:** `ugy uj` a nyitáskor kiírja és `tapasztalat_nyitaskor` mezőbe menti; minden `ugy inditas|folytat` prompt
  „KORÁBBI TAPASZTALAT” szakaszt kap (más ügyekből, ≤6 tétel, ≤1800 karakter, idf-súlyos szóegyezés; a tételek >12%-ában előforduló általános szavak nem számítanak, ≥2 közös szó és ≥6 pont kell;
  a sikertelen megközelítés jelölve: „ne ismételd vakon”). Kikapcsolás: `UGY_TAPASZTALAT=0`.
- **Mérés:** `tapasztalat.py meres` (hány promptba került, hány tétel); a megtakarítás (kihagyott ismételt munka, hívás, idő, két
  költségkönyv) mérése a következő kör. Még nincs: ellentmondás-jelölés, régi (Windows-) archívum, ügygazda-kezdeményezés bekötése.
- **3. kör – „ez már volt” a tervezésben (ügygazda):** az önálló kezdeményezés tervét az ügygazda a tapasztalati memóriával
  összeveti (`tapasztalat.ismetles_gyanu`, ≥14 pont = erősen rokon; mért küszöb 54 ügyön). Találatnál a tervező EGYSZER visszakapja
  („ismétlés-gyanú – ez már volt: forrás [típus]: …”), és mást választ, vagy a forrásra hivatkozva indokolja az eltérést.
  Mérés: `~/.local/state/project-s/tapasztalat/megtakaritas.jsonl` (`ismetles_gyanu`, `elkerult_ismetles`, `elteres_indoklassal`),
  összesítés: `tapasztalat.py meres` → `tervezesi_jelzesek`.
- **4. kör – ellentmondás-jelölés (követelmény d):** a `kinyer` az ugyanarról szóló (a nem-sablon kulcsszavak Jaccard ≥ 0,5),
  ellentétes kimenetű tételeket (bevált/tanulság ↔ sikertelen) kölcsönösen jelöli (`ellentmond`), a régebbi `ellentmondasos`
  állapotot kap; egyik sem íródik felül. A prompt „⚠ ELLENTMONDÁS ezzel: …” jelzéssel adja át. Mért (257 tétel): 77 pár, 34
  ellentmondásos tétel, mind ügyön belüli (korábbi sikertelen próba ↔ későbbi elfogadott) – a sablonszűrő nélkül 581 hamis pár lett volna.
- **5. kör – felhasználás mérése és záró próba:** `tapasztalat.py proba` (exit 0 = teljesült) azt nézi, hogy egy VALÓDI psx-feladat,
  amely tapasztalatot kapott, a végső válaszában (`attempts/*/agent-*.txt`) hivatkozik-e a kapott forrás feladatára/ügyére
  (≥6 karakteres azonosító, szóhatárral), vagy kifejezetten a korábbi tapasztalatra. A `meres` `felhasznalta` mezője ugyanezt
  számolja. Állapot 2026-11-15 10:10 UTC: 1 valódi feladat kapott tapasztalatot, hivatkozás nincs → a próba `nem teljesült`.
- **Helyi modell minőségmérés (2026-11-15, 5 rutinfeladat, gépi elfogadás, 0 USD):** f3 (ERROR-sorok számlálása) és f4 (hibás függvény
  javítása) elfogadva 1. kísérletre; f1/f2/f5 nem dőlt el: (1) a párhuzamos kérések az egyhelyes szervert túlterhelték → `-np 3 -c 49152`;
  (2) a 60 s-os olvasási időkorlát CPU-n időtúllépést adott, amit a ciklus „overloaded”-nek vett és 10 percre letiltotta a `local`-t →
  szolgáltatónkénti `keres_timeout_s`/`olvasasi_timeout_s` (`modellek.json`, ollama: 600 s); (3) minőség: a 4B modell 12 fordulón át
  ugyanazt a hibás egysoros `python -c "…; with …"` parancsot ismételte. A `tomeg` láncba csak az élesített időkorlát utáni újramérés után kerülhet.
- **Újramérés az élesített javítások után (2026-11-15 10:31 UTC, `elo-20261115-103039`):** 3/5 elfogadva 1. kísérletre (f3, f4, f5),
  túlterhelés/letiltás nem volt. f2: hamis sikerjelentés (a modell kész munkát írt, a fájl nem jött létre – a gépi elfogadás kiszűrte);
  f1: 600 s alatt csak 4 modellkérés fért bele (CPU, 3 párhuzamos feladat, ~3,5k tokenes prompt) → időtúllépés.
  **Döntés:** a `local` NEM kerül automatikusan a `tomeg` tartalékláncba (60%, hamis sikerjelentés); kifejezett `providers: ["local"]`-lal,
  gépi elfogadási szabállyal, rövid rutinlépésre használható. Előrelépés: GPU (meglévő jóváhagyási úton) vagy erősebb modell mérése.
- **Független ellenőr tartaléklánca (2026-11-15):** az `ugy` ellenőr-lépése `tartalek_lanc: [codex, copilot, claude]`-ot kap (a lépésre
  szabott lánc felülírja a típusláncot; a szerző gyártója a `psx.lanc`-ban kimarad; DeepSeek nem). Ok: a Codex 2026-11-18-ig limites volt,
  az `ugy-cel-duplikacio-ellenor` ellenőrzése 3 napig várt volna, és ez az ügygazda (max. 1 saját ügy) minden új kezdeményezését is megállította.
- **6. kör – tervezési szintű felhasználás a záró próbában (2026-11-15):** a `proba` az elkerült ismétlést is számolja, ha a tapasztalat
  miatt valódi, önálló (`onallo`) új ügy indult; a megtakarítás az elkerült rokon ügy tényleges ráfordítása a psx-naplókból (feladatszám,
  pénz és előfizetéses érték külön, perc). **Élesben, magától (11:31 UTC):** az ügygazda `psx-dontes-ok-feltaras` terve 28,8 ponttal az
  elakadt `akadaly-ismetlodes-csoportosito`-ra hasonlított → a tervező másikat választott (`ellenorzok-regresszio-futtato`). Becsült
  megtakarítás: 3 feladat, 0 USD pénz, ~1,05 USD előfizetéses érték, ~185 perc (becslés: az ismétlés ugyanannyiba került volna).
  A végrehajtói szintű felhasználás (válaszban hivatkozás) 8 tapasztalatos promptból 0 – ez külön mérőszám marad.
