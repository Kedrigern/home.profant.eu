# DB

## SQL

## PostgreSQL

```bash
sudo -S -u postgres psql   # pq super user
```

```console 
$ pwd
$ sudo -S -u postgres psql   # pq super user
```

```console
$ sudo systemctl enable postgresql
$ sudo postgresql-setup --initdb --unit postgresql
$ sudo systemctl start postgresql
$ sudo -u postgres psql
# CREATE USER my_user WITH PASSWORD 'my_passwd';
# CREATE DATABASE my_db OWNER my_user;
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
