# 🛡️ Pentest Cheat-Sheet — Prüfungsedition

> Für die praktische Prüfung: **Recon → Service-Enumeration → Initial Access → PrivEsc → Flags/Beweise → Bericht**.
> Ziel ist nicht maximale Vollständigkeit, sondern ein schneller, zuverlässiger roter Faden.

## Platzhalter einmal setzen

```bash
export RHOST=<TARGET-IP>
export LHOST=$(ip -o -4 addr show tun0 2>/dev/null | awk '{print $4}' | cut -d/ -f1)
[ -z "$LHOST" ] && export LHOST=$(ip -o -4 addr show eth0 | awk '{print $4}' | cut -d/ -f1)
export LPORT=4444

mkdir -p ~/exam/$RHOST/{nmap,enum,exploits,loot,screenshots,notes}
cd ~/exam/$RHOST
```

**In Befehlen anpassen:** `$RHOST`, `$LHOST`, `$LPORT`, `<PORTS>`, `<USER>`, `<PASS>`, `<PATH>`, `<PARAM>`, `<SERVICE>`, `<TASK>`.

---

## Prüfungsflow

```text
1. Beide Hosts scannen und Ports notieren.
2. Web und SMB zuerst prüfen; Hinweise/Dateien vor Exploits priorisieren.
3. Produkt + Version feststellen, dann PoC lesen und klein testen.
4. Initial Access: Auth Bypass, Credentials, Upload/Webshell, Command Injection oder Service-PoC.
5. Weak Shell stabilisieren und user.txt sichern.
6. PrivEsc manuell beginnen; danach LinPEAS/PrivescCheck.
7. root.txt direkt lesen, sobald erhöhte Rechte/euid vorhanden sind.
8. Für jeden entscheidenden Schritt Befehl, Ausgabe und Screenshot sichern.
```

### Minimaler Scan

```bash
nmap -p- -T5 $RHOST -oN nmap/allports.txt
nmap -sCV -p <PORTS> $RHOST -oN nmap/detail.txt
```

> **Hinweis Prof:** Nicht im ersten Exploit festbeißen. Erst Ports, Versionen, Webpfade, Shares, Dateien und Hinweise vollständig erfassen.

---

## Entscheidung nach Port/Fund

| Fund | Direkt weiter |
|---|---|
| `80/443/8080` | [Web-Enumeration](./01-Exploration/Services/web.md) → [Web-Exploitation](./02-Exploitation/Web/README.md) |
| Login/Reset/Cookies | **Auth Bypass priorisieren** → [Web-Exploitation](./02-Exploitation/Web/README.md) |
| `445/139` oder SMB-Hinweis | [SMB](./01-Exploration/Services/smb.md): Guest/Null, Shares, Backups, Benutzer, NTLMv2 |
| `1433` | [MSSQL / T-SQL](./01-Exploration/Services/mssql.md) |
| Hash / `.kdbx` / PDF / ZIP | [Passwörter & Hashes](./02-Exploitation/Passwords/README.md) |
| Python-PoC | [PoC lesen und als curl nachbauen](./99-Resources/python-exploit-zu-curl.md) |
| Weak/Web-Shell | [Shell Upgrade](./99-Resources/shell-upgrade.md) |
| Linux Low-Priv | [Linux PrivEsc](./03-PrivEsc/Linux/README.md) |
| Windows Low-Priv | [Windows PrivEsc](./03-PrivEsc/Windows/README.md) |
| PCAP/Traffic | [PCAP & Forensik-Artefakte](./04-Post-Exploitation/Loot-Sources/pcap-und-forensik-artefakte.md) |
| Metasploit-Modul/Exploit | [Metasploit-Anleitung](./99-Resources/metasploit.md): Suche, Optionen, Target, Payload, Sessions |
| msfvenom/Payload | [Payload, OS/Architektur, staged/stageless](./99-Resources/msfvenom.md) |
| Bericht/IPT-001 | [Reporting](./05-Reporting/README.md) |

---

## Wenn 20–30 Minuten nichts vorangeht

```text
□ Gibt es einen zweiten offenen Dienst?
□ robots.txt, Source, JS, Cookies, Redirects und Fehlermeldungen geprüft?
□ Größere Gobuster-Wordlist benutzt?
□ SMB Guest/Null und alle Shares rekursiv geladen?
□ Hinweise zu Usern, Firma, Anwendung oder Passwortkandidaten gesammelt?
□ Default-Credentials und Passwort-Reuse geprüft?
□ Produkt/Version korrekt und PoC für richtiges OS/Architektur gewählt?
□ PoC-Request zunächst mit id/whoami statt Reverse Shell getestet?
□ Exploit-Error auf Python-Version, Dependency, Pfad, Payload und Listener geprüft?
```

---

## Bewusster Prüfungsfokus

**Priorität:** Authentication Bypass, Web-/PHP-Shell, `curl`-Trigger, SMB-Dateien/Backups, kleine Passwortlisten, NetNTLMv2, MD5/Standard-Hashes, Shell-Upgrade, Linux SUID/`sudo -l`, Windows Services/Scheduled Tasks/PrivescCheck, PowerShell-History und IPT-001.

**Nicht im Fokus:** XSS, SQL-Injection, Burp Suite, vollständige UDP-Enumeration, Active Directory und komplexes Pivoting. Spezialkapitel bleiben im Repository, sind aber nicht Teil des roten Prüfungswegs.

---

## Hauptordner

- [01-Exploration](./01-Exploration/README.md) — Ports, Services und Hinweise
- [02-Exploitation](./02-Exploitation/README.md) — Initial Access
- [03-PrivEsc](./03-PrivEsc/README.md) — Linux/Windows Privilege Escalation
- [04-Post-Exploitation](./04-Post-Exploitation/README.md) — Loot, Hashes, Flags und Beweise
- [05-Reporting](./05-Reporting/README.md) — DemoCorp/IPT-001
- [99-Resources](./99-Resources/README.md) — Payloads, Encoding, Transfers und Troubleshooting
