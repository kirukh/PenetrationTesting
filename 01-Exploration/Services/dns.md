# 🌍 DNS (53)

## Basics

```bash
# Reverse Lookup
dig -x $RHOST
nslookup $RHOST

# Forward
dig @$RHOST domain.local any
host -t any domain.local $RHOST
```

## Zone Transfer (oft Goldgrube!)

```bash
dig axfr @$RHOST domain.local
host -l domain.local $RHOST
nmap -p53 --script dns-zone-transfer --script-args dns-zone-transfer.domain=domain.local $RHOST
```

## Subdomain Brute

```bash
dnsenum --enum domain.local
fierce --domain domain.local
gobuster dns -d domain.local -w subdomains.txt
```

## NSE

```bash
nmap -p53 --script "dns-*" $RHOST
```
