<!-- Automatikus tükör a privát Projekt S műhelyrepóból (scripts/docs_tukor.py). Személyes és hálózati adatok kitakarva. Ne itt szerkeszd. -->

# S Bridge 3.0 – a telefon hídja

A telefonon futó S Bridge app **maga kérdezi le** a szervert (pull, tailneten, eszköztokennel); a szerver nem nyit
kapcsolatot a telefon felé. Szerver: `scripts/sbridge_szerver.py` (`project-s-sbridge.service`, 127.0.0.1:8841,
kívülről `tailscale serve` :8840, tailnet only). App: `~/project-s-android/s-bridge-3` (ág: `sbridge-3.0`), telepíthető
APK: `~/project-s-android/kiadas/`. Kezelőfelület: Control Center → Telefon (`/cc#/sbridge`).

## Mit tud (3.0)
| Csoport | Típusok | Feltétel |
|---|---|---|
| megfigyelő | `ping`, `allapot`, `kepessegek`, `ertesites` | mindig kiadható |
| a telefon tartalmát olvassa | `kepernyokep`, `ui_dump`, `ertesitesek`, `appok`, `fajl_lista` | jóváhagyás |
| a telefont vezérli | `koppint`, `hosszan`, `huz`, `elem_koppint`, `szoveg`, `gomb`, `app_indit`, `url_nyit`, `vagolap`, `hang`, `rezges`, `csenget`, `zseblampa`, `tts`, `ebreszt`, `fajl_le`, `fajl_fel` | jóváhagyás |

- **Jóváhagyás:** `sbridge jovahagy <jid> <típus,…> [--perc] [--db]`, majd `sbridge feladat <típus> '<json>' --jid <jid>`
  (fájl a telefonra: `sbridge feladat fajl_le '{}' --jid <jid> --melleklet <fájl>`).
  A Control Centerből indított parancs egyszeri, 10 perces jóváhagyást kap; a Főnök/végrehajtók telefonos munkájához
  toti ad tartós jóváhagyást (Telefon → Jóváhagyások). A szerver jóváhagyás nélkül ilyen feladatot nem ad ki
  (`elutasitva`), az app pedig a jóváhagyásjelző (`jid`) nélkül érkezőt nem hajtja végre.
- **Helyi kontroll a telefonon:** az app „Leállítás” gombja / értesítése, a szerveroldali vészleállító, és a
  Kisegítő lehetőségek kapcsolója – kikapcsolva minden távvezérlés és képernyőolvasás megszűnik.
- **Azonnali kézbesítés:** hosszú lekérdezés (`var_mp` ≤ 25); emulátoron mért köridő ~0,3 mp.
- **Fájlok:** a képernyőkép, UI-fa, értesítés- és applista, mappalista és a telefonról kért fájl a
  `$SBRIDGE_DIR/fajlok/<feladat>.<kit>` alá kerül (0600); szerver → telefon: `fajl_le` (melléklet a `kimeno/` tárból).
- **Életben maradás:** a rendszer által kötött szolgáltatások (akadálymentesítés, értesítés-hozzáférés) és egy 15 perces
  ébresztő újraindítják a kapcsolatot, ha a rendszer kilőtte a folyamatot (csak ha a felhasználó nem állította le).
- A 2.0-s app változatlanul működik (ping, állapot, értesítés); a szerver új mezőit figyelmen kívül hagyja.

## Telepítés a telefonra
Control Center → Telefon → „APK letöltése a telefonra” → megnyitás (frissíti a meglévő `.dev` csomagot, azonos
aláírással) → az appban: token beillesztése (ha kell), „Távvezérlés bekapcsolása”, igény szerint értesítésolvasás és
fájlátvitel, akkuoptimalizálás kikapcsolása, MIUI automatikus indítás.

## Próbák
```sh
python3 scripts/test_sbridge_szerver.py                       # szerver (a tesztek.sh része)
# valódi app Android 13 emulátoron, elkülönített állapottal (az APK-t SB_E2E_URL=http://127.0.0.1:8851-gyel kell fordítani):
python3 scripts/sbridge_e2e_emulator.py <app-debug.apk> --jelentes ki.json
```
Emulátor: `~/.local/opt/android/sdk` (AVD: `sb33`, Android 13), indítás:
`emulator -avd sb33 -no-window -no-audio -gpu swiftshader_indirect`.

## Megőrzés és kötések
- A feltöltött fájlokból a legfrissebb 60 darab / 200 MB marad meg, és csak amíg a feladatuk a sorban van; a kimenő
  melléklet a kézbesítés lezárultával törlődik (`selejtez`).
- A kiadott feladat a **tokenhez** kötött (nem az eszköz nevéhez): azonos nevű új token nem veszi át a régi feladatait.
- A kezelőfelület csak akkor ad ki 3.0-s (olvasó / vezérlő) feladatot, ha a FUTÓ szolgáltatás a `/health`-en `v3`-at
  hirdet – a jóváhagyást a futó példány kényszeríti ki, ezért régi példánynak ilyen feladat nem megy.

## Élesítés
A szolgáltatás a `~/.local/opt/sbridge/sbridge_szerver.py` másolatot futtatja (`python3 -I`): a kapun átment változat
után `install -m 600 scripts/sbridge_szerver.py ~/.local/opt/sbridge/ && systemctl --user restart project-s-sbridge`.
