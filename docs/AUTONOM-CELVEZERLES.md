<!-- Automatikus tükör a privát Projekt S műhelyrepóból (scripts/docs_tukor.py). Személyes és hálózati adatok kitakarva. Ne itt szerkeszd. -->

# S autonóm célvezérlése (toti jóváhagyta, 2026-11-14)

**Cél:** S nem azért dolgozik állandóan, hanem hogy önállóan felismerje, MIKOR és MIVEL érdemes dolgoznia. Mindennek mérhető haszna van.
Egy rendszer: a meglévő **ügygazda** (olcsó, állandó felügyelet), **főnök** (drága gondolkodás), **ügy-műhely** (`ugy`, `ugyek/`) és **szuper-végrehajtó** (psx loop, alügynökök,
eszközjegyzék) együtt – NEM építünk párhuzamos irányítási rendszert; ami hiányzik, azt a meglévő komponensekbe építjük.

## Rétegek
- **Ügygazda = felügyelet.** Szabályokkal és olcsó modellel figyeli az eseményeket (feladat kész/elbukott, limit feloldódás, új ötlet, elmaradt vállalás, hosszan álló ügy, új levél/lehetőség),
  és csak akkor ébreszti a főnököt, ha van jelölt, amelynek várható haszna meghaladja a küszöböt. Egyébként nem gondolkodik drága modellel.
- **Főnök = döntés és kezdeményezés.** Helyzetértékelés → jelöltek → önálló választás indoklással → végrehajtás a szuper-végrehajtóval → mérés → tanulás.
- **Tartalék:** időzített felügyelet marad (óránkénti szívverés + a meglévő időzítők), de NINCS párhuzamos főnökindítás (zár/azonosító: egyszerre egy főnök-kezdeményezés), nincs végtelen
  újrapróbálkozás (korlátos kísérlet, majd állapot: ELAKADT), és nincs céltalan önfejlesztés (alapító lap nélkül nincs kezdeményezés).

## Állandó célok (tartósak; a módszert a főnök szabadon választja)
1. A rendszer működésének és értékeinek megőrzése (tesztek zöldek, mentés és visszaállítás rendben, szolgáltatások futnak, élesítés/visszalépés működik).
2. Meglévő ügyek eredményes befejezése.
3. Valódi bevételi lehetőségek keresése – önállóan is keres (web, piac, NEAR, ügyfél-/megbízás-források), nem csak a megadott ötleteket dolgozza fel.
4. A feladatokhoz szükséges új képességek megépítése – csak konkrét akadály/igény miatt.
5. Elmentett ötletek aktív feldolgozása és megvalósítása.
Minden célhoz mérőszám tartozik (`docs/CELOK.md`, géppel olvasható; a főnök bővítheti, a célok sorrendjét/lényegét toti hagyja jóvá).

## Kezdeményezés és mérhető haszon
- Minden önálló kezdeményezés **alapító lapot** kap: melyik célt szolgálja, hipotézis, mérőszám és időpont, kockázat, visszafordíthatóság, költségkeret, a választás indoklása (miért ez, miért most, mi volt a többi jelölt).
- Lezáráskor **mérés**; a nyilvántartásban (kezdeményezés-napló) az eredmény, a költség, a tanulság. Ha nem hozott mérhető hasznot, rögzítjük, és a hasonlók lejjebb kerülnek.
- A főnök **önállóan indíthat belső, visszafordítható munkát** (kód a fejlesztési példányban, ügyek, dokumentáció, kutatás, mérés). A meglévő jóváhagyási határok érvényben maradnak:
  toti dönt pénzköltésről/regisztrációról, külső beküldésről, OVH-ról, tűzfal/SSH szűkítéséről, fiókbelépésről; a laptop leállítása előtt egyeztetés; titok nem kerül a repóba.
  Kód- és élesítésváltozásnál a független átnézés marad.

## Költség – két külön könyvelés
- **Tényleges pénzkiadás** (fizetős API-hívás, pl. OpenRouter): kezdetben **legfeljebb 2 USD kezdeményezésenként**; e fölött a döntési sorba kerül. Külön könyvelve.
- **Előfizetéses kapacitás** (Claude Pro, Codex/ChatGPT, Copilot): számított érték, nem pénzkiadás; a kapacitást ki kell használni, a limit természetes korlát (tartaléklánc, várakozás).
  Külön könyvelve, a kettőt a jelentés nem keveri.

## Tanulás (a döntésben használt, nem csak napló)
A kezdeményezés-napló eredményei, költségei, hibái és tanulságai a következő helyzetértékelés BEMENETEI: a jelöltek rangsorolása előtt a főnök lekéri a hasonló korábbi kezdeményezések kimenetét
(mi vált be, mi nem, mennyibe került, mennyi idő volt), és az indoklásban hivatkozik rájuk. A rangsor pontszáma: fontosság × várható érték ÷ (költség + kockázat), a várható értéket a korábbi mért eredmények korrigálják.

## Állapotok és lezárási kapu
A főnök-szál záró állapota külön, géppel kezelt érték (nem szöveg-egyezés):
- `BACKLOG_KESZ` – a kijelölt teendőlista minden tétele lezárva;
- `ELKESZULT` – a kezdeményezés célja teljesült, a mérés kész;
- `ELAKADT` – korlátos kísérletek után sem halad (ok és szükséges döntés megnevezve);
- `NINCS_ERTELMES_TEENDO` – helyzetértékelés után nincs a küszöböt meghaladó jelölt (a felügyelet alszik, eseményre ébred).
A lezárás **géppel ellenőrzött bizonyítékokon** alapul: a főnök a vállalásait (pl. „élesítés után lefuttatom a docs-tükröt") gépi listába írja; minden vállaláshoz bizonyíték-ellenőrzés tartozik
(commit/élesítési címke, fájl/hash, a tükör-repó új commitja, tesztkimenet). Amíg van bizonyíték nélküli vállalás, a szál nem zárulhat (állapota: `FUGGO_VALLALAS`, és a felügyelet visszaadja).
**Első valós teszteset:** a nyilvános dokumentáció-tükör (`scripts/docs_tukor.py`) elmaradt frissítése (2026-11-14: a főnök vállalta, a szál közben leállt).

## Mérföldkövek
1. **Lezárási kapu** (első, szűk): vállalás-nyilvántartás, bizonyíték-ellenőrzők, a négy állapot a `fonok_auto.py`-ban és a panelen; a docs-tükör eset a valós teszt.
   **1. kör kész (2026-11-14, fejlesztési példány):** `scripts/lezarasi_kapu.py` (`vallal` / `zar` / `allapot`; szálanként
   `~/.local/state/project-s/fonok-auto/kapu/<szál>.json`), bizonyítékok: `commit:`, `cimke:`, `fajl:[:sha256]`, `tukor`, `parancs:`.
   A `fonok_auto.py` a gépi záró állapotot használja; a régi „BACKLOG KÉSZ” szöveg csak kérésként számít, a kapun át.
   Függő vállalásnál `FUGGO_VALLALAS`, a következő kör promptja a függő listát kapja (ELAKADT függő mellett is jelenthető).
   Valós eset: a `tukor` vállalás „elavult (20:40 < 20:55)” → `docs_tukor.py` lefuttatva → teljesült. Teszt: `scripts/test_lezarasi_kapu.py` (7).
   **2. kör kész (2026-11-14):** panel – a beszélgetés oldalán „Lezárási kapu” doboz (záró állapot, kért ↔ tényleges, vállalások
   bizonyítékkal; függőnél nyitva). A panel `parancs:` bizonyítékot nem futtat. Valós hiba javítva: a régi szöveges jelzés
   egy szövegközi idézetre is lezárt; most csak az utolsó nem üres sor számít (`regi_kesz_jelzes`). Teszt: 9.
2. **Célfájl, alapító lap, kezdeményezés-napló** (`docs/CELOK.md`, nyilvántartás), két külön költségkönyv.
   **1. kör kész (2026-11-14, fejlesztési példány):** `docs/CELOK.md` (géppel olvasható json-blokk, 5 cél mérőszámokkal),
   `scripts/kezdemenyezes.py` (`uj <lap.json>` / `koltseg <id> penz|elofizetes …` / `lezar <id> <kimenet> "<mérés>" "<tanulság>"` / `lista`).
   Állapot: `~/.local/state/project-s/kezdemenyezes/` (lapok/, naplo.jsonl, koltseg-penz.jsonl, koltseg-elofizetes.jsonl).
   2 USD fölötti pénzkeret vagy keret-túllépés → `dontesre_var` (az ügygazda döntési sorába kötése a 3. mérföldkő része).
   Első valós bejegyzés: `k20261114-215204-348e` (docs-tükör eset, `siker`, 0 USD). Teszt: `scripts/test_kezdemenyezes.py` (4).
3. **Ügygazda-ébresztés:** olcsó eseményfigyelés, egy-főnök-zár, korlátos újrapróbálás, tartalék időzítő.
4. **Helyzetértékelés és önálló lehetőségkeresés**, aktív ötletfeldolgozás (értékbecslés, N napon belüli döntés).
5. **Tanulás a döntésben** (korábbi eredmények a rangsorolásban, hivatkozva).
6. **Végső elfogadási próba (nem mesterséges):** a rendszer MAGA talál egy korábban nem kijelölt, hasznos kezdeményezést, indokolja a választást (jelöltek, rangsor, korábbi tanulságok),
   végrehajtja az engedélyezett kereteken belül, és méri az eredményt; független ellenőrzés + élesben igazolva. A főnök a mérföldköveket önállóan viszi végig, független ellenőrzéssel és éles próbákkal.

## Működési elv a megvalósításhoz
Ne építs új párhuzamos rendszert: a `scripts/ugygazda.py` (védett; ellenőrzött élesítés, toti tartós felhatalmazásával), `scripts/fonok_auto.py`, `executor/beszelgetes.py`, `executor/psx.py`, az `ugy` bővítése.
A cél nem a folyamatos munka. A rendszernek mindig tudnia kell indokolni, MIÉRT ez a következő lépés, és utólag mérni, hogy megérte-e.
