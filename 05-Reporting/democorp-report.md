# 🏢 DemoCorp-Bericht — Aufbau (Hinweis **IPT-001**)

> **Prof-Hinweis:** Der **DemoCorp-Bericht** mit dem Hinweis **IPT-001** **kommt in der
> Prüfung dran**. Diese Datei hält die exakt erwartete Struktur fest — so gliederst du
> deinen eigenen Bericht. Findings werden **pro VM** gruppiert, jedes Finding bekommt
> Risk + Begründung + PoC + Referenzen + betroffene Systeme + Remediation.

---

## 0. Deckblatt / Kopf

- Titel, Datum, Version, Klassifizierung (z.B. *Confidential*)
- **Ansprechpartner**: kurz wer ist der Kontakt auf Kundenseite, **worum geht's** (1–2 Sätze).
- Tester-Name, Engagement-ID (z.B. **IPT-001**).

## 1. Scope

- Welche Systeme/Netze/IP-Bereiche waren in-scope (und was **nicht**).
- Zeitraum, Art des Tests (z.B. *Internal Penetration Test* — daher **IPT**-001).
- Annahmen/Einschränkungen (z.B. „nur Bordmittel", „kein DoS").

## 2. Executive Summary

Management-tauglich, **ohne Fachjargon**: Was wurde getestet, wie war die Gesamtlage,
wie viele Findings in welcher Schwere, die wichtigste Kernaussage zuerst.

| | Anzahl |
|---|---|
| 🔴 Critical | x |
| 🟠 High | x |
| 🟡 Medium | x |
| 🔵 Low | x |

## 3. Risk-Faktoren — Likelihood & Impact

Erkläre **wie** ihr Risiko bewertet: Risk = Funktion aus **Likelihood**
(Eintrittswahrscheinlichkeit) und **Impact** (Schadenshöhe). Kleine Matrix einfügen:

| Likelihood ↓ / Impact → | Low | Medium | High |
|---|---|---|---|
| **High** | Medium | High | Critical |
| **Medium** | Low | Medium | High |
| **Low** | Low | Low | Medium |

→ Jedes Finding bekommt später **beide** Werte + den resultierenden Risk.

## 4. Walkthrough (Angriffskette von Anfang bis Ende)

Erzählerisch der **komplette Weg**: Recon → Initial Access → PrivEsc → Ziel — so, dass ein
Leser den Pfad nachvollziehen kann. Ideal mit nummerierten Schritten und Verweisen auf die
Findings (z.B. „durch Finding 2.1 …").

## 5. Findings — **pro VM gruppiert**

> Struktur: **VM1 — alle Findings**, dann **VM2 — alle Findings**, …
> Jedes einzelne Finding nach folgendem Schema:

```
### Finding VM1-1: <Titel der Schwachstelle>

**Risk:** 🟠 High        Likelihood: Medium | Impact: High
**Betroffene Systeme:** 192.168.x.x (VM1)  [ggf. mehrere, wenn dieselbe Schwachstelle
                        auf weiteren Systemen existiert -> hier auflisten]

**Beschreibung / Warum dieser Risk?**
Kurze Erklärung der Schwachstelle UND die Begründung, *warum* sie genau diesen
Risk-Faktor hat (z.B. „unauthentifiziert aus dem Netz ausnutzbar -> Likelihood High;
führt zu Codeausführung als root -> Impact High").

**Proof of Concept (PoC):**
Befehle/Requests + Screenshot-Verweise, die den Fund reproduzierbar belegen.
    nmap -p- -T5 192.168.x.x
    <exploit/curl/msfvenom-Schritt>
    -> Ergebnis (z.B. Shell als www-data / Inhalt root.txt)

**Referenzen:**
- CVE-XXXX-YYYY
- HackTricks / Exploit-DB / Hersteller-Advisory

**Remediation:**
Konkret, was dagegen zu tun ist (Patch, Konfig härten, Passwort-Policy, Dienst entfernen…).
```

> **Mehrfach betroffen?** Wenn dieselbe Schwachstelle auf mehreren Systemen auftritt, im Feld
> *Betroffene Systeme* alle nennen — nicht für jedes System neu schreiben.

## 6. Remediation-Übersicht

Sammeltabelle über alle Findings (Priorität ↔ Maßnahme), als schnelle To-do-Liste.

| Priorität | Finding | Maßnahme |
|---|---|---|
| 1 | VM1-1 | Patchen auf Version … |
| 2 | VM2-1 | Default-Passwort ändern, … |

## 7. Action-Plan

> **Prof-Hinweis:** Am Ende ein Action-Plan: **was jetzt sofort** zu tun ist **und worauf in
> Zukunft** weiter zu achten ist.

- **Sofort (Quick Wins):** kritische/High-Findings schließen (Patches, Passwörter, exponierte Dienste).
- **Mittelfristig:** Härtung, Segmentierung, Monitoring, Logging.
- **Langfristig / Prozess:** regelmäßige Updates, Awareness, wiederkehrende Pentests,
  Secure-Configuration-Baselines.

---

## ✅ Checkliste (DemoCorp-Stil)

- [ ] Ansprechpartner + Worum-geht's
- [ ] Scope
- [ ] Executive Summary
- [ ] Risk-Faktoren (Likelihood + Impact erklärt)
- [ ] Walkthrough (Anfang → Ende)
- [ ] Findings **pro VM**, je mit Risk-Begründung + PoC + Referenzen + betroffene Systeme
- [ ] Remediation je Finding (+ Übersicht)
- [ ] Action-Plan (jetzt + Zukunft)

→ Technisches Notizen-/Report-Template: [report-template.md](./report-template.md)
