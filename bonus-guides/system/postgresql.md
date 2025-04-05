---
layout:
  title:
    visible: true
  description:
    visible: false
  tableOfContents:
    visible: true
  outline:
    visible: true
  pagination:
    visible: true
---

# PostgreSQL

PostgreSQL is a powerful, open source object-relational database system that uses and extends the SQL language combined with many features that safely store and scale the most complicated data workloads.

{% hint style="warning" %}
Difficulty: Medium
{% endhint %}

<figure><img src="../../.gitbook/assets/PostgreSQL-Logo-white.png" alt="" width="563"><figcaption></figcaption></figure>

## Installation

### Install PostgreSQL using the apt package manager

* With user `admin`, update and upgrade your OS. Press "**y**" and `enter` or directly `enter` when the prompt asks you

```bash
sudo apt update && sudo apt full-upgrade
```

* Import the repository signing key

{% code overflow="wrap" %}
```bash
sudo install -d /usr/share/postgresql-common/pgdg
```
{% endcode %}

{% code overflow="wrap" %}
```bash
sudo curl -o /usr/share/postgresql-common/pgdg/apt.postgresql.org.asc --fail https://www.postgresql.org/media/keys/ACCC4CF8.asc
```
{% endcode %}

Expected output:

```
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  4812  100  4812    0     0   5453      0 --:--:-- --:--:-- --:--:--  5449
```

* Create the repository configuration file

{% code overflow="wrap" %}
```bash
sudo sh -c 'echo "deb [signed-by=/usr/share/postgresql-common/pgdg/apt.postgresql.org.asc] https://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'
```
{% endcode %}

* Update the package lists and install the latest version of PostgreSQL. Press "**y**" and `enter` or directly `enter` when the prompt asks you

```bash
sudo apt update && sudo apt install postgresql postgresql-contrib
```

* Check the correct installation of PostgreSQL

```bash
psql -V
```

**Example** of expected output:

```
psql (PostgreSQL) 17.4 (Debian 17.4-1.pgdg120+2)
```

* Ensure PostgreSQL is running and listening on the default port `5432`

```bash
sudo ss -tulpn | grep postgres
```

Expected output:

<pre><code><strong>tcp   LISTEN 0      200        127.0.0.1:5432       0.0.0.0:*    users:(("postgres",pid=2532748,fd=7))
</strong>tcp   LISTEN 0      200            [::1]:5432          [::]:*    users:(("postgres",pid=2532748,fd=6))
</code></pre>

* You can monitor general logs by the systemd journal. You can exit monitoring at any time with `Ctrl-C`

```bash
journalctl -fu postgresql
```

**Example** of expected output:

```
Mar 26 08:56:51 ramix systemd[1]: Starting postgresql.service - PostgreSQL RDBMS...
Mar 26 08:56:51 ramix systemd[1]: Finished postgresql.service - PostgreSQL RDBMS.
```

* And the sub-instance and specific cluster logs. You can exit monitoring at any time with `Ctrl-C`

```bash
journalctl -fu postgresql@17-main
```

**Example** of expected output:

```
Mar 26 08:57:02 ramix systemd[1]: Starting postgresql@17-main.service - PostgreSQL Cluster 17-main...
Mar 26 08:57:05 ramix systemd[1]: Started postgresql@17-main.service - PostgreSQL Cluster 17-main.
```

### Create data folder

* Create the dedicated PostgreSQL data folder

```bash
sudo mkdir -p /data/postgresdb/17
```

* Assign as the owner to the `postgres` user

```bash
sudo chown -R postgres:postgres /data/postgresdb
```

* Assign permissions of the data folder only to the `postgres` user

```bash
sudo chmod -R 700 /data/postgresdb
```

* With user `postgres`, create a new cluster in the dedicated folder

```bash
sudo -u postgres /usr/lib/postgresql/17/bin/initdb -D /data/postgresdb/17
```

<details>

<summary><strong>Example</strong> of expected output ⬇️</summary>

```
The files belonging to this database system will be owned by user "postgres".
This user must also own the server process.

The database cluster will be initialized with locale "en_US.UTF-8".
The default database encoding has accordingly been set to "UTF8".
The default text search configuration will be set to "english".

Data page checksums are disabled.

fixing permissions on existing directory /data/postgresdb17 ... ok
creating subdirectories ... ok
selecting dynamic shared memory implementation ... posix
selecting default "max_connections" ... 100
selecting default "shared_buffers" ... 128MB
selecting default time zone ... Etc/UTC
creating configuration files ... ok
running bootstrap script ... ok
performing post-bootstrap initialization ... ok
syncing data to disk ... ok

initdb: warning: enabling "trust" authentication for local connections
initdb: hint: You can change this by editing pg_hba.conf or using the option -A, or --auth-local and --auth-host, the next time you run initdb.

Success. You can now start the database server using:

    /usr/lib/postgresql/17/bin/pg_ctl -D /data/postgresdb/17 -l logfile start
```

</details>

* Edit the PostgreSQL data directory in configuration, to redirect the store to the new location

```bash
sudo nano +42 /etc/postgresql/17/main/postgresql.conf --linenumbers
```

* Replace the `line 42` with `/var/lib/postgresql/17/main` to the next. Save and exit

<pre><code><strong>data_directory = '/data/postgresdb/17'
</strong></code></pre>

* Restart PostgreSQL to apply changes and monitor the correct status of the main instance and sub-instance monitoring sessions before

<pre class="language-bash"><code class="lang-bash"><strong>sudo systemctl restart postgresql
</strong></code></pre>

* You can monitor the PostgreSQL main instance by the systemd journal and check the log output. You can exit the monitoring at any time with `Ctrl-C`

```bash
journalctl -fu postgresql
```

Expected output:

```
Nov 08 11:51:10 ramix systemd[1]: Stopped PostgreSQL RDBMS.
Nov 08 11:51:10 ramix systemd[1]: Stopping PostgreSQL RDBMS...
Nov 08 11:51:13 ramix systemd[1]: Starting PostgreSQL RDBMS...
Nov 08 11:51:13 ramix systemd[1]: Finished PostgreSQL RDBMS.
```

* You can monitor the PostgreSQL sub-instance by the systemd journal and check the log output. You can exit monitoring at any time with `Ctrl-C`

```bash
journalctl -fu postgresql@17-main
```

**Example** of the expected output:

```
Nov 08 11:51:10 ramix systemd[1]: Stopping PostgreSQL Cluster 17-main...
Nov 08 11:51:11 ramix systemd[1]: postgresql@17-main.service: Succeeded.
Nov 08 11:51:11 ramix systemd[1]: Stopped PostgreSQL Cluster 17-main.
Nov 08 11:51:11 ramix systemd[1]: postgresql@17-main.service: Consumed 1h 10min 8.677s CPU time.
Nov 08 11:51:11 ramix systemd[1]: Starting PostgreSQL Cluster 17-main...
Nov 08 11:51:13 ramix systemd[1]: Started PostgreSQL Cluster 17-main.
```

* You can check if the cluster is in status "online" by

```bash
pg_lsclusters
```

Expected output:

```
Ver Cluster Port Status Owner    Data directory       Log file
17  main    5432 online <unknown> /data/postgresdb/17  /var/log/postgresql/postgresql-17-main.log
```

{% hint style="info" %}
**(Optional)** -> If you want, you can **disable the autoboot** option for PostgreSQL **(not recommended)** using:

```bash
sudo systemctl disable postgresql
```

Expected output:

```
Synchronizing state of postgresql.service with SysV service script with /lib/systemd/systemd-sysv-install.
Executing: /lib/systemd/systemd-sysv-install disable postgresql
Removed /etc/systemd/system/multi-user.target.wants/postgresql.service.
```
{% endhint %}

#### Validation

* Ensure PostgreSQL is listening on the default relational database port

```bash
sudo ss -tulpn | grep postgres
```

Expected output:

```
tcp   LISTEN 0      200        127.0.0.1:5432       0.0.0.0:*    users:(("postgres",pid=2532748,fd=7))
tcp   LISTEN 0      200            [::1]:5432          [::]:*    users:(("postgres",pid=2532748,fd=6))
```

### Create a PostgreSQL user account

* Create a new database `admin` user and assign the password "`admin`" with the automatically created user for the PostgreSQL installation, called `postgres`

<pre class="language-bash"><code class="lang-bash"><strong>sudo -u postgres psql -c "CREATE ROLE admin WITH LOGIN CREATEDB PASSWORD 'admin';"
</strong></code></pre>

Expected output:

```
CREATE ROLE
```

{% hint style="success" %}
Congrats! You have PostgreSQL ready to use as a database backend for another software
{% endhint %}

## Extras (optional)

### Some useful PostgreSQL commands

* With user `admin`, enter the PostgreSQL CLI with the user `postgres`. The prompt should change to `postgres=#`

```bash
sudo -u postgres psql
```

**Example** of expected output:

```
psql (16.3 (Ubuntu 16.3-1.pgdg22.04+1))
Type "help" for help.

postgres=#
```

{% hint style="info" %}
Type `\q` command and enter to exit PostgreSQL CLI, and exit to come back to the `admin` user
{% endhint %}

#### List the global existing users and roles associated

* Type the next command and enter

```sql
\du
```

**Example** of expected output:

```
                             List of roles
 Role name |                         Attributes
-----------+------------------------------------------------------------
 admin     | Create DB
 postgres  | Superuser, Create role, Create DB, Replication, Bypass RLS
```

#### List the existing global databases

* Type the next command and enter

```sql
\l
```

**Example** of expected output:

```
     Name     |  Owner   | Encoding | Locale Provider |   Collate   |    Ctype    | ICU Locale | ICU Rules |   Access privileges
--------------+----------+----------+-----------------+-------------+-------------+------------+-----------+-----------------------
 btcpay       | admin    | UTF8     | libc            | en_US.UTF-8 | en_US.UTF-8 |            |           |
 lndb         | admin    | UTF8     | libc            | en_US.UTF-8 | en_US.UTF-8 |            |           |
 nbxplorer    | admin    | UTF8     | libc            | en_US.UTF-8 | en_US.UTF-8 |            |           |
 nostrelay    | admin    | UTF8     | libc            | en_US.UTF-8 | en_US.UTF-8 |            |           |
 postgres     | postgres | UTF8     | libc            | en_US.UTF-8 | en_US.UTF-8 |            |           |
 template0    | postgres | UTF8     | libc            | en_US.UTF-8 | en_US.UTF-8 |            |           | =c/postgres          +
              |          |          |                 |             |             |            |           | postgres=CTc/postgres
 template1    | postgres | UTF8     | libc            | en_US.UTF-8 | en_US.UTF-8 |            |           | =c/postgres          +
              |          |          |                 |             |             |            |           | postgres=CTc/postgres
(8 rows)
```

#### List tables inside a specific database

* Connect to a specific database, type the next command, and enter. The prompt should change to the name of the database. Example: `lndb=#`

```sql
\c <NAMEOFDATABASE>
```

{% hint style="info" %}
Replace `<NAMEOFDATABASE`> to the specific name of the database
{% endhint %}

**Example:**

```bash
\c lndb
```

**Expected output:**

```
You are now connected to database "lndb" as user "postgres".
```

* List tables

```sql
\dt
```

**Example of expected output:**

```
             List of relations
 Schema |       Name       | Type  | Owner
--------+------------------+-------+-------
 public | channeldb_kv     | table | admin
 public | decayedlogdb_kv  | table | admin
 public | macaroondb_kv    | table | admin
 public | towerclientdb_kv | table | admin
 public | towerserverdb_kv | table | admin
 public | walletdb_kv      | table | admin
(6 rows)
```

#### **View the size of a specific database**

* Type the next command and enter

```sql
SELECT pg_size_pretty(pg_database_size('<NAMEOFDATABASE>'));
```

{% hint style="info" %}
Replace `<NAMEOFDATABASE`> to the specific name of the database
{% endhint %}

**Example:**

```sql
SELECT pg_size_pretty(pg_database_size('lndb'));
```

**Example** of expected output:

```
 pg_size_pretty
----------------
 546 MB
(1 row)
```

#### **View the size of a specific table inside a database**

* Enter a specific database with

```sql
\c <NAMEOFDATABASE>
```

{% hint style="info" %}
Replace `<NAMEOFDATABASE>` to the specific name of the database
{% endhint %}

**Example:**

```sql
\c lndb
```

* View the size of a specific table

```sql
SELECT pg_size_pretty(pg_total_relation_size('<NAMEOFTABLE>'));
```

{% hint style="info" %}
Replace `<NAMEOFTABLE>` to the specific name of the database
{% endhint %}

**Example:**

```sql
SELECT pg_size_pretty(pg_total_relation_size('channeldb_kv'));
```

**Example** of expected output:

```
 pg_size_pretty
----------------
 457 MB
(1 row)
```

#### **Show if there is content inside a table**

Get a quick view of the data stored in a table without having to retrieve all the records. Useful after a data migration, for example.

* Enter a specific database

```
\c <NAMEOFDATABASE>
```

Example:

```bash
\c lndb
```

* Type the next command to make a request and obtain the data

```sql
 SELECT * FROM <NAMEOFTABLE> LIMIT 10;
```

{% hint style="info" %}
Replace `<NAMEOFTABLE>` to the specific name of the database
{% endhint %}

#### **Example:**

```sql
 SELECT * FROM channeldb_kv LIMIT 10;
```

#### **Delete** a specific database

* Type the next command and enter

```sql
DROP DATABASE <NAMEOFDATABASE>;
```

{% hint style="info" %}
Replace `<NAMEOFTABLE>` to the specific name of the table
{% endhint %}

Example:

```sql
DROP DATABASE lndb;
```

#### Expected output:

```
DROP DATABASE
```

#### Delete a table inside a specific database

{% hint style="warning" %}
Stop the service related to this database before the action, i.e: `sudo systemctl stop lnd`
{% endhint %}

* Enter a specific database with

```sql
\c <NAMEOFDATABASE>
```

{% hint style="info" %}
Replace `<NAMEOFDATABASE>` to the specific name of the database
{% endhint %}

**Example:**

```sql
\c lndb
```

* Delete a specific table

{% hint style="warning" %}
Stop the service related to this table and database before the action, i.e: `sudo systemctl stop lnd`
{% endhint %}

```sql
DROP TABLE <NAMEOFTABLE>;
```

{% hint style="info" %}
Replace `<NAMEOFTABLE>` to the specific name of the table
{% endhint %}

{% hint style="danger" %}
Warning: this command is especially dangerous, do it at your own risk
{% endhint %}

Example:

```sql
DROP TABLE towerclientdb_kv;
```

#### Delete existing users

* In some hypothetical situation, you might want to delete an existing user of PostgreSQL. Type the next command

```sql
DROP ROLE <user>;
```

{% hint style="info" %}
Replace `<user>` to the desired user
{% endhint %}

Example:

```sql
DROP ROLE admin;
```

{% hint style="danger" %}
Warning: this command is especially dangerous, do it at your own risk
{% endhint %}

## Upgrade

The latest release can be found on the [official PostgreSQL web page](https://www.postgresql.org/ftp/source/).

* To upgrade, type this command. Press "y" and enter, or directly enter when the prompt asks you

```bash
sudo apt update && sudo apt full-upgrade
```

### Migrate to a major version <a href="#upgrade-to-major-version" id="upgrade-to-major-version"></a>

* With user `admin`, ensure you followed [the previous Upgrade](postgresql.md#upgrade) section

#### **PostgreSQL server migration**

* Stop all existing clusters

```bash
sudo systemctl stop postgresql@16-main postgresql@17-main
```

* Create a new database destination folder for the new v17 cluster, ready for migration from v16

{% hint style="info" %}
This could change in the future with the next releases, for example, you will need to replace v16 with v17 and v17 with v18
{% endhint %}

```bash
sudo mkdir /data/postgresdb/17
```

* Assign the owner as the postgres user

```bash
sudo chown postgres:postgres /data/postgresdb/17
```

* Assign the correct permissions

```bash
sudo chmod 700 /data/postgresdb/17
```

* Delete the cluster created by default for v17

```bash
sudo -u postgres pg_dropcluster 17 main
```

* Update the systemd

```bash
sudo systemctl daemon-reload
```

* Start the migration with the PostgreSQL migration tool

```
sudo -u postgres pg_upgradecluster 16 main /data/postgresdb/17
```

{% hint style="info" %}
⌛ This may take a lot of time depending on the existing database size (the nostr relay database especially) and your machine's performance; it is recommended to use [tmux](https://github.com/tmux/tmux). Wait until the prompt shows up again
{% endhint %}

<details>

<summary>Example of expected output 👇</summary>

```
Restarting old cluster with restricted connections...
Notice: extra pg_ctl/postgres options given, bypassing systemctl for start operation
Creating new PostgreSQL cluster 17/main ...
/usr/lib/postgresql/17/bin/initdb -D /data/postgresdb/17 --auth-local peer --auth-host scram-sha-256 --no-instructions --encoding UTF8 --lc-collate en_US.UTF-8 --lc-ctype en_US.UTF-8 --locale-provider libc
The files belonging to this database system will be owned by user "postgres".
This user must also own the server process.

The database cluster will be initialized with locale "en_US.UTF-8".
The default text search configuration will be set to "english".

Data page checksums are disabled.

[...]
Starting upgraded cluster on port 5432...
Warning: the cluster will not be running as a systemd service. Consider using systemctl:
  sudo systemctl start postgresql@17-main
Running finish phase upgrade hook scripts ...
vacuumdb: processing database "postgres": Generating minimal optimizer statistics (1 target)
vacuumdb: processing database "template1": Generating minimal optimizer statistics (1 target)
vacuumdb: processing database "postgres": Generating medium optimizer statistics (10 targets)
vacuumdb: processing database "template1": Generating medium optimizer statistics (10 targets)
vacuumdb: processing database "postgres": Generating default (full) optimizer statistics
vacuumdb: processing database "template1": Generating default (full) optimizer statistics

Success. Please check that the upgraded cluster works. If it does,
you can remove the old cluster with
    pg_dropcluster 16 main

Ver Cluster Port Status Owner    Data directory      Log file
16  main    5433 down   postgres /data/postgresdb/16 /var/log/postgresql/postgresql-16-main.log
Ver Cluster Port Status Owner    Data directory      Log file
17  main    5432 online postgres /data/postgresdb/17 /var/log/postgresql/postgresql-17-main.log
```

</details>

* Reload the systemd again

```bash
sudo systemctl daemon-reload
```

* List the clusters to show the state

```bash
pg_lsclusters
```

Example of expected output:

```
Ver Cluster Port Status  Owner     Data directory      Log file
16  main    5433 down   <unknown> /data/postgresdb/16 /var/log/postgresql/postgresql-16-main.log
17  main    5432 online  <unknown> /data/postgresdb/17 /var/log/postgresql/postgresql-17-main.log
```

{% hint style="info" %}
> Don't worry about the output of the "Owner" column

> Note how the old 16 cluster has automatically gone into status "down" after the migration
{% endhint %}

* Stop the version 17 cluster using the `pg_ctlcluster` tool, to then be able to run it and manage it with `systemd`

```bash
sudo pg_ctlcluster 17 main stop
```

* List the clusters again to show the state

```bash
pg_lsclusters
```

Example of expected output:

```
Ver Cluster Port Status Owner     Data directory      Log file
16  main    5433 down   <unknown> /data/postgresdb/16 /var/log/postgresql/postgresql-16-main.log
17  main    5432 down   <unknown> /data/postgresdb/17 /var/log/postgresql/postgresql-17-main.log
```

{% hint style="info" %}
Note how the version 17 cluster has gone into status "down"
{% endhint %}

* Start the new version 17 cluster with systemd

```bash
sudo systemctl start postgresql@17-main
```

* Monitor the logs of the PostgreSQL version 17 cluster to ensure that it is working fine with `systemd.` Press Ctrl + C to continue with the steps

```bash
journalctl -fu postgresql@17-main
```

Example of expected output:

```
minibolt systemd[1]: Starting PostgreSQL Cluster 17-main...
minibolt systemd[1]: Started PostgreSQL Cluster 17-main.
```

* List the clusters again to show the state

```bash
pg_lsclusters
```

Example of expected output:

```
Ver Cluster Port Status Owner     Data directory      Log file
16  main    5433 down   <unknown> /data/postgresdb/16 /var/log/postgresql/postgresql-16-main.log
17  main    5432 online <unknown> /data/postgresdb/17 /var/log/postgresql/postgresql-17-main.log
```

{% hint style="info" %}
Note how the version 17 cluster has come back into the status "online"
{% endhint %}

* Delete the version 16 (old and disused) cluster

```bash
sudo pg_dropcluster 16 main
```

* List again the clusters to check the correct deletion

```bash
pg_lsclusters
```

Example of expected output:

```
Ver Cluster Port Status Owner     Data directory      Log file
17  main    5432 online <unknown> /data/postgresdb/17 /var/log/postgresql/postgresql-17-main.log
```

{% hint style="info" %}
Note how it no longer appears in version 16 (old and disused) cluster
{% endhint %}

#### **Check the PostgreSQL server version in use**

* With the user `admin`, enter the psql (PostgreSQL CLI)

```bash
sudo -u postgres psql
```

* Enter the next command to get the server version

```sql
SELECT version();
```

Example of expected output:

<pre><code>                                                              version
-----------------------------------------------------------------------------------------------------------------------------------
PostgreSQL <a data-footnote-ref href="#user-content-fn-1">17.2</a> (Ubuntu 17.2-1.pgdg22.04+1) on x86_64-pc-linux-gnu, compiled by gcc (Ubuntu 11.4.0-1ubuntu1~22.04) 11.4.0, 64-bit
(1 row)
</code></pre>

{% hint style="info" %}
Check the previous version in use is now PostgreSQL 17.2 (the latest and current version of the PostgreSQL server at this moment)
{% endhint %}

* Come back to the `admin` user bash prompt

```sql
/q
```

* Delete unnecessary packages. Press "y" and enter, or directly enter when the prompt asks you

```bash
sudo apt autoremove
```

{% hint style="success" %}
That's it! You have updated PostgreSQL to the major version immediately higher
{% endhint %}

## Uninstall

### Uninstall the PostgreSQL package and configuration

* With user `admin`, stop and disable the PostgreSQL service

```bash
sudo systemctl stop postgresql && sudo systemctl disable postgresql
```

* Uninstall PostgreSQL using the apt package manager

```bash
sudo apt remove postgresql postgresql-* --purge
```

* Uninstall possible unnecessary dependencies

```bash
sudo apt autoremove
```

* Delete configuration files and data

{% code overflow="wrap" %}
```bash
sudo rm -rf /etc/postgresql/ && sudo rm -rf /etc/postgresql-common/ && sudo rm -rf /var/lib/postgresql/ && sudo rm -rf /var/log/postgresql/ && sudo rm -rf /usr/lib/postgresql/ && sudo rm -rf /usr/share/postgresql/
```
{% endcode %}

### Uninstall postgres user

* Delete the postgres user. Don't worry about `userdel: bitcoind mail spool (/var/mail/bitcoind) not found` output, the uninstall has been successful

```bash
sudo userdel -rf postgres
```

* Delete postgres group

```bash
sudo groupdel postgres
```

* Delete the complete `postgresdb` directory

```bash
sudo rm -rf /data/postgresdb
```

## Port reference

<table><thead><tr><th align="center">Port</th><th width="100">Protocol<select><option value="LjEORAGqFfCk" label="TCP" color="blue"></option><option value="0Q37zvNddYdf" label="SSL" color="blue"></option><option value="vDkMZIWjORx7" label="UDP" color="blue"></option></select></th><th align="center">Use</th></tr></thead><tbody><tr><td align="center">5432</td><td><span data-option="LjEORAGqFfCk">TCP</span></td><td align="center">Default relational DB port</td></tr></tbody></table>

[^1]: Check this
