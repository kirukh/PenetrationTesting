# Changelog

## v2.0 — Neuordnung & Vollabdeckung des Prüfungs-Labs (CY-B-26)

Dieses Release integriert alle im PenTest-Bericht dokumentierten Methoden in die
phasenbasierte Struktur und verlinkt sie durchgängig untereinander.

### Neue Dateien
- `01-Exploration/Services/vnc.md` — VNC (5900), schwache PWs (Prüfungsfrage 4).
- `02-Exploitation/Network-Devices/pfsense.md` — Firewall-Übernahme: Admin-Cred-Hunt,
  WebGUI-Login, Diagnostics→Command Prompt (=root), NAT-Port-Forward, Segmentierung
  aushebeln (Prüfungsfragen 2 & 3).
- `02-Exploitation/Windows/eternalblue-ms17-010.md` — EternalBlue komplett: Metasploit
  **und** manueller AutoBlue-Weg (`shell_prep.sh` → `sc_x64_msf.bin` → `listener_prep.sh`
  → `eternalblue_exploit7.py`) (Prüfungsfrage 6).
- `02-Exploitation/Windows/command-injection-windows.md` — Windows OS Command Injection
  (`&`-Chaining) + Metasploit `web_delivery` (in-memory, AMSI/AV-Bypass) (VM17).
- `02-Exploitation/Linux/xz-backdoor-und-exposed-backdoors.md` — exponierter Port-1337-
  Backdoor + CVE-2024-3094 (xzbot) → SUID-bash → root (VM16).
- `99-Resources/av-evasion.md` — Defender umgehen: LOLBin-binPath, mingw-Cross-Compile,
  In-Memory-PowerShell (Prüfungsfrage 5, `iphlpsvc`/`jenny`).
- `99-Resources/reverse-shells.md` — Reverse-Shell-Payloads (bisher nur verlinkt, jetzt vorhanden).
- `04-Post-Exploitation/Loot-Sources/pcap-und-forensik-artefakte.md` — PCAP-Creds
  (Wireshark/tshark, VM2) + SAM/SYSTEM aus Forensik-ZIP → Pass-the-Hash (VM17),
  inkl. `impacket-secretsdump` Case-Sensitivity-Falle.
- `04-Post-Exploitation/Pivoting/firewall-pivot-und-route.md` — Device-Pivot vs.
  Host-Pivot, Vergleichstabelle (Prüfungsfrage 2).

### Erweiterte/aktualisierte Dateien
- `02-Exploitation/Passwords/README.md` — OSINT-gezielter rockyou-Filter
  (`grep -i -E "love|you"`) für VM6 (`jenny:iloveyou`).
- `03-PrivEsc/Linux/README.md` — VM16-SUID-bash-Notiz; Lab-Beispiele präzisiert.
- `03-PrivEsc/Windows/README.md` — `iphlpsvc`-binPath-Hijack + HiveNightmare-Beispiel,
  Verweise auf AV-Evasion, pfSense, Forensik-Loot.
- `04-Post-Exploitation/Pivoting/README.md` — vollständiger Chisel-SOCKS5-Workflow über
  dual-homed Windows-Pivot (SMB-Transfer der `chisel.exe`, proxychains-Config).
- `04-Post-Exploitation/Windows/README.md` — `secretsdump` Case-Sensitivity-Gotcha + Forensik-Link.
- `99-Resources/shell-upgrade.md` — `HOME=/root`-Falle bei vom Web-Prozess geerbter Umgebung.
- Diverse Index-READMEs (Services, Exploration, Exploitation, Web, Resources, Haupt-README)
  um Links auf die neuen Inhalte ergänzt; Entscheidungsbaum erweitert.

### Struktur
- Bewährte phasenbasierte Gliederung beibehalten (01→05 + 99).
- Zwei neue Unterordner: `02-Exploitation/Network-Devices/` und
  `04-Post-Exploitation/Loot-Sources/`.
