# Penetration Test Report — Template

## 1. Management Summary

Im Rahmen der praktischen Prüfung wurde das Zielsystem `<IP>` untersucht. Dabei wurden offene Dienste identifiziert, Schwachstellen ausgenutzt und die Flags `user.txt` sowie `root.txt`/Administrator-Flag nachgewiesen.

## 2. Scope

| System | IP | Rolle | Status |
|---|---|---|---|
| Target 1 | `<IP>` | Linux/Windows | user/root erreicht |

## 3. Methodik

```bash
nmap -p- -T5 <IP>
nmap -sCV -p <PORTS> <IP>
```

Weitere Schritte: Service-Enumeration, Suche nach sensiblen Informationen, Exploitation, Privilege Escalation, Evidence-Sicherung.

## 4. Angriffspfad

1. Recon: `<Ports/Services>`
2. Initial Access: `<wie Zugriff erhalten>`
3. User-Flag: `<Pfad>`
4. PrivEsc: `<Schwachstelle>`
5. Root/Admin-Flag: `<Pfad>`

## 5. Findings

### Finding 1 — `<Titel>`

**Severity:** High  
**Betroffenes System:** `<IP>`  
**Komponente:** `<Service>`

#### Beschreibung
`<Beschreibung>`

#### Nachweis / PoC

```bash
<Befehle>
```

#### Auswirkung
`<Impact>`

#### Behebung
`<Remediation>`

## 6. Evidence

- Screenshot `whoami/id`
- Screenshot `hostname/ipconfig/ip a`
- Screenshot `user.txt`
- Screenshot `root.txt`

## 7. Empfehlungen

- Secrets nicht in Dateien/Backups/Shares speichern
- Zugriffskontrollen serverseitig prüfen
- Service-/Task-/SUID-Rechte härten
- SMB Guest deaktivieren
- Starke Passwörter/MFA/Lockout
