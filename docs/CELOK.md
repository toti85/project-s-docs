<!-- Automatikus tükör a privát Projekt S műhelyrepóból (scripts/docs_tukor.py). Személyes és hálózati adatok kitakarva. Ne itt szerkeszd. -->

# Állandó célok és mérőszámok (docs/AUTONOM-CELVEZERLES.md)

Géppel olvasható: a lenti `json` blokkot a `scripts/kezdemenyezes.py` olvassa (minden alapító lapnak egy itt szereplő célt kell szolgálnia).
A főnök mérőszámot bővíthet; a célok sorrendjét és lényegét toti hagyja jóvá. A sorrend a fontosság (1 = legfontosabb).

```json
{
 "valtozat": 1,
 "jovahagyta": "toti, 2026-11-14 (docs/AUTONOM-CELVEZERLES.md)",
 "celok": [
  {"id": "megorzes", "fontossag": 5, "cim": "A rendszer működésének és értékeinek megőrzése",
   "meroszamok": ["scripts/tesztek.sh ZÖLD", "napi visszaállítási próba RENDBEN", "szolgáltatások futnak", "élesítés/visszalépés működik"]},
  {"id": "ugyek-befejezese", "fontossag": 4, "cim": "Meglévő ügyek eredményes befejezése",
   "meroszamok": ["lezárt ügyek száma", "elakadt/nyitott ügyek száma és kora"]},
  {"id": "bevetel", "fontossag": 4, "cim": "Valódi bevételi lehetőségek keresése (önállóan is)",
   "meroszamok": ["talált, ellenőrzött lehetőségek száma", "beadott ajánlatok (toti jóváhagyásával)", "tényleges bevétel USD"]},
  {"id": "kepesseg", "fontossag": 3, "cim": "Szükséges új képességek megépítése – csak konkrét akadály/igény miatt",
   "meroszamok": ["megszüntetett akadályok száma (ugygazda döntési sor)", "élesített képesség, éles próbával"]},
  {"id": "otletek", "fontossag": 3, "cim": "Elmentett ötletek aktív feldolgozása és megvalósítása",
   "meroszamok": ["feldolgozott ötletek aránya", "ötlet → döntés átfutási ideje (nap)"]}
 ]
}
```
