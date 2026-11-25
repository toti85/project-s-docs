<!-- Automatikus tükör a privát Projekt S műhelyrepóból (scripts/docs_tukor.py). Személyes és hálózati adatok kitakarva. Ne itt szerkeszd. -->

# Telefon-doktor

Android-telefon vizsgálata és javítása a szerverről, adb-n át (`scripts/telefon_doktor.py`). Az első eset: toti Redmi Note 13 Prója
(MIUI 14), ahol a ChatGPT és a Claude app „összeomlását” valójában a rendszer memóriahiány miatti leállítása és a MIUI zároláskori
tisztítása okozta – nem a tárhely.

## Használat
```
telefon_doktor.py csatlakozik <ip:port> [--parosit <ip:port> <kód>]   # Fejlesztői beállítások → Vezeték nélküli hibakeresés
telefon_doktor.py vizsgal [--json] [--app <csomag>]                   # csak olvas; a jelentés: ~/.local/state/project-s/telefon/
telefon_doktor.py ved <csomag> …                                      # akkukivétel, háttérfutás, automatikus indítás, zárolás a legutóbbiak közt
telefon_doktor.py zarolasi-tisztitas ki|<mp>                          # MIUI: memóriatisztítás zároláskor
telefon_doktor.py leallit|tilt|engedelyez <csomag>
telefon_doktor.py valtozasok | visszaallit
```

## Elvek
- A `vizsgal` semmit nem módosít. Megmutatja: valódi összeomlások (ok szerint), a rendszer általi leállítások oka appként
  (memóriahiány, zároláskori tisztítás), lefagyások, memória, tárhely, legnagyobb fogyasztók, és ezekből javaslatokat ad.
- Minden módosítás a régi értékkel naplózódik (`valtozasok.jsonl`), a `visszaallit` fordított sorrendben visszavonja.
- Appot nem távolít el és adatot nem töröl; a `tilt` visszafordítható, rendszerösszetevőre nem alkalmazható.
- A telefonra kerülő értékek ellenőrzöttek (csomagnév-minta, számtartomány); helyi shell nincs a hívásban.

## Korlát és következő lépés
A vezeték nélküli hibakeresés portja változik, és a kapcsolat Wi-Fi-váltáskor megszűnik – ezért ez ma kézi indítású eszköz.
Tartós megoldás: ugyanezek a vizsgálatok és javítások az S Bridge appon át (az már folyamatosan kapcsolódik a szerverhez),
így a Control Center rendszeresen futtathatja, és a Főnök / ügygazda is indíthatja.
