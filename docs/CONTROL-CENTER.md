<!-- Automatikus tükör a privát Projekt S műhelyrepóból (scripts/docs_tukor.py). Személyes és hálózati adatok kitakarva. Ne itt szerkeszd. -->

# Control Center 2.0 (`/cc`)

A panel új, egyoldalas felülete: valós idejű áttekintés és irányítás telefonról és gépről. Ugyanabban a
szolgáltatásban fut (`project-s-panel.service`), ugyanazzal a hitelesítéssel (tailnet + Tailscale-identitás), és a
meglévő psx / ügygazda / rendszerlépés / S Bridge függvényeit hívja – **nincs külön irányítórendszer és nincs külön
állapot**. A klasszikus oldalak változatlanok.

| Fájl | Szerep |
|---|---|
| `mcp/cc_adat.py` | adatréteg, csak olvas: psx-eseménynaplók, ügygazda, élesítési napló, mentések, S Bridge, systemd, /proc |
| `mcp/cc.py` | HTTP: `/cc`, `/cc/s/*`, `/cc/api/*`, élő folyam (SSE), író műveletek + hash-láncolt műveletnapló |
| `mcp/cc_eszkoz.py` | terminál, fájlok, S Bridge-vezérlés, böngésző, ügyrészlet, feladatindítás, beállítások |
| `mcp/static/cc/` | a felület (építőlépés nélküli JS + CSS; külső függőség nincs) |
| `executor/tests/test_panel_cc.py` | egységtesztek elkülönített állapottal |
| `scripts/cc_bongeszo_proba.py` | valódi böngészős próba (mobil + asztali; `--izolalt` a műveletekhez, `--elo URL` a bejáráshoz) |

## Nézetek
Áttekintés (teendők, riasztások, futó feladatok, idővonal) · Feladatok és feladatrészlet (idővonal, lépések, szabályok,
kimenet, döntés) · Ügyek · Kapcsolati térkép (ki kinek dolgozik, élő állapottal) · Ügygazda · Rendszerlépések ·
Modellek és költség · Önállóság · Élesítés és mentés (visszaállítási próbák, címkék) · Rendszer (erőforrás, egységek,
journal) · Napló (minden forrásban keres) · Értesítések · S Bridge (telefon) · Terminál · Fájlok · Böngésző ·
Főnök-szálak · Beállítások.

## Élő működés
A `/cc/api/elo` SSE-folyam 2 mp-enként összeveti a források ujjlenyomatát, és csak a megváltozott nézetek nevét küldi;
a kliens azokat tölti újra. Megszakadáskor magától újracsatlakozik (EventSource), három hiba után 5 mp-es lekérdezésre
vált, alvásból visszatérve azonnal frissít. A folyam 10 percenként szándékosan zár (nem marad árva szál).

## Adatok hitelessége
Minden szám a szerver állapotából számolódik; ami nem olvasható, az „–” vagy hibaüzenet – becsült vagy kitalált érték
nincs. A költségnél a fizetős (OpenRouter stb.) és az előfizetés-egyenértékű összeg külön látszik.

## Író műveletek és jogosultság
- `POST /cc/api/muvelet` (JSON) – munkamenet-token (`X-CC-Token`), Origin-ellenőrzés, kötelező JSON tartalomtípus.
- Minden művelet a `~/.local/state/project-s/cc/muveletek.jsonl` hash-láncolt naplóba és a panel-auditba kerül
  (fájltartalom, eszköztoken nem). Ellenőrzés: Beállítások → Műveletnapló, keresés: Napló → Control Center műveletek.
- Feladat: leállítás, elfogadás/elutasítás, folytatás, új kísérlet, új feladat (ügyhöz kötve `ugy.inditas`).
- **Terminál:** `zárt` mód = a végrehajtókkal azonos Landlock-zár (`executor/zar.py --teljes`: titok és sudo nélkül);
  `teljes` mód = zár nélkül, a panel felhasználójaként; root = rendszerlépés-kérés → jóváhagyás. Időkorlát (alap 120 mp,
  legfeljebb 1 óra), 8 MB kimenetkorlát, leállítható; titokgyanús környezeti változók nem öröklődnek.
- **Fájlok:** olvasás a saját könyvtár alatt a titokmappák (`~/.config`, `~/.ssh`, …) és titoknevű fájlok kivételével;
  írás csak a munkaterületeken (`kozos/`, `ugyek/`, `~/work`, média, `/tmp`). Az éles repó kódja innen **nem írható** –
  az a független átnézés + élesítési kapu útja. Törlés és felülírás előtt a régi változat a kukába kerül
  (`~/.local/state/project-s/cc/kuka/`).
- **S Bridge:** megfigyelő parancs azonnal; író parancs egyszeri, 10 perces, arra az egy feladatra szóló jóváhagyással;
  vészleállító; eszköztoken kiadása (egyszer látható) és visszavonása.

## Kezdőlap és visszaállítás
A panel `/` címe a Control Centerre irányít. Visszaváltás: CC → Beállítások → Kezdőlap → Klasszikus panel (azonnal hat,
`~/.local/state/project-s/cc/beallitasok.json`). A klasszikus feladatlista mindig elérhető: `/regi`. A teljes kód
visszaállítása: `python3 scripts/elesites.py visszaallit <elozo-címke>`.

## Tesztelés
```sh
python3 -m unittest executor.tests.test_panel_cc            # a scripts/tesztek.sh része
~/.local/opt/bongeszo-venv/bin/python scripts/cc_bongeszo_proba.py --izolalt
~/.local/opt/bongeszo-venv/bin/python scripts/cc_bongeszo_proba.py --elo http://127.0.0.1:8820
```
A paneltesztek fájlneve `test_panel_*` legyen: a közös futásban a `test_kerelem` után töltődnek be, különben a psx
modulszintű `PSX_HOME`-ja az élő állapotra mutatna.
