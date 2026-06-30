# 🚀 Phase 3 — Privilege Escalation

> Du hast eine Shell. Aber als Low-Priv-User. Ziel: **root** (Linux) bzw. **SYSTEM/Administrator** (Windows).

## 📂 Unterverzeichnisse

### [→ Linux PrivEsc](./Linux/README.md)
SUID, sudo, Capabilities, Cron, Kernel, GTFOBins.

### [→ Windows PrivEsc](./Windows/README.md)
Token, Service, Registry, GPP, AlwaysInstallElevated, etc.

## 🛠️ Automatisierte Tools (immer als erstes laufen lassen!)

| OS | Tool | Befehl |
|---|---|---|
| Linux | LinPEAS | `curl -L https://github.com/peass-ng/PEASS-ng/releases/latest/download/linpeas.sh \| bash` |
| Linux | LinEnum | `wget .../LinEnum.sh && bash LinEnum.sh` |
| Linux | linux-exploit-suggester | `./linux-exploit-suggester.sh` |
| Windows | WinPEAS | `winpeas.exe` |
| Windows | PowerUp | `Import-Module PowerUp.ps1; Invoke-AllChecks` |
| Windows | PrivescCheck | `Import-Module .\PrivescCheck.ps1; Invoke-PrivescCheck` |
| Windows | Seatbelt | `Seatbelt.exe -group=all` |
| Windows | windows-exploit-suggester | gegen `systeminfo`-Output |

## 📌 Shell-Stabilisierung (zuerst!)

→ [99-Resources/shell-upgrade.md](../99-Resources/shell-upgrade.md)

## 🧭 Prof-Ablauf (so vorgehen)

`wer/wo bin ich` (`id`, `whoami`, `hostname`) → andere User? Dateien/Mails mit Hinweisen?
→ `sudo -l` und Ergebnisse bei [GTFOBins](https://gtfobins.github.io/) prüfen → **linpeas
*und* PrivescCheck** laufen lassen → reused Passwords testen → **SUID-Binaries** suchen →
GTFOBins. Kompakte Schritt-für-Schritt-Variante: [Prüfungs-Spickzettel §8](../00-Pruefung/README.md).
