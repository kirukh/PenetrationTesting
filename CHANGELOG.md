# Changelog

## v3.0 — Fokus auf die Prüfung (CY-B-26), Aufräumen

### Entfernt (laut Vorgabe nicht prüfungsrelevant)
- **AV-/Defender-Evasion** komplett (`99-Resources/av-evasion.md` + alle Verweise).
- **Pivoting** komplett (`04-Post-Exploitation/Pivoting/` inkl. Chisel/Ligolo/SOCKS,
  `firewall-pivot-und-route.md`) — beide Prüfungsmaschinen sind direkt von Kali erreichbar.
- **Firewall/pfSense-Thema** (`02-Exploitation/Network-Devices/pfsense.md`) — keine Rolle.
- Zugehörige Querverweise in README, Index-READMEs, msfvenom, file-transfer, ssh, command-injection,
  Windows-/Linux-Post-Exploit und PrivEsc bereinigt bzw. neutral umformuliert.

### Neu — Prüfungs-Schwerpunkt (Prof-Hinweise erklärt)
- **`00-Pruefung/README.md`** — zentraler Spickzettel: ordnet **alle** Prof-Hinweise nach
  Prüfungsablauf und erklärt die Anwendung (Recon `-p- -T5`, sensible Infos sammeln,
  Passwort-Listen aus Hinweisen → Hydra, NTLMv2 `-m 5600` muss geknackt werden / kein PtH,
  Authentication Bypass, SMB 445/137, msfvenom staged/stageless, EternalBlue-Python,
  Shell-Upgrade, PrivEsc-Ablauf inkl. Scheduled Task / schwache Services / SUID-Read-root,
  Meterpreter `hashdump`, Referenz-Seiten).
- **`00-Pruefung/troubleshooting.md`** — Python-Version umstellen, C/mingw kompilieren,
  Error-Mindset, **curl als Exploit-Trigger** statt Python-PoC.
- **`05-Reporting/democorp-report.md`** — exakte Berichtsstruktur **IPT-001**: Ansprechpartner,
  Scope, Executive Summary, Risk-Faktoren (Likelihood+Impact-Matrix), Walkthrough, Findings
  **pro VM** mit Risk-Begründung + PoC + Referenzen + betroffene Systeme, Remediation, Action-Plan.

### Erweitert
- `99-Resources/msfvenom.md` — ausführlicher staged-vs-stageless-Hinweis (Listener muss passen).
- `03-PrivEsc/README.md` — Prof-Ablauf (wer/wo bin ich → sudo -l/GTFOBins → linpeas+PrivescCheck → SUID).
- `03-PrivEsc/Windows/README.md` — Scheduled-Task-/Service-PrivEsc als prüfungsrelevant markiert,
  Windows-LPE-Cookbook verlinkt; Stored-Creds neutral (ohne Firewall).
- Haupt-`README.md` — Prüfungs-Spickzettel ganz oben, Entscheidungsbaum auf den Prüfungs-Scope getrimmt.

### Beibehalten
- Phasenstruktur 00 → 05 + 99. PCAP/Forensik-Loot bleibt (Netzwerk-Traffic kommt laut Prof dran).
