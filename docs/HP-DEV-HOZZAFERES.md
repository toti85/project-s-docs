<!-- Automatikus tükör a privát Projekt S műhelyrepóból (scripts/docs_tukor.py). Személyes és hálózati adatok kitakarva. Ne itt szerkeszd. -->

# hp-dev hozzáférés – tényleges állapot

2026-11-14. **Előkészített integráció, nem igazolt működő távoli hozzáférés.**
A szerverkonzolos korábbi sikert a felhasználó közölte; ez nem plugin/PSX bizonyíték.

## Megoldás és határok

Célzott vizsgálat: `mcp/server.py`, `mcp/README.md`, `executor/psx.py`,
`executor/zar.py`, `scripts/elesites.py`, mért `KEPESSEGEK.md`.
Meglévő távoli gép/parancsfuttató API nem található ezekben. A plugin támogatott
`task_submit` útja változatlanul PSX-feladatot indít. Új általános fájlfelderítés nincs.

A `scripts/hp_dev.py hp-dev whoami`, `... hp-dev hostname`,
`... hp-dev sudo -n whoami` kliens a `/run/project-s-hp-dev/control.sock`
socketet hívja. A rendszer-szolgáltatás sagentként fut, root tulajdonú kódból,
0700 socketkönyvtárban, kernel SO_PEERCRED sagent ellenőrzéssel.
Ez explicit, három műveletes képesség minden sagent végrehajtónak; nem általános shell.
A Landlock titoktiltása és a Projekt S felhatalmazási szabályzata változatlan.
A teljes laptopadminisztráció általános parancsútja **még nincs megvalósítva**;
az engedélyezés újragondolása nélkül nem vezettem be tetszőleges sudo-végrehajtót.

Fix SSH-cél: `kriszti35@<tailnet-ip>`; IdentityFile:
`/home/sagent/.ssh/project_s_hp_dev`. Privát kulcsot nem olvastam/másoltam.
Nincs SSH-agent, felhasználói SSH-konfiguráció vagy jelszó fallback.
A telepítő az ED25519 kulcsot csak a megadott
`SHA256:f7ug08hL1O4Dtcyg3TDQQwekGQqZx4vgDmdht03ZZvI`
ujjlenyomat egyezésekor telepíti; SSH StrictHostKeyChecking=yes.
A root-tulajdonú known_hosts külön `/etc/project-s/hp-dev.known_hosts`.
A kérés, socket és SSH időkorlátos. A szolgáltatás naplója kizárólag művelet/siker;
SSH stderr és tetszőleges távoli kimenet nem jut naplóba vagy klienshez.
Csak a várt három érték kerülhet sikeres válaszba.

## Bizonyíték és folyamat

`python3 -m unittest discover -s executor/tests -p test_hp_dev.py`: 4 teszt PASS.
Valódi Unix socket és kernel peer-azonosítás tesztelve; az SSH ebben mockolt,
tehát ez nem laptopkapcsolat. Cél/extra paraméter/parancsinjektálás elutasítása is tesztelt.
A teljes csomag első futása socket-útvonalhosszon hibázott; a teszt rövid,
explicit /tmp mappát használó javítása után újrafuttatva (eredmény lent).

Valódi Project-S plugin `task_submit` útból indított próba:
`t20261114-103844-e54622`, három engedélyezett művelet, emberi elfogadással.
Eredmény: needs_acceptance; mindhárom indítás rc=2, Permission denied a másik
feladat védett munkafájljának olvasásakor. SSH nem indult; nincs kapcsolatbizonyíték.
Telepítés után a kliens hivatalos útja `/usr/local/lib/project-s/hp_dev.py`;
ez a meglévő rendszerkönyvtár-olvasási engedélyen át elérhető, zárbővítés nélkül.

A `hp-zbook-linux-atallas` ügyet a meglévő `ugy allapot` nem találta.
A plugin nem kínál ügyregisztrációs API-t. A helyi `ugy uj` automatikusan pushol,
ezért a kifejezett push-tilalom miatt nem hívtam meg, és nem kerültem meg.

## Telepítés és hátralevő munka

Független, más gyártójú PSX-átnézés és az eredeti `scripts/elesites.py` kapu kell.
A telepítő kizárólag az éles `/home/sagent/project-s-workshop` példányból fut;
megköveteli a változást tartalmazó, naplózott ellenőrzött élesítést.
Root-lépés (a git-objektumból, nem a munkafáról): `GIT_NO_REPLACE_OBJECTS=1 git -C /home/sagent/project-s-workshop show <commit>:scripts/hp_dev_install.sh | sudo GIT_NO_REPLACE_OBJECTS=1 bash -s -- <commit>`.
Ez hostkulcsot ellenőriz, root tulajdonú kódot/unitot telepít, szolgáltatást indít;
sem SSH-kulcsot, sem laptopadatot nem módosít.
A pontos rendszerlépés-kérés az alábbi folytatási bejegyzésben szerepel.

Élesítés előtt: az eredeti élesítő automatikusan pushol; a jelen feladat push-tilalma
miatt nem indítható így. Nem módosítottam megkerülésként az élesítőt.
Jóváhagyott telepítés után ismét a pluginból indítandó a három próba.
Elvárt kimenet sorrendben: `kriszti35`, `kriszti35-HP-ZBook-17-G5`, `root`.
Csak ennek PSX-bizonyítékával nevezhető működőnek. Driver/játék/reboot/adatmódosítás nincs.

## Folytatási bejegyzés

- Teljes `scripts/tesztek.sh`: **ZÖLD**, exit 0 (javítás utáni futás).
- Telepítés: **megtörtént** (2026-11-14, `2cd9598` commitból, a fenti git-objektumos paranccsal; a korábbi r20261114-104011-73eb2c rendszerlépés-kérés és a munkafás
  parancs elavult, NEM használandó).
- A végső dokumentációs commit után a meglévő PSX-átnézési mintával külön,
  Claude/Copilot (Anthropic, tehát más gyártó a szerző OpenAI-hoz képest) feladat kerül beküldésre;
  azonosítója a végrehajtói összefoglalóban. Elfogadását nem állítom előre.
- Az átnézés önmagában nem telepítés. Élesítés/push-tilalom rendezése,
  rendszerlépés, majd a telepített klienssel valódi plugin–PSX újrapróba szükséges.

## Frissítés 2026-11-14 (átnézés utáni javítás, toti: a laptop teljes sudo-adminisztrálása engedélyezve)
- A telepítő (`scripts/hp_dev_install.sh <commit>`) root-ként csak a commitolt, origin/main-ben lévő tartalomból telepít (`git show`); a sagent által
  írható munkafát és az `elesites.jsonl`-t nem olvassa (az átnézés 1–2. kifogása). A hostkulcs a szerveren ellenőrzött `~/.ssh/known_hosts_hp_dev`.
- Az egységre `CapabilityBoundingSet=` és `ProtectKernel*/ProtectControlGroups` került.
- Az adminisztrációt (illesztőprogram, energiabeállítás, alkalmazások) a Claude Code-munkamenet közvetlenül végzi a `~/bin/hp` segéddel
  (szigorú hostkulcs-ellenőrzéssel); a végrehajtóknak/pluginnak a három művelet marad – általános parancsút nincs bekötve (döntés kérdése).
- 2. átnézés (t20261114-105953-7cc435) kifogásai javítva: `refs/replace` kizárva (`--no-replace-objects`, teljes commit-azonosító), a root-lépés a
  git-objektumból fut (nem a sagent által írható munkafáról).

## Laptop beállítások és mérések (2026-11-14, `~/bin/hp`-vel, a szerverről)
- **Energia:** hálózati tápról nincs automatikus altatás (`sleep-inactive-ac-type='nothing'` a felhasználónál + `/etc/dconf/db/gdm.d/90-project-s-power`
  és `/etc/dconf/profile/gdm` a bejelentkező képernyőre); akkumulátoron 15 perc után alvás marad; kijelző-kikapcsolás (idle-delay 300 s) és képernyőzár változatlan;
  a fedélkezelés (lecsukott fedél = suspend) érintetlen. Az indok: 10:15-kor a gép hálózati tápról is elaludt (GNOME idle-ág). Újraindítás után is érvényes [MÉRT].
- **NVIDIA:** nvidia-driver-580 (580.178.04, Pascal utolsó támogatott ága; `ubuntu-drivers`: „recommended”), `linux-modules-nvidia-580-7.0.0-43-generic` (Canonical által aláírt modul,
  Secure Boot ki), nouveau tiltva, `prime-select nvidia`; `nvidia-smi` működik, a gnome-shell és a Roblox Studio a GPU-n fut [MÉRT]. Újraindítás nem kellett.
- **Sober:** flatpak 1.8.0 (rendszerszintű), indítóikon: `~/Desktop/Sober.desktop` („Sober Roblox (NVIDIA)”, megbízható, `flatpak run org.vinegarhq.Sober`).
  Hiba volt: a sandboxban nem volt NVIDIA-bővítmény, a Sober az Intel UHD 630-on futott (napló 11:37) → telepítve `org.freedesktop.Platform.GL(32).nvidia-580-178-04//1.4`,
  és `flatpak override` az alkalmazásra: `MESA_VK_DEVICE_SELECT=10de:1cba!`, `DRI_PRIME=1`, `__NV_PRIME_RENDER_OFFLOAD=1`, `__GLX_VENDOR_LIBRARY_NAME=nvidia`, `__VK_LAYER_NV_optimus=NVIDIA_only`.
  Mérés: a Sober naplója „Vulkan Device: Quadro P2000, Vendor 10de Device 1cba, Driver 580.x, API 1.4.312”, és a folyamat az `nvidia-smi`-ben a GPU-n [MÉRT 12:03 CET].
  Játékmentések, süti, `~/.var/app/org.vinegarhq.Sober` érintetlen. Visszavonás: `flatpak override --reset org.vinegarhq.Sober`.
- **Híd igazolása a feladat útján:** psx-feladat t20261114-110425-6e817e (zárolt Codex): whoami=kriszti35, hostname=kriszti35-HP-ZBook-17-G5, sudo=root; a híd szerveroldali naplója
  ugyanezt három `ok:true` bejegyzéssel mutatja. A zárból a `~/.ssh` és a privát kulcs olvasása tiltott (Permission denied) [MÉRT].
