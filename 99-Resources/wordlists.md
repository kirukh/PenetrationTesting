# 📜 Wordlists — Wo liegt was?

## Kali Standard

```bash
ls /usr/share/wordlists/
ls /usr/share/seclists/                    # apt install seclists
```

## Top Picks

### Passwörter

```
/usr/share/wordlists/rockyou.txt                                                    # 14M, der Klassiker
/usr/share/seclists/Passwords/Common-Credentials/10-million-password-list-top-1000000.txt
/usr/share/seclists/Passwords/Common-Credentials/10-million-password-list-top-10000.txt
/usr/share/seclists/Passwords/Leaked-Databases/rockyou-75.txt                       # gefiltert
/usr/share/seclists/Passwords/probable-v2-top12000.txt                              # Spraying
```

### User

```
/usr/share/seclists/Usernames/Names/names.txt
/usr/share/seclists/Usernames/top-usernames-shortlist.txt
/usr/share/seclists/Usernames/cirt-default-usernames.txt
/usr/share/seclists/Usernames/xato-net-10-million-usernames.txt
```

### Web Directories

```
/usr/share/wordlists/dirb/common.txt                                                # klein, schnell
/usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt                        # mittel
/usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt               # mein Favorit
/usr/share/seclists/Discovery/Web-Content/big.txt                                   # groß
/usr/share/seclists/Discovery/Web-Content/raft-medium-words.txt                     # für Param-Brute
```

### Web Files

```
/usr/share/seclists/Discovery/Web-Content/raft-medium-files.txt
/usr/share/seclists/Discovery/Web-Content/quickhits.txt                             # known-vuln paths
```

### DNS / Subdomains

```
/usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt
/usr/share/seclists/Discovery/DNS/subdomains-top1million-20000.txt
/usr/share/seclists/Discovery/DNS/fierce-hostlist.txt
```

### Default-Creds (Routers, Services)

```
/usr/share/seclists/Passwords/Default-Credentials/default-passwords.csv
```

### SNMP Communities

```
/usr/share/seclists/Discovery/SNMP/common-snmp-community-strings.txt
```

## Custom Wordlists erstellen

```bash
# CeWL (von Webseite)
cewl http://$RHOST/ -m 6 -w custom.txt
cewl http://$RHOST/ -d 3 -m 5 -w custom.txt              # tiefer crawl

# Mit Mutationen
hashcat --stdout custom.txt -r /usr/share/hashcat/rules/best64.rule > custom_mutated.txt

# Username-Generator (vorname.nachname etc.)
username-anarchy -i users.txt
```

## Quick Brute (kleine, schnelle Listen)

```
/usr/share/seclists/Passwords/Common-Credentials/best110.txt
/usr/share/seclists/Passwords/Common-Credentials/best15.txt
/usr/share/seclists/Passwords/probable-v2-top1575.txt
```

## Verfügbarkeits-Check

```bash
# Falls SecLists fehlt:
sudo apt install seclists

# Falls rockyou gzipped ist:
sudo gunzip /usr/share/wordlists/rockyou.txt.gz
```
