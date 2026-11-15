<!-- Automatikus tükör a privát Projekt S műhelyrepóból (scripts/docs_tukor.py). Személyes és hálózati adatok kitakarva. Ne itt szerkeszd. -->

# Főnök-teendőlista (toti, 2026-11-13 éjjel: „amíg alszom, menjen minél több automatikusan”)

Sorrend szerint haladj; egyszerre egy tétel; minden tétel: fejlesztési példány → teszt (scripts/tesztek.sh ZÖLD) → commit → a rendszer
független átnézést indít és az ügygazda automatikusan élesít. Hatáskör: kutatás, tervezés, fejlesztés, tesztelés, rutinszerű
helyreállítás, feladatindítás. NEM a tiéd: pénz/kifizetés, új fiók/regisztráció/külső vállalás, root-lépés, a felhatalmazást rögzítő
ugygazda/szabalyzat.json módosítása. Ha elakadsz: konkrét akadályjelentés a válaszodban, majd a következő tétel.
Válaszod végén mindig: „Következő lépés: …”. Ha minden tétel kész vagy elakadt: „BACKLOG KÉSZ”.

1. **Tesztcsomag a zárban is zöld.** A független átnézők szűk zárban futtatják a scripts/tesztek.sh-t, és ott néha piros
   (ModuleNotFoundError, útvonalak, ideiglenes mappa). Futtasd a csomagot a szűk zárban (`python3 executor/zar.py --ir <példány> --ir /tmp
   --olvas … -- scripts/tesztek.sh`), és tedd robusztussá: relatív/`__file__` alapú útvonalak, TMPDIR használata, nincs beégetett éles út.
   **Elkészült (2026-11-13, fejlesztési példány; független átnézésre vár):** a tesztfuttató saját, takarított
   TMPDIR-t és Python-gyorsítótárat használ; tiltott örökölt TMPDIR esetén /tmp alá vált. Az offline próbák
   nem írnak a repó gyökerébe, a helyi HTTP-jelentésteszt nem függ a telepített böngészőtől.
   Reprodukálható szűk záras ellenőrzés (a példány gyökeréből):
   `python3 executor/zar.py --ir "$PWD" --ir /tmp --olvas "$PWD" -- scripts/tesztek.sh`.
   Az `--olvas` szükséges: a védett fájlokat tartalmazó könyvtárakra a zár nem ad teljes írásjogot,
   ezért az önálló olvasási engedély biztosítja a könyvtárlistázást és a Python-importokat.

2. **Reggeli jelentés toti-nak.** `scripts/reggeli_jelentes.py`: rövid magyar összefoglaló az elmúlt 12 órából (önállóan elfogadott
   feladatok, élesítések/visszaállítások az elesites.jsonl-ból, ügygazda-elakadások és döntési sor, NEAR: ajánlat/kiosztás/üzenet/
   kifizetés, hibák, főnök-körök), push-értesítéssel (ertesites.background) és fájlba (~/.local/state/project-s/reggeli/<nap>.md).
   Teszttel. Az ügygazda ciklusa hívja naponta egyszer 05:30 UTC után (ugygazda.py ciklus – a változtatás az ellenőrzött élesítésen megy át).
   **Elkészült (2026-11-13, fejlesztési példány; ellenőrzött élesítésre vár):**
   `scripts/reggeli_jelentes.py` az elmúlt 12 órát összegzi, a hiányos forrásokat jelzi.
   Az ügygazda 05:30 UTC után meghívja; fájlzár és a kész napi fájl akadályozza a többszörös futást.
   A push egyszeri, best effort; célja az Ügygazda paneloldal. A teljes jelentés:
   `~/.local/state/project-s/reggeli/<nap>.md`. Kikapcsolt ügygazda nem indít jelentést.
   Tesztek: időablak, kifizetési státusz, emberi elfogadás kizárása, forráshibák, újraindítás,
   párhuzamosság, értesítési/írási hiba és ciklusbekötés (hálózat nélkül).

3. **OpenRouter főnök-modellek próbája.** A katalógusból (`openrouter_katalogus.lista()`) válassz ~20 eszközképes, népszerű/olcsó
   modellt (pl. openai/gpt-5*, moonshotai/kimi-k*, qwen coder, deepseek, google/gemini), és futtasd rájuk `proba()`-t; összesen legfeljebb
   1 USD. A panel `/c/modellek` így mutatja az igazoltakat. Eredmény: rövid jelentés a válaszodban.
   **AKADÁLYOZOTT (2026-11-13):** a meglévő kliens hitelesítés-ellenőrzése a fejlesztési környezetből
   `PermissionError` hibát ad. Fizetős hívás nem indult (0 USD); 20 katalógusjelölt előkészítve,
   működésük nincs igazolva. A `proba()` token-/összköltség-korlát nélkül fut, ezt a legfeljebb 1 USD-s
   próba előtt rendezni kell. Részletek: `docs/OPENROUTER-FONOK-PROBA-2026-11-13.md`.

4. **Dokumentáció.** docs/UGYGAZDA.md és docs/RENDSZER.md: a főnök-folyamat (fejlesztési példány, átnézés, automatikus élesítés,
   visszaállítás, reggeli jelentés), a FOLYTATAS.md „hol tartunk” frissítése.
   **Elkészült (2026-11-13, fejlesztési példány):** `docs/UGYGAZDA.md`, `docs/RENDSZER.md`,
   `FOLYTATAS.md` frissítve a kódban ellenőrzött folyamattal, az ághoz kötött átnézés feltételével,
   élesítés/visszaállítás korlátaival és a reggeli jelentés részleteivel. A fejlesztési és éles állapot külön jelölve;
   az első két kör elmaradt élesítésének oka a naplóból ellenőrizve.

5. **Videó (ügy video-shorts).** A double-wish renderelés a 668. képkockánál megszakadt: `~/video-shorts/blender/render-dw.sh`
   folytatható (kész képeket kihagyja). Indítsd `nice -n 19`-cel háttérben (nohup), figyeld, majd `finish-dw.sh`; igazold, hogy a
   double-wish.mp4 létrejött (ffprobe: hossz, felbontás). Ez nem repóváltozás; az eredményt ugy rogzit-tel rögzítsd a video-shorts ügyben.

   **AKADÁLYOZOTT / korábbi készre jelentés (2026-11-13):** az éles ügyiratban a videó már
   2026-11-12 17:57-kor készre jelentett. Most a két szkript olvasása és mindkét MP4 `ffprobe` vizsgálata
   `Permission denied` hibával állt le; friss hossz/felbontás nem igazolt, új render nem indult.
   Részletek: `ugyek/video-shorts/eredmenyek/2026-11-13-backlog-ellenorzes.md`.
   A friss ellenőrzéshez a médiát jogosultan elérő meglévő környezet kell, nem új render vakon.

**A kör vége:** 1., 2., 4. fejlesztési tétel elkészült; 3. és 5. akadályozott.
A fejlesztések élesítése külön feltételekhez kötött; az akadályozott tételek nincsenek késznek minősítve.

6. **Az ügygazda saját ügyeinek hibás „hova”-ja (új, 2026-11-13 03:15).** Az éjjel mind a három saját kezdeményezésű ügy (near-leadas-csomagellenor,
   near-ciklus-elszamolas, near-kivalasztas-proba) kimerítette a javítási keretet, mert a tervező által megadott „hova” nem fájl:
   `eredmenyek/` (mappa), `eredmenyek/near-ciklus-elszamolas` (kiterjesztés nélkül); ilyenkor a `letezik:` és az automatikusan hozzáadott
   `allitasok:<…>-ellenorzes.md` szabály sosem teljesülhet (igazolt 0, cáfolt 0, ellenőrizhetetlen 0). Javítsd: (a) `ugygazda.py`
   `terv_ervenyes` és a kezdeményezés érvényesítése: a „hova” csak relatív FÁJLÚT lehet kiterjesztéssel, nem végződhet „/”-re (hibaüzenet a
   tervezőnek, a meglévő egyszeri újrakérdezéssel); (b) `ugy.py inditas`: az automatikus független ellenőr/állításkapu csak `.md` jelentésre kerül;
   kódot/adatot (py, json) eredményül adó feladatnál a gépi szabály a futtatás (parancs:) legyen. Teszt: a három eredeti hibás „hova” esetre.
   Mielőtt dolgozol: `git pull --no-edit origin main` a fejlesztési példányodban (az éles main azóta előrébb van).

   **Elkészült (fejlesztési példány, 2026-11-13):** közös relatív fájlút-ellenőrzés kiterjesztéssel;
   hibás tervnél a meglévő egyszeri újrakérdezés kapja a pontos hibaüzenetet, kezdeményezésnél is.
   Nem `.md` eredményhez `parancs:` kell; automatikus Markdown-szerkezet, idézetkapu és szöveges
   ellenőr/állításkapu csak `.md` jelentésre kerül. Meglévő mappára mutató fájlnév sem fogadható el.
   Regresszió: `scripts/test_celut.py` (a három eredeti ügy célútja, újrakérdezés mindkét tervezői ágon,
   indítás `.md`/`.py`/`.json` eredménnyel). Pontosítás: a `near-kivalasztas-proba` tényleges célútja
   `eredmenyek/kivalasztas-proba.md`, ez szabályos és továbbra is elfogadható; az első két ügy útja hibás.
   A régi ügyek eredményeit nem minősítettük át és javítási keretüket nem állítottuk vissza.

7. **Az automatikus élesítés ideiglenes akadályai nem lehetnek végleges hibák (új, 2026-11-13 04:00).** Éjjel egy elfogadott átnézés
   órákig nem élesedett: (a) a napi generált `docs/KEPESSEGEK.md` (mentes.sh → kepessegek.py) nem commitolt módosításként a kódkönyvtár
   tisztaságvizsgálatán megbuktatta az élesítést; (b) egy kézi/automata próbálkozás „nem commitolt változás” miatt `auto_vegleges_hiba`
   eseményt írt, és az `auto()` ettől az átnézést örökre kihagyta. Javítsd az `elesites.py`-ban: az ideiglenes okok (nem commitolt
   változás, futó feladat, ütköző helyi elem) ne legyenek végleges hibák (újrapróba a következő körben, a naplóban látható okkal, ismétlődő
   azonos ok 6 óra után értesítés); a generált `docs/KEPESSEGEK.md` ne akadályozza az élesítést (a tisztaságvizsgálat kihagyja, vagy a
   kepessegek.py commitolja magát – válaszd a kisebb kockázatút, teszttel). Előtte: `git pull --no-edit origin main`.

   **Elkészült (fejlesztési példány, 2026-11-13):** az átmeneti akadály kilépési kódja 2,
   az `auto()` nem véglegesíti, hanem minden következő körben próbálja, okkal naplózva (`auto_varakozas`).
   Ugyanaz az akadálytípus 6 óra után egyszer szálüzenetet és push-jelzést kap; okváltás új időablakot indít,
   a várakozás/értesítés állapota naplóból visszaáll. A régi, ismert átmeneti okból tévesen véglegesített
   hibák újrapróbálhatók; a valódi végleges hibák és jóváhagyási korlátok megmaradnak.
   A tisztaságvizsgálat csak a generált `docs/KEPESSEGEK.md` fájlt hagyja ki; azt nem commitolja és nem törli.
   Ütköző helyi tartalmat a Git továbbra sem írhat felül (ilyen ütközésnél várakozik).
   Teszt: valódi ideiglenes Git-repókban leltármegőrzés, piszkos kód/ignorált fájl utáni sikeres újrapróba;
   hamis órával naplóból folytatott várakozás, okváltás, egyszeri 6 órás jelzés és régi hibák feloldása.

8. **SÜRGŐS – a végrehajtók zárában a repó testvér-moduljai nem importálhatók (2026-11-13 22:00).** Reprodukálható: `python3 executor/zar.py --teljes
   --ir $PWD/kozos -- bash -c 'cd $PWD && ls scripts; python3 -c "import sys;sys.path[:0]=[\"scripts\",\"executor\"];import celut,web,psx"'` →
   „cannot open directory 'scripts'" / ModuleNotFoundError: celut, web, psx. Ok: az `irhato_agak` (executor/zar.py) a védett fájlt tartalmazó
   szülőket (repó gyökere, scripts/, executor/) szétbontja, de magukra a szülőkre nem ad olvasási/listázási jogot (READ_DIR), csak a gyerekekre.
   Következmény: a zárban futó végrehajtók és független átnézők nem tudják futtatni a tesztcsomagot („PIROS”), miközben a psx elfogadási
   `parancs:` szabálya zár nélkül fut („exit 0”) – ez az ellentmondás. Javítás (egy másolaton már igazolva): `irhato_agak` gyűjtse a szétbontott
   szülőket (`szulok` lista), és az `alkalmaz` adjon rájuk `olvas` jogot (írás nélkül). Elvárt: (a) importok és `ugy lista` működnek a zárban;
   (b) a védett fájl írása, új fájl létrehozása a védett fájlt tartalmazó szülőben továbbra is tiltott; (c) a nem védett meglévő fájl írható;
   (d) REGRESSZIÓS TESZT a tesztcsomagban (scripts/tesztek.sh): `zar.py --teljes -- python3 -c "import celut, web, psx, ugy"` a repó gyökeréből
   és `ls scripts`, hogy a jövőben a zár-hiba a zár nélküli elfogadási futásban is pirosat adjon; (e) tesztek.sh a zárban is ZÖLD.
   Előtte: `git pull --no-edit origin main`. Második rész (9. tétel lent) külön commit.
   **Elkészült (fejlesztési példány, 2026-11-13):** az `irhato_agak` az írható ágak mellett a szétbontott
   szülőket is visszaadja; az `alkalmaz` ezekre csak olvasás/listázás/futtatás jogot ad, írásjogot nem.
   Új regresszió: `scripts/test_zar_import.py`, a `scripts/tesztek.sh` része. Valódi Landlock alatt teszteli
   a `--teljes` importokat (`celut`, `web`, `psx`, `ugy`), az `ls scripts` és az `ugy lista` futását,
   továbbá elkülönített fixture-rel a védett írás/csere és új fájl tiltását, a meglévő nem védett fájl írhatóságát.
   A beágyazott teljes zár próbája explicit `--ir <példány>` utat használ, külön `--olvas <példány>` nélkül;
   a külső zár home-listázási korlátozását a belső zár nem tudja feloldani. A pusztán `--ir kozos` alak itt
   emiatt továbbra sem elég; ez különbözik a szétbontott szülők most javított READ_DIR-hiányától.
   Teljes csomag normál és szűk záras futása ellenőrizve; a szűk próba külön repo-olvasási engedély nélkül:
   `python3 executor/zar.py --ir "$PWD" --ir /tmp -- scripts/tesztek.sh`.
   A mentési hibát (9. tétel) ez a commit nem módosítja.

9. **Visszaállítási próba: abszolút symlink a mentésben.** A napi próba (scripts/visszaallitas_proba.py) 2026-11-13 03:17 óta HIBA:
   `AbsoluteLinkError … ugyek/titok-vedelem/munka/fixture/alias is a link to an absolute path` (a titokvédelmi próba fixture-je került a mentésbe).
   Javítás: a kibontás az abszolút symlinkeket NE hozza létre (kihagyja, naplózza: név + cél), minden mást a `data_filter` szerint; a hiba csak akkor
   legyen HIBA, ha nem symlink-szűrési eredetű. Teszt: szintetikus tar abszolút linkkel (kihagyva, naplózva, a többi ép), valamint a valódi
   archívum. A korábbi gazdátlan próbálkozás: ugyek/mentes-abszolut-link-javitas/munka/gazdatlan-modositas-visszaallitas_proba.patch (referencia, nem
   átveendő: a valódi archívumra nem hatott).
   **Kódjavítás elkészült; valódi archívumos ellenőrzés AKADÁLYOZOTT (2026-11-13):**
   a kibontás minden abszolút symlinket kihagy, névvel és céllal naplózza a `kibontas.jsonl`-ben és
   az `osszegzes.json` `kibontasi_naplo` mezőjében. Nincs fixture-engedélylista vagy külön CLI-kapcsoló.
   A tag saját útvonala is `data_filter`-ellenőrzést kap; a relatív kilépő link, abszolút hardlink és
   egyéb szűrési/naplózási hiba továbbra is hibát okoz. Az eredeti teljességi és naplóellenőrzések megmaradnak.
   Regresszió: `scripts/test_visszaallitas_proba.py` (6 teszt), a teljes tesztcsomagba bekötve;
   szintetikus teljes archívum öt valódi psx-ellenőrzéssel és `ugy lista` futtatással RENDBEN,
   elrontott naplóval HIBA. A valódi `/home/sagent/mentes` könyvtár olvasása `PermissionError` (errno 13),
   ezért a valódi archívumon a javítást ebből a fejlesztési környezetből nem ellenőriztem.
   Az éles napi próba eredményét ez a fejlesztési futás nem minősíti át; jogosult környezetből újra kell futtatni.

10. **Figyelő: nem commitolt kódváltozás az éles repóban.** Ha a kódkönyvtárakban (executor/scripts/mcp/install/docs/ugygazda) nem commitolt követett
   módosítás van 15 percnél tovább, az ügygazda jelezzen (döntési tétel + egyszeri értesítés, a módosított fájlok listájával) – nem blokkoló.
   Ez a helyzet órákig akadályozta az automatikus élesítést.

   **Elkészült (fejlesztési példány, 2026-11-13):** a figyelő Gitből, csak olvasással méri a követett
   index- és munkafamódosításokat a hat megadott könyvtárban; az új, nem követett fájlok nem tartoznak ide.
   Fájlonként menti az első észlelés idejét, 15 percnél tovább fennálló módosításnál `figyelmeztetes`
   döntési tételt és egyszeri push-jelzést készít; a teljes aktuális fájllista a döntési sorban van.
   Tiszta repónál a jelzés és a mérés törlődik, új epizód ismét jelezhet. Mérési hiba nem jelent tisztaságot;
   nem törli a korábbi figyelmeztetést. A figyelő nem commitol, nem javít és nem állít le ügyet.
   Az `ugygazda` push-azonosítót a feldolgozó most elfogadja; a jelzés a `/ugygazda` oldalra mutat.
   Regresszió: 15 perces határ, egyszeri/tartós értesítési állapot, fájllistaváltás, tisztulás/új epizód,
   mérési hiba melletti továbbműködés, valódi Gitben staged/unstaged/törlés/átnevezés és különleges fájlnevek.

11. **Szuper-végrehajtó (toti, 2026-11-14; terv: `docs/SZUPER-VEGREHAJTO.md`).** A főnök TELJES hozzáféréssel dolgozik (zár nélkül, titkok és sudo elérhetők), nincs hívásplafon; haladj, ne ellenőrizz túl.
   Építési sorrend a tervben (2. közös ügynöki hurok → 3. eszközök közvetlenül → 4. alügynökök → 5. beolvasztás → 6. önfejlesztés). Egy kör = egy kicsi, működő, tesztelt lépés
   (kód a fejlesztési példányban, `scripts/tesztek.sh` zöld, commit; az élesítés ellenőrzött). Első lépés: mérd fel a meglévő adaptereket (`executor/agent.py`, `modell.py`, `beszelgetes.py`
   agy_*), írd le a közös hurok minimális kialakítását `docs/SZUPER-VEGREHAJTO.md` „Hurok-terv" szakaszába, majd valósítsd meg a legkisebb használható változatot a psx-ben (egy új, opcionális
   `kind: "loop"` lépés), és igazold egy valódi többlépéses feladattal. Maradó szabályok: titok nem kerül a repóba, pénz/regisztráció/külső beküldés/OVH/tűzfal toti döntése, laptop leállítása előtt egyeztetés.

   **1. kör elkészült (fejlesztési példány, 2026-11-14; ellenőrzött élesítésre vár):** felmérés és „Hurok-terv” a
   `docs/SZUPER-VEGREHAJTO.md`-ben; a psx új, opcionális `kind: "loop"` lépése (`goal`, `check`, `max_rounds`): ügynök
   (meglévő tartaléklánc) → ellenőrző parancs → bukásnál exit-kód + kimenet vissza a következő körbe; körök naplózva
   (`loop_round`/`loop_check`), naplóból folytatható. Teszt: `executor/tests/test_loop.py` (4, hamis ügynökkel).
   Valódi próba (or-olcso, külön PSX_HOME): `t20261114-165551-0ed2aa` 1 körben, `t20261114-165615-3433a0`
   átnézői visszajelzés után a 2. körben javított → mindkettő `accepted`, összesen ~0,0024 USD.
   Következő kör: eszközjegyzék a hurok alá (3. lépés) / `ps-chat inditas` hurokkal.
   **2. kör elkészült (fejlesztési példány, 2026-11-14):** `ps-chat inditas … --hurok "<ellenőrző parancs>" [--korok 3]`
   ügy nélküli indításnál psx `loop` lépést küld (legfeljebb 10 kör); ügyhöz kötve egyelőre egyértelmű hibát ad.
   Leírás: `docs/ASSZISZTENS.md`; teszt: `executor/tests/test_beszelgetes.py` (+2). Az élesítés feltétele a psx `loop`
   lépés élesítése is (az éles psx enélkül elutasítja). Következő kör: eszközjegyzék a hurok alá (terv 3. lépés).
   **Élesítve (2026-11-14 17:10 UTC):** `f0059b6` (átnézés `t20261114-170140-171730` elfogadva), címke
   `elo-20261114-171059`, visszaállítás: `scripts/elesites.py visszaallit elozo-20261114-171059`. Éles próba az éles
   felügyelőn: `t20261114-171158-7dd44b` (or-olcso) – 1. kör exit 5 (átnézői kérés), 2. kör exit 0 → `accepted`.
   **3. kör elkészült (fejlesztési példány, 2026-11-14):** eszközjegyzék `executor/eszkozjegyzek.py` (9 eszköz, egységes
   hívás és JSONL-napló titokkitakarással, OpenAI tools-export, CLI). Teszt: `executor/tests/test_eszkozjegyzek.py` (5).
   Valódi próba: shell, `ugy lista`, laptop-híd (hostname/uptime) ok; webes keresés ekkor kapcsolat-visszautasítást kapott
   (helyesen `ok: false`). Következő kör: a főnök (`beszelgetes.py`) / hurok ezen a jegyzéken keresztül hív eszközt.
   Az 1. átnézés (`t20261114-172244-7694a3`) elutasította: a szóközzel / JSON-ban megadott titok (`password foobar`,
   `"password": "x"`) nem volt kitakarva. Javítva + regressziós teszt; a `sys.path`-duplikálás és a sémaépítő mellékhatása is.
   A 2. átnézés (`t20261114-173256-71a9bf`) a kulcsszó nélküli, puszta titkot kifogásolta: a napló most a szerver ismert
   titkainak pontos értékét is kitakarja (`~/.config/project-s/**/*.env`, titoknevű környezeti változók; csak memóriában).
   Naplóépítési hiba sem dob; a `fajl_olvas` csak a kért sorokig olvas. A „halott teszt” kifogás téves (az érték szerepel a bemenetben).

12. **Dokumentáció naprakészen tartása (toti, 2026-11-14).** A 11. tétel MINDEN körénél, ha viselkedés/felület változik, ugyanabban a commitban frissítsd az érintett leírást
   (`docs/SZUPER-VEGREHAJTO.md`, a főnök kontextusa a `beszelgetes.py`-ban). A 11. tétel lezárásakor (vagy ha a `docs/RENDSZER.md` 7 napnál régebbi, és a kód közben változott): frissítsd a
   `docs/RENDSZER.md`-t (zár nélküli főnök, `loop` lépés, eszközjegyzék, alügynökök, ami készült), a `docs/HP-DEV-HOZZAFERES.md` és a `docs/KEPESSEGEK.md` hivatkozásait; futtasd a `scripts/docs_tukor.py`-t
   (nyilvános tükör, titokellenőrzéssel), és a válaszban sorold fel, mi frissült. Az elavult leírás hiba, nem kozmetika.
   **11. tétel, 3. kör – átnézve:** `t20261114-174325-28604f` elfogadva; az élesítés összefésülési ütközés miatt nem futott
   (a `main` közben a 12. tétellel bővült). Összefésülve, a főnök kontextusa (`beszelgetes.py`) a hurokkal és az
   eszközjegyzékkel frissítve (12. tétel szerint) → új átnézés, utána élesítés.
   **3. kör élesítve (2026-11-14 18:11 UTC):** `d4762d0` (átnézés `t20261114-180622-21f54a`), címke `elo-20261114-181144`;
   éles próba: `eszkozjegyzek.py lista` 9 eszköz, `hiv ugy lista` ok, naplózva.
   **4. kör elkészült (fejlesztési példány, 2026-11-14):** a saját ügynöki ciklus teljes jognál a jegyzék `psx`/`ugy`
   eszközét használja (egységes napló, szándékrekord, megszakadás után nem ismétel; laptop kimarad). Teszt:
   `executor/tests/test_agent_jegyzek.py` (4). Valódi próba: `t20261114-182144-51c740` (or-olcso, `loop` lépés, teljes jog) –
   az `ugy lista` eszközhívásból írt fájl az 1. körben átment, `accepted`, 0,014 USD. Dokumentáció: `docs/SZUPER-VEGREHAJTO.md`.
   **4. kör élesítve (2026-11-14 18:36 UTC):** `ab275c5` (átnézés `t20261114-182544-19a0c3`), címke `elo-20261114-183611`.
   **5. kör elkészült (fejlesztési példány, 2026-11-14):** alügynök-csoportok `executor/alugynok.py` (indit/allapot/begyujt,
   worktree-izoláció, `loop` ellenőrzéssel) + `alugynok` eszköz a jegyzékben (csak a főnöknek); a saját ciklus rendszerpromptja
   teljes jognál helyesen írja le a `run`-t (az előző átnézés megjegyzése). Teszt: `executor/tests/test_alugynok.py` (3).
   Valódi próba az éles felügyelőn: csoport `proba-20261114-1` – `t20261114-184148-b9a545`, `t20261114-184148-ab0f0f`
   párhuzamosan, mindkettő `accepted`, összesen 0,0017 USD. Főnök-kontextus és `docs/SZUPER-VEGREHAJTO.md` frissítve.
   Az átnézés (`t20261114-184352-90837c`) 2 jogos hibát talált: részleges beküldésnél árva feladatok (a csoportfájl nem íródott
   ki), és a `begyujt` git-időtúllépésnél leállt. Javítva: előzetes érvényesítés (hibás listából semmi nem indul), a csoportfájl
   minden beküldés után íródik, a `_diff` hibája szövegként jelenik meg. +2 regressziós teszt.
   **5. kör élesítve (2026-11-14 19:03 UTC):** `6a474ea` (átnézés `t20261114-185632-4056a1`), címke `elo-20261114-190341`.
   **6. kör elkészült (fejlesztési példány, 2026-11-14):** beolvasztás – `tipus` (kod/kutatas/tomeg) szerinti modellánc a psx-ben
   és az alügynököknél, a `loop` 2 bukás után gyártót vált. Az 5. kör átnézésének 3 megjegyzése javítva (git-hiba látszik az
   összesítőben, a modellválasz kitakarva, a `run`-leírás teljes jognál ellentmondásmentes). Teszt: +2 `test_loop`, +2
   `test_alugynok`. Valódi próba: `t20261114-191218-6e7ee0` (`tipus: kod`; codex limitben → claude átvette) `accepted`.
   Az átnézés (`t20261114-191356-457b57`) elutasította: a `tipus: kutatas` (claude, codex) nem kap automatikus `max_cost_usd`-t.
   Ez szándékos és a korábbi működéssel azonos: az előfizetéses végrehajtók külön névleges keretet kapnak
   (`max_elofizetes_usd`, alap 5), a `max_cost_usd` csak az OpenRouter fizetős végrehajtójára vonatkozik. Kódkommentben,
   `FIZETOS_OPENROUTER` állandóval és regressziós teszttel rögzítve; új átnézés.
   **6. kör élesítve (2026-11-14 19:45 UTC):** `f958fcb` (átnézés `t20261114-193231-430f80`), címke `elo-20261114-194515`.
   **7. kör – a 11. tétel LEZÁRÁSA (2026-11-14):** éles, ember nélküli mérés: alügynök-csoport `fonok-onallo-20261114`
   (`tipus: kod` → codex lehűlésben, claude átvette; `tipus: tomeg` → or-olcso), mindkettő hurokkal 1. körben `accepted`.
   `KEPESSEGEK.md`: új „Főnök önállóan” szakasz (`scripts/kepessegek.py`). 12. tétel szerint frissítve: `docs/RENDSZER.md`
   (főnök zár nélkül, `loop`, `tipus`, eszközjegyzék, alügynökök), `docs/HP-DEV-HOZZAFERES.md` (jelenlegi állapot + laptop-eszköz),
   `docs/SZUPER-VEGREHAJTO.md` (Mérés). Apró javítás: a git-figyelmeztetés nem jelenik meg hibaként az összesítőben.
   **11. tétel: KÉSZ (első változat).** Továbbfejlesztési jelöltek a `docs/SZUPER-VEGREHAJTO.md` „Mérés” szakaszában.

13. **S autonóm célvezérlése (toti jóváhagyta, 2026-11-14; terv és szabályok: `docs/AUTONOM-CELVEZERLES.md` – olvasd el teljesen, az a mérvadó).**
   Építsd meg a mérföldköveket sorrendben (1. lezárási kapu → 2. célfájl/alapító lap/napló + külön költségkönyv → 3. ügygazda-ébresztés → 4. helyzetértékelés/önálló lehetőségkeresés/ötletfeldolgozás →
   5. tanulás a döntésben → 6. végső, NEM mesterséges elfogadási próba). Egy kör = egy kicsi, működő, tesztelt lépés; kód a fejlesztési példányban, független átnézés, ellenőrzött élesítés, éles próba.
   Nincs párhuzamos irányítási rendszer: a meglévő ügygazdát/főnököt/ügy-műhelyt/psx-et bővítsd. Az első valós teszteset a docs-tükör elmaradt frissítése: a lezárási kapunak fel kell ismernie és be kell hajtania.
   Emberhez csak valódi elakadásnál/szükséges döntésnél fordulj (döntési sor); a `BACKLOG KÉSZ` itt csak akkor jelenthető, ha a 6. mérföldkő géppel ellenőrzött bizonyítékokkal teljesült.
   **1. mérföldkő, 1. kör kész (fejlesztési példány, 2026-11-14):** lezárási kapu `scripts/lezarasi_kapu.py` + `fonok_auto.py` bekötés
   (gépi záró állapot, `FUGGO_VALLALAS`, függő lista a következő körbe). Docs-tükör valós eset felismerve és behajtva. Következő kör: panel.
   **2. kör kész (fejlesztési példány, 2026-11-14):** panel-doboz a szál oldalán; a szövegközi kész-idézet téves lezárása javítva
   (csak az utolsó sor számít). Következő: 2. mérföldkő (`docs/CELOK.md`, alapító lap, kezdeményezés-napló, két költségkönyv).
   **2. mérföldkő, 1. kör kész (fejlesztési példány, 2026-11-14):** `docs/CELOK.md`, `scripts/kezdemenyezes.py` (alapító lap, napló, két
   külön költségkönyv, 2 USD küszöb); első valós lap lezárva (`k20261114-215204-348e`). A panel a nem futtatott parancs-bizonyítékot
   nem számolja függőnek. Következő: 3. mérföldkő (ügygazda-ébresztés, egy-főnök-zár, döntési sor bekötése).
   **3. mérföldkő, 1. kör kész (fejlesztési példány, 2026-11-14):** `scripts/ebresztes.py` az ügygazda ciklusában (pontozás, küszöb,
   egy-főnök-zár, ≤3 próba → ELAKADT döntési tétel, döntésre váró kezdeményezés a döntési sorban, óránkénti szívverés).
   Következő: éles próba, majd a 4. mérföldkő (helyzetértékelés, önálló lehetőségkeresés, ötletek).
   **3. mérföldkő élesítve és élesben igazolva (2026-11-14 22:31 UTC):** `6734abc`, címke `elo-20261114-223038`; az éles ügygazda-ciklus szívverést írt.
   **4. mérföldkő, 1. kör kész (fejlesztési példány):** ötlettár 3 napos döntési határidővel, ébresztés-forrásként.
   Következő: az ügygazda saját kezdeményezéseinek alapító lapra kötése, helyzetértékelés a kezdeményezés-naplóval.
   **4. mérföldkő, 2. kör kész (fejlesztési példány, 2026-11-14):** ügygazda-kezdeményezés → alapító lap, lezárás méréssel, tanulságok
   a tervező bemenetében; 39 régi lap pótolva; tesztszivárgás javítva. Első ötlet feldolgozva (`o20261114-223422-8ac0` → `k20261114-225545-1aba`).
   **4. mérföldkő 2. kör élesítve, élesben igazolva (2026-11-14 23:11 UTC):** `7fe2ef9`, `elo-20261114-231040`; az éles ciklus 28 saját ügy kezdeményezését zárta méréssel.
   **5. mérföldkő, 1. kör kész (fejlesztési példány):** korábbi eredményekkel korrigált pontozás + hivatkozás a promptban.
   Következő: 6. mérföldkő (végső, nem mesterséges elfogadási próba).
   **5. mérföldkő élesítve (2026-11-14 23:20 UTC):** `895d799`, `elo-20261114-232036`.
   **6. mérföldkő előkészítve (fejlesztési példány):** gépi próba `scripts/celvezerles_proba.py`; az elakadt saját ügy többé nem
   blokkolja az önálló kezdeményezést. Ezután a rendszernek MAGÁNAK kell kezdeményeznie – a próba addig `nem teljesült`.
   **6. mérföldkő élesben folyamatban (2026-11-14 23:50 UTC):** a rendszer magától kezdeményezett (`elfogadasi-parancs-relativ-utak`), próba 3/4;
   az ügy által feltárt `parancs:`-útvonal gyökérok javítva (fejlesztési példány, tervérvényesítés + prompt).
   **6. mérföldkő TELJESÜLT (2026-11-14 23:50 UTC, élesben, géppel ellenőrizve):** `scripts/celvezerles_proba.py` exit 0.
   A rendszer magától kezdeményezte az `elfogadasi-parancs-relativ-utak` ügyet (lap `k20261114-233150-dd4c`; 4 mérlegelt jelölt,
   3 korábbi kezdeményezés-hivatkozás), végrehajtotta (1. feladat elutasítva, célzott javítás `t20261114-234023-e777de` accepted,
   6 gépi ellenőrzés), maga zárta le, a mérés `siker`, 0 USD pénzkiadás / 1 USD keret. Megszorítás: az elfogadás a `munka/`-alá másolt
   példánnyal ment át (azonos fájl) – épp ezt a gyökérokot javítja a `parancs_ut_hiba` tervérvényesítés.
   **13. tétel: mind a 6 mérföldkő kész.** A szál záró állapota a lezárási kapuban: `BACKLOG_KESZ` (a függő vállalások bizonyítása után).
   Toti döntésére vár: `sajat-elakadt:akadaly-ismetlodes-csoportosito` (lezárás vagy folytatás); a `docs/CELOK.md` célsorrendjének jóváhagyása.

14. **Kapacitásgazda → helyi intelligencia; a történeti tudásbázis előkészítése párhuzamosan (toti+GPT jóváhagyta, 2026-11-15; irány: `docs/AUTONOM-CELVEZERLES.md` „Jóváhagyott irányok").**
   A főnök a meglévő célvezérlés szerint mérlegel (alapító lap, indoklás, korábbi tanulságok), nem kell mindent egyszerre. A `docs/CELOK.md`-t NE módosítsd. Prioritás: (1) **kapacitásgazda** (ötlet o20261115-054035-4d98,
   az o20261115-053120-e2a2 tartaléklánc-ötlettel együtt: központi szolgáltató/limit/reset/költség/erősség nyilvántartás, feladattípus szerinti választás, gyors váltás; tényleges pénz és előfizetéses érték külön),
   (2) **helyi intelligencia** (o20261115-054035-3040; első próba: felhasználói szintű CPU-s llama.cpp + kis/közepes modell mérés, a helyi végpont; a távoli hozzáférést ne szakítsd meg; GPU-meghajtó/sudo csak a meglévő jóváhagyási úton),
   (3) a **történeti tudásbázis** (o20261115-054642-385d) előkészítése és az alábbi követelmények szerinti megvalósítása – párhuzamosan, kis lépésekben.
   **Tudásbázis = aktív tapasztalati memória** (nem dokumentumtár). Követelmények: (a) új ügy indításakor (`ugy uj`/ügygazda kezdeményezés) automatikus keresés a releváns korábbi tapasztalatok között; (b) ismerje fel a már megoldott
   problémákat és a korábban sikertelen megközelítéseket (és mondja meg a végrehajtónak: „ez már volt, így járt"); (c) lezárt ügyből/kezdeményezésből automatikus tapasztalat-kinyerés (állítás: forrás, dátum, típus, környezet, állapot);
   (d) régi és új információ ellentmondását jelölje (nem írja felül csendben); (e) a végrehajtónak CSAK az adott feladathoz szükséges, szűrt tudás menjen át (méretkorlát, relevancia, nincs titok); (f) mérje a megtakarítást:
   kihagyott ismételt munka, modellhívás, idő, költség (két külön költségkönyv szerint), és a mérés kerüljön vissza a tanulásba. A régi dokumentumok TÖRTÉNETI források: a régi Windows-környezet megoldásait Linuxon újra kell ellenőrizni, mielőtt döntés épül rájuk.
   **Biztonság:** a nyers archívum elkülönítve marad a karanténban (`~/.local/state/project-s/archivum-karanten`, 0700, repón kívül); a titokszűrés helyben, MODELLHÍVÁS NÉLKÜL fut (`scripts/titok_ellenor.py` mintái + a belépési adat/token/jelszó/e-mail/IP minták bővítése), a találatokat kitakarja, és a tartalom csak sikeres
   ellenőrzés után adható tovább. Az archívum fájljai még nincsenek a szerveren (toti/GPT tölti fel); addig a már meglévő forrásokból (ugyek/*/eredmenyek, lezárt kezdeményezések, forrasok/) építsd a memóriát.
   Ne építs feleslegesen új alrendszert: az ügygazdát, a kezdeményezés-nyilvántartást és az állításkezelést bővítsd. Kód: fejlesztési példány, független átnézés, ellenőrzött élesítés, éles próba (a végső próba: egy valódi új ügynél a
   rendszer MAGA találja meg és használja a korábbi tapasztalatot, és a megtakarítás mérhető).
   **14/1. kapacitásgazda, 1. kör kész (fejlesztési példány, 2026-11-15):** `executor/kapacitas.py` (tábla + választás) és psx-tartaléklánc
   (`psx.lanc`, független ellenőr gyártója védve); kiinduló mérés: 12,66 óra modellvárakozás/7 nap. Következő: élesítés után a
   várakozási óra összevetése, majd 2. helyi intelligencia (CPU-s llama.cpp mérés), párhuzamosan a tudásbázis előkészítése.
   **14/1. élesítve (2026-11-15 06:31 UTC):** `c1091b3`, `elo-20261115-063111`.
   **14/2. helyi intelligencia, 1. kör kész (2026-11-15):** llama.cpp CPU + Qwen3-4B Q4_K_M a `local` végponton (`project-s-llama.service`),
   ~12 tok/s; éles psx-próba accepted 0 USD; díjmentes szolgáltató pénzkeret-hibája javítva. Következő: a `local` bevonása a `tomeg`
   tartalékláncba mért minőség után, és a tudásbázis előkészítése (14/3).
   **14/2. élesítve (2026-11-15 06:42 UTC):** `be0dcda`, `elo-20261115-064221`; éles próba 0 USD kerettel accepted (`t20261115-064317-a81521`).
   Átfedés (megszűnt): a `b20261114-205523-9a10` szál `scripts/kapacitasgazda.py`-ja visszavonva (`fa04a20`, `21aeba8`); a mért sikerarány-ötlet a
   `executor/kapacitas.py` későbbi bővítése lehet (a psx-tartaléklánc a mért sikerarányt használja).
   **14/3. tapasztalati memória, 1. kör kész (fejlesztési példány):** `scripts/tapasztalat.py` (253 tétel a meglévő ügyekből és
   kezdeményezésekből, helyi titokszűrés, keresés, `ugy uj`/feladatprompt bekötve, használatmérés). Következő: ellentmondás-jelölés,
   megtakarítás-mérés, ügygazda-kezdeményezés bekötése; végső próba: valódi új ügy maga használja a tapasztalatot.
   **14/3. 1. kör élesítve (2026-11-15 07:05 UTC):** `6a9c531`, `elo-20261115-070524`; éles tár 257 tétel, 0 titok miatt kihagyva, könyvtár 0700.
   Éles próbán a relevancia gyenge volt (általános szavak) → 2. kör: gyakori szavak kizárása + küszöb; a 4B modell álneve javítva
   (`qwen3-4b-instruct`). A `helyi-modell-vegpont-diagnozis` saját ügy oka (nem futó helyi végpont) ezzel megszűnt.
   **14/3. 2. kör élesítve (2026-11-15 07:24 UTC):** `7e9a839`, `elo-20261115-072440`; llama-unit frissítve, éles próba `t20261115-072550-6620a7` accepted.
   **14/3. 3. kör kész (fejlesztési példány):** ügygazda-tervezés ismétlés-gyanúval + megtakarítás-napló. Következő: élesítés, majd
   ellentmondás-jelölés és a végső próba (valódi új ügy maga használja a tapasztalatot, mért megtakarítással).

   **Tanulság (2026-11-15, szál b20261114-205523-9a10):** a lezárt 13.-as szál a tükör-vállalás újranyílása miatt felvette a 14. tételt, holott annak
   saját szála van (b20261115-060236-7705, `c1091b3` élesben) – a duplikátum visszavonva, a kapu javítva (bizonyított vállalás nem nyílik újra).
   **14/3. 3. kör élesítve (2026-11-15 09:46 UTC):** `e6b199b`, `elo-20261115-094601`. **4. kör kész (fejlesztési példány):** ellentmondás-jelölés
   (77 pár, mind ügyön belüli; sablonszűrővel). Hátra: záró próba valódi új ügyön mért megtakarítással; `local` a `tomeg` láncba mérés után.
   **14/3. 4. kör élesítve (2026-11-15 10:02 UTC):** `b8e3b67`, `elo-20261115-100239`. **5. kör kész (fejlesztési példány):** gépi záró próba
   `scripts/tapasztalat.py proba` – a végső próba akkor teljesül, ha egy valódi új ügy maga használja a tapasztalatot (most: nem teljesült).
   **14/2. helyi modell, 2. kör (2026-11-15 10:31 UTC, `elo-20261115-103039`):** 3 párhuzamos hely + szolgáltatónkénti időkorlát élesítve;
   újramérés 3/5 (hamis sikerjelentés 1, időtúllépés 1) → a `local` nem kerül automatikusan a `tomeg` láncba. 14/3 záró próba
   (`scripts/tapasztalat.py proba`) valódi ügyre vár; régi archívum: a fájlok feltöltésére vár (toti/GPT).
   **14/3. záró próba (2026-11-15):** `scripts/tapasztalat.py proba` exit 0 – a rendszer magától kerülte el egy elakadt ügy megismétlését
   (`psx-dontes-ok-feltaras` → `ellenorzok-regresszio-futtato`), becsült megtakarítás 3 feladat / ~1,05 USD előfizetéses érték / ~185 perc.
   Végrehajtói hivatkozás a kapott tapasztalatra: 0/8 (gyenge pont, továbbfejlesztési jelölt). Régi archívum: feltöltésre vár (toti/GPT).
   **14/3. régi archívum karanténban (2026-11-15, `t20261115-173321-d99bc2`):** a feltöltött ZIP (sha256 `1851f91b…`, 4 fájl: 2 md, 1 json, 1 docx)
   a `~/.local/state/project-s/archivum-karanten`-ben (0700/0600), a közös mappából törölve, gitben sosem volt. Helyi szűrés (`titok_ellenor.kitakar`):
   12 IPv4 kitakarva, maradt gyanú 0, utóellenőrzés 0 → szűrés bizonyított. Szűrt jelöltek: `szurt/jeloltek.jsonl` (4 tétel, `tapasztalat.py`
   tételformátum, `tortenet-ellenorizetlen`, `regi-windows`) – a tárba még NEM kerültek. Következő: `tapasztalat.py kinyer` bővítése a jelöltek
   beolvasására (alacsonyabb rang, frissebb tétel elsőbbsége, ellentmondás-jelölés) fejlesztési példányban; feldolgozó: `kozos/archivum_karanten.py`.

