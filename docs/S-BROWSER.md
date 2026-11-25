<!-- Automatikus tükör a privát Projekt S műhelyrepóból (scripts/docs_tukor.py). Személyes és hálózati adatok kitakarva. Ne itt szerkeszd. -->

# S-Browser – önálló böngészős munkafolyamatok

Valódi Chromium (Playwright) a szerveren, amelyet a Projekt-S ügynökei (psx-végrehajtók, Főnök) lépésenként vezérelnek:
hétköznapi internetes munkafolyamatok – keresés, űrlapok, többlépéses ügyintézés, adatgyűjtés, le- és feltöltés.
Démon: `scripts/sbrowser.py serve` (`project-s-sbrowser.service`, csak 127.0.0.1:8861); parancs: `sbrowser`.

## Az ügynök szemszögéből
```sh
sbrowser nyit https://pelda.hu --cel "időpontfoglalás"   # → munkamenet-azonosító + pillanatkép
sbrowser ir <id> 4 "keresett szöveg" --enter             # a [4] mezőbe ír, Enter
sbrowser kattint <id> 12                                  # a [12] elem megnyomása
sbrowser valaszt <id> 7 "Budapest" · sbrowser gomb <id> PageDown · sbrowser gorget <id> le · sbrowser vissza <id>
sbrowser szoveg <id>                                      # az oldal teljes szövege
sbrowser kep <id> ki.png · sbrowser feltolt <id> 9 fajl.pdf · sbrowser bezar <id> · sbrowser lista
sbrowser ember <id> "be kell jelentkezni"                 # átadás totinak; a parancs megvárja a végét
```
Minden művelet a lap friss **pillanatképét** adja vissza: cím, URL, a látható kezelőelemek számozva
(`[szám] szerep „név” = érték`), címsorok és a szöveg eleje – a következő lépés ebből dönthető el, képfelismerés nélkül.
A számok a legutóbbi pillanatképre érvényesek. Eszközjegyzék: `bongeszo`; képességtár: `kepesseg mutat bongeszo`.

## Ami az ember kezében marad (a démon tartatja be)
| Helyzet | Mi történik |
|---|---|
| jelszó- vagy bankkártyamező kitöltése | az ügynök hívása 423-mal megáll, a lap **átadódik totinak** |
| jelszavas / kártyás űrlap elküldése Enterrel | 423 |
| fizetést, megrendelést véglegesítő gomb (Fizetés, Megrendelem, Pay now, Place order …) | 423 + átadás – pénzköltés toti döntése |
| captcha, kétlépcsős kód, megerősítés | az ügynök kéri: `sbrowser ember <id> "miért"` |

Átadáskor toti értesítést kap; a Control Center **Böngésző** nézetében (`/cc#/bongeszo`) látja a lap élő képét, a képre
koppintva kattint, rejtett mezőből gépel, majd „Kész – visszaadom” (megjegyzéssel) vagy „Nem – visszaadom”. Átvétel alatt
az ügynök nem nyúlhat a laphoz. Az emberi műveleteket a démon csak a panel titkával fogadja el
(`~/.config/project-s/sbrowser.titok` – a végrehajtók zárja nem engedi olvasni).

## Védelem
- **Belső címek tiltva** (127.0.0.1, 10/8, 192.168/16, 100.64/10 – tailnet, `*.ts.net`, `.local`, `file:`): az ügynök a
  böngészőn át nem éri el a panelt és a szerver belső szolgáltatásait. A szűrés minden kérésre érvényes (beágyazott is).
- **Feltöltés** csak a munkaterületekről (kozos/, ugyek/, ~/work, /tmp), titoknevű fájl nem – a démon zár nélkül fut, ő szűr.
- **Profil** (sütik, belépések): `~/.local/share/project-s/sbrowser/<profil>` – a végrehajtók zárja nem olvashatja.
  A belépések megmaradnak, így egy egyszeri emberi bejelentkezés után az ügynök önállóan dolgozhat az adott oldalon.
- **Napló:** `~/.local/state/project-s/sbrowser/naplo.jsonl`, hash-láncolt; a címek lekérdezés nélkül (`?…`), az emberi
  átvétel alatt begépelt szöveg egyáltalán nem kerül bele. A kimenetekben a titokgyanús címparaméterek kitakarva.
- Letöltések: `kozos/bejovo/bongeszo/<munkamenet>/`. Legfeljebb 6 munkamenet; 45 perc tétlenség után a lap bezárul.

## Korlátok (ismert)
Csak a fő keret elemei számozottak (beágyazott iframe tartalma nem); a fizetésgomb-felismerés feliratalapú (magyar, angol,
német minták) – ismeretlen nyelvű vagy képes gombnál nem fog, ezért pénzmozgással járó oldalon az ügynök feladata előre
átadni; robotszűrős oldalak (captcha) emberi lépést igényelnek.

## Próbák és üzemeltetés
```sh
~/.local/opt/bongeszo-venv/bin/python scripts/test_sbrowser.py     # szabályok + valódi Chromium helyi próbaoldalon (tesztek.sh része)
~/.local/opt/bongeszo-venv/bin/python scripts/sbrowser_e2e.py      # ügynök → démon → emberi átvétel a Control Centerben → vissza
systemctl --user status project-s-sbrowser · sbrowser allapot
```
Telepítés: `ln -s ~/project-s-workshop/install/systemd-user/project-s-sbrowser.service ~/.config/systemd/user/ &&
systemctl --user enable --now project-s-sbrowser`; parancs: `~/bin/sbrowser` → `python3 scripts/sbrowser.py "$@"`.
A démon kódjának élesítése után: `systemctl --user restart project-s-sbrowser` (a nyitott lapok bezárulnak, a profil megmarad).
