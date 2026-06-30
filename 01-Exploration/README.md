# 🔎 Phase 1 — Exploration / Recon

> **Ziel:** Information sammeln, ohne (noch) zu exploiten.
> **Output:** Liste offener Ports, Services, Versionen, OS, evtl. User/Shares.

---

## 🚀 Step 1 — Initial Nmap Scan

```bash
# Variablen
export RHOST=10.10.10.10

# Schneller First-Look (alle Ports, kein Service-Detect, schnell)
sudo nmap -p- --min-rate=2000 -oN nmap_quick.txt $RHOST

# Detail-Scan auf gefundene Ports (PORTS=22,80,445 z.B.)
sudo nmap -sS -sV -sC -O -p $PORTS -oN nmap_detail.txt $RHOST

# Alles-in-einem (langsamer, aber bequem)
sudo nmap -sS -sV -sC -O -A -p- --min-rate=1000 -oN nmap_full.txt $RHOST

# UDP (oft vergessen! Top 100 reicht meistens)
sudo nmap -sU --top-ports 100 -oN nmap_udp.txt $RHOST
```

### Nmap Flags – was tun sie?
| Flag | Bedeutung |
|---|---|
| `-sS` | TCP SYN Scan (stealth, default als root) |
| `-sV` | Service-Version-Detection |
| `-sC` | Default-Scripts (NSE) |
| `-O` | OS-Detection |
| `-A` | Aggressive (= `-sV -sC -O --traceroute`) |
| `-p-` | alle 65535 Ports |
| `--min-rate=1000` | Mindest-Pakete/Sek (Speed-Boost) |
| `-oN` | Output Normal (lesbar) |
| `-oA` | Output All (3 Formate auf einmal) |

---

## 🪟 vs 🐧 — Welches OS?

Wenn du nicht weißt was du vor dir hast:

| Indikator | Wahrscheinlich |
|---|---|
| Port 445, 139, 3389, 5985 | **Windows** |
| Port 22 (OpenSSH), 111, manchmal 80/Apache | **Linux** |
| TTL ~128 in ping | Windows |
| TTL ~64 in ping | Linux |
| `Server: Microsoft-IIS/...` | Windows |
| `Server: Apache/.. (Ubuntu)` | Linux |

→ Dann gehst du je nach Ergebnis in:

## 📂 Unterverzeichnisse

### [→ Windows Enumeration](./Windows/README.md)
Wenn das Ziel eine **Windows-Maschine** ist (Server, AD, etc.).

### [→ Linux Enumeration](./Linux/README.md)
Wenn das Ziel eine **Linux-Maschine** ist.

### [→ Services (Port-spezifisch)](./Services/README.md)
Egal ob Win oder Lin — nach Port/Service sortiert:
- [FTP (21)](./Services/ftp.md)
- [SSH (22)](./Services/ssh.md)
- [SMTP (25)](./Services/smtp.md)
- [DNS (53)](./Services/dns.md)
- [HTTP/HTTPS (80/443)](./Services/web.md)
- [SMB (139/445)](./Services/smb.md)
- [SNMP (161 UDP)](./Services/snmp.md)
- [LDAP (389/636)](./Services/ldap.md)
- [RDP (3389)](./Services/rdp.md)
- [VNC (5900)](./Services/vnc.md)
- [WinRM (5985/5986)](./Services/winrm.md)

---

## 📝 Notes-Template (für jede Maschine)

```
TARGET: 10.10.10.10
OS:
OPEN PORTS:
  - 22/tcp  ssh    OpenSSH 8.2
  - 80/tcp  http   Apache 2.4.41
  - 445/tcp smb    Samba 4.x
USERS FOUND:
SHARES:
CREDS:
NOTES:
```
