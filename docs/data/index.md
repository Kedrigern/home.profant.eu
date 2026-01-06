---
title: SQL
icon: fontawesome/solid/database
---

```sql
SELECT col1 , col2 FROM table_name WHERE condition ORDER BY col1 DESC LIMIT 10;
INSERT INTO table_name (col1, col2) VALUES (val1, val2);
UPDATE table_name SET col1 = val1, col2 = val2 WHERE condition;
DELETE FROM table_name WHERE condition;
CREATE TABLE table_name (col1 datatype, col2 datatype, ...);
ALTER TABLE table_name ADD column_name datatype;
ALTER TABLE table_name DROP COLUMN column_name; 
```

## PostgreSQL

`$` - shell, `#` - psql

#### PG super user
```console 
$ sudo -S -u postgres psql
```

#### DB and users

```console
# CREATE USER my_user WITH PASSWORD 'my_passwd';
# CREATE DATABASE my_db OWNER my_user;
# ALTER USER user_name WITH PASSWORD 'new_password';
```

#### PSQL

```console
# \l         -- list dbs
# \c db_name -- connect to db
# \dt        -- list tables
# \d table_name -- describe table
# \q         -- quit
```

#### Backup and restore

```console
$ pg_dump my_db > my_db
$ pg_dumpall > outfile
$ psql dbname < my_db
```

#### PG setup
```console
$ sudo systemctl enable postgresql
$ sudo postgresql-setup --initdb --unit postgresql
$ sudo systemctl start postgresql
$ sudo -u postgres psql
```

Config: `/var/lib/pgsql/data/pg_hba.conf`:

Key are these lines:

```commandline
# IPv4 local connections:
host    all             all             127.0.0.1/32            ident
# IPv6 local connections:
host    all             all             ::1/128                 ident 
```

Change column `method` from `ident` to `md5`.

## PostgREST

https://docs.postgrest.org/en/v14/
