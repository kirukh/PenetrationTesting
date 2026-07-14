# 🚀 Phase 3 — Privilege Escalation

> Sobald du eine Low-Priv-Shell hast: stabilisieren, Identität prüfen, Flags suchen, dann systematisch PrivEsc.

---

## Sofort nach Initial Access

```bash
whoami || id
hostname
pwd
ip a 2>/dev/null || ipconfig
```

Dann:

- Shell stabilisieren: [shell-upgrade.md](../99-Resources/shell-upgrade.md)
- Linux: [Linux PrivEsc](./Linux/README.md)
- Windows: [Windows PrivEsc](./Windows/README.md)

> **Hinweis Prof:** Nicht nur Tools laufen lassen. Erst manuell: Wer bin ich? Welche User gibt es? Welche Dateien geben Hinweise? Gibt es `sudo -l`, SUID, Services, Scheduled Tasks, History?

---

## Flags früh suchen

```bash
# Linux
find / -iname 'user.txt' -o -iname 'root.txt' 2>/dev/null

# Windows
where /r C:\ user.txt 2>nul
where /r C:\ root.txt 2>nul
```

> **Hinweis Prof:** Wenn du mit euid/root-Rechten oder einem speziellen Exploit nur Dateien lesen kannst, direkt `root.txt` lesen und dokumentieren.
