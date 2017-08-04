# Postgresql Admin Notes

## Connecting through a tunnel

Create the SSH tunnel:

```bash
$ ssh -nNT -L 3333:ra-sam.cdko7wofihzp.us-west-2.rds.amazonaws.com:5432 deploy@w1.rpm
```

```bash
$ psql -h localhost -p 3333 -U postgres -W ra_sam
```

## Getting database size:

For the whole database:

```sql
SELECT datname, pg_size_pretty(pg_database_size(datname))
FROM pg_database
WHERE datname = 'ra_sam_development'
ORDER BY pg_database_size(datname) DESC;
```

Getting amounts for individual tables:

```sql
SELECT
  table_name as "table",
  pg_size_pretty(pg_relation_size(table_name)) as "size"
FROM information_schema.tables
WHERE table_schema = 'public'
ORDER BY pg_relation_size(table_name) DESC;
```

## Showing long running postgres processes

https://medium.com/little-programming-joys/finding-and-killing-long-running-queries-on-postgres-7c4f0449e86d

```sql
SELECT pid, now() - pg_stat_activity.query_start AS duration, left(query, 80), state
FROM pg_stat_activity
WHERE now() - pg_stat_activity.query_start > interval '5 minutes';
```

## What's blocking a running query

```sql
SELECT * FROM pg_blocking_pids(4711);
```

## Kill a hung query

First find the PID of the process

```sql
SELECT pg_terminate_backend(<pid>);
```

## PSQL queries to increase the Postgres log level

```sql
ALTER SYSTEM SET log_min_messages = 'INFO';
ALTER SYSTEM SET log_statement = 'all';
SELECT pg_reload_conf();
```

## Killing idle sessions

Starting with Postgres 9.6, you can use the `idle_in_transaction_session_timeout` to set the amount of time a transaction will take before it is killed.

This query will set the time to 2.5 seconds

```sql
SET idle_in_transaction_session_timeout TO 2500;
```

## Files

### Asking the database where it keeps the log file:

This could be useful, however when I tried it the records didn't have a location for the log file on my system.

```sql
SELECT *
FROM pg_settings
WHERE category IN( 'Reporting and Logging / Where to Log' , 'File Locations')
ORDER BY category, name;
```

### Log File

Using the Postgres.app on the Mac: `~/Library/Application\ Support/Postgres/var-9.5/postgres-server.log`

Using Homebrew: `/usr/local/var/log/postgres.log`

### Config file

Using the Postgres.app on the Mac: `~/Library/Application\ Support/Postgres/var-9.5/postgresql.conf`

Using Homebrew: `/usr/local/var/postgres/postgresql.conf`

### Host based authentication file

Using the Postgres.app on the Mac: `~/Library/Application\ Support/Postgres/var-9.5/pg_hba.conf`

Using Homebrew: `/usr/local/var/postgres/pg_hba.conf`

**Setting the slow query log:**

Set the `log_statement` value to `all`.

Set the `log_min_duration_statement` statement to a number of milliseconds, say `250`.

Queries that last longer than that get logged to the log file:

## Setting up pgbadger for log analysis

[pgbadger](https://github.com/dalibo/pgbadger) will analyze the log file and spit out a report with interesting information about postgres, like a list of the slowest queries.

**Install:**

```bash
$ brew install pgbadger
```

**Config file settings:**

Add to config file `/usr/local/var/postgres/postgres.conf`

```text
log_min_duration_statement = 1
log_duration = off
log_line_prefix = '%t [%p]: [%l-1] user=%u,db=%d,app=%a,client=%h '
log_statement = 'none'
```

Make sure you restart the server.  Run some queries and then generate the report with:

```bash
$ pgbadger /usr/local/var/log/postgres.log
```

The report will be an html file named `out.html`.

## PostgreSQL's explain analyze made readable

You can use the [explain.depesz.com](http://explain.depesz.com) to break down the results of an `EXPLAIN ANALYZE` query in a more human readable fashion.