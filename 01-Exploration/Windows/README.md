# 🪟 Windows Enumeration

> Du hast erkannt: das Ziel ist Windows. Jetzt: was läuft genau drauf?

---

## Typische Windows-Ports

| Port | Service | Notizen |
|---|---|---|
| 53 | DNS | oft Domain Controller |
| 88 | Kerberos | → **Domain Controller!** |
| 135 | RPC | Endpoint Mapper |
| 139, 445 | SMB / NetBIOS | Shares, Null-Session, EternalBlue |
| 389, 636 | LDAP / LDAPS | AD-Daten |
| 464 | Kerberos pw change | DC |
| 593 | RPC over HTTP | |
| 3268, 3269 | Global Catalog | DC |
| 3389 | RDP | Remote Desktop |
| 5900 | VNC | grafischer Zugriff (schwache PWs!) |
| 5985, 5986 | WinRM | für `evil-winrm` |

→ Wenn du **88 + 389 + 445** siehst: das ist mit hoher Wahrscheinlichkeit ein **Domain Controller**.

---

## 🎯 First Moves (in Reihenfolge)

### 1. SMB-Check (immer zuerst!)
```bash
# Null-Session?
smbclient -L //$RHOST/ -N
enum4linux -a $RHOST
enum4linux-ng -A $RHOST
nxc smb $RHOST -u '' -p ''        # nxc = netexec (Nachfolger von crackmapexec)
```
→ Details: [SMB Enum](../Services/smb.md)

### 2. AD / Domain Controller Check
```bash
# Wenn Port 88/389 offen
nxc ldap $RHOST
ldapsearch -x -H ldap://$RHOST -s base namingcontexts

# RID Brute (User-Liste!)
nxc smb $RHOST -u 'guest' -p '' --rid-brute
```
→ Details: [LDAP](../Services/ldap.md)

### 3. WinRM offen?
```bash
nxc winrm $RHOST -u 'user' -p 'pass'
# Wenn ja → später: evil-winrm -i $RHOST -u user -p pass
```
→ Details: [WinRM](../Services/winrm.md)

### 4. RDP offen?
```bash
nmap -p 3389 --script rdp-enum-encryption,rdp-vuln-ms12-020 $RHOST
```
→ Details: [RDP](../Services/rdp.md)

---

## 🚨 Quick Wins (immer testen!)

```bash
# EternalBlue (MS17-010)
nmap -p445 --script smb-vuln-ms17-010 $RHOST

# SMBGhost (CVE-2020-0796)
nmap -p445 --script smb-protocols $RHOST

# Zerologon (auf DC!)
nxc smb $RHOST -M zerologon

# PrintNightmare
nxc smb $RHOST -M printnightmare
```
→ EternalBlue komplett (Metasploit + manuell/AutoBlue): [eternalblue-ms17-010.md](../../02-Exploitation/Windows/eternalblue-ms17-010.md)

---

## 🔎 User-Enumeration (wenn AD)

```bash
# Kerberos User-Brute (kein Lockout!)
kerbrute userenum --dc $RHOST -d domain.local users.txt

# AS-REP Roasting (User ohne Pre-Auth)
GetNPUsers.py domain.local/ -dc-ip $RHOST -usersfile users.txt -no-pass

# Kerberoasting (mit Creds)
GetUserSPNs.py domain.local/user:pass -dc-ip $RHOST -request
```

---

## ➡️ Wenn du Creds hast

Geh zu [Exploitation → Windows](../../02-Exploitation/Windows/README.md)

## ➡️ Wenn du eine Shell hast

Geh zu [PrivEsc → Windows](../../03-PrivEsc/Windows/README.md)
