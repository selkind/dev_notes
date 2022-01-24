# Dev notes

## This is a place to store notes on solutions to problems I've come across so that I don't have to solve them twice

### I'll try to organize topics by tags

#### Postgres setup on wsl

- installation is pretty straight forward with apt
- start up db with sudo service start postgresql (lots of guides will use systemctl to inspect the service status, but systemd isn't installed on wsl)
- use 'sudo psql -u postgres' to login initially (using root as user postgres should mean no pw auth is required)
- immediately change postgres's pw to something secure
- pg auth configuration is in /etc/postgresql/<version>/main/
    - relevant files are pg_hba.conf and postgres.conf. postgres.conf runs on startup so any changes need to be paired with a sudo service postgresql restart command
- in postgres.conf there's a commented out password_encryption var. If it is uncommented, it needs to be consistent with the auth method for the relevant rows in the pg_hba.conf file
- manually create a new db as user postgres for a new app
    - unsure how using docker volumes and/or alembic would alter this process
- create a new user for the app with a password so that the app connection isn't via a superuser.
- once the db is created, as a super user, grant privileges to all tables in schema public to the user
created for the app
    - something like 'GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA public TO <user>;'
    
### NOTE that the fastapi cookie cutter has a healthcheck defined by the command 'pg_isready -U <user>' This user needs to be consistent with the user defined by docker secrets.

## Postgres setup in docker-compose
Shockingly, the default postgres image only sets up a single superuser for the database based on default values
or the env vars POSTGRES_USER, POSTGRES_DB, and POSTGRES_PASSWORD.

This is great for database administration, but terrible for web security. Ultimately I need a user with
limited privileges that my web service can connect as for API queries. In order to mitigate the damage of
sql injection attacks, I do not want my web server to have super user privileges (duh).
[This github issue outlines the problem pretty clearly](https://github.com/docker-library/postgres/issues/175)

Based on a post within this discussion, I created a file within the docker-entrypoint-initdb.d directory
that looks like this 

```bash
#!/bin/sh

set -e

# get the postgres user or set it to a default value
if [ -n $POSTGRES_USER ]; then pg_user=$POSTGRES_USER; else pg_user="postgres"; fi
# get the postgres db or set it to a default value
if [ -n $POSTGRES_DB ]; then pg_db=$POSTGRES_DB; else pg_db=$POSTGRES_USER; fi

if [ -n "$POSTGRES_NON_ROOT_USER" ]; then
psql -v ON_ERROR_STOP=1 --username "$pg_user" --dbname "$pg_db" <<-EOSQL
    CREATE USER $POSTGRES_NON_ROOT_USER WITH encrypted password '$POSTGRES_NON_ROOT_USER_PASSWORD';
    GRANT CREATE, CONNECT ON DATABASE $pg_db TO $POSTGRES_NON_ROOT_USER;
    ALTER DEFAULT PRIVILEGES IN SCHEMA public GRANT SELECT, UPDATE, INSERT, DELETE, REFERENCES ON TABLES TO
    $POSTGRES_NON_ROOT_USER;
EOSQL
fi
```

This does exactly what I want. Uses two extra env vars to configure a user with the right level of privileges 
for a web server.
