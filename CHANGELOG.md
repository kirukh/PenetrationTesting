# Changelog

## 2026-07-14 — Metasploit-Prüfungsanleitung

- Neue kompakte Metasploit-Anleitung mit vollständigem Prüfungsflow: `search` → `info` → `use` → Optionen → Target/Payload → `check` → `run` → Sessions.
- Erklärungen zu `RHOSTS`, `RPORT`, `TARGETURI`, `LHOST`, `LPORT`, `SSL`, `VHOST`, globalen Variablen und Betriebssystem-/Architekturwahl ergänzt.
- Session- und Meterpreter-Grundbefehle, `hashdump`, `getsystem`, Dateiübertragung und normale Shell dokumentiert.
- `multi/handler` für exakt passende msfvenom-Payloads sowie `web_delivery` für bereits bestätigte Web-Command-Injection ergänzt.
- Fehlerdiagnose für „Exploit completed, but no session was created“, falsche Standard-Payloads und Webpfade eingebaut.

## 2026-07-14 — Prüfungsedition 10/10

- Haupt-README als kompakte Prüfungsnavigation mit klaren Platzhaltern, Entscheidungsbaum und 20–30-Minuten-Wechselregel überarbeitet.
- Web-Kapitel entdoppelt: Enumeration getrennt von Authentication Bypass, Reset-Manipulation, Cookie-Tampering, Command Injection und Upload-Bypässen.
- SMB um Port-Einordnung, rekursiven Loot-Workflow, Responder-Erfassung, NTLMSSP-PCAP-Auswertung und klare Trennung NetNTLMv2/lokaler NTLM erweitert.
- Passwortkapitel auf kleine, hinweisbasierte Kandidatenlisten und die prüfungsrelevanten Hash-/Dateiformate reduziert.
- Windows-PrivEsc für Windows 11 konkretisiert: PowerShell-History/AppData, PrivescCheck, Service-ACL, writable Service Binary, Unquoted Service Path und Scheduled Tasks.
- PCAP-Kapitel auf Klartext-Credentials, NetNTLMv2 und SAM/SYSTEM-Forensik-Artefakte fokussiert.
- msfvenom auf relevante Windows/Linux/PHP-Payloads, Shellcode und staged/stageless reduziert.
- Nicht prüfungsrelevante Spezialthemen aus der Hauptnavigation entfernt, bleiben aber im Repository erhalten.

## 2026-07-14 — Übungsnotizen und Prof-Hinweise vertieft

- Auth-Bypass-Kapitel um `ffuf`-Username-Enumeration, Multi-Wordlist-Login-Fuzzing, Passwort-Reset-Parameter-Manipulation und Cookie-Tampering erweitert.
- Bei allen ffuf-Beispielen erklärt, welche URL, Feldnamen, Match-/Filterwerte und Platzhalter in der Prüfung angepasst werden müssen.
- File-Upload-Kapitel um eine kompakte Bypass-Matrix ergänzt: alternative PHP-Endungen, doppelte Endungen, Case, MIME-Type, Magic Bytes, Windows/IIS-Normalisierung und Apache-`.htaccess`.
- Linux-SUID-Enumeration um eine Liste auffälliger Binaries, Schnellfilter und Analyse von Custom-Binaries erweitert.
- Neues MSSQL-/T-SQL-Kapitel für Port 1433: `impacket-mssqlclient`, SQL- vs. Windows-Auth, Datenbanken/Tabellen, Rollen, Impersonation, Linked Servers und `xp_cmdshell`.
- Nmap-Flow bleibt minimal; zusätzlich gezielter Windows-Port-Recheck als Fallback dokumentiert, falls der schnelle `-T5`-Scan unplausibel wirkt.
- SMB-/NTLMv2-Erklärung präzisiert: NetNTLMv2 aus Traffic muss geknackt werden, lokaler NTLM aus SAM kann je nach Dienst per Pass-the-Hash genutzt werden.

## 2026-07-08 — Prüfungsflow überarbeitet

- Zentralen separaten Prüfungs-Spickzettel entfernt.
- Prof-Hinweise direkt in die passenden Kapitel eingearbeitet, jeweils mit **Hinweis Prof** markiert.
- Nmap auf prüfungsrelevanten Minimalflow reduziert: `nmap -p- -T5` + danach `nmap -sCV -p <ports>`.
- Nicht benötigten Windows-Legacy-Exploit entfernt und Verweise gelöscht.
- Neues Kapitel: `99-Resources/python-exploit-zu-curl.md` für Python-PoC → curl-Trigger.
- SMB-Kapitel auf Prüfung fokussiert: 445/137, Shares, Backups, NTLMv2, SAM/SYSTEM.
- Web-Kapitel auf Auth Bypass, Gobuster, whatweb, curl, URL-Encoding/Base64 und Webshells fokussiert.
- Windows PrivEsc auf Services, Scheduled Tasks, PrivescCheck, PowerShell-History und Rechte-zum-Flag-Lesen erweitert.
- Linux PrivEsc auf `ls -la`, versteckte Dateien, `/home`, `/opt`, `/var`, SUID/GTFOBins und `sudo -l` fokussiert.
- Reporting auf DemoCorp/IPT-001-Gliederung angepasst.
