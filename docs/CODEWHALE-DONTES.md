<!-- Automatikus tükör a privát Projekt S műhelyrepóból (scripts/docs_tukor.py). Személyes és hálózati adatok kitakarva. Ne itt szerkeszd. -->

# Döntés-előkészítés: CodeWhale fogyasztás-elkülönítés és titokelérés

_2026-11-12. Döntés: toti. A jelenlegi zár nélküli működést ez az anyag NEM módosítja._

## Tények
- [LEÍRÁS – `executor/psx.py` CodeWhaleAdapter] A CodeWhale `exec --auto` (Full Access) módban, **zár nélkül** fut
  (toti döntése, 2026-11-08), a **közös OpenRouter-kulccsal** (ugyanaz, amit az `or-olcso` használ). A psx költségkerete és
  árplafonja itt nem érvényesül; a költséget a nyilvános árlistából becsüli.
- [MÉRT 2026-11-12 10:08 UTC, toti-server, `sagent`] A felhasználó jogaival olvasható: `~/.config/project-s/oauth.env`,
  `near.env` (NEAR-token, írásra is jó), `openrouter.env`, `telegram/` (munkamenet), `~/.config/gh/hosts.yml`
  (GitHub-token). → A zár nélküli CodeWhale **mindezt olvashatja**.
- [MÉRT 2026-11-12, psx-naplók] Eddig 6 CodeWhale-költségesemény, összesen ~0,12 USD (becsült), ismeretlen költségű nincs.

## Mit old meg a külön API-kulcs, és mit nem
- **Megoldja:** a CodeWhale fogyasztása külön mérhető, és az OpenRouteren a kulcsra **külön kreditlimit** tehető (a
  kiadás felső határa akkor is érvényes, ha a psx-keret nem).
- **Nem oldja meg:** a többi titok (NEAR-, GitHub-, Telegram-, OAuth-token) olvashatóságát. Zár nélkül a folyamat a
  felhasználó minden fájlját eléri; a külön kulcs ezt önmagában nem védi.

## Lehetőségek (toti választ)
| | Mi változik | Előny | Ár |
|---|---|---|---|
| A | Marad minden | nincs korlátozás | közös kulcs, nincs külön limit, minden titok olvasható |
| B | Külön OpenRouter-kulcs kreditlimittel (toti hozza létre az OpenRouteren) | mérhető, felülről korlátos költség; a zár nélküli működés marad | a titokelérés nem változik |
| C | B + „teljes” Landlock-zár a CodeWhale-re is (mint a többi végrehajtónál: mindenhez hozzáfér, kivéve a titokmappákat) | a titkok védve | GitHub-push a `~/.config/gh` tiltása miatt nem menne – külön, szűk kivétel kellene (pl. csak a repó deploy-kulcsa) |

**Javaslat:** B most (nem korlátoz, csak mér és felülről határol), C külön döntésként, ha a CodeWhale-t külső tartalommal
(weboldal, idegen feladatleírás) dolgoztatjuk – ott a beágyazott utasítás kockázata a titkokra valós.
