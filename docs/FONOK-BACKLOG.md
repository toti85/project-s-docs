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
