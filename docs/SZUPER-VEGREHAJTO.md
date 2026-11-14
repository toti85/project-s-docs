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
