<!-- Automatikus tükör a privát Projekt S műhelyrepóból (scripts/docs_tukor.py). Személyes és hálózati adatok kitakarva. Ne itt szerkeszd. -->

# Folytatás – hol tartunk

_Utolsó frissítés: 2026-11-11 20:00 UTC_

> **A teljes rendszer leírása: `docs/RENDSZER.md`.** **Új munkamenet:** a `CLAUDE.md` automatikusan betöltődik. Utána: `ugy lista`, majd a kért ügy
> (`ugy allapot <ügy>`, `ugy folytat <ügy>`). Rendszerellenőrzés: `systemctl status project-s-rc`,
> `scripts/gpt-kapcsolat.sh status`.

## ⏭️ HOL TARTUNK (2026-11-11 este) – a panel beszélgetéséből folytatható
- **Bevétel / ügynökpiac (ügy `ugynok-piac-felmeres`):** döntések: `eredmenyek/2026-11-11-dontesek.md`. Kiesett: Taskmarket,
  MoltJobs, toku, dealwork, általános kutatás; Moltbook parkol; kifizetési út nem akadály. **Következő: NEAR Agent Market**
  vevői keresletének csak olvasó felmérése. Listázás/jelentkezés/befizetés: toti döntése, még NEM.
  NEAR-belépés KÉSZ (2026-11-11, panel /near; az eszközkódos utat a NEAR csak adminoknak engedi). A token mcp-jogköre írásra is
  jó lenne → olvasni KIZÁRÓLAG `near-olvaso`-val (engedélylista + readOnlyHint). 35 nyitott vevői feladat látszott.
  Felmérés kész (`eredmenyek/near-kereslet-2.md`): 6 db „[internal research]” 15 USDC-s feladat illik; fedezet csak kiosztáskor.
  **NEAR-munkás kész** (panel `/near/munkas`, `scripts/near_munkas.py`, időzítő `project-s-near.timer`): toti BEKAPCSOLÁSA után
  önállóan keres, ajánl, elkészít (Claude idegen-tartalom módban), ellenőriz (Codex + gépi idézet), lead; keret NINCS (toti: előfizetésből dolgozik; körönként ≤10 új ajánlat),
  több feladatra is jelentkezhet. Időzítő BE, automatikus ajánlat BE (toti, 2026-11-12; kódos munkára csak javaslat). Választás: cím/címke előszűrés → teljes leírás besorolása
  (`scripts/near_minoseg.py`, előfizetéses Claude eszközök nélkül: vallalhato/nem/bizonytalan; bizonytalan → értesít, nem vállal);
  a vevő ajánlatban kért kérdéseire a Claude írja az ajánlatot. Leadás: vevői feltételek tételesen + idézet + alátámasztás (mind kell).
  Életút-napló (ajánlat→kiosztás→leadás→elfogadás→kifizetés) + ráfordítás. Napló: ~/.local/state/project-s/near/.
- **Termékjelölt: ellenőrző szolgáltatás** (`ellenoriz-jelentes`, panel /ellenorzes): jelenlét (gépi) + alátámasztás (modell
  véleménye, külön oszlop). Pontosság: ügy `ellenorzo-pontossag`. A `t20261111-173756-f4dda7` végső elfogadása NYITOTT: a GPT
  folyó szöveges jelentését (linkekkel exportálva) kell lefuttatni rajta.
- **Végrehajtó-választás:** webes kutatás = Claude, ellenőr = Codex; DeepSeek csak kód/rutin (webes kutatásban többször
  kitalált idézeteket). Ha a Claude-keret elfogy, a feladat vár és magától folytatja (`waiting_model`).
- **Videótár / torrent:** működik (Jellyfin, panel /dl, /fel). Telegram (ATA): belépés toti 2FA-jelszavára vár.

## Most használható
- **🔔 Telefonos értesítés (2026-11-10):** Web Push a panel-alkalmazásba (döntésre vár / kész / elutasítva / sikertelen /
  emberre vár, új rendszerlépés). `ertesites proba|allapot`. A Codex készítette, átnézve, toti telefonján igazolva.
- **🎬 Videótár (2026-11-09, ügy `media-konyvtar`):** Jellyfin, `/srv/media` (1 TB), csak tailnet: `http://<tailnet-ip>:8096`
  / `https://<tailnet-gép>:8920`; nézés: **Otthon** felhasználó jelszó nélkül. Próbavideó: Big Buck Bunny (CC BY).
  Torrent: panel `/dl` (qBittorrent, kategória → videótár). Feltöltés: panel `/fel` (videótár, .torrent, ügy, közös mappa). Következő: TV Tailscale-be (toti), <helyi-ip> mesék
  (`smb-belepes`), panel Média oldal, NVIDIA hardveres átkódolás.
- **💬 Beszélgető asszisztens (2026-11-08):** panel → Beszélgetés (`/c`). Codex (ChatGPT-előfizetés) / Claude / DeepSeek,
  menet közben váltható; ismeri a rendszert, utánanéz, és toti kérésére feladatot indít, root-lépést kér (jóváhagyás
  toti „jóváhagyom” üzenetével vagy a /r oldalon). `docs/VEGREHAJTO.md` 14.
- **Panel (2026-11-08):** `https://<tailnet-gép>:8443` (Tailscale kell; Androidon telepíthető „Projekt S”
  alkalmazás). Jelentések formázott dokumentumként, futás alatt készült fájlok kattinthatók, újraellenőrzés gomb,
  Telegram-keresés (`/tg`), **Rendszerlépések** (`/r`: a végrehajtók root-kérései toti jóváhagyásával). Végrehajtók: DeepSeek
  (saját ciklus), Claude, Copilot, Codex, CodeWhale (zár nélkül) – teljes jog a titkok kivételével. `docs/VEGREHAJTO.md` 11–13.
- **Erősebb végrehajtó (2026-11-05):** 5 független végrehajtó (DeepSeek/ingyenes saját ciklus, Claude, Copilot, **Codex** – belépés:
  SSH `codex-belepes`), saját ciklusban `web_search`/`web_fetch` (+ Playwright-böngésző) **pillanatkép-bizonyítékkal**,
  `idezetek:` elfogadási szabály (az idézeteket a rendszer függetlenül újra letölti), panelen **független ellenőr** (másik
  gyártó cáfolni próbál). Éles próba: a DeepSeek 2 forrás-hamis idézetét a gépi ellenőrzés kiszűrte (`ugyek/proba-bizonyitek`).
  Előfizetéses végrehajtók külön névleges kerettel (`max_elofizetes_usd`, alap 5). Leírás: `docs/VEGREHAJTO.md` 11.
  Telepítési felhatalmazás (toti, 11-05): a kísérleti szerveren telepítés kérdés nélkül; nyitott: AppArmor bwrap-profil (toti).
- **Szolgáltatók kifejezett választással (2026-11-05):** a panel modellválasztójában DeepSeek / ingyenes / **Claude** (előfizetés:
  webkeresés, erős elemzés) / **Copilot** (előfizetés: kód). Próba: Claude webkutatás `t20261105-094605-2940d8` (accepted;
  Metaculus-oldalak egy része 403, a FutureEval-forrásoldal és a GitHub elérhető → `ugyek/bevetel-kutatas/munka/metaculus-futureeval-2026-11-05.md`),
  Copilot kód+teszt `t20261105-094605-ad7311` (accepted, 9,5 kredit). GPT: nincs Codex CLI/node; út: OpenRouter GPT-profil (fizetős, toti döntése).
  A Claude/Copilot nem a Landlock-zár alatt fut (saját jogosultságkezelés).
- **ATA kereskedési kutatás (2026-11-04, ügy `ata-kereskedesi-kutatas`):** csak olvasó Telegram-kliens (Telethon, saját fiók),
  `ata beallit|belepes|allapot|azonosit|hozzajarulas|gyujt|figyel|proba` (SSH-ból is: `/usr/local/bin/ata`). Titkok:
  `~/.config/project-s/telegram/` (700/600), adatok `~/.local/share/project-s/ata/` – a psx-zár ezeket nem éri el.
  Szintetikusan tesztelve (8 OK); **valódi belépés toti lépésére vár**; tartalomgyűjtés és AI-elemzés a Telegram Content
  Licensing/AI-feltétele miatt **csak a trader rögzített hozzájárulásával**. Leírás: `docs/ATA.md`.
- **Ügyek a panelen (2026-11-04):** `https://<tailnet-gép>:8443/u` – ügy kiválasztása, állapot, feladatok,
  eredményfájlok olvasása/letöltése, **Folytatás** természetes nyelvű céllal (ugyanaz az `ugy` + psx; az ügy előzményei és
  olvasható forrásai automatikusan a promptba kerülnek; kötelező jelentés: Elkészült / Ellenőrizve / Bizonytalan / Következő lépés).
  Leírás: `docs/VEGREHAJTO.md` 10. szakasz. Az ügynök által indított folyamatok kernelszintű írási zár alatt futnak (Landlock, `executor/zar.py`; 10. szakasz).
- **Claude-tól független elérés (2026-11-04):** MCP `https://<tailnet-gép>/mcp` → psx (task_submit/list/status/
  log/result/cancel/decide; alapmodell DeepSeek V4 Flash, kis kerettel), szerveroldalon végponttól végpontig igazolva;
  **telefonos ChatGPT-ből még nem igazolt** (a korábbi kísérletből egyetlen OAuth-hívás sem érkezett – connector újracsatlakoztatása kell).
  SSH: `toti@<tailnet-ip>` (Tailscale), majd `psx …` / `ugy …`. Leírás: `docs/VEGREHAJTO.md` 9. szakasz.
  **ChatGPT mobilon a Project-S plugin csak „számítógépen” használható** (platformkorlát) → telefonos út: tailnet-panel
  `https://<tailnet-gép>:8443` (Tailscale bekapcsolva a telefonon; csak tailnet, Funnel nélkül).
  **Telefonról igazolva (2026-11-04):** beküldés → DeepSeek → bírálat → elfogadás a panelen; ügyhöz kötött beküldés is.
- **Saját ügynöki ciklus – valódi modellel igazolva (2026-11-04):** `or-free` (cohere/north-mini-code:free) egy hibajavítást
  elfogadottan elvégzett; a naplóelemzést nem (6/6 kísérlet, csővezetékes parancs ismétlése → leállás). `or-olcso`
  (deepseek/deepseek-v4-flash, kötelező költségkerettel) a naplóelemzést 1 kísérletben, 0,0044 USD-ért elfogadottan elvégezte.
  Részletek, mérés, javított hibák: `docs/VEGREHAJTO.md` 8. szakasz. Kulcs: `~/.config/project-s/openrouter.env` (600).
- **psx végrehajtó (2026-11-04)** – `executor/psx.py`, felügyelő: `project-s-executor.service` (user, max 3 párhuzamos).
  Tartós, hash-láncolt eseménynapló feladatonként; kísérletek külön systemd-unitban (független a Claude-munkamenettől);
  idempotens beküldés; lépésenkénti ellenőrzőpont, nem idempotens lépést nem ismétel vakon; modellimitnél munkacsomaggal
  átad (claude → copilot), ha nincs modell: `waiting_model` és automatikus folytatás; idő-, költség-, kísérlet- és
  párhuzamossági korlát; elfogadás csak feladathoz rögzített szabállyal. Az `ugy inditas` ezt használja.
  Próbák (mind a 6 + korlátok) és döntések: `docs/VEGREHAJTO.md`. Naplók: `~/.local/state/project-s/psx/tasks/<id>/`.
- **Ügykezelő (`ugy`)** – telefonról (Remote Control `toti-server-tmux` vagy SSH) ügy + cél →
  tartós állapot (`ugyek/<ügy>/allapot.json` + `UGY.md`) → háttérfeladat (`ps-job-<id>`,
  `claude -p`) → eredmény `eredmenyek/`-be → állapot, következő lépés, commit + push →
  `ugy allapot` rövid beszámoló. Leírás: `docs/UGYKEZELES.md`.
- **Végrehajtva ≠ elfogadva (2026-11-04):** indításkor `--elvart` / `--hova` / `--ellenorzes`;
  a végrehajtó sikeres kilépése után független ellenőrzés → `elfogadva` | `hianyos` | `elfogadasra_var`
  (szubjektív eredmény, ez az alap); döntés: `ugy elfogad` / `ugy elutasit`. Bemutató: `ugyek/proba-elfogadas/`.
- **Három ügy:** `project-s-ujjaepites`, `szerver-es-kapcsolatok`, `bevetel-kutatas`.
- **Teljes próba kész (2026-11-03):** a `szerver-es-kapcsolatok` ügyben
  1. `ugy inditas` → szerverleltár + állapotjelentés (háttérfeladat, ~1,5 perc) →
     `eredmenyek/20261103-231657-14da.md`, nyers adatok `munka/leltar-2026-11-03.txt`;
  2. egy **új, előzmény nélküli** `claude -p` munkamenet a „Folytasd a szerver ügyet” kérésre
     a `CLAUDE.md` alapján megtalálta és folytatta (root-ellenőrzés: SSH, ufw, SMART, mentés) →
     `eredmenyek/2026-11-03-root-ellenorzes.md`, következő lépés rögzítve.
  3. Leállítás (`ugy leallit`) ideiglenes ügyön kipróbálva → `cancelled`, rögzítve.
- **Mentés:** `scripts/mentes.sh` → `~/mentes/` (napi 03:17, `project-s-mentes.timer`, 14 nap);
  első kézi mentés kész. Az ügyek a GitHubon is megvannak.
- **MCP (ChatGPT „Project S” plugin)** fut – psx-alapú eszközök, minden végrehajtó választható; csak PC-n (`mcp/README.md`).

## Nyitott döntések (toti)
0. **Vond vissza az OpenRouteren** a nyilvános `project-s-unified` repó előzményében maradt régi kulcsokat (3 különböző érték;
   a jelenlegi kódból eltávolítva: `8fe850a`). Javasolt a mostani kulcs cseréje is, mert a beszélgetésben szerepelt.
1. SSH: kulcs a telefonra, utána jelszavas belépés tiltása (vagy csak Tailscale-ről).
2. ufw bekapcsolása (tailscale0 + LAN, 22-es port).
3. Különálló mentési cél: **Google Drive előkészítve (2026-11-08)** – `drive-belepes` (egyszer, toti; SSH 53682-es port-továbbítással), utána a napi mentés titkosítva (rclone crypt, csak drive.file jog) a Drive `ProjektS-mentes` mappájába, 30 nap.
   A visszafejtő jelszót toti mentse el: `cat ~/.config/project-s/mentes-jelszo`. Korábban: csak ugyanarra a lemezre ment. **Ne** a `/dev/sda1` (`Data3`) legyen: azon a
   2026-09-i fertőzés titkosított fájljai vannak (a DL380 azóta újratelepítve – ez a toti-server).
4. GitHub OAuth Client secret cseréje (chatben szerepelt).
5. A DEV laptop átnézése az augusztusi canonical mag után, amikor elérhető.
6. Az OVH VPS-1 (10-17-én egy hónapra fizetve) sorsa – a Linux-műhely most a toti-serveren fut.
Megjegyzés: a folytató munkamenet telepítette a `smartmontools` csomagot (SMART-olvasáshoz).

## Következő munka (javaslat)
- **Biztonság:** a Landlock-zár kiterjesztése a `claude`/`copilot` adapterekre (írható konfigurációs könyvtárakkal), ha újra használjuk őket.
- **Végrehajtó:** (1) MCP `task_*` átállítása psx-re (a `mcp/jobs.py` ma már csak a régi feladatokhoz és az MCP-hez kell);
  (2) OpenCode-adapter, ha toti belép egy szolgáltatóhoz (`opencode auth login`); (3) reboot utáni folytatás éles próbája.
- **Bevétel / Plether – LEZÁRVA, 0 jelölt (2026-11-04):** 6 átnézési kör 0 megalapozott jelölt; a 4 fuzz-FAIL mind PASS a CI-s
  Foundry v1.5.1-gyel (4/4 seed); a 3. kör (DeepSeek) „potenciális jelöltje” érvénytelen (atomikus revert). Döntés és bizonyítékok:
  `ugyek/bevetel-kutatas/eredmenyek/2026-11-04-plether-dontes.md`. Beküldés nem történt. Zárás 11-16 – új bizonyíték nélkül nem folytatjuk.
  Elfogadásra vár (toti, panel): `t20261104-143045-5b483b`, `t20261104-125111-a70aa2`, `t20261104-070542-c3b344`, `t20261104-070502-9bf260`.
0. Kész: történeti dokumentumok feldolgozva; végrehajtva ≠ elfogadva megvalósítva.
   **Data3 lezárva:** a nyers bájtkeresés 2026-11-04 00:30-kor végigfutott, egyetlen találat sincs (részletek:
   `ugyek/project-s-ujjaepites/UGY.md`). A Data3 csak olvashatóan csatolva maradt (`/mnt/data3`, blockdev ro;
   leválasztás: `sudo umount /mnt/data3`). Az augusztusi mag: DEV laptop, ha elérhető.
1. **Most:** `bevetel-kutatas` – a Plether lezárva; a döntési lap 2. változata (`munka/dontesi-lap-2.md`) toti elfogadására vár.
   Javasolt egyetlen próba: J1 (b) – 10 nyitott Metaculus Fall 2026 kérdés (toti bemásolja a
   `kozos/bejovo/metaculus-kerdesek-1.md` fájlba; a Metaculus a szerverről 403) → időbélyegzett, utólag nem
   módosított becslések, beküldés nélkül. **Most nincs API-költés és nincs regisztráció; a J3 parkol.**
2. A `ugy` műveletek MCP-eszközként (`case_list/status/start/continue`) – így később GPT is ügyet kezelhet.
3. Befejezéskor értesítés a telefonra (pl. a Remote Control-munkamenetbe vagy push-értesítés).

## Ne felejtsd
- Nagyobb Project-S-rendszert (SAI, tanácsrendszer, modellrouter) most nem építünk.
- Hitelesítő adatok csak a szerveren (`~/.config/...`), a repóba soha.
- A háttérfeladatok nem kapnak sudo-t (`NoNewPrivileges`); root-igényű lépés: közvetlenül a munkamenetben, utána `ugy rogzit`.
- A meglévő `project-s` tmux-munkamenetet ne töröld.
- Régi kód: `~/src/project-s-unified` (nem fut itt; célzott átnézés: `ugyek/project-s-ujjaepites/UGY.md`).
