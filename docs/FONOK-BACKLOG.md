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
2. **Reggeli jelentés toti-nak.** `scripts/reggeli_jelentes.py`: rövid magyar összefoglaló az elmúlt 12 órából (önállóan elfogadott
   feladatok, élesítések/visszaállítások az elesites.jsonl-ból, ügygazda-elakadások és döntési sor, NEAR: ajánlat/kiosztás/üzenet/
   kifizetés, hibák, főnök-körök), push-értesítéssel (ertesites.background) és fájlba (~/.local/state/project-s/reggeli/<nap>.md).
   Teszttel. Az ügygazda ciklusa hívja naponta egyszer 05:30 UTC után (ugygazda.py ciklus – a változtatás az ellenőrzött élesítésen megy át).
3. **OpenRouter főnök-modellek próbája.** A katalógusból (`openrouter_katalogus.lista()`) válassz ~20 eszközképes, népszerű/olcsó
   modellt (pl. openai/gpt-5*, moonshotai/kimi-k*, qwen coder, deepseek, google/gemini), és futtasd rájuk `proba()`-t; összesen legfeljebb
   1 USD. A panel `/c/modellek` így mutatja az igazoltakat. Eredmény: rövid jelentés a válaszodban.
4. **Dokumentáció.** docs/UGYGAZDA.md és docs/RENDSZER.md: a főnök-folyamat (fejlesztési példány, átnézés, automatikus élesítés,
   visszaállítás, reggeli jelentés), a FOLYTATAS.md „hol tartunk” frissítése.
5. **Videó (ügy video-shorts).** A double-wish renderelés a 668. képkockánál megszakadt: `~/video-shorts/blender/render-dw.sh`
   folytatható (kész képeket kihagyja). Indítsd `nice -n 19`-cel háttérben (nohup), figyeld, majd `finish-dw.sh`; igazold, hogy a
   double-wish.mp4 létrejött (ffprobe: hossz, felbontás). Ez nem repóváltozás; az eredményt ugy rogzit-tel rögzítsd a video-shorts ügyben.
