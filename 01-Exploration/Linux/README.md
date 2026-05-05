# 🐧 Linux Enumeration

> Du hast erkannt: das Ziel ist Linux. Jetzt: was läuft genau drauf?

---

## Typische Linux-Ports

| Port | Service | Notizen |
|---|---|---|
| 21 | FTP | Anonymous Login? Version exploitable? |
| 22 | SSH | Key-Auth? Bruteforce? |
| 25 | SMTP | VRFY für User-Enum |
| 53 | DNS | Zone Transfer? |
| 80, 443 | HTTP/HTTPS | **fast immer interessant** |
| 110, 143 | POP3, IMAP | Mail |
| 111 | rpcbind | NFS? |
| 139, 445 | Samba | Wie Windows-SMB |
| 161 | SNMP | community strings |
| 2049 | NFS | mountbar? |
| 3306 | MySQL | leere root-Passwörter? |
| 5432 | PostgreSQL | |
| 6379 | Redis | oft no-auth |
| 8080, 8000 | HTTP-Alt | Tomcat, Jenkins, etc. |

---

## 🎯 First Moves

### 1. Web ist meistens der Einstieg
Wenn 80/443/8080/etc. offen → [Web Enumeration](../Services/web.md)

### 2. SSH-Banner checken
```bash
nc -nv $RHOST 22
ssh $RHOST                # Banner zeigt manchmal Distro
```
→ [SSH](../Services/ssh.md)

### 3. NFS prüfen
```bash
showmount -e $RHOST
# Wenn Mount offen:
sudo mount -t nfs $RHOST:/share /mnt/nfs -o nolock
```

### 4. SMB (Samba auf Linux)
```bash
smbclient -L //$RHOST/ -N
enum4linux-ng -A $RHOST
```
→ [SMB](../Services/smb.md)

### 5. SNMP (oft vergessen, oft Goldgrube)
```bash
snmpwalk -v2c -c public $RHOST
onesixtyone -c communities.txt $RHOST
```
→ [SNMP](../Services/snmp.md)

---

## 🚨 Quick Wins

```bash
# Anonymous FTP
ftp $RHOST                   # User: anonymous, PW: leer

# Redis ohne Auth
redis-cli -h $RHOST

# MySQL mit leerem PW
mysql -h $RHOST -u root

# rsync ohne Auth
rsync rsync://$RHOST/
```

---

## 📋 Banner Grabbing

```bash
# Schnell auf einen Port:
nc -nv $RHOST 80
curl -I http://$RHOST/

# Mit nmap:
nmap -sV --version-intensity 9 -p $PORT $RHOST

# whatweb (für Webseiten)
whatweb http://$RHOST/
```

---

## ➡️ Wenn du Creds hast

Geh zu [Exploitation → Linux](../../02-Exploitation/Linux/README.md)

## ➡️ Wenn du eine Shell hast

Geh zu [PrivEsc → Linux](../../03-PrivEsc/Linux/README.md)
