<!-- Automatikus tükör a privát Projekt S műhelyrepóból (scripts/docs_tukor.py). Személyes és hálózati adatok kitakarva. Ne itt szerkeszd. -->

# Szerver-képességek (generált: 2026-11-12 10:18 UTC, `scripts/kepessegek.py`)

> Mért állapot. Ha ez ellentmond egy régebbi ügyiratnak vagy javaslatlistának, EZ az érvényes.

## Végrehajtók (`psx providers`)
```
claude   elérhető  ok
copilot  elérhető  ok
codex    elérhető  ok
codewhale elérhető  ok
or-free  elérhető  ok
or-olcso elérhető  ok
local    NEM elérhető  a végpont nem válaszol (http://127.0.0.1:11434/v1): URLError
proba    NEM elérhető  a végpont nem válaszol (http://127.0.0.1:18080/v1): URLError
```

## Eszközök

- ✅ **Böngésző (Playwright + Chromium, fej nélkül)** – `~/.local/opt/bongeszo-venv/bin/python` – a saját ciklus `web_fetch browser=true`; venv: ~/.local/opt/bongeszo-venv, böngésző: ~/.cache/ms-playwright

- ✅ **Codex CLI** – `~/.local/opt/codex/bin/codex` – belépve, ha ~/.codex/auth.json létezik: ✅

- ✅ **CodeWhale** – `~/.local/opt/codewhale/codewhale-linux-x64/codewhale` – zár nélkül, teljes joggal (toti döntése)

- ✅ **Foundry v1.5.1 (Plether CI-verzió)** – `~/.foundry-1.5.1/bin/forge`

- ✅ **Telethon (ATA, csak olvasó)** – `~/.local/opt/ata-venv/bin/python` – munkamenet-fájl (a belépést nem garantálja): ✅ (belépés: `ata allapot`)

- ✅ **Jellyfin videótár (/srv/media, tailnet: http://<tailnet-ip>:8096, https …:8920)** – `/usr/lib/jellyfin/bin/jellyfin`

- ✅ **Torrentkliens (qBittorrent, API 127.0.0.1:8085, panel /dl)** – `/usr/bin/qbittorrent-nox`

- ✅ **rendszerlépés-kérés (root, toti jóváhagyásával)** – `~/bin/rendszerlepes` – `rendszerlepes ker "…" --miert "…"`

- ✅ `rclone`

- ✅ `bwrap`

- ✅ `gh`

- ✅ `git`

- ✅ `python3`

- ✅ `pdftotext`

- ❌ `node`

## Telefonos értesítések
- Web Push: `~/bin/ertesites allapot`, próba: `ertesites proba`
- VAPID-kulcs létezik: ✅ (csak létezésvizsgálat)
- Feliratkozások: 1

## Mentés
- helyi: `~/mentes` (napi 03:17, 14 nap): 10 mentés
- Google Drive (titkosított): ❌ még nincs bekötve (`drive-belepes`, toti)
- visszaállítási próba: ✅ RENDBEN (2026-11-12T09:46:40+00:00, 55/55 napló, `scripts/visszaallitas_proba.py`)

## Ügygazda (önálló irányító)
- állapot: bekapcsolva; saját ügyek: near-hiba-gyokerok; döntési sor: 17; elakadás (összes/megoldott): 0/0
- leírás: `docs/UGYGAZDA.md`; NEAR külső műveletek: `scripts/near_kulso.py`; ügynapló: `ugy naplo ellenoriz`

## Jog és zár
- Alapértelmezés: `teljes` jog (írás/futtatás mindenhol) a titkok/fiókadatok és sudo kivételével (`executor/zar.py` TELJES_KIVETEL); root csak `rendszerlepes`-kéréssel. Részletek: `docs/VEGREHAJTO.md` 11–13.
