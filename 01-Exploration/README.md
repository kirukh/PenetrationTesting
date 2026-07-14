# 🔎 Phase 1 — Exploration / Recon

> Ziel: In den ersten 10–15 Minuten die Angriffsfläche beider Maschinen erfassen und danach gezielt arbeiten.

## 1. Standardscan

```bash
export RHOST=<TARGET-IP>
nmap -p- -T5 $RHOST -oN nmap_all.txt
nmap -sCV -p <PORTS> $RHOST -oN nmap_detail.txt
```

- `-p-`: alle **TCP**-Ports.
- `-T5`: schneller Prüfungs-Scan im lokalen Lab.
- `-sCV`: Versionserkennung und Standard-Skripte nur auf offenen Ports.

> **Hinweis Prof:** Kein vollständiger UDP-Scan. Nicht 20 Scanvarianten lernen.

## 2. Windows-Recheck nur bei unplausiblem Ergebnis

`-T5`, Paketverlust oder eine Firewall können einzelne Antworten verschlucken. Wenn ein Hinweis auf Windows/SMB/MSSQL besteht, erwartete Ports gezielt und etwas langsamer nachprüfen:

```bash
nmap -Pn -sCV -T4 -p 80,135,139,445,1433,3389,5985,5986 $RHOST \
  --reason -oN nmap_windows_recheck.txt
```

- Liste nur um **konkret erwartete** Ports ergänzen.
- Port `137` ist normalerweise NetBIOS Name Service über UDP. Kein Full-UDP-Scan; nur bei einer ausdrücklichen Aufgaben-/Professorvorgabe gezielt: `sudo nmap -sU -p 137 $RHOST`.

## 3. Service priorisieren

| Port | Prüfen |
|---|---|
| 80/443/8080 | [Web](./Services/web.md): Source, Login, Reset, Cookies, Upload, `/dev`, Backups |
| 445/139 | [SMB](./Services/smb.md): Guest/Null, Shares, Dateien, RID-Brute, NTLMv2 |
| 1433 | [MSSQL](./Services/mssql.md): Login, Datenbanken, Impersonation, `xp_cmdshell` |
| 22 | [SSH](./Services/ssh.md): gefundene/selbst gebaute Passwörter und Keys |
| 21 | [FTP](./Services/ftp.md): Anonymous, Backups, Upload |
| 3389 | [RDP](./Services/rdp.md): gefundene Credentials |
| 5985/5986 | [WinRM](./Services/winrm.md): Passwort oder lokaler NTLM-Hash |
| 5900 | [VNC](./Services/vnc.md): schwaches/gefundenes Passwort |

## 4. Produkt, Version und PoC

```bash
whatweb http://$RHOST/
searchsploit '<PRODUKT> <VERSION>'
grep -iE 'http|smb|ssh|ftp|mssql|winrm|version|title' nmap_detail.txt
```

PoC vor Ausführung lesen:

```bash
sed -n '1,220p' exploit.py
grep -nEi 'requests|url|headers|cookies|data|params|payload|cmd|lhost|lport' exploit.py
```

→ HTTP-PoC vereinfachen: [Python-Exploit → curl](../99-Resources/python-exploit-zu-curl.md)

## 5. Sofortnotizen

```markdown
# Target: <IP>
OS/Architektur:
Ports/Versionen:
Webpfade/Cookies:
SMB-Shares/Dateien:
User/Anwendungsnamen:
Passwörter/Hashes:
PoC-Kandidaten:
Nächster Test:
```

> **Hinweis Prof:** E-Mails, Textdateien, Webseiten, Backups, App-Daten und Shell-History liefern oft den nächsten Schritt.
