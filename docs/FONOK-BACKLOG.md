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
