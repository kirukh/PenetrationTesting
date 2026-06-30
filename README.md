# 🛡️ Pentest Cheat-Sheet

> Mein persönliches Cheat-Sheet für Pentesting-Prüfungen.
> Aufbau folgt dem typischen Pentest-Flow: **Recon → Exploit → PrivEsc → Post-Exploit → Report**.

---

## 🎯 ZUERST LESEN: [Prüfungs-Spickzettel (CY-B-26)](./00-Pruefung/README.md)

Destillat aller Hinweise des Profs — geordnet, erklärt und in die Kapitel verlinkt.
Enthält u.a.: **Authentication Bypass**, **SMB (445/137)**, **NTLMv2/MD5-Cracking**,
**eigene Wortlisten aus Hinweisen**, **msfvenom (staged/stageless)**, **EternalBlue (Python)**,
**Scheduled Tasks/Services** & **SUID** für PrivEsc, **sensible Infos in Dateien/Backups/Mails**,
und der **DemoCorp-Bericht IPT-001**.
Begleitend: [Exploit-Troubleshooting (Python-Versionen, Kompilieren, curl-Trigger)](./00-Pruefung/troubleshooting.md).

> **Rahmen:** je **eine Linux- und eine Windows-11-Maschine**, beide **direkt von Kali**
> erreichbar. **Kein** Pivoting, **keine** Firewall, **kein** Evasion, **kein** UDP-Scan,
> **kein** XSS/SQLi/Burp.

---

## 🚦 Workflow (so navigierst du dich durch die Prüfung)

```
1. Exploration   → was ist da? welche OS, welche Ports, welche Services?
2. Exploitation  → wie komme ich rein? (Initial Access)
3. PrivEsc       → wie werde ich root / SYSTEM / Admin?
4. Post-Exploit  → loot, flags, sensible Daten
5. Reporting     → sauber dokumentieren (DemoCorp-Stil!)
```

---

## 📂 Phasen

### 0. [Prüfungs-Spickzettel](./00-Pruefung/README.md)
> Die Prof-Hinweise, erklärt. Hier anfangen.

### 1. [Exploration / Recon](./01-Exploration/README.md)
> Hier startest du **immer**. Ziel: OS erkennen, offene Ports, Services, Versionen.

### 2. [Exploitation](./02-Exploitation/README.md)
> Erst hier rein, **wenn du in Phase 1 was Konkretes gefunden hast** (Web, SMB, Service-Version).

### 3. [Privilege Escalation](./03-PrivEsc/README.md)
> Du hast einen Low-Priv Shell? Dann hier weiter.

### 4. [Post-Exploitation](./04-Post-Exploitation/README.md)
> Loot, Hashes, sensible Daten. Inkl. [PCAP/Forensik-Loot](./04-Post-Exploitation/Loot-Sources/pcap-und-forensik-artefakte.md).

### 5. [Reporting](./05-Reporting/README.md)
> Templates + **[DemoCorp-Bericht IPT-001](./05-Reporting/democorp-report.md)** (kommt dran!).

### 99. [Resources & Cheats](./99-Resources/README.md)
> Reverse-Shells, Encoding, Wordlists, msfvenom, Steganografie, schnelle Referenzen.

---

## ⚡ Quick Start in der Prüfung

```bash
# 1. Variablen setzen
export RHOST=10.10.10.10
export LHOST=$(ip a show tun0 | grep -oP '(?<=inet\s)\d+(\.\d+){3}')
export LPORT=4444

# 2. Erster Recon (kein UDP nötig)
nmap -p- -T5 $RHOST -oN nmap_all.txt
nmap -sV -sC -p <ports> $RHOST -oN nmap_detail.txt

# 3. Notes-File anlegen
mkdir -p ~/exam/$RHOST && cd ~/exam/$RHOST
```

---

## 🧭 Entscheidungsbaum

| Was siehst du? | Geh zu |
|---|---|
| nmap zeigt Port 445/139/137 (SMB) | [SMB Enum](./01-Exploration/Services/smb.md) |
| nmap zeigt Port 80/443 (Web) | [Web Enum](./01-Exploration/Services/web.md) |
| Login-Maske / geschützter Bereich (Web) | [Authentication Bypass](./02-Exploitation/Web/README.md) |
| nmap zeigt Port 21 (FTP) | [FTP](./01-Exploration/Services/ftp.md) |
| nmap zeigt Port 22 (SSH) | [SSH](./01-Exploration/Services/ssh.md) |
| nmap zeigt Port 3389 (RDP) | [RDP](./01-Exploration/Services/rdp.md) |
| nmap zeigt Port 5900 (VNC) | [VNC](./01-Exploration/Services/vnc.md) |
| Port 445 + altes Windows (SMBv1) | [EternalBlue MS17-010](./02-Exploitation/Windows/eternalblue-ms17-010.md) |
| OS = Windows | [Windows Enum](./01-Exploration/Windows/README.md) |
| OS = Linux | [Linux Enum](./01-Exploration/Linux/README.md) |
| Hash gefunden (NTLMv2/MD5/…) | [Passwords & Hashes](./02-Exploitation/Passwords/README.md) |
| Nur Hinweise, kein Passwort | [eigene Wortliste bauen → Hydra](./00-Pruefung/README.md) |
| Exploit wirft Error / Py2 vs Py3 | [Troubleshooting](./00-Pruefung/troubleshooting.md) |
| .pcap / SAM-SYSTEM-Hives gefunden | [PCAP & Forensik-Loot](./04-Post-Exploitation/Loot-Sources/pcap-und-forensik-artefakte.md) |
| Low-Priv-Shell auf Windows | [Windows PrivEsc](./03-PrivEsc/Windows/README.md) |
| Low-Priv-Shell auf Linux | [Linux PrivEsc](./03-PrivEsc/Linux/README.md) |
| Weak/dumme Shell | [Shell-Upgrade](./99-Resources/shell-upgrade.md) |
| Bericht schreiben | [DemoCorp / IPT-001](./05-Reporting/democorp-report.md) |
