---
title: SQL & PostgreSQL Cheat Sheet
icon: fontawesome/solid/database
tags:
  - data
  - database
  - sql
  - postgresql
---

## Základní SQL operace

```sql
-- Výběr: Seřadí sestupně podle col1, vrátí prvních 10 záznamů splňujících podmínku
SELECT col1, col2 FROM table_name WHERE col1 = 'hodnota' ORDER BY col1 DESC LIMIT 10;

-- Vložení: Nový řádek se specifickými hodnotami
INSERT INTO table_name (col1, col2) VALUES ('text', 123);

-- Aktualizace: Změní data pouze u řádků splňujících WHERE (POZOR na vynechání WHERE!)
UPDATE table_name SET col1 = 'nova_hodnota' WHERE id = 5;

-- Smazání: Smaže řádky splňující podmínku
DELETE FROM table_name WHERE id = 5;

-- Tvorba tabulky: Definice sloupců včetně datových typů a primárního klíče
CREATE TABLE table_name (
    id SERIAL PRIMARY KEY,
    col1 VARCHAR(255) NOT NULL,
    col2 INTEGER DEFAULT 0
);

-- Modifikace tabulky: Přidání a smazání sloupce
ALTER TABLE table_name ADD COLUMN new_column_name VARCHAR(50);
ALTER TABLE table_name DROP COLUMN old_column_name;
```

## PostgreSQL

`$` - shell, `#` - psql

#### 1. Přihlášení (Superuser)
```console 
$ sudo -S -u postgres psql
```

#### 2. Správa DB a Uživatelů

Při pouhém `CREATE DATABASE ... OWNER ...` v novějších verzích PG nemusí mít uživatel plná práva ve schématu public. Bezpečný a funkční postup:

```sql
-- 1. Vytvoření uživatele s heslem
CREATE USER my_user WITH PASSWORD 'moje_silne_heslo';

-- 2. Vytvoření databáze přiřazené uživateli
CREATE DATABASE my_db OWNER my_user;

-- 3. Připojení do nové DB a explicitní udělení práv (nutné pro frameworky jako Hibernate, Prisma, atd.)
\c my_db
GRANT ALL PRIVILEGES ON SCHEMA public TO my_user;

-- Změna hesla stávajícího uživatele
ALTER USER my_user WITH PASSWORD 'nove_heslo';
```

#### 3. Nastavení lokálního přihlašování (Jméno + Heslo)

Aby se frameworky (např. Node.js, Python, PHP, Java) připojily přes localhost pomocí jména a hesla, je nutné změnit autentizační metodu z ident/peer na heslo.

Konfigurační soubor: `/var/lib/pgsql/data/pg_hba.conf` (nebo `/etc/postgresql/.../pg_hba.conf`)

Změň hodnotu ve sloupci METHOD z ident (nebo scram-sha-256 pokud framework starší šifrování neumí, případně naopak na modernější scram-sha-256 místo md5):

```plaintext
# TYPE  DATABASE        USER            ADDRESS                 METHOD
# Lokální připojení přes soket (přihlášení přes 'psql' v konzoli)
local   all             all                                     peer

# IPv4 lokální připojení (pro frameworky běžící na localhost / 127.0.0.1)
host    all             all             127.0.0.1/32            scram-sha-256
# IPv6 lokální připojení
host    all             all             ::1/128                 scram-sha-256
```

Poznámka: Pokud tvůj framework/ovladač nepodporuje moderní scram-sha-256, použij md5.

Po změně konfigurace vždy restartuj/načti znovu PostgreSQL:

```console
$ sudo systemctl reload postgresql
```

ConnectionString pro frameworky pak vypadá takto: `postgresql+psycopg://my_user:moje_silne_heslo@localhost:5432/my_db`

Ověření připojení: `psql -h 127.0.0.1 -U <user> -d <db_name>`

4. Psql příkazy

```console
\l            -- Seznam všech databází
\c db_name    -- Připojení k vybrané databázi
\dn           -- Seznam schémat
\dt           -- Seznam tabulek v aktuální DB
\d table_name -- Detailní popis struktury tabulky (sloupce, indexy)
\q            -- Ukončení psql konzole
```

#### 5. Záloha a Obnova (Backup / Restore)

```console
# Záloha jedné konkrétní DB do souboru
$ pg_dump my_db > my_db_backup.sql

# Záloha kompletně všech DB (včetně uživatelů a práv)
$ pg_dumpall > all_dbs_backup.sql

# Obnova databáze ze souboru (DB již musí existovat)
$ psql my_db < my_db_backup.sql
```

#### 6. Instalace a První spuštění (Fedora/RHEL)

```console
$ sudo dnf install postgresql-server postgresql-contrib
$ sudo postgresql-setup --initdb --unit postgresql
$ sudo systemctl enable --now postgresql
$ sudo systemctl start postgresql
$ sudo -u postgres psql
```

### TIPS 

- [PostgREST](https://docs.postgrest.org/) API přímo nad PG
- [Restic](https://github.com/restic/restic) zálohování
