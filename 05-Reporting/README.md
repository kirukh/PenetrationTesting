# 📝 Phase 5 — Reporting

> **Hinweis Prof:** DemoCorp / IPT-001 kommt dran. Der Bericht muss die technische Kette sauber erklären: Was war offen, wie wurde es ausgenutzt, welche Rechte wurden erreicht, welche Flags/Beweise gibt es, wie behebt man es?

---

## 1. Während der Prüfung direkt mitschreiben

```bash
mkdir -p ~/exam/$RHOST/{nmap,enum,exploits,loot,screenshots,notes}
cd ~/exam/$RHOST
script -a notes/terminal.log   # alle Terminal-Ein-/Ausgaben mitschreiben
nano notes/notes.md
```

Template:

```markdown
# Target: <IP>
OS: Linux / Windows
Hostname:
Datum:

## Recon
- nmap all ports:
- nmap -sCV:
- Web/SMB/Services:

## Attack Path
1. Recon-Fund:
2. Initial Access:
3. User-Flag:
4. PrivEsc:
5. Root/Admin-Flag:

## Evidence
- Screenshot whoami/id:
- Screenshot Flag:
- wichtige Befehle:

## Credentials / Hashes
| Quelle | User | Passwort/Hash | Nutzung |
|---|---|---|---|

## Finding-Ideen
| Titel | Severity | Betroffenes System | Impact |
|---|---|---|---|
```

---

## 2. Proof-Screenshot

Jeder Proof sollte enthalten:

```text
hostname
whoami / id
IP-Adresse
Flag-Inhalt
```

Linux:

```bash
hostname; whoami; id; ip a | grep inet; cat /root/root.txt
```

Windows:

```cmd
hostname && whoami && ipconfig && type C:\Users\Administrator\Desktop\root.txt
```

---

## 3. Finding-Struktur nach IPT-001 / DemoCorp

```markdown
### Finding: <kurzer Titel>

**Severity:** Critical / High / Medium / Low  
**Betroffenes System:** <IP / Hostname>  
**Komponente:** Web / SMB / Windows Service / SUID / Credentials

#### Beschreibung
Kurz erklären, was die Schwachstelle ist.

#### Auswirkung
Was kann ein Angreifer dadurch erreichen? Initial Access, Credential Disclosure, PrivEsc, root/Admin.

#### Nachweis / PoC
Befehle, Screenshots, Outputs. Keine riesigen Logs, nur relevante Evidence.

#### Angriffspfad
1. ...
2. ...
3. ...

#### Behebung
Konkrete Maßnahmen.
```

---

## 4. Kurzbeispiele für Prüfungs-Findings

| Thema | Titel |
|---|---|
| Web Auth Bypass | Fehlende serverseitige Zugriffskontrolle erlaubt Admin-Zugriff |
| Web Command Injection | Ungefilterte Eingabe führt zu Remote Code Execution |
| SMB Share | Guest-lesbarer SMB-Share enthält sensible Backups |
| NTLMv2 | Abfangbarer NTLMv2-Hash ermöglicht nach Crack Login |
| Windows Service | Schwache Service-Berechtigungen erlauben lokale Rechteausweitung |
| Scheduled Task | Schreibbares Task-Skript wird mit höheren Rechten ausgeführt |
| Linux SUID | SUID-Binary erlaubt Lesen von root.txt / root-Shell |
| Hardcoded Credentials | Klartext-Passwort in Datei/Binary ermöglicht Benutzerwechsel |

---

## 5. Zeitmanagement

- 0–15 min: beide Hosts scannen, Ports und erste Priorität festlegen
- 15–80 min: Maschine A — Initial Access, User-Flag, PrivEsc
- 80–145 min: Maschine B — Initial Access, User-Flag, PrivEsc
- 145–170 min: offene PrivEsc-/Flag-Schritte und alternative Wege
- 170–180 min: Screenshots, Befehle, Credentials und Flags vollständig sichern

> Wenn du 25–30 Minuten ohne neue Information festhängst: Stand notieren, anderen Dienst/Angriffsweg oder die zweite Maschine bearbeiten.

---

## Vorlagen

- [DemoCorp/IPT-001 Beispiel](./democorp-report.md)
- [Report Template](./report-template.md)
