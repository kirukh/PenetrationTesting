# 🪟 Windows Exploration

> Fokus: Windows 11, SMB/NetBIOS, WinRM/RDP, lokale User, Shares, NTLMv2, Backup-Dateien.

## Typische Ports

| Port | Dienst | Was prüfen? |
|---|---|---|
| 135 | MSRPC | Windows-Hinweis |
| 137/139 | NetBIOS | SMB/Name Service |
| 445 | SMB | Shares, Guest, User, Signing, Backups |
| 3389 | RDP | Login mit Creds |
| 5985/5986 | WinRM | `evil-winrm` mit Creds/Hash |

> **Hinweis Prof:** Port 445 und 137 sind wichtig. SMB wird wahrscheinlich auf Windows relevant.

## Schnellcheck

```bash
nxc smb $RHOST
nxc smb $RHOST -u guest -p '' --shares
smbclient -L //$RHOST/ -N
```

## Nach Credentials

```bash
nxc smb $RHOST -u user -p 'pass'
evil-winrm -i $RHOST -u user -p 'pass'
xfreerdp /u:user /p:'pass' /v:$RHOST /cert:ignore
```

## NTLMv2

```bash
hashcat -m 5600 ntlmv2.txt /usr/share/wordlists/rockyou.txt
```

> NTLMv2 zuerst cracken, dann Passwort benutzen. Kein Pass-the-Hash.
