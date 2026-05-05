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

## Manuelle Checks (nicht vergessen!)

- `robots.txt`, `sitemap.xml`
- `.git/` (→ `git-dumper http://$RHOST/.git/ ./loot`)
- `.env`, `.bak`, `.old`, `.swp`
- `/admin`, `/login`, `/api`, `/dev`, `/test`
- HTML-Source: Kommentare, JS-Files (oft Endpoints/Keys drin!)
- Cookies & Auth-Header
- Standard-Creds (`admin:admin`, `admin:password`, `tomcat:tomcat`)

## Burp / Proxy

```bash
# Firefox auf 127.0.0.1:8080 stellen
# Alle Requests durch Burp → Repeater & Intruder nutzen
```

## ➡️ Vulns gefunden?

- SQLi → [Web Exploitation](../../02-Exploitation/Web/README.md)
- LFI/RFI → [Web Exploitation](../../02-Exploitation/Web/README.md)
- File Upload → [Web Exploitation](../../02-Exploitation/Web/README.md)
- Admin-Panel → Default-Creds, dann CMS-Exploit
