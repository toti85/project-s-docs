<!-- Automatikus tükör a privát Projekt S műhelyrepóból (scripts/docs_tukor.py). Személyes és hálózati adatok kitakarva. Ne itt szerkeszd. -->

# Project-S workshop

Saját szerveren futó, telefonról irányítható AI-munkarendszer: ügyek → végrehajtók (Claude, Codex, Copilot, DeepSeek,
CodeWhale) → végrehajtótól független ellenőrzés → emberi döntés → tartós ügyirat.

**A teljes rendszer leírása: [`docs/RENDSZER.md`](docs/RENDSZER.md)** (önmagában érthető, AI-olvasónak is).
Hol tartunk: [`FOLYTATAS.md`](FOLYTATAS.md) · mért állapot: [`docs/KEPESSEGEK.md`](docs/KEPESSEGEK.md).

## Környezet
| Elem | Érték |
|---|---|
| Szerver | `toti-server` (Ubuntu 24.04), munkafelhasználó `sagent` |
| Elérés | Tailscale `<tailnet-ip>` / `<tailnet-gép>`; LAN `<helyi-ip>` |
| Panel | `https://<tailnet-gép>:8443` (csak tailnet) |
| MCP (ChatGPT) | `https://<tailnet-gép>/mcp` (Funnel + GitHub OAuth) |
| Munkamappa | `/home/sagent/project-s-workshop` · repó: `github.com/toti85/project-s-workshop` (privát) |

## Elérés
- **Panel** (telefon, tailnet): feladatok, ügyek, beszélgető asszisztens, rendszerlépések, Telegram-keresés.
- **SSH**: `toti@<tailnet-ip>`, majd `ugy`, `psx`, `ata`, `rendszerlepes`, `kepessegek` (a `sagent` fiók alatt futnak).
- **Claude Code Remote Control**: `project-s-rc.service` → `toti-server-tmux` (`~/bin/project-s-rc.sh status`).
- **ChatGPT**: „Project S” plugin (MCP) – PC-n.

## Mappaszerkezet
- `executor/` – psx végrehajtó, saját ügynöki ciklus, elfogadási kapu, webes bizonyítékréteg, Landlock-zár, beszélgetés
- `scripts/` – `ugy` (ügykezelő), `rendszerlepes`, `kepessegek`, mentés, segédek
- `mcp/` – panel (`panel.py`) és MCP-szerver (`server.py`)
- `ata/` – Telegram-kliens (csak olvasó)
- `ugyek/` – ügyek (tartós állapot + eredmények) · `kozos/` – közös munkatér
- `docs/` – leírások (`RENDSZER.md`, `VEGREHAJTO.md`, `UGYKEZELES.md`, `ASSZISZTENS.md`, `ATA.md`, `KEPESSEGEK.md`)
- `install/` – systemd-unitok és telepítő

## Szabály
Jelszó, token, privát kulcs **nem** kerülhet a repóba (`.gitignore`, `scripts/titok_ellenor.py` pre-commit hook).
