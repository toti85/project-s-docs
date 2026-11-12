<!-- Automatikus tükör a privát Projekt S műhelyrepóból (scripts/docs_tukor.py). Személyes és hálózati adatok kitakarva. Ne itt szerkeszd. -->

# ATA kereskedési kutatás – Telegram-kapcsolat és kutatási alap

Ügy: `ugyek/ata-kereskedesi-kutatas/`. Kód: `ata/`. Parancs: `ata` (SSH-ból bármely felhasználóval: `/usr/local/bin/ata`
→ `sudo -iu sagent`). Telethon 1.45.0 a `~/.local/opt/ata-venv` felhasználói venvben (rendszercsomag nem kellett).

## Mit csinál és mit NEM
- A felhasználó **saját fiókjával**, meglévő tagsága alapján, **csak olvas**. Minden API-kérés engedélylistán megy át
  (`ata/kliens.py`): üzenetküldés, csatlakozás, meghívó elfogadása, **olvasottnak jelölés**, reakció, törlés, profil-/
  státuszmódosítás, kijelentkezés → a kliens el sem küldi (`TiltottKeres`, unit-teszttel igazolva). Belépési kérések csak
  `ata belepes` alatt.
- Azonosításnál (`ata azonosit`) csak a saját találatok **metaadatai**: cím, chat-ID, típus, felhasználónév, taglétszám,
  Telegram scam/fake/verified jelzés. Módszer: `contacts.Search` „saját találatok” (nem a teljes párbeszédlista);
  tartalék: `--dialogusokbol` (a válaszban érkező utolsó üzeneteket eldobjuk, nem mentjük). Üzenettartalmat nem olvas.
  A név **nem** azonosít személyt: a trader személye nem ellenőrzött.
- Tartalomgyűjtés (`ata gyujt`, `ata figyel`) és AI-elemzés **csak rögzített hozzájárulással** fut (`ata hozzajarulas`).

## Keresés a saját csevegésekben (2026-11-08)
`ata keres "szöveg" [--csak csoport|csatorna] [--chat ID]` és a panel **Telegram-keresés** oldala (`/tg`): a Telegram saját
keresőjével azonos funkció (`messages.searchGlobal` / `messages.search`) az összes csevegésedben. Csak megjelenít – nem ment,
nem jelöl olvasottnak, AI nem dolgozza fel, a psx-háttérfeladatok nem érik el. Találatonként: csevegés, idő, küldő, szöveg,
„megnyitás a Telegramban” hivatkozás (nyilvános/csatorna esetén).

## Titkok és elkülönítés
| Mi | Hol | Jog |
|---|---|---|
| api_id / api_hash | `~/.config/project-s/telegram/api.env` | 600, mappa 700 |
| Telegram-munkamenet | `~/.config/project-s/telegram/ata.session` | 600 |
| források, hozzájárulások, adatbázis, képek | `~/.local/share/project-s/ata/` | 700 |
A telefonszám, a belépőkód és a kétlépcsős jelszó **rejtett beviteli mezőben** megy a Telethonnak; nem tárolódik, nem naplózódik.
A repóba csak kód, dokumentáció és szintetikus tesztadat kerül (`scripts/titok_ellenor.py` pre-commit).

**psx-háttérfeladatok nem érik el:** az ügynök által indított folyamatok Landlock-zár alatt futnak (`executor/zar.py`), amely
a `~/.config` és `~/.local/share/project-s` mappát **nem olvashatja** – és ezek olvasható gyökérként vagy munkakönyvtárként sem
adhatók meg (zar.py 126, `agent.Ciklus` PermissionError; teszt: `ata/tests/test_ata.py::Elkulonites`). A korábban dokumentált
„munkamappán kívüli írás” rést ez a zár zárta be. **Nem fedett:** a psx `cmd` lépései és a `claude`/`copilot` adapter (a sagent
jogaival futnak) – ezért ATA-gyűjtést nem psx-feladatként, hanem saját unitként (`ata/ata-figyelo.service`) futtatunk.
Erősebb elkülönítés (külön Unix-felhasználó a munkamenetnek) lehetséges; rendszermódosítás, ezért toti döntése.

## Telegram-feltételek (ellenőrizve 2026-11-04, elsődleges forrás)
- **API Terms of Service** (core.telegram.org/api/terms) 1.5: az API-használatra a *Terms of Service for Content Licensing and
  AI Scraping* is vonatkozik; tilos a platformról szerzett adatot AI/ML fejlesztésére, javítására vagy **telepítésére**
  („deployment”) használni, gyűjteni vagy aggregálni. 1.4: tilos a felhasználó tudta nélküli művelet és az olvasottsági
  állapot manipulálása (mi nem jelölünk olvasottnak, és nem is akadályozunk semmit). 2.1: saját api_id kell.
  2.3: az alkalmazás neve nem tartalmazhatja a „Telegram” szót (kivéve „Unofficial …”).
- **Content Licensing and AI Scraping** (telegram.org/tos/content-licensing): a felhasználói tartalomhoz csak a platform
  szokásos, rendeltetésszerű felhasználói használatán túl, egy legitim harmadik féltől származó kliens működtetéséhez szükséges
  mértékben szabad hozzáférni; be kell tartani a tartalom jogosultjának szerzői feltételeit. AI-felhasználás (scraping,
  indexelés, aggregálás, validálás, benchmark, telepítés) tilos; **kivétel**: ha minden érintett felhasználó egyenként
  **kifejezett, tájékozott, megerősítő és folyamatos** hozzájárulást ad, az adott tartalomra és csatornára/csevegésre
  korlátozva; más kontextusra nem vihető át.
- **Következmény:** a csoporttagság nem hozzájárulás. A 90 napos gyűjtéshez és az AI-elemzéshez a trader (és csoportban minden
  érintett szerző) igazolható hozzájárulása és a csatorna szerzői feltételeinek ismerete kell. Ezt csak toti szerezheti be;
  rögzítés: `ata hozzajarulas rogzit --forras <ID> --tipus gyujtes|ai --igazolas "…" --kor "…" --ki toti`.
  Addig: szintetikus próba. (Jogi tanácsnak nem minősül; a feltételek értelmezése bizonytalan pont: a kézi, nem AI-s
  személyes kutatási tárolás határa nincs egyértelműen kimondva → ezt is a hozzájárulás mögé tettük.)

## Adatmodell és feldolgozás (`ata/tarolo.py`, `ata/gyujto.py`, `ata/kinyeres.py`)
- Üzenet: forrás, üzenet-ID, eredeti időpont, szerző/aláírás, válaszkapcsolat, album-ID, első látás és módja.
- Változat: szöveg (képaláírás is), szerkesztési idő, látás ideje, mód (`tortenet`/`elo`), `eredeti_ismert`
  (történetiben szerkesztett üzenetnél 0 – az eredeti nem ismert). Az élő követés minden látott változatot külön megőriz.
- Média: chartképek ideiglenes fájlba, sha256, atomikus átnevezés; albumok egy elemzési egységben.
- Történeti gyűjtés: 90 nap, a legrégebbitől, minden üzenet után ellenőrzőpont → megszakítás után folytatható; ismételt
  futás nem duplikál; FloodWait: a kliens 120 s-ig magától vár, hosszabbnál kivárjuk és ellenőrzőpontról folytatjuk.
- Traderhez köthető: csatornában minden poszt; csoportban/privátban csak a `forrasok.json` `szerzok` listája (toti adja meg).
- Kinyerés: instrumentum, idősík, irány, belépési zóna, stop, célárak, feltételek, indoklás, módosítás/lezárás; kategória
  `terv` / `jelzes` / `modositas` / `eredmeny`; minden kitöltött mező forrásüzenetre (forrás + üzenet-ID-k + idézet) hivatkozik;
  ami nincs a szövegben, `null`. Kép: AI-hozzájárulás nélkül „nem elemezve”, árszint nem olvasódik ki.
- Kockázati értékek, stratégia, végrehajtó: még nincs (következő szakasz: szimuláció díjjal, csúszással, tényleges jelzésidővel;
  a trader módszere és toti kockázatvállalása külön paraméterként).

## Próba (2026-11-04, szintetikus)
`ata proba` / `~/.local/opt/ata-venv/bin/python -m unittest ata.tests.test_ata` (8 teszt OK): megszakított gyűjtés (3/6) →
folytatás (3) → újraindítás 0 új → teljes újraolvasás 6 duplikált, 0 új, 0 új kép → élő szerkesztés új változat, ugyanaz
újra 0 → album 2 képpel egy jelzésben → kategóriák terv/jelzés/módosítás/eredmény/terv → történetiben szerkesztett eredeti
ismeretlen → írási kérések tiltva → hozzájárulás nélkül gyűjtés/AI tiltva → psx-zár nem éri el a titokmappát.
**Valódi Telegram-kapcsolat még nincs kipróbálva** (api_id és belépés kell).

## Belépés (toti, telefonról)
1. Telefonos böngészőben: https://my.telegram.org → telefonszám nemzetközi formában → a Telegram-alkalmazásba érkező kódot
   a weboldalra írd → **API development tools**.
2. „Create new application”: App title `Project-S ATA kutatas` (a „Telegram” szó nem lehet benne), Short name `psata01`,
   URL üres, Platform **Other**, Description `Unofficial read-only research client` → **Create application**.
   (Ha „ERROR” jön: VPN/reklámblokkoló ki, vagy másik böngésző.) Az **App api_id** és **App api_hash** megjelenik – ne küldd el senkinek.
3. SSH (pl. Termius): `ssh toti@<tailnet-ip>`, majd `ata beallit` (a toti sudo-jelszava, utána api_id, api_hash),
   `ata belepes` (telefonszám, a Telegramba érkező kód, kétlépcsős jelszó – rejtve), végül `ata azonosit` és `ata allapot`.
Az új belépésről a Telegram értesítést küld; a munkamenet a Telegram Beállítások → Eszközök alatt bármikor megszüntethető.
