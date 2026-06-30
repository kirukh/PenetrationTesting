# 📡 SNMP (161/UDP)

> SNMP wird bei UDP-Scans oft übersehen — und liefert massiv Informationen.

## Community Strings finden

```bash
# Brute Communities
onesixtyone -c /usr/share/seclists/Discovery/SNMP/common-snmp-community-strings.txt $RHOST

# nmap
nmap -sU -p161 --script snmp-brute $RHOST
```

Standard-Communities: `public`, `private`, `community`, `manager`

## Walk (alles auslesen)

```bash
snmpwalk -v2c -c public $RHOST
snmpwalk -v1 -c public $RHOST

# Spezifische OIDs
snmpwalk -v2c -c public $RHOST 1.3.6.1.4.1.77.1.2.25      # Windows User
snmpwalk -v2c -c public $RHOST 1.3.6.1.2.1.25.4.2.1.2     # Running Processes
snmpwalk -v2c -c public $RHOST 1.3.6.1.2.1.25.6.3.1.2     # Installed Software
snmpwalk -v2c -c public $RHOST 1.3.6.1.2.1.6.13.1.3       # TCP Listening Ports
```

## Enum-Tool

```bash
snmp-check $RHOST -c public
braa public@$RHOST:1.3.6.*
```

## Wichtige OIDs (Spickzettel)

| OID | Was |
|---|---|
| `1.3.6.1.2.1.25.1.6.0` | System Processes |
| `1.3.6.1.2.1.25.4.2.1.2` | Running Programs |
| `1.3.6.1.2.1.25.4.2.1.4` | Process Paths |
| `1.3.6.1.2.1.25.2.3.1.4` | Storage Units |
| `1.3.6.1.2.1.25.6.3.1.2` | Software Names |
| `1.3.6.1.4.1.77.1.2.25` | User Accounts |
| `1.3.6.1.2.1.6.13.1.3` | TCP Local Ports |
