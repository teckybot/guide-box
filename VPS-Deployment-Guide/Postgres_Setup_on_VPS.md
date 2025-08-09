## Setup PostgreSQL on VPS

Install PostgreSQL and Dependencies

```bash
  sudo apt install postgresql postgresql-contrib -y
```
#### Explanation:
-postgresql: The core database system
    postgresql-contrib: Useful additional utilities and extensions

Start and Enable PostgreSQL Service

```bash
  sudo systemctl start postgresql
```
```bash
  sudo systemctl enable postgresql
```

#### Explanation:
- start: Starts the PostgreSQL service immediately.
    enable: Enables PostgreSQL to start automatically on system boot.


Verify PostgreSQL Status

```bash
  sudo systemctl status postgresql
```
#### Explanation:
- Shows whether the PostgreSQL service is running and any potential issues.

Switch to postgres User and Access PostgreSQL Shell

```bash
  sudo -i -u postgres
```
```bash
  psql
```
#### Explanation:
- sudo -i -u postgres: Switches to the postgres system user.
- psql: Starts the PostgreSQL interactive terminal (psql) as the postgres user.

#### Use Cases:
- Perform a series of PostgreSQL commands in a session.
- Create databases, users, and roles interactively.
- Run and test SQL queries directly in the terminal.

Enter PostgreSQL Shell (Alternative Single-Command Method)

```bash
  sudo -u postgres psql
```
#### Explanation:
- Directly launches the psql shell as the postgres user (no full login session).
- Ideal for quickly accessing the DB shell for short tasks.

PostgreSQL commands

- CREATE DATABASE your_db_name;
- CREATE USER your_user_name WITH ENCRYPTED PASSWORD  'your_password';
- ALTER ROLE your_user_name SET client_encoding TO 'utf8';
- ALTER ROLE your_user_name SET default_transaction_isolation TO 'read committed';
- ALTER ROLE your_user_name SET timezone TO 'UTC';
- GRANT ALL PRIVILEGES ON DATABASE your_db_name TO your_user_name;

Switch to the New Database and Grant Schema Privileges

- \c your_db_name
   GRANT ALL PRIVILEGES ON SCHEMA public TO your_user_name;

Exit PostgreSQL Prompt

```bash
  \q
```