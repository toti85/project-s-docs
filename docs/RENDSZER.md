<!-- Automatikus tükör a privát Projekt S műhelyrepóból (scripts/docs_tukor.py). Személyes és hálózati adatok kitakarva. Ne itt szerkeszd. -->

# Projekt S – a teljes rendszer leírása (AI-olvasónak is)

_Állapot: 2026-11-11. Repó: `github.com/toti85/project-s-workshop` (privát). Ez a dokumentum önmagában érthető; a
részletek a hivatkozott fájlokban vannak. A mért, napi frissülő állapot: `docs/KEPESSEGEK.md`._

## 1. Mi ez, és kinek készült
A Projekt S egy **saját szerveren futó, telefonról irányítható AI-munkarendszer** egy felhasználónak (toti; magyar,
Kárpátalja; jellemzően telefonról dolgozik). Feladatot lehet adni természetes nyelven; különböző AI-„végrehajtók”
(Claude, Codex, Copilot, DeepSeek, CodeWhale) elvégzik a szerveren; a rendszer a végrehajtótól **függetlenül ellenőrzi**
az eredményt; a döntés (elfogadás/elutasítás) mindig az emberé; és minden nyom tartósan megmarad (ügyirat, napló, git).

Alapelvek:
1. **Végrehajtva ≠ elfogadva.** A végrehajtó „kész” jelzése nem bizonyíték; előre rögzített gépi szabályok és ember dönt.
2. **Bizonyíték, nem önbevallás.** Webes állításhoz szó szerinti idézet + mentett pillanatkép; kódhoz futtatott teszt.
3. **Tartós állapot a szerveren**, nem a beszélgetésben: bármelyik munkamenet (Claude, ChatGPT, panel) folytathatja.
4. **Titok soha nem kerül repóba vagy AI elé**; a végrehajtók OS-szintű zárban futnak (kivétel: a szándékosan zár nélküli CodeWhale).
5. **Kicsi, összerakható részek** a meglévő eszközökre építve; nincs párhuzamos infrastruktúra.

## 2. Hardver és hálózat
- Gép: `toti-server` (HP DL380 Gen9, Ubuntu 24.04, 72 mag, 125 GB RAM), felhasználó `sagent` (jelszó nélküli sudo; a
  végrehajtók NEM kapnak sudót). LAN `<helyi-ip>`.
- Elérés: **Tailscale** (`<tailnet-ip>`, `<tailnet-gép>`) – csak a saját tailnetből.
  - Panel: `https://<tailnet-gép>:8443` (tailnet-only, Tailscale-identitással; Androidon telepíthető PWA).
  - MCP (ChatGPT-connector): `https://<tailnet-gép>/mcp` (Tailscale Funnel, GitHub OAuth).
  - SSH: `toti@<tailnet-ip>`, onnan a parancsok (`ugy`, `psx`, `ata`, …) a `sagent` fiók alatt futnak.
  - Claude Code Remote Control: `project-s-rc.service` (tmux `toti-server-tmux`) – ezen keresztül beszél toti Claude-dal.
- Minden szolgáltatás újraindítás után magától indul (systemd; user-unitok `Linger=yes`).

## 3. Rétegek áttekintése
```
 toti (telefon / PC)
   ├─ Panel (web, tailnet)  ── feladatok, ügyek, beszélgetés, rendszerlépések, Telegram-keresés
   ├─ ChatGPT (PC) ── Project S plugin = MCP ──┐
   ├─ Claude (Remote Control / SSH) ───────────┤
   └─ SSH parancsok (ugy, psx, …) ─────────────┤
                                               ▼
                      Ügykezelő `ugy`  (ügyek: tartós állapot, ügyirat, git)
                                               ▼
                      psx végrehajtó  (feladat → lépések → végrehajtó → ellenőrzés)
                       │  végrehajtók: or-olcso(DeepSeek) · or-free · claude · codex · copilot · codewhale
                       │  zár: executor/zar.py (Landlock) · root csak `rendszerlepes` + toti jóváhagyás
                       ▼
                      Elfogadási kapu (executor/acceptance.py) → toti döntése → ügyirat + git
```

## 4. Ügyek (`scripts/ugy.py`, parancs: `ugy`; leírás: `docs/UGYKEZELES.md`)
- Egy **ügy** = egy hosszabb cél (pl. `bevetel-kutatas`, `ata-kereskedesi-kutatas`, `project-s-ujjaepites`).
  Mappa: `ugyek/<ügy>/` – `allapot.json` (gépi forrás: állapot, cél, következő lépés, feladatok), `UGY.md` (ügyirat,
  generált fejléc + napló), `eredmenyek/` (jelentések), `munka/` (munkaanyag).
- Fő parancsok: `ugy lista`, `ugy uj`, `ugy allapot <ügy>`, `ugy inditas <ügy> "<feladat>" --elvart … --ellenorzes …`,
  `ugy folytat`, `ugy elfogad|elutasit <ügy> [id]`, `ugy rogzit` (kézi munka rögzítése), `ugy lezar`.
- Ügyhöz kötött feladat promptja automatikusan tartalmazza az ügy célját, következő lépését, a legutóbbi eredményeket,
  az olvasható forrásokat (`olvashato` az allapot.json-ban), és megköveteli a jelentés szerkezetét:
  `## Elkészült / ## Ellenőrizve / ## Bizonytalan / ## Következő lépés`.
- A feladat végén hook frissíti az ügyet (állapot, összefoglaló), commitol és pushol.

## 5. A psx végrehajtó (`executor/psx.py`, parancs: `psx`; leírás: `docs/VEGREHAJTO.md`)
- **Feladat (spec):** cím, munkakönyvtár (`dir` / `worktree` / `empty`), lépések (`cmd` vagy `agent`), elfogadási
  szabályok, korlátok (idő, költség, kísérletszám).
- **Napló:** feladatonként append-only, hash-láncolt `events.jsonl` (`~/.local/state/project-s/psx/tasks/<id>/`); az
  állapot ebből számolt. Felügyelő: `project-s-executor.service` (max 3 párhuzamos), kísérletenként külön systemd-unit.
- **Életciklus:** `queued → running → verifying → accepted | needs_acceptance | rejected`; közben `waiting_model`
  (limit/nincs modell → magától folytatja), `waiting_human`, `failed`, `cancelled`. Lépésenkénti ellenőrzőpont;
  megszakítás után onnan folytat; limitnél munkacsomaggal átad a következő megadott szolgáltatónak.
- **Végrehajtók** (`psx providers`):
  | név | mi | keret | zár |
  |---|---|---|---|
  | `or-olcso` | saját ügynöki ciklus (`executor/agent.py`) DeepSeek V4 Flash-sel az OpenRouteren, árplafonnal | OpenRouter, feladatonkénti USD-keret | Landlock |
  | `or-free` | ugyanez, csak ingyenes modellekkel | 0 | Landlock |
  | `claude` | Claude Code headless (`claude -p`) | toti Claude Pro kerete (közös a beszélgetéseivel) | saját engedélykezelés |
  | `codex` | OpenAI Codex CLI (`codex exec`) | toti ChatGPT-előfizetése | Landlock (a teljes folyamat) |
  | `copilot` | GitHub Copilot CLI | Copilot-kredit | Landlock |
  | `codewhale` | CodeWhale (DeepSeek), `exec --auto` | OpenRouter, keret NEM érvényesül | **nincs** (toti döntése) |
- **Saját ciklus eszközei:** fájl listázás/olvasás/keresés/írás/szerkesztés, `run` (shell, csővezeték), `web_search`,
  `web_fetch` (urllib vagy Playwright-böngésző) **pillanatképpel**, szándékrekord + helyreállítás, titokszűrés,
  fordulókeret („kerettel zárás”), kérés-határidő.
- **Jog („teljes” az alapértelmezés):** írás/futtatás bárhol a gépen, KIVÉVE `zar.TELJES_KIVETEL` (kulcsok `~/.config`,
  `~/.ssh`, fiókadatok `~/.codex` `~/.claude*` `~/.copilot`, ATA-adat, mentések, psx-napló/bizonyíték); sudo nincs.
- **Root-lépés:** `rendszerlepes ker "<parancs>" --miert "…"` → panel `/r` (vagy beszélgetésben toti „jóváhagyom”
  üzenete) → `sudo` futtatás 15 perc korláttal, sha256-egyezéssel; hiteles napló `~/.local/state/project-s/rendszerlepes/`.

## 6. Elfogadási kapu és bizonyítékréteg (`executor/acceptance.py`, `executor/web.py`)
- Szabályok: `letezik:`, `min-meret:`, `tartalmaz:`, `nem-tartalmaz:`, `json:`, `parancs:`, **`idezetek:<fájl>[:min]`**, `ember`.
- `idezetek:` a jelentés `[[idézet: "…" | URL]]` jelöléseit **függetlenül** ellenőrzi: a végrehajtó eredeti pillanatképe
  és egy friss letöltés is; ítélet `igazolt` / `hamis` (az oldal megvolt, a szöveg nincs benne) / `ellenőrizhetetlen`
  (emberi döntés). Csak a szöveg jelenlétét igazolja – azt, hogy az idézet alátámasztja-e az állítást, külön kell nézni.
- **Független ellenőr lépés:** egy másik gyártó modellje (kutatásnál: Claude kutat, Codex ellenőriz) cáfolni próbálja a jelentést (IGAZOLT/CÁFOLT/ELLENŐRIZHETETLEN
  táblázat); cáfolt állítás → hiányos. Ez külön vélemény, nem bizonyíték.
- Eddigi valós találatok: DeepSeek forrás-hamis idézetei (kivonatot tulajdonított egy 403-as oldalnak), ellenőr-modell
  „összerakott” idézetei, Claude önbevallott „15 ellenőrzött idézete” (valójában 0) – mindet a gépi kapu fogta meg.
- Ítélet: minden gépi szabály teljesül + nincs `ember` → `accepted`; van `ember` → `needs_acceptance`; bármi bukik → `rejected`/hiányos.

## 7. Felületek
- **Panel** (`mcp/panel.py`, `project-s-panel.service`): feladatlista, beküldés (végrehajtó-választás), feladat-részlet
  (összefoglaló formázva, jelentésfájl-linkek, futás alatt készült fájlok, gépi ellenőrzés, **újraellenőrzés** gomb,
  elfogadás/elutasítás/leállítás), ügyek (állapot, fájlok dokumentumnézetben, „Folytatás” természetes nyelvű céllal,
  független ellenőr jelölőnégyzet), **Beszélgetés** (`/c`), **Rendszerlépések** (`/r`), **Telegram-keresés** (`/tg`).
  Védelem: Tailscale-identitás (`<e-mail>`), CSRF + Origin, idempotens beküldés.
- **Beszélgető asszisztens** (`executor/beszelgetes.py`, útmutató `docs/ASSZISZTENS.md`): szálak, agy választható és
  menet közben váltható – **Codex** (ChatGPT-előfizetés, folytatható munkamenet), **Claude** (Pro), **DeepSeek**.
  Ismeri a rendszert (útmutató + FOLYTATAS + KEPESSEGEK + ügylista + ügyirat), utánanéz, és toti kérésére/rábólintására
  műveletet hajt végre (`ps-chat inditas|leallit|root-ker|root-jovahagy`) a megbízható feldolgozón át. Root csak toti
  saját, mentett „jóváhagyom” üzenetére. Ha a beszélgetésben kód változik, automatikusan indul a teljes tesztcsomag + egy másik gyártó kódátnézése. Tárolás csak a szerveren (`~/.local/state/project-s/beszelgetesek/`).
- **MCP** (`mcp/server.py`, `mcp/README.md`): ChatGPT „Project S” plugin – `task_submit/list/status/log/result/cancel/decide`,
  `workspace_read/write`; ugyanazon a psx-állományon. ChatGPT-ben csak PC-n használható (mobilon platformkorlát).
- **SSH-parancsok:** `ugy`, `psx`, `ata`, `rendszerlepes`, `kepessegek`, `codex-belepes`, `drive-belepes`, `smb-belepes`.

## 8. Mellékrendszerek
- **ATA / Telegram** (`ata/`, `docs/ATA.md`): csak olvasó Telethon-kliens toti fiókjával (engedélylistás API-hívások),
  keresés a saját csevegésekben (`ata keres`, panel `/tg`, nem ment, AI nem dolgozza fel). Belépés: `ata belepes`
  (kóddal) vagy `ata belepes --qr` (QR; képként a panelen: `/tg/qr`); a fiókon kétlépcsős jelszó van beállítva. Tartalomgyűjtés és AI-elemzés
  a Telegram „Content Licensing and AI Scraping” feltétele miatt csak az érintettek rögzített hozzájárulásával.
- **Képességleltár** (`scripts/kepessegek.py` → `docs/KEPESSEGEK.md`): mért állapot (végrehajtók, eszközök, mentés);
  minden végrehajtó és az asszisztens erre hivatkozik, hogy ne írjon elavult állapotot.
- **Mentés:** napi 03:17 `scripts/mentes.sh` → `~/mentes` (14 nap; titkok nélkül); utána `scripts/mentes-drive.sh`
  titkosítva (rclone crypt, csak `drive.file` jog) a Google Drive `ProjektS-mentes` mappába, 30 nap – bekötés:
  `drive-belepes` (toti, egyszer). Visszaállítási próba még nem volt.
- **Videótár (Jellyfin)** (ügy: `ugyek/media-konyvtar/`): Jellyfin 12.4, 1 TB-os médiakötet `/srv/media`
  (`Mesek/`, `Filmek/`, `Sorozatok/`), csak a Tailscale-címen és helyben figyel (`http://<tailnet-ip>:8096`,
  HTTPS `https://<tailnet-gép>:8920`), a helyi LAN felől nem érhető el. Felhasználók: **Otthon** (jelszó
  nélkül, nézéshez), `toti` (rendszergazda; jelszó + API-kulcs: `~/.config/project-s/jellyfin.env`). Kliensek: Android TV,
  Android, böngésző. Hardveres átkódolás (NVIDIA Tesla M60) még nincs bekapcsolva (meghajtó kell).
- **Torrentkliens** (qBittorrent 4.6, `project-s-torrent.service`): webes felület csak helyben (`127.0.0.1:8085`), tailneten
  `https://<tailnet-gép>:8090`; panel `/dl` (magnet/URL hozzáadása kategóriával, haladás, szünet, törlés).
  Kategóriák (automatikus torrentkezeléssel) → `/srv/media/Filmek|Mesek|Sorozatok`, a félkész fájlok `/srv/media/Letoltesek/.reszleges` alatt; a Jellyfin a kész
  fájlt magától felveszi. Az asszisztens a helyi API-n át kezeli. Tartalmat toti választ; zárt warez-tracker automatizálása nincs.
- **Telefonos értesítés** (`scripts/ertesites.py`, Web Push): a panel főoldalán bekapcsolható; értesít a feladat
  needs_acceptance/accepted/rejected/failed/waiting_human állapotánál és új rendszerlépés-kérésnél (csak azonosító, rövid cím,
  állapot). VAPID-kulcs: `~/.config/project-s/ertesites/` (600). `ertesites proba|allapot`.
- **Feltöltés** (panel `/fel`): több fájl egyszerre, haladásjelzővel, fájlonként ≤60 GB, a szerver darabonként a lemezre írja.
  Célok: videótár (Mesék/Filmek/Sorozatok), `.torrent` → letöltés a választott kategóriába, `kozos/bejovo`, bármely ügy
  `munka/feltoltes/` mappája. Azonos nevet nem ír felül.
- **Jelentés-ellenőrző** (`ellenoriz-jelentes`, panel `/ellenorzes`): külső jelentés idézet–URL párjai és linkjei →
  jelenlét (gépi: igazolt / nincs az oldalon (most) / ellenőrizhetetlen / link él·halott / eszközhiba) + alátámasztás (másik
  gyártó modellje: igen/részben/nem/nem eldönthető, külön oszlop). Kutatási ügyfeladatoknál az elfogadáskor automatikusan
  elkészül (tájékoztató). Pontosság: ügy `ellenorzo-pontossag` (30 eset; ismert korlát: jelenlét ≠ alátámasztás).
- **NEAR Agent Market:** belépés a panel `/near` oldalán (webes OAuth + PKCE, a visszairányított címet toti beilleszti); token
  `~/.config/project-s/near.env` (az mcp-jogkör írásra is jó!) → olvasás csak `near-olvaso`-val (engedélylista, readOnlyHint).
- **NEAR-munkás** (`scripts/near_munkas.py`, panel `/near/munkas`, időzítő): toti bekapcsolása után a kereten belül önállóan
  keres, besorol (teljes leírás, Claude eszközök nélkül: vallalhato/nem/bizonytalan), ajánl, elkészít, ellenőriz
  (vevői feltételek + idézet + alátámasztás) és lead; életút- és ráfordításnapló; vevői/pénzes műveletet nem végez; Claude idegen-tartalom módban (nincs shell, titokmappa tiltva).
- **Nyilvános dokumentációs tükör:** github.com/toti85/project-s-docs (`scripts/docs_tukor.py`): a `docs/`, `FOLYTATAS.md`,
  README és az újjáépítés ügyirata, e-mail/IP/tailnet-címek kitakarva, titokellenőrzéssel – a GPT- és a Claude-projekt ezt olvassa.
  Frissítés: `python3 scripts/docs_tukor.py` (időzítő: `install/systemd-user/project-s-docs-tukor.timer`).
- **Tesztcsomag:** `scripts/tesztek.sh` (végrehajtó + ATA tesztek + szintaxis); a beszélgetésben készült kódnál automatikusan fut.
- **SMB-hozzáférés:** `smb-belepes [IP]` – Windows-megosztás belépési adatai a repón kívülre (pl. a <helyi-ip> mesemappája).
- **Titokvédelem:** `scripts/titok_ellenor.py` pre-commit hook; titkok: `~/.config/project-s/` (600), soha nem repóba.

## 9. Bevételi kutatás – eddigi eredmények (`ugyek/bevetel-kutatas/`)
- Plether (Immunefi Boost) audit: 6 kör, 0 jelölt, a 4 fuzz-FAIL eszközverzió-eltérés (Foundry v1.5.1-gyel PASS) → lezárva.
- Pénzdíjas GitHub-issue (Expensify #104504): túlzsúfolt (~35 javaslat) → nem folytatva. HackenProof: bejelentkezett
  nézet botvédelem mögött, megkerülés tilos. Metaculus FutureEval botverseny: díjalap 50k USD/szezon, AI a műfaj része –
  nyitott jelölt (regisztráció toti döntése; a kifizetési út nem akadály).

## 10. Szabályok röviden (toti döntései)
- A kifizetési út (<helyi> fogadás) nem akadály: magyar bankszámla vagy kriptotárca (toti, 2026-11-11).
- Titok, token, jelszó soha nem kerül repóba/chatbe/AI elé. Külső beküldés, regisztráció, pénzköltés, fiókbelépés: toti.
- A toti-server kísérleti gép: telepítés és rendszerbeállítás kérdezés nélkül mehet (utólag jelezni).
- Végrehajtók teljes joggal (titkok és sudo nélkül); CodeWhale zár nélkül; root csak jóváhagyással.
- OVH-t nem módosítjuk. A `project-s` tmux-munkamenetet nem töröljük. Git-előzményt nem írunk át.
- Nagy „önálló AI”-rendszert (SAI, tanácsrendszer, modellrouter) nem építünk; kicsi, ellenőrizhető lépésekben haladunk.

## 11. Nyitott pontok
- Hálózati proxy domain-engedélylistával a saját ciklusnak; ügyenkénti eseménynapló; visszaállítási próba.
- Telegram-belépés (toti; kétlépcsős jelszó kell), Drive-bekötés (toti), MCP-próba ChatGPT-ből az új végrehajtókkal.
- Videótár: TV Tailscale-be léptetése (toti), a <helyi-ip> mesék átmásolása (`smb-belepes`), panel Média oldal +
  „Lejátszás a TV-n”, NVIDIA-meghajtó.
- A régi canonical mag (DEV laptop) sorsa; a 09-15-i review kétágú migrációs döntése.

## 12. Hol mi van (fájltérkép)
| Út | Mi |
|---|---|
| `CLAUDE.md` | utasítás új Claude Code-munkamenetnek |
| `FOLYTATAS.md` | hol tartunk, nyitott döntések |
| `docs/RENDSZER.md` | ez a leírás |
| `docs/VEGREHAJTO.md` | psx részletesen (1–14. szakasz: döntések, próbák, mérések) |
| `docs/UGYKEZELES.md` · `docs/ASSZISZTENS.md` · `docs/ATA.md` · `docs/KEPESSEGEK.md` | ügykezelés · asszisztens · Telegram · mért állapot |
| `executor/` | `psx.py` (végrehajtó), `agent.py` (saját ciklus), `modell.py` (OpenRouter/helyi modellfelület), `acceptance.py`, `web.py`, `bongeszo.py`, `zar.py` (Landlock), `beszelgetes.py`, `tests/` |
| `scripts/` | `ugy.py`, `rendszerlepes.py`, `kepessegek.py`, `mentes*.sh`, `drive-belepes.sh`, `titok_ellenor.py`, `project-s-rc.sh` |
| `mcp/` | `panel.py` (panel), `server.py` (MCP), régi `jobs.py`/`runner.py` (az MCP előtti végrehajtó, ma nem ezt használjuk) |
| `ata/` | Telegram-kliens és kutatási tároló |
| `install/` | systemd-unitok (`systemd-user/`, `systemd/project-s-rc.service`), root-telepítő |
| `ugyek/` · `kozos/` | ügyek · közös munkatér (bejovo/kimeno/jegyzetek) |
| `/srv/media` | videótár (Jellyfin-könyvtárak) |
| `~/.local/state/project-s/` | psx-feladatok, beszélgetések, rendszerlépés-napló (nem repó) |
| `~/.config/project-s/` | titkok (nem repó) |

- Külső jelentések idézet- és linkellenőrzése: `ellenoriz-jelentes <fájl>`, panel: `/ellenorzes`; bizonyítékcsomaggal, háttérben (részletek: [VEGREHAJTO.md](VEGREHAJTO.md#külső-jelentés-ellenőrzése)).
