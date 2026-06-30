# 🌐 HTTP / HTTPS (80/443/8080/...)

> Webserver = häufigster Initial-Access-Vector. Immer gründlich enumerieren.

## Step 1 — Was läuft da?

```bash
# Banner / Tech-Stack
curl -I http://$RHOST/
whatweb http://$RHOST/
whatweb -v http://$RHOST/

# nmap http-scripts
nmap -p80 --script="http-*" $RHOST
nmap -p80 --script=http-enum,http-headers,http-methods,http-title $RHOST

# Wappalyzer im Browser nicht vergessen!
```

## Step 2 — Directory & File Brute

```bash
# gobuster (schnell)
gobuster dir -u http://$RHOST/ -w /usr/share/wordlists/dirb/common.txt
gobuster dir -u http://$RHOST/ -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -x php,html,txt,bak -t 50

# feroxbuster (rekursiv, mein Favorit)
feroxbuster -u http://$RHOST/ -x php,html,txt,bak

# ffuf (sehr schnell, flexibel)
ffuf -u http://$RHOST/FUZZ -w /usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt -e .php,.html,.txt
ffuf -u http://$RHOST/FUZZ -w wordlist.txt -fc 404                  # filter status
ffuf -u http://$RHOST/FUZZ -w wordlist.txt -fs 1234                  # filter size

# dirsearch
dirsearch -u http://$RHOST/ -e php,html,txt,bak,old,zip
```

> **Lab-Tipp:** feroxbuster fand in mehreren VMs die entscheidenden Pfade:
> VM1 → `/dev`, `/dev/shell`, `/admin`; VM8 → `/S3cr3t`, `/S3cr3t/wp-content`;
> VM2 → `/weblog/wp-admin`, `/phpmyadmin`. Immer **rekursiv** scannen.

## Step 3 — Vhost / Subdomain Brute

```bash
# Vhost (gleiche IP, anderer Hostname → andere Seite)
ffuf -u http://$RHOST/ -H "Host: FUZZ.target.local" -w subdomains.txt -fs 1234

# DNS-Subdomains
gobuster dns -d target.com -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt
```

## Step 4 — Parameter-Brute

```bash
# Welche GET-Params nimmt die Seite an?
ffuf -u "http://$RHOST/page.php?FUZZ=test" -w /usr/share/seclists/Discovery/Web-Content/burp-parameter-names.txt -fs 1234

# Arjun (intelligenter)
arjun -u http://$RHOST/page.php
```

## Step 5 — Vuln-Scanner

```bash
# nikto (alt aber gut)
nikto -h http://$RHOST/ -o nikto.txt

# nuclei (modern, viele Templates)
nuclei -u http://$RHOST/
nuclei -u http://$RHOST/ -t cves/ -severity critical,high
```

## CMS-spezifisch

```bash
# WordPress
wpscan --url http://$RHOST/ --enumerate u,p,t
wpscan --url http://$RHOST/ --enumerate u --api-token YOUR_TOKEN

# Drupal
droopescan scan drupal -u http://$RHOST/

# Joomla
joomscan -u http://$RHOST/
```

→ WordPress kam im Lab 4× vor: kompletter Workflow in
[WordPress-Angriffe](../../02-Exploitation/Web/wordpress.md)

## Manuelle Checks (nicht vergessen!) — oft die eigentlichen Quick-Wins

- `robots.txt`, `sitemap.xml` → enthalten oft Pfade zu Wortlisten / Keys / versteckten Dirs
- `.git/` (→ `git-dumper http://$RHOST/.git/ ./loot`)
- `.env`, `.bak`, `.old`, `.swp`
- `/admin`, `/login`, `/api`, `/dev`, `/test`
- **HTML-Source ansehen:** Kommentare (!), JS-Files (oft Endpoints/Keys/Hashes drin!)
- Cookies & Auth-Header
- Standard-Creds (`admin:admin`, `admin:password`, `tomcat:tomcat`)

### 🔍 Sensible Daten im Quellcode / in statischen Dateien

Eine der häufigsten "leisen" Schwachstellen: Credentials oder Hashes liegen
offen im ausgelieferten HTML/JS oder in scheinbar harmlosen Dateien.

```bash
# HTML-Kommentare anzeigen (Hashes, TODOs, Test-Creds)
curl -s http://$RHOST/dev | grep -oP '<!--.*?-->'
curl -s http://$RHOST/ | grep -iE 'pass|user|hash|key|todo'

# Gefundene Hashes identifizieren und cracken
hashid '<hash>'
hashcat -m 100 hashes.txt rockyou.txt        # SHA1 z.B.
# → siehe Passwords/README.md
```

> **Lab-Beispiel VM1 (Bulldog):** Unter `/dev` lagen im HTML auskommentierte
> SHA1-Hashes mehrerer Mitarbeiter ("Need these for testing"). Zwei davon
> crackten zu `bulldog` / `bulldoglover` → Login unter `/admin`.

### Base64-/trivial kodierte Geheimnisse

"Security through Obscurity" — kodiert, aber nicht verschlüsselt. Trivial
umkehrbar. Tritt z.B. in Lizenz-/Info-Seiten oder versteckten Endpunkten auf.

```bash
# Auf der Seite gefundenen String dekodieren
echo 'ZWxsaW90OkVSMjgtMDY1Mgo=' | base64 -d
# → elliot:ER28-0652

# Endpunkte / Dateien, die so was enthalten:
curl -s http://$RHOST/license | strings | grep -E '^[A-Za-z0-9+/]{16,}={0,2}$'
```

> **Lab-Beispiel VM3 (MrRobot):** Unter `/license` lag tief im Text ein
> Base64-String, der zu `elliot:ER28-0652` dekodierte — das Login-Passwort
> für den WordPress-Brute-Force.

### robots.txt als Wortlisten-/Key-Quelle

```bash
curl -s http://$RHOST/robots.txt
# Referenzierte Dateien runterladen:
wget http://$RHOST/fsocity.dic
wget http://$RHOST/key-1-of-3.txt
# Wortliste vor Brute-Force entduplizieren:
sort -u fsocity.dic > fsocity_clean.dic
```

> **Lab-Beispiel VM3:** `robots.txt` verwies auf `fsocity.dic` (Wortliste
> für den Login-Brute) und `key-1-of-3.txt` (erste Flag).

## Burp / Proxy

```bash
# Firefox auf 127.0.0.1:8080 stellen
# Alle Requests durch Burp → Repeater & Intruder nutzen
```

## ➡️ Vulns gefunden?

- SQLi → [Web Exploitation](../../02-Exploitation/Web/README.md)
- LFI/RFI → [Web Exploitation](../../02-Exploitation/Web/README.md)
- File Upload → [Web Exploitation](../../02-Exploitation/Web/README.md)
- Command Injection (Linux **oder** Windows) → [Web Exploitation](../../02-Exploitation/Web/README.md) · [Windows Command Injection](../../02-Exploitation/Windows/command-injection-windows.md)
- WordPress → [WordPress-Workflow](../../02-Exploitation/Web/wordpress.md)
- Admin-Panel → Default-Creds, dann CMS-Exploit
- Hashes/Base64-Creds gefunden → [Passwords](../../02-Exploitation/Passwords/README.md)
