Data migration from PG9.2 (and up) to PG16 (copy databases and users and upgrade all).

## Tips

As [`pg_upgrade` docs](https://www.postgresql.org/docs/16/pgupgrade.html) mention `pg_upgrade` is faster then doing dump and restore (and it really is). It also seems like a safer process as `pg_upgrade` seem to check a lot of things on the way and provides solutions for most problems encountered. At least that was my experience.

**Important tip**: Start with an empty installation of PG16. Do not create any users/roles in your new server. In the process PG users are also migrated so you will get conflicts. If you already have some data in PG16 you can use steps described in *Re-starting migration* (assuming you can throw away this data).

*Tip*: You might need to correct some stuff, `pg_upgrade` should provide scripts. Check upgrade logs in case the message provided by `pg_upgrade` was not clear.

## Migration

Backup config files first:
```bash
cd /etc/postgresql/16/main/
cp pg_hba.conf pg_hba.conf.default
cp postgresql.conf postgresql.conf.default
```

Note that we are assuming port `5416` for the new server.
```bash
su postgres

mkdir ~/pg_mig
cd ~/pg_mig

# SET PATH=C:\Program Files\PostgreSQL\16\bin;%PATH%

# Test version and auth:
psql -V
#psql -U postgres -p 5416

# Stop servers:
service postgresql@10-main stop
service postgresql@16-main stop

# Check data and bin directory:
ls /var/lib/postgresql/*/main
ls /usr/lib/postgresql/*/bin

# Run upgrade:
#??? /usr/lib/postgresql/16/bin/pg_upgrade -U postgres --old-datadir "/var/lib/postgresql/10/main"  --old-bindir "/usr/lib/postgresql/10/bin"  --new-datadir "/var/lib/postgresql/16/main"  --new-bindir "/usr/lib/postgresql/16/bin"
```

After migration you might need to run extra scripts (as instructed by `pg_upgrade`).

## Re-starting migration

Step 1. Copy `postgresql.conf` and `pg_hba.conf` to some other dir.

Step 2. Remove old `data` dir.

Step 3. Create fresh data folder:
```bash
#??? initdb --pgdata="/var/lib/postgresql/16/main" --auth=md5 --username=postgres --pwprompt --icu-locale="pl-PL" --locale-provider=icu --encoding="UTF8" --locale="pl_PL.utf8" --lc-messages="C"
```

Step 4. Adjust port in `postgresql.conf`.

Step 5. Migrate:
```bash
#??? /usr/lib/postgresql/16/bin/pg_upgrade -U postgres --old-datadir "/var/lib/postgresql/10/main"  --old-bindir "/usr/lib/postgresql/10/bin"  --new-datadir "/var/lib/postgresql/16/main"  --new-bindir "/usr/lib/postgresql/16/bin"
```
