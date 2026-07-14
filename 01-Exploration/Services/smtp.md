# ✉️ SMTP (25)

## Banner & Basics

```bash
nc -nv $RHOST 25
nmap -p25 --script smtp-commands,smtp-enum-users,smtp-vuln-* $RHOST
```

## User-Enumeration (VRFY/EXPN/RCPT)

```bash
# Manuell
nc $RHOST 25
> HELO test
> VRFY root
> VRFY admin

# Mit smtp-user-enum
smtp-user-enum -M VRFY -U users.txt -t $RHOST
smtp-user-enum -M EXPN -U users.txt -t $RHOST
smtp-user-enum -M RCPT -U users.txt -t $RHOST
```

## Open Relay Test

```bash
nmap -p25 --script smtp-open-relay $RHOST
```

## Bekannte Vulns

- Exim < 4.91 (CVE-2019-10149) — RCE
- Postfix mit Sasl-Misconfig
