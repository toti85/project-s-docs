<!-- Automatikus tükör a privát Projekt S műhelyrepóból (scripts/docs_tukor.py). Személyes és hálózati adatok kitakarva. Ne itt szerkeszd. -->

# Ügygazda – a Projekt S önálló megfigyelője és irányítója

_Állapot: 2026-11-12. Forrás: toti tartós megbízása (2026-11-12): „tartsátok működésben és fejlesszétek a Projekt S-t,
vigyétek tovább a bevételi és egyéb kijelölt ügyeket, építsétek meg a hiányzó képességeket”. A rendszer méretét a feladat
indokolja; a korábbi „nagy önálló AI-rendszert nem építünk” szabály ezzel megszűnt._

Bizonyítékcímkék: **[MÉRT]** friss mérés (dátum, környezet, futásazonosító) · **[TESZT]** automatikus teszt
(`scripts/tesztek.sh`) · **[KORÁBBI]** korábbi mérés/napló · **[LEÍRÁS]** csak dokumentált/tervezett, nem mért.

## Szerepek
- **Külső fejlesztő/irányító** (Claude Code munkamenet, toti megbízásából): fejleszti a rendszert, köztük az ügygazdát.
- **Ügygazda** (`scripts/ugygazda.py`, időzítő `project-s-ugygazda.timer`, 10 percenként): a szerveren önállóan futó
  irányító. A meglévő elemeket használja: `psx` (végrehajtó), `ugy` (ügykezelő), NEAR-munkás, mentés, értesítés.

## Három képesség
| Réteg | Mit csinál | Modellhívás |
|---|---|---|
| **figyel** (üzemeltetés) | szolgáltatások (`project-s-executor/panel/mcp/torrent`), időzítők (NEAR, mentés: ütemez-e, mikor futott), mentés kora + **napi helyi visszaállítási próba**, psx: beragadt `queued`, halott `running`, lejárt `waiting_model` (a még érvényes modellkorlát **indokolt várakozás**, nem elakadás), `waiting_human`; NEAR bizonytalan műveletek; döntésre váró eredmények; strukturált állapotexport a végrehajtóknak (`allapot/rendszer.json`) | nincs |
| **folytat** (ügyfolytatás) | önálló ügyekben (`szabalyzat.json: onallo_ugyek` + saját ügyek): elfogadott eredmény → tervező dönt: következő feladat / kész / toti kell; hiányos/bukott feladat → újrapróba **másik végrehajtóval** (előtte: az eredeti futás lezárult, külső NEAR-hatás nem történt; 72 óránál régebbi bukást nem próbál újra) | csak elfogadott eredménynél (eltérés), napi kerettel |
| **kezdeményez** | a tartós célokból (`bevetel`, `kepesseg`, `megbizhatosag`) legfeljebb **1 saját ügy** egyszerre, 12 óránként legfeljebb egy új; az első feladatnak gépi elfogadási szabálya kell | igen, szabályozott gyakorisággal |

## Elakadáskezelés
- Elakadásonként legfeljebb **3 érdemben eltérő** javítási módszer (pl. `restart` → `reset-failed+start` →
  `daemon-reload+restart`); ugyanaz a módszer nem ismétlődik, az azonos hiba ismétlődése nem új megoldás.
- Kimerüléskor: a ciklus lezárul, **akadályjelentés** (`~/.local/state/project-s/ugygazda/akadalyok/`), döntési sor,
  push-értesítés; az ügygazda más ügyeken dolgozik tovább. Ha a hiba megszűnik, magától lezárja.

## Önállóság és határok
- Kérdés nélkül: kutatás, tervezés, fejlesztés, tesztelés, rutinszerű helyreállítás, feladatindítás, egyértelmű
  ügyfolytatás. A NEAR-ajánlatadás és leadás a munkás meglévő felhatalmazásával megy (nincs új jóváhagyási lépés).
- **Toti engedélye kell:** új külső vállalás (új piac, fiók, szerződés), nem engedélyezett kiadás, saját felhatalmazás
  vagy elfogadási szabály módosítása, root-művelet (`rendszerlepes` – a jóváhagyást nem kerüli meg).
- **Saját felhatalmazás védelme:** a védett fájlokat (`ugygazda/szabalyzat.json`, `scripts/ugygazda.py`, `CLAUDE.md`,
  `executor/zar.py`, `executor/acceptance.py`, `scripts/rendszerlepes.py`, `scripts/near_kulso.py`) az ügygazda nem
  módosíthatja: minden általa indított feladatra automatikusan `parancs:… ugygazda.py vedett-ellenor <alap-commit>` szabály
  kerül → ha védett fájl változott, a feladat hiányos lesz, és toti döntésére megy. A tervező a védett fájlt érintő tervet
  eleve elutasítja. Minden változás git-commit → visszaállítható.
- **Elfogadás:** gépi szabállyal igazolható munka automatikusan elfogadódik (psx); a tartalmi mérlegelést igénylő
  eredményekhez a döntési sorban rövid javaslat + bizonyíték (gépi szabályok, jelentés, összefoglaló) áll.
- **Külső tartalom:** az ügygazda strukturált állapotot kap; a weboldalak és feladatleírások feldolgozását végrehajtókra
  delegálja (NEAR: Claude idegen-tartalom módban). A beágyazott utasítás nem felhatalmazás.

## Bizonyítékalapú elfogadás (toti/GPT, 2026-11-12)
A független ellenőr minden állításhoz szerepet ad: **dontesi** (ár, jogosultság, finanszírozás, vállalhatóság – mind
IGAZOLT kell, ellentmondás nélkül), **erdemi** (legfeljebb 20% ELLENŐRIZHETETLEN – induló működési küszöb), **feltaro**
(lezárható „tájékoztató” eredményként; ebből az ügygazda nem indít vállalást). Szabály: `allitasok:<ellenorzes.md>[:0.20]`
(`executor/allitasok.py`), kutatási feladatokra automatikusan. Bizonyítékhiány → célzott javítás. Nem nyilvános API-adat
(webes forrás nélkül) csak mentett válasszal igazolt: `allapot/api-valaszok/<forrás>/<UTC>-<rekord>.json` (14 nap).

## Futásidejű védelem (toti jóváhagyása, 2026-11-12)
Minden zárolt végrehajtó (a CodeWhale is) Landlock-zárban fut: titokmappák nem olvashatók, a védett fájlok
(`ugygazda/szabalyzat.json` listája + kötelezők) nem írhatók – nem csak utólag ellenőrizve. [MÉRT 2026-11-12]: titok olvasás
exit 1, védett írás exit 1, nem védett írás exit 0.

## Tartós felhatalmazás védett fájlokra (toti, 2026-11-13)
A védett fájl érintése önmagában nem kér új engedélyt: a már felhatalmazott feladatok módosításai az ellenőrzött élesítésen mennek át
(`scripts/elesites.py`: pontosan az átnézett commit, más gyártó független átnézése, zöld éles teszt, futó feladat nélkül, utólagos
szolgáltatás-ellenőrzés, automatikus visszaállítás), és az élesítő rögzíti a jóváhagyást (`ugygazda vedett-jovahagy`, „ellenőrzött
élesítés … toti tartós felhatalmazása”). Kivétel: a felhatalmazást rögzítő fájl (`jovahagyast_igenylo_fajlok`, jelenleg
`ugygazda/szabalyzat.json`) – ennek módosítása toti kifejezett jóváhagyását kéri (`elesit … --vedett-jovahagyva "<toti>"`).
Az ügygazda saját ciklushibáját is jelzi (döntési tétel + push), a következő sikeres ciklus feloldja; mindkét esemény a naplóban marad.

## Napló, összesítő, kikapcsolás
- Minden intézkedés: `~/.local/state/project-s/ugygazda/esemenyek.jsonl`, szereplő `ugygazda`.
- Napi összesítő (18 UTC után, push): `…/ugygazda/napi/<nap>.md` – figyelő körök, elakadások, sikeres javítások,
  indított/újrapróbált/kezdeményezett feladatok, ráfordítás (ismeretlen költség jelölve), NEAR-összesítő, döntési sor.
- Kikapcsolás: panel **/ugygazda** gomb vagy `ugygazda ki` → csak megfigyel és naplóz, nem avatkozik be. Vissza: `ugygazda be`.
- Parancsok: `ugygazda figyel | folytat | kezdemenyez [--most] | ciklus | napi [NAP] | allapot | dontesek | ki | be`.

## Kapcsolódó, ugyanebben a körben készült elemek
- **Visszaállítási próba** (`scripts/visszaallitas_proba.py`): a legutóbbi mentés elkülönített könyvtárba bontva;
  `psx verify-journal` minden feladatra a visszaállított útvonallal (`PSX_HOME`), `ugy lista` (`UGY_UGYEK`), teljesség
  (mentés előtti élő feladatok/ügyek, NEAR-állapot), élő állomány-ujjlenyomat előtte/utána; nyers kimenetek és kódok:
  `~/.local/state/project-s/visszaallitas/<idő>/`. Hiányos mentés → HIBA (kilépési kód 1).
  [MÉRT 2026-11-12 09:32 UTC, toti-server, archívum `project-s-20261112-031723.tar.gz`]: 55/55 napló rendben, 11/11 ügy
  listázható, hiány nincs, élő állomány nem változott. [TESZT/kézi]: elrontott napló + törölt feladat → HIBA, kód 1.
- **NEAR külső műveletek** (`scripts/near_kulso.py`): `kulso_szandek` → hívás → `kulso_eredmeny` / `kulso_bizonytalan`
  (op-azonosító, idempotencia-kulcs, külső azonosító, `kulso_hatas=True`); időtúllépés után **egyeztetés** (pl. saját
  ajánlatok listája), bizonytalanság esetén nincs vak újrabeadás; kikapcsolásjelző (`near-munkas ki|be`, panel) minden
  új külső művelet előtt; a már elküldött művelet egyeztetése kikapcsolva is megy; körzár a párhuzamos futás ellen; az
  értesítés hibája nem vált ki újrabeadást; napi összesítő: `near-munkas napi`. [TESZT] 7 teszt.
- **Ügyszintű eseménynapló** (`scripts/ugynaplo.py`, `ugyek/<ügy>/esemenyek.jsonl`): eseményazonosító, folytonos
  sorszám, szereplő, változás; kezdő esemény a meglévő ügyek jelenlegi állapotáról (korábbi történet nélkül); zár;
  `ugy naplo ellenoriz` → sorszám, egyediség, visszajátszás = allapot.json. Saját hash-lánc nincs; a git a commitolt
  történetet védi, ez nem teljes körű átírásvédelem. [TESZT] 5 teszt; [MÉRT 2026-11-12] 11/11 ügy rendben.

## A NEAR-felhatalmazás pontos terjedelme
A munkás önállóan: nyitott feladatok olvasása és besorolása; **ajánlattétel** (bármilyen pénznemben – toti, 2026-11-12; USD-hez a NEAR Stripe-bekötést kér, addig a munkás naponta újrapróbál; kutatási típusú, „vállalható”
feladatra, Claude-írta + Codex-ellenőrzött szöveggel, a feladat kiírt díjával); ajánlathoz kiegészítő üzenet; elnyert munka
indítása, haladásjelzés, elkészítés, ellenőrzés és **leadás** (csak ha a vevő feltételei, az idézetek és az alátámasztás
mind teljesülnek); saját ajánlat visszavonása (panelről). **Nem:** pénz befizetése, vevői művelet (feladás, kiosztás,
elfogadás), vita, új fiók/piac. **Keret:** dollárplafon nincs (toti, 2026-11-11); korlát a körönkénti legfeljebb 10 új
ajánlat. A korábbi „kereten belül” megfogalmazások erre értendők.

### Saját ciklushiba jelzése

A figyelő ellenőrzi a `project-s-ugygazda.service` `Result` értékét. Hibás eredménynél,
vagy a ciklusból kiszökő kivételnél azonnal ment egy `akadaly:ugygazda-ciklus` döntési tételt,
és egyszer push-értesítést küld. A `/ugygazda` döntési sorában megjelenik a hiba és a
`journalctl --user -u project-s-ugygazda.service -n 20` titokszűrt vége.
A tétel teljes sikeres ciklus után automatikusan eltűnik; a folyamatban lévő ciklus
`Result=success` értéke önmagában nem törli. A figyelő külön futtatva a már befejezett,
sikeres systemd-egység alapján is töröl. Nem olvasható systemd-állapot nem igazol sikert.
A kivétel továbbra is hibás kilépést okoz; a jelzés nem rejti el a systemd elől.
