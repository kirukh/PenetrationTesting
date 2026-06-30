# 📝 Phase 5 — Reporting

> In der Prüfung gewinnt nicht nur, wer reinkommt — sondern wer es **dokumentieren** kann.

## 🗂️ Ordner-Setup pro Maschine

```bash
mkdir -p ~/exam/$RHOST/{nmap,enum,exploits,loot,screenshots,notes}
cd ~/exam/$RHOST
```

## 📋 Live-Notes Template (`notes.md`)

```markdown
# Target: 10.10.10.10
**Date:** 2026-XX-XX
**OS:** Windows Server 2019 / Linux Ubuntu 20.04

---

## Recon
- nmap: ports 22, 80, 445 offen
- OS: ...
- Tech-Stack: ...

## Open Ports
| Port | Service | Version | Notes |
|---|---|---|---|
| 22 | SSH | OpenSSH 8.2 | Bruteforce versucht — fail |
| 80 | HTTP | Apache 2.4.41 | WordPress 5.7 |
| 445 | SMB | Samba 4.5 | Anon-Login: ja |

## Findings
1. **Anonymous SMB Login** → User-Liste extrahiert
2. **WordPress 5.7** → 1 plugin mit known CVE
3. ...

## Exploitation Path
1. SMB Anon → users.txt extrahiert
2. AS-REP Roasting auf user1 → Hash gecrackt: `Winter2024!`
3. WinRM-Login → Low-Priv-Shell
4. SeImpersonatePrivilege → GodPotato → SYSTEM

## Credentials Found
| User | Password / Hash | Source |
|---|---|---|
| user1 | Winter2024! | AS-REP Roast + Hashcat |
| admin | aad3b435b51404eeaad3b435b51404ee:... | secretsdump |

## Flags
- user.txt: `flag{...}` — `C:\Users\user1\Desktop\user.txt`
- root.txt: `flag{...}` — `C:\Users\Administrator\Desktop\root.txt`
```

## 📸 Screenshot-Konventionen

Jeder Screenshot sollte folgendes enthalten:
1. **Hostname** des Targets (`hostname`)
2. **Whoami / id**
3. **IP-Adresse** (`ip a` / `ipconfig`)
4. **Den Flag-Inhalt** (`type C:\...\root.txt` oder `cat /root/root.txt`)

```bash
# Linux Proof-Befehl
echo "===" ; hostname ; whoami ; id ; ip a | grep inet ; cat /root/root.txt

# Windows Proof
hostname && whoami && ipconfig && type C:\Users\Administrator\Desktop\root.txt
```

## 📄 Final Report Template

→ siehe [report-template.md](./report-template.md)

## ⏱️ Zeitmanagement-Tipps

| Phase | Zeit (Beispiel 4h-Prüfung) |
|---|---|
| Recon | 30-45 min pro Box |
| Erste Exploitation-Versuche | 30-60 min |
| PrivEsc | 30-60 min |
| Reporting / Screenshots | 30 min am Ende reservieren! |

**Faustregel:** Wenn du 30 min an einer Sache hängst → Screenshot machen, ins Notizfeld "stuck on X", **andere Box anfangen**, später zurückkommen.
