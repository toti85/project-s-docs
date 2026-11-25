<!-- Automatikus tükör a privát Projekt S műhelyrepóból (scripts/docs_tukor.py). Személyes és hálózati adatok kitakarva. Ne itt szerkeszd. -->

# Szerver-képességek (generált: 2026-11-25 03:17 UTC, `scripts/kepessegek.py`)

> Mért állapot. Ha ez ellentmond egy régebbi ügyiratnak vagy javaslatlistának, EZ az érvényes.

## Végrehajtók (`psx providers`)
```
claude   elérhető  ok
copilot  elérhető  ok
codex    NEM elérhető  lehűlés 2026-11-25T10:44:00+00:00-ig
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

- ✅ `node`

## Telefonos értesítések
- Web Push: `~/bin/ertesites allapot`, próba: `ertesites proba`
- VAPID-kulcs létezik: ✅ (csak létezésvizsgálat)
- Feliratkozások: 1

## Mentés
- helyi: `~/mentes` (napi 03:17, 14 nap): 16 mentés
- Google Drive (titkosított): ✅ bekötve
- visszaállítási próba: ✅ RENDBEN (2026-11-24T23:12:29+00:00, 845/845 napló, `scripts/visszaallitas_proba.py`)

## Ügygazda (önálló irányító)
- állapot: bekapcsolva; saját ügyek: dontesi-sor-konzisztencia, akadaly-ismetlodes-csoportosito, node-hordozhato-telepites, mentes-drive-kvotahiba, nyitott-dontesek-kor-kimutatas, proba-vegpont-18080-diagnozis, akadaly-dontes-lezaras-javaslat, near-export-mezohiany-kimutatas, ugygazda-allapotfajl-olvashatosag, szolgaltatas-path-gh-node-lefedettseg, szolgaltatas-path-alkalmazas-csomag, path-javito-diff-alkalmazasi-kerelem-tervezet, psx-hibaosztaly-ismetlodes-rangsor, mentes-drive-rclone-korlat-alkalmazas, psx-hibaosztaly-ismetlodes-rangsor-ellenor, szolgaltatas-port-elo-proba, mentes-drive-rclone-korlat-javaslat-ellenor, kepesseg-gh-jogosultsag-ellenor, elfogadasi-szabaly-alak-ellenor, kezdemenyezes-cel-eloszlas-merleg, near-ajanlat-elozetes-kapu-elo-export-futtatas, visszaallitas-tmpdir-diff-alkalmazas-proba, mentes-300mb-kizaras-alkalmazas, onallosag-napi-igazolas-rogzito, onallosag-1-nap-igazolas-futtatas; döntési sor: 60; elakadás (összes/megoldott): 2/2
- leírás: `docs/UGYGAZDA.md`; NEAR külső műveletek: `scripts/near_kulso.py`; ügynapló: `ugy naplo ellenoriz`

## Főnök önállóan (szuper-végrehajtó)
- építőelemek: ✅ ellenőrző hurok (psx `loop`), ✅ eszközjegyzék, ✅ alügynök-csoportok, ✅ modellválasztás (`tipus`)
- csoport `proba-20261114-1`: 2/2 elfogadva (accepted, accepted)
- csoport `fonok-onallo-20261114`: 2/2 elfogadva (accepted, accepted)
- leírás: `docs/SZUPER-VEGREHAJTO.md`; napló: `~/.local/state/project-s/eszkozok/naplo.jsonl`

## Jog és zár
- Alapértelmezés: `teljes` jog (írás/futtatás mindenhol) a titkok/fiókadatok és sudo kivételével (`executor/zar.py` TELJES_KIVETEL); root csak `rendszerlepes`-kéréssel. Részletek: `docs/VEGREHAJTO.md` 11–13.
- A főnök (beszélgető asszisztens) zár nélkül, titkokkal és sudo-val fut (toti, 2026-11-14; `FONOK_ZAR=1` visszakapcsol).
