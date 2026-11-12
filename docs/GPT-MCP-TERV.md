<!-- Automatikus tükör a privát Projekt S műhelyrepóból (scripts/docs_tukor.py). Személyes és hálózati adatok kitakarva. Ne itt szerkeszd. -->

> **Történeti terv (2026-11-03).** A mai állapot: `mcp/README.md` és `docs/RENDSZER.md` – az MCP már a psx-en fut
> (`task_submit` stb.), a ChatGPT-plugin PC-n használható, mobilon nem.

# Terv: GPT ⇄ toti-server végrehajtó (MCP)

_Állapot: 1–4. lépcső kész (MCP-szerver, GitHub OAuth, systemd-unit, Funnel). Hátra: ChatGPT-connector + éles próba. Utolsó frissítés: 2026-11-03._

## Végcél
A ChatGPT-beszélgetés **feladatot indíthat** a szerver végrehajtóján, **állapotot kérdezhet le**,
**eredményt olvashat** és **leállíthatja** a feladatot. A fájlmegosztás (`kozos/`) csak köztes lépés.

## Architektúra

```
ChatGPT (connector, OAuth)
   │ HTTPS  (Tailscale Funnel: https://toti-server.<tailnet>.ts.net/mcp)
   ▼
[oauth-kapu]  ── OAuth 2.1 + PKCE, csak a toti85 GitHub-fiók   (127.0.0.1:8443)
   ▼
[project-s-mcp]  Python FastMCP, Streamable HTTP              (127.0.0.1:8808)
   │  feladat-nyilvántartás: ~/.local/state/project-s/jobs/<id>/
   ▼
[végrehajtó]  systemd-run --user --unit=ps-job-<id>  (sagent, cgroup-korlátokkal)
   │  futtatható: (a) Claude Code headless:  claude -p "<feladat>" --output-format stream-json
   │              (b) fehérlistás szkriptek:  ~/project-s-workshop/scripts/allowed/*
   ▼
~/project-s-workshop  (munkakönyvtár; git-ben követve)
```

Kész komponensek:
| Réteg | Komponens | Miért |
|---|---|---|
| MCP-szerver | `mcp` Python SDK (FastMCP), Streamable HTTP transport | hivatalos, kevés saját kód |
| Hitelesítés | FastMCP beépített OAuth-támogatása (auth provider + GitHub OAuth App), alternatíva: `oauth2-proxy` | ChatGPT-connector OAuth-ot vár |
| Folyamatkezelés | `systemd-run --user` (linger már be van kapcsolva) | izolált unit, leállítás, napló, erőforrás-korlát ingyen |
| Végrehajtó motor | `claude -p` (Claude Code headless) | már telepítve és hitelesítve van |
| Nyilvános HTTPS | Tailscale Funnel | tanúsítvány + port-nyitás nélkül |

## MCP-eszközök (tools)

| Eszköz | Bemenet | Kimenet | Megjegyzés |
|---|---|---|---|
| `task_start` | `prompt: str`, `kind: "claude"\|"script"`, `script?: str`, `args?: list[str]`, `timeout_min?: int (≤120)` | `{job_id, state:"queued"}` | `script` csak a fehérlistáról; egyszerre max. 2 futó feladat |
| `task_status` | `job_id` | `{state: queued\|running\|succeeded\|failed\|cancelled\|timeout, started_at, ended_at, exit_code, last_lines[20]}` | `systemctl --user show ps-job-<id>` + állapotfájl |
| `task_result` | `job_id`, `offset?: int`, `limit?: int (≤64 KB)` | `{text, next_offset, done}` | kimenet lapozva; Claude esetén a záró `result` üzenet külön mezőben |
| `task_cancel` | `job_id`, `reason?: str` | `{state:"cancelled"}` | `systemctl --user stop ps-job-<id>` (SIGTERM → 15 mp → SIGKILL) |
| `task_list` | `state?: str`, `limit?: int` | `[{job_id, state, title, started_at}]` | utolsó 50 feladat |
| `workspace_read` | `path` (a `kozos/` alatt) | fájltartalom | köztes lépés, csak olvasás |
| `workspace_write` | `path` (a `kozos/bejovo/` alatt), `content` | `{ok}` | köztes lépés |

Állapotgép: `queued → running → succeeded | failed | timeout`, bármikor `→ cancelled`.
Minden feladat könyvtára: `meta.json` (ki, mikor, mit), `output.log`, `result.json`, `exit_code`.

### Végrehajtó korlátok
- `systemd-run --user -p MemoryMax=4G -p CPUQuota=200% -p RuntimeMaxSec=<timeout>`
- `claude -p` engedélymódja: `acceptEdits` vagy `auto`, munkakönyvtár `~/project-s-workshop`;
  sudo-t a GPT-feladatok **nem** kapnak (külön, jelszavas sudo nélküli szolgáltatásfiók: később `psjob`).
- Minden indítás naplózva: `~/.local/state/project-s/audit.log` (időpont, OAuth-alany, eszköz, bemenet hash-e).

## Hitelesítés – a HTTPS bekapcsolása ELŐTT
Sorrend kötelező: **1 → 2 → 3, és csak utána Funnel.**

1. **GitHub OAuth App** létrehozása (toti85 fiók → Settings → Developer settings → OAuth Apps).
   Callback: a ChatGPT-connector által megadott redirect URI.
   Client ID/secret: `~/.config/project-s/oauth.env` (`chmod 600`), **nem** a repóban.
2. **OAuth-réteg** a `project-s-mcp` előtt: csak a `toti85` GitHub-felhasználó (numerikus ID-vel
   ellenőrizve) kaphat hozzáférési tokent; token élettartam ≤ 1 óra, frissítő token visszavonható.
3. **Helyi teszt** a Funnel előtt, csak `127.0.0.1`-en:
   - token nélkül → `401`;
   - más GitHub-fiókkal → `403`;
   - `task_start` → `task_status` → `task_result` → `task_cancel` végigfut;
   - `@modelcontextprotocol/inspector`-ral a tools lista helyes.
4. **Csak ezután:** `sudo tailscale funnel --bg --https=443 http://127.0.0.1:8443`, majd
   ChatGPT → Settings → Connectors (Developer mode) → új MCP-connector a Funnel-URL-lel, OAuth.

Vészleállítás: `sudo tailscale funnel reset` (azonnal megszűnik a nyilvános elérés).

## Lépcsők és becsült idő
| # | Lépés | Állapot |
|---|---|---|
| 0 | ChatGPT GitHub-connector → repó olvasása | azonnal, telepítés nélkül |
| 1 | `project-s-mcp` váz (FastMCP) + `task_*` eszközök, helyi teszt | 1–2 óra |
| 2 | GitHub OAuth App + OAuth-réteg, 401/403-tesztek | 1 óra |
| 3 | systemd-unit `project-s-mcp.service` | 15 perc |
| 4 | Tailscale Funnel + ChatGPT-connector | 15 perc |
| 5 | Külön `psjob` futtatófiók, szigorúbb jogok | később |

## Nyitott kérdések
- Melyik ChatGPT-csomag érhető el (a Developer mode / egyéni MCP-connector csomagfüggő)?
- A GPT-feladatok írhatnak-e közvetlenül a `main` ágra, vagy csak `gpt/<job_id>` ágra (ajánlott)?
