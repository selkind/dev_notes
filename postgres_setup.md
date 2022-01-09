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
