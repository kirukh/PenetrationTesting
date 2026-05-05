# 🛡️ Pentest Cheat-Sheet

> Mein persönliches Cheat-Sheet für Pentesting-Prüfungen.
> Aufbau folgt dem typischen Pentest-Flow: **Recon → Exploit → PrivEsc → Post-Exploit → Report**.

---

## 🚦 Workflow (so navigierst du dich durch die Prüfung)

```
1. Exploration   → was ist da? welche OS, welche Ports, welche Services?
2. Exploitation  → wie komme ich rein? (Initial Access)
3. PrivEsc       → wie werde ich root / SYSTEM / Admin?
4. Post-Exploit  → loot, persistence, pivoting
5. Reporting     → sauber dokumentieren
```

---

## 📂 Phasen

### 1. [Exploration / Recon](./01-Exploration/README.md)
> Hier startest du **immer**. Ziel: OS erkennen, offene Ports, Services, Versionen.

### 2. [Exploitation](./02-Exploitation/README.md)
> Erst hier rein, **wenn du in Phase 1 was Konkretes gefunden hast** (z.B. SMB offen, Webserver, FTP-Anon-Login).

### 3. [Privilege Escalation](./03-PrivEsc/README.md)
> Du hast einen Low-Priv Shell? Dann hier weiter.

### 4. [Post-Exploitation](./04-Post-Exploitation/README.md)
> Loot, Hashes dumpen, Pivoting auf andere Hosts.

### 5. [Reporting](./05-Reporting/README.md)
> Templates für Notizen während der Prüfung & finalen Report.

### 99. [Resources & Cheats](./99-Resources/README.md)
> Reverse-Shell-Cheats, Encoding, Wordlists, schnelle Referenzen.

---

## ⚡ Quick Start in der Prüfung

```bash
# 1. Variablen setzen
export RHOST=10.10.10.10
export LHOST=$(ip a show tun0 | grep -oP '(?<=inet\s)\d+(\.\d+){3}')
export LPORT=4444

# 2. Erster Recon
sudo nmap -sS -sV -sC -O -p- --min-rate=1000 -oN nmap_full.txt $RHOST

# 3. Notes-File anlegen
mkdir -p ~/exam/$RHOST && cd ~/exam/$RHOST
```

---

## 🧭 Entscheidungsbaum

| Was siehst du? | Geh zu |
|---|---|
| nmap zeigt Port 445/139 (SMB) | [SMB Enum](./01-Exploration/Services/smb.md) |
| nmap zeigt Port 80/443 (Web) | [Web Enum](./01-Exploration/Services/web.md) |
| nmap zeigt Port 21 (FTP) | [FTP](./01-Exploration/Services/ftp.md) |
| nmap zeigt Port 22 (SSH) | [SSH](./01-Exploration/Services/ssh.md) |
| nmap zeigt Port 3389 (RDP) | [RDP](./01-Exploration/Services/rdp.md) |
| OS = Windows | [Windows Enum](./01-Exploration/Windows/README.md) |
| OS = Linux | [Linux Enum](./01-Exploration/Linux/README.md) |
| Du hast eine Low-Priv-Shell auf Windows | [Windows PrivEsc](./03-PrivEsc/Windows/README.md) |
| Du hast eine Low-Priv-Shell auf Linux | [Linux PrivEsc](./03-PrivEsc/Linux/README.md) |
