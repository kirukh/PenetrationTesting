# 🌐 HTTP / HTTPS — Enumeration

> **Prüfungsfokus:** Tech-Stack, größere Wordlist, Source/JS, Login/Reset/Cookies, sensible Dateien und passende PoCs. Exploitation steht zentral unter [Web Exploitation](../../02-Exploitation/Web/README.md).

## 1. Basisrequest und Tech-Stack

```bash
curl -i http://$RHOST/
curl -k -i https://$RHOST/
whatweb http://$RHOST/
```

Achte auf:

```text
Server / Framework / CMS / Version
Set-Cookie, Location, WWW-Authenticate
Form-Feldnamen und Zielpfad
Kommentare, Benutzernamen, E-Mails, App-Namen
```

## 2. Verzeichnisse und Dateien

```bash
gobuster dir -u http://$RHOST/ \
  -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt \
  -x php,txt,bak,old,zip,html,config -t 40 -o gobuster.txt
```

- Bei HTTPS `-k` ergänzen.
- Bei Unterpfad erneut scannen: `-u http://$RHOST/<PFAD>/`.
- Interessant: `/admin`, `/login`, `/dev`, `/test`, `/backup`, `/uploads`, `/old`, `/api`, `/dashboard`, `/phpmyadmin`, `/robots.txt`.

> **Hinweis Prof:** Nicht nur `common.txt`; eine größere Wordlist verwenden.

## 3. Manuelle Checks

```bash
curl -s http://$RHOST/robots.txt
curl -s http://$RHOST/sitemap.xml
curl -s http://$RHOST/ | tee index.html

grep -niE 'pass|pwd|user|mail|key|token|hash|backup|todo|admin|debug' index.html
grep -oP '<!--.*?-->' index.html
```

JavaScript/Endpoints:

```bash
grep -oE 'src="[^"]+\.js' index.html | cut -d'"' -f2
curl -s http://$RHOST/<SCRIPT.js> | grep -niE 'api|admin|login|reset|token|secret|upload|debug'
```

Virtual Host nur bei Domain-Hinweisen testen:

```bash
ffuf -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt \
  -u http://$RHOST/ -H 'Host: FUZZ.<DOMAIN>' -fs <BASELINE_SIZE>
```

`<DOMAIN>` aus Webseite/Zertifikat übernehmen; `<BASELINE_SIZE>` ist die Größe der normalen Fehlseite.

## 4. Auth-Fläche erkennen

```bash
# Form-Felder und Ziel-URL ansehen
curl -s http://$RHOST/<LOGIN> | grep -niE '<form|input|name=|action=|method='

# Normalen Fehlversuch als Baseline speichern
curl -i -c cookies.txt -X POST http://$RHOST/<LOGIN> \
  -H 'Content-Type: application/x-www-form-urlencoded' \
  -d 'username=test&password=test' | tee login_fail.txt
```

Notiere Statuscode, Länge, Fehltext, Redirect und Cookies. Damit werden `ffuf`-Filter korrekt gesetzt.

→ Username-Enumeration, Reset-Manipulation und Cookie-Tampering: [Web Exploitation](../../02-Exploitation/Web/README.md)

## 5. Produkt/Version recherchieren

```bash
searchsploit '<PRODUKT> <VERSION>'
searchsploit -x <EXPLOIT-ID>
```

Bei Python-PoC:

```bash
sed -n '1,220p' exploit.py
grep -nEi 'requests|url|headers|cookies|data|params|json|payload|cmd' exploit.py
```

→ [Python-PoC als curl nachbauen](../../99-Resources/python-exploit-zu-curl.md)

## Nicht priorisieren

XSS, SQL-Injection und Burp Suite wurden ausdrücklich nicht als Prüfungsfokus genannt. Erst die oben genannten Wege ausschöpfen.
