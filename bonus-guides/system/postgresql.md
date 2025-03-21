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

{% hint style="danger" %}
Status: Not tested on RaMiX
{% endhint %}

{% hint style="warning" %}
Difficulty: Medium
{% endhint %}

<figure><img src="../../.gitbook/assets/PostgreSQL-Logo-white.png" alt="" width="563"><figcaption></figcaption></figure>

## Installation

### Install PostgreSQL using the apt package manager

* With user `admin`, update and upgrade your OS

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

* Check the correct installation of the PostgreSQL

```bash
psql -V
```

**Example** of expected output:

```
psql (PostgreSQL) 15.3 (Ubuntu 15.3-1.pgdg22.04+1)
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
May 31 13:51:11 ramix systemd[1]: Finished PostgreSQL RDBMS.
```

* And the sub-instance and specific cluster logs. You can exit monitoring at any time with `Ctrl-C`

```bash
journalctl -fu postgresql@17-main
```

**Example** of expected output:

```
May 31 13:51:18 ramix systemd[1]: Starting PostgreSQL Cluster 17-main...
May 31 13:51:21 ramix systemd[1]: Started PostgreSQL Cluster 17-main.
```

### Create data folder

* Create the dedicated PostgreSQL data folder

```bash
sudo mkdir /data/postgresdb/17
```

* Assign as the owner to the `postgres` user

<pre class="language-bash"><code class="lang-bash"><strong>sudo chown postgres:postgres /data/postgresdb/17
</strong></code></pre>

* Assign permissions of the data folder only to the `postgres` user

<pre class="language-bash"><code class="lang-bash"><strong>sudo chmod -R 700 /data/postgresdb/17
</strong></code></pre>

* With user `postgres`, create a new cluster on the dedicated folder

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

* Edit the PostgreSQL data directory on configuration, to redirect the store to the new location

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

* You can monitor the PostgreSQL sub-instance by the systemd journal and check log output. You can exit monitoring at any time with `Ctrl-C`

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

* You can check if the cluster is on status "online" by

```bash
pg_lsclusters
```

Expected output:

```
Ver Cluster Port Status Owner    Data directory       Log file
17  main    5432 online postgres /data/postgresdb/17  /var/log/postgresql/postgresql-17-main.log
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
Congrats! You have PostgreSQL ready to use as a database backend by another software
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
Type `\q` command and enter to exit PostgreSQL CLI and exit to come back to the `admin` user
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

#### List the global existing databases

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

#### List tables inside of a specific database

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

#### **Detele** a specific database

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

#### Delete a table inside of a specific database

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

Example:

```sql
DROP TABLE towerclientdb_kv;
```

## Upgrade

The latest release can be found on the [official PostgreSQL web page](https://www.postgresql.org/ftp/source/).

* To upgrade, type this command

```bash
sudo apt update && sudo apt full-upgrade
```

## Uninstall

### Uninstall PostgreSQL package and configuration

* With user `admin`, stop and disable the postgres service

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
