<!-- Automatikus tükör a privát Projekt S műhelyrepóból (scripts/docs_tukor.py). Személyes és hálózati adatok kitakarva. Ne itt szerkeszd. -->

# Ügykezelés – telefonról indított, tartós munka

## Folyamat (2026-11-08)
```
toti (panel: ügy → Folytatás / beszélgetés: „indítsd” / SSH: ugy inditas / ChatGPT: MCP)
  ▼
ugy inditas <ügy> "<feladat>" [--elvart …] [--ellenorzes …]     scripts/ugy.py  (~/bin/ugy)
  │  allapot.json: feladat felvéve (fut); UGY.md napló; commit+push
  │  prompt: ügy célja, következő lépés, legutóbbi eredmények, olvasható források, kötelező jelentésszerkezet
  ▼
psx feladat (executor/psx.py) – lépés: munka [+ ellenor: másik gyártó modellje]
  │  végrehajtó: or-olcso | claude | codex | copilot | codewhale | or-free; zár, keret, folytathatóság
  ▼
elfogadási kapu (letezik, tartalmaz…, idezetek, nem-tartalmaz CÁFOLT) → hook: ugy _psx_szinkron
  │  eredmenyek/<id>.md, allapot.json (vegrehajtva → elfogadva | elfogadasra_var | hianyos), UGY.md; commit+push
  ▼
toti: ugy elfogad | elutasit  (vagy panel)          ugy allapot <ügy> – rövid beszámoló
```
A munkamenet a feladat alatt bezárható; az állapot a szerveren és a GitHubon marad. Részletek: `docs/RENDSZER.md`,
`docs/VEGREHAJTO.md`. (A 2026-11-04 előtti feladatok még a régi `ps-job-<id>` / `mcp/jobs.py` úton futottak.)

## Ügy szerkezete
```
ugyek/<azonosito>/
  allapot.json   gépi állapot (forrás): id, nev, cel, allapot, kovetkezo_lepes,
                 utolso_osszefoglalo, feladatok[{job_id, feladat, allapot, kezdes, vege,
                 osszefoglalo, eredmeny}], frissitve
  UGY.md         ügyirat: generált fejléc (<!-- ALLAPOT:... -->), Anyagok, Napló (utolsó szakasz)
  eredmenyek/    <job_id>.md – mentett eredmények
  munka/         a feladatok munkamappája (pl. nyers adatok)
```
Ügyállapotok: `nyitott` · `fut` · `elfogadasra_var` (van döntésre váró feladat) · `elakadt` (a végrehajtás
sikertelen) · `lezart`.

## Végrehajtva ≠ elfogadva (2026-11-04)
A blueprint `EXECUTION_RECORDED ≠ ACCEPTED` elvének kicsi változata. Indításkor rögzítjük:
`--elvart` (mi az elvárt eredmény), `--hova` (hova kerül), `--ellenorzes` (hogyan döntjük el, hogy megfelel).
```
fut → vegrehajtva → elfogadva        minden gépi ellenőrzés teljesült, és nincs „ember” szabály
                  → elfogadasra_var  gépi ellenőrzés rendben, emberi döntés kell (szubjektív eredmény; alap)
                  → hianyos          legalább egy ellenőrzés nem teljesült
    → sikertelen                      a végrehajtó hibával / időtúllépéssel / leállítva ért véget
elfogadasra_var | hianyos → elfogadva | elutasitva     (ugy elfogad / ugy elutasit [--ki …])
```
- A végrehajtó sikeres kilépése csak `vegrehajtva`; a lezárás (`_lezaras`) **a végrehajtótól függetlenül**
  ellenőrzi a tényleges eredményt: a `--hova` fájl megléte mindig, továbbá a szabályok
  (`letezik:`, `min-meret:`, `tartalmaz:`, `json:út:kulcsok`, `parancs:` a `munka/`-ban, `ember`).
- A végrehajtó „Ügyállapot” szakasza csak javaslat (`javasolt_ugyallapot`), nem zárja le az ügyet.
- Az ellenőrzés eredménye a feladatnál (`ellenorzes_eredmeny`) és a jelentés végén is szerepel.
  Újrafuttatás: `ugy ellenoriz <ügy> [job_id]`.
- A közvetlen munkamenetes munka (`ugy rogzit`) `elfogadasra_var` állapotba kerül.
- Régi (2026-11-03-i) feladatok: `vegrehajtva` állapotra migrálva, „elfogadás nem volt rögzítve” megjegyzéssel.
- Bemutató: `ugyek/proba-elfogadas/` – 1 automatikusan elfogadott (JSON + értéktartomány), 1 hiányos → elutasított.

A feladat-prompt (`prompt_keszit`) tartalmazza az ügy célját, a rögzített következő lépést és
az utolsó 3 feladat összefoglalóját; a válasz végén kötelező szakaszok: `## Összefoglaló`,
`## Következő lépés`, `## Ügyállapot` – ezekből frissül az ügy.

## Honnan jön (project-s-unified)
A Continuous Mind `goal_queue.json` cél-sémáját és állapotlogikáját, valamint az executor
„prompt → futtatás → kimenet fájlba → ellenőrzött lezárás” mintáját vettük át, kód nélkül.
Részletek: `ugyek/project-s-ujjaepites/UGY.md`.

## Futási adatok és mentés
- Feladatok: `~/.local/state/project-s/jobs/<id>/` (meta, state, progress.log, output.log, result.json,
  on_done.log); audit: `~/.local/state/project-s/audit.log`.
- Az ügyek (eredményekkel) a git-repóban → GitHub.
- Mentés: `scripts/mentes.sh` – napi tar.gz a `~/mentes/` alá (14 napig), a
  `project-s-mentes.timer` (user) futtatja. Visszaállítás: `tar -xzf <fájl> -C ~`.
  Titkokat (`~/.config/project-s`) **nem** ment.
