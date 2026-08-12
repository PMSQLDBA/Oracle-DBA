# Lesson 4/10 — Core Oracle DBA Administration

**Strictly Lesson 4 of 10.**

This is where you begin doing the **daily work of an Oracle DBA**: managing users, privileges, schemas, tablespaces, datafiles, undo, redo, quotas, and database health.

Oracle's official 19c documentation organizes these areas under database administration, security, storage, and managing database structures. The commands below are representative 19c administration commands; always verify syntax and behavior against your exact Oracle release before using them in production.

---

# 4.1 The DBA's Core Responsibilities

After installation and configuration, a DBA continuously manages:

```text
Users
  ↓
Security
  ↓
Schemas
  ↓
Tablespaces
  ↓
Datafiles
  ↓
Undo
  ↓
Redo
  ↓
Storage
  ↓
Database Health
```

A production DBA should be able to answer at any moment:

> Who is connected?
> What are they allowed to do?
> Where is their data stored?
> How much storage remains?
> Is undo healthy?
> Are redo logs healthy?
> Are there blocked sessions?
> Is the database approaching a capacity limit?

---

# 4.2 Users

An Oracle database user is an account that can authenticate to the database.

Create one:

```sql
CREATE USER appuser
IDENTIFIED BY "StrongPassword";
```

Initially, the user generally cannot do much.

Give login capability:

```sql
GRANT CREATE SESSION TO appuser;
```

Now the user can establish a database session.

---

# 4.3 User Authentication

Check users:

```sql
SELECT username,
       account_status,
       default_tablespace,
       temporary_tablespace
FROM dba_users
ORDER BY username;
```

Important columns:

```text
USERNAME
ACCOUNT_STATUS
DEFAULT_TABLESPACE
TEMPORARY_TABLESPACE
```

Possible account states include:

```text
OPEN
LOCKED
EXPIRED
EXPIRED & LOCKED
```

---

# 4.4 Unlocking a User

If an account is locked:

```sql
ALTER USER appuser ACCOUNT UNLOCK;
```

You can change a password:

```sql
ALTER USER appuser
IDENTIFIED BY "NewStrongPassword";
```

In production, don't expose passwords in scripts, shell history, tickets, or chat.

---

# 4.5 Roles

Instead of granting dozens of privileges individually to every user, use roles.

Create:

```sql
CREATE ROLE app_role;
```

Grant privileges to the role:

```sql
GRANT CREATE SESSION TO app_role;
```

Then:

```sql
GRANT app_role TO appuser;
```

Conceptually:

```text
                APP_ROLE
               /   |   \
              /    |    \
          Priv1  Priv2  Priv3
             |
             v
          APPUSER
```

This makes privilege management much easier.

---

# 4.6 System Privileges vs Object Privileges

This distinction is essential.

### System privilege

Allows a user to perform a type of database operation.

Example:

```sql
GRANT CREATE TABLE TO appuser;
```

### Object privilege

Allows access to a particular object.

Example:

```sql
GRANT SELECT
ON hr.employees
TO appuser;
```

Think:

```text
SYSTEM PRIVILEGE
      |
      +-- "What kind of operation can I perform?"

OBJECT PRIVILEGE
      |
      +-- "Which specific object can I access?"
```

---

# 4.7 Least Privilege

A professional DBA should follow:

> **Give users only the privileges they actually require.**

Avoid giving application accounts excessive privileges such as:

```sql
GRANT DBA TO appuser;
```

unless there is a legitimate administrative requirement.

A typical application architecture is closer to:

```text
Application User
      |
      +-- CREATE SESSION
      |
      +-- SELECT on required tables
      |
      +-- INSERT/UPDATE/DELETE where required
```

rather than:

```text
Application User
      |
      +-- DBA
```

---

# 4.8 Checking Privileges

User system privileges:

```sql
SELECT *
FROM dba_sys_privs
WHERE grantee = 'APPUSER';
```

Roles:

```sql
SELECT *
FROM dba_role_privs
WHERE grantee = 'APPUSER';
```

Object privileges:

```sql
SELECT *
FROM dba_tab_privs
WHERE grantee = 'APPUSER';
```

These views become very useful during security troubleshooting.

---

# 4.9 Revoking Privileges

If a privilege is no longer required:

```sql
REVOKE CREATE TABLE
FROM appuser;
```

For object privileges:

```sql
REVOKE SELECT
ON hr.employees
FROM appuser;
```

---

# 4.10 Schemas

Remember the concept from Lesson 1:

```text
USER
  |
  v
SCHEMA
  |
  +-- TABLE
  +-- INDEX
  +-- VIEW
  +-- PROCEDURE
  +-- PACKAGE
  +-- SEQUENCE
```

If you create:

```sql
CREATE USER APPUSER IDENTIFIED BY "Password";
```

Oracle creates the user account and associated schema namespace.

If `APPUSER` creates:

```sql
CREATE TABLE employees (...);
```

the object belongs to:

```text
APPUSER.EMPLOYEES
```

---

# 4.11 Tablespaces

Now we move into storage.

A tablespace is a logical storage container.

```text
DATABASE
   |
   +-- SYSTEM
   +-- SYSAUX
   +-- UNDOTBS1
   +-- TEMP
   +-- USERS
   +-- APP_DATA
```

Query them:

```sql
SELECT tablespace_name,
       status,
       contents,
       extent_management,
       segment_space_management
FROM dba_tablespaces
ORDER BY tablespace_name;
```

---

# 4.12 Tablespace Types

Important categories:

### Permanent

Stores normal database objects.

```text
USERS
APP_DATA
```

### Temporary

Used for temporary operations.

```text
TEMP
```

### Undo

Stores undo information.

```text
UNDOTBS1
```

You need to understand all three.

---

# 4.13 Datafiles

Tablespaces are physically implemented using datafiles.

Example:

```text
APP_DATA
   |
   +-- app_data01.dbf
   +-- app_data02.dbf
```

Find datafiles:

```sql
SELECT file_id,
       file_name,
       tablespace_name,
       bytes/1024/1024 AS size_mb,
       autoextensible
FROM dba_data_files
ORDER BY tablespace_name, file_id;
```

---

# 4.14 Temporary Files

Temporary tablespaces use temporary files.

Check:

```sql
SELECT file_id,
       file_name,
       tablespace_name,
       bytes/1024/1024 AS size_mb
FROM dba_temp_files;
```

---

# 4.15 Creating a Tablespace

Example:

```sql
CREATE TABLESPACE app_data
DATAFILE '/u02/oradata/ORCL/app_data01.dbf'
SIZE 1G
AUTOEXTEND ON
NEXT 100M
MAXSIZE 10G;
```

This means:

```text
Initial size = 1 GB
Autoextend = ON
Growth = 100 MB
Maximum = 10 GB
```

**Production warning:** blindly enabling unlimited autoextend can eventually consume an entire filesystem. Storage growth must be planned.

---

# 4.16 Adding a Datafile

Suppose:

```text
APP_DATA
```

needs more space.

You can add a datafile:

```sql
ALTER TABLESPACE app_data
ADD DATAFILE
'/u02/oradata/ORCL/app_data02.dbf'
SIZE 1G;
```

Now:

```text
APP_DATA
 |
 +-- app_data01.dbf
 |
 +-- app_data02.dbf
```

---

# 4.17 Resizing a Datafile

For example:

```sql
ALTER DATABASE DATAFILE
'/u02/oradata/ORCL/app_data01.dbf'
RESIZE 2G;
```

But don't assume every datafile can safely be resized downward.

Oracle cannot shrink a datafile below the space occupied by objects located toward the end of that file.

This is why capacity analysis comes before resizing.

---

# 4.18 Autoextend

Check:

```sql
SELECT file_name,
       bytes/1024/1024 AS size_mb,
       autoextensible,
       increment_by,
       maxbytes/1024/1024 AS max_mb
FROM dba_data_files;
```

You should understand:

```text
Current Size
Autoextend?
Next Increment
Maximum Size
```

---

# 4.19 Tablespace Usage

One of your most important daily DBA checks is tablespace capacity.

A simple approach:

```sql
SELECT tablespace_name,
       used_percent
FROM dba_tablespace_usage_metrics
ORDER BY used_percent DESC;
```

You might discover:

```text
APP_DATA       91%
USERS          62%
SYSTEM         43%
SYSAUX         51%
```

A DBA should investigate high usage **before** it becomes an outage.

---

# 4.20 Capacity Planning

Don't think:

> "The tablespace is 90% full, therefore add 1 GB."

Instead:

```text
Current growth
     ↓
Daily growth rate
     ↓
Available filesystem/storage
     ↓
Autoextend settings
     ↓
Expected workload
     ↓
Required capacity
```

The question is:

> **When will this resource run out?**

not merely:

> **How full is it right now?**

---

# 4.21 Default Tablespace

Users can have a default tablespace.

Check:

```sql
SELECT username,
       default_tablespace,
       temporary_tablespace
FROM dba_users;
```

Create a user with a specific default:

```sql
CREATE USER appuser
IDENTIFIED BY "StrongPassword"
DEFAULT TABLESPACE app_data
TEMPORARY TABLESPACE temp;
```

This is preferable to allowing application objects to accidentally end up in an inappropriate tablespace.

---

# 4.22 Quotas

A user may have a quota on a tablespace.

Example:

```sql
ALTER USER appuser
QUOTA 5G ON app_data;
```

Or:

```sql
ALTER USER appuser
QUOTA UNLIMITED ON app_data;
```

Be careful with `UNLIMITED`.

You can inspect quotas:

```sql
SELECT username,
       tablespace_name,
       max_bytes
FROM dba_ts_quotas
ORDER BY username;
```

---

# 4.23 Temporary Tablespaces

Temporary space is used for operations such as:

* Sorts
* Hash operations
* Temporary segments
* Certain query operations

Check:

```sql
SELECT username,
       temporary_tablespace
FROM dba_users;
```

Check tempfiles:

```sql
SELECT tablespace_name,
       file_name,
       bytes/1024/1024 AS size_mb
FROM dba_temp_files;
```

A temporary space problem can cause seemingly unrelated SQL operations to fail.

---

# 4.24 Undo

Undo is one of the most important Oracle concepts.

Undo allows Oracle to:

* Roll back transactions
* Provide read consistency
* Support certain recovery operations
* Maintain consistent query results

Conceptually:

```text
Transaction
     |
     +-- Changes data
     |
     +-- Undo information
             |
             v
         Undo Tablespace
```

Check:

```sql
SHOW PARAMETER undo;
```

You may see:

```text
undo_management
undo_tablespace
```

---

# 4.25 Undo Tablespace

Find undo tablespaces:

```sql
SELECT tablespace_name,
       status,
       contents
FROM dba_tablespaces
WHERE contents = 'UNDO';
```

Current undo tablespace:

```sql
SHOW PARAMETER undo_tablespace;
```

---

# 4.26 Redo

Undo and redo are **not the same thing**.

This is a common interview question.

### Undo

Helps Oracle reverse changes and maintain read consistency.

### Redo

Records changes so Oracle can reproduce them during recovery.

Think:

```text
CHANGE
  |
  +---- UNDO
  |
  +---- REDO
```

---

# 4.27 Online Redo Logs

Check:

```sql
SELECT group#,
       thread#,
       sequence#,
       bytes/1024/1024 AS size_mb,
       members,
       status
FROM v$log
ORDER BY group#;
```

Find members:

```sql
SELECT group#,
       member
FROM v$logfile
ORDER BY group#;
```

Example:

```text
GROUP 1
  |
  +-- redo01a.log
  +-- redo01b.log
```

---

# 4.28 Redo Groups and Members

Think:

```text
Redo Group 1
  |
  +-- Member A
  +-- Member B

Redo Group 2
  |
  +-- Member A
  +-- Member B
```

Multiple members provide multiplexing.

The goal is redundancy.

If one redo member is lost but another member in the same group remains available, Oracle can continue operating depending on the exact failure.

---

# 4.29 Redo Log Status

Common statuses include:

```text
CURRENT
ACTIVE
INACTIVE
```

You should **never randomly delete redo log files**.

Redo management requires understanding:

* Groups
* Members
* Status
* Checkpoints
* Archive status
* Database mode
* Recovery requirements

We'll handle detailed redo administration and recovery implications in **Lesson 5**.

---

# 4.30 Log Switch

Oracle periodically switches from one redo log group to another.

Conceptually:

```text
Group 1
   ↓
Group 2
   ↓
Group 3
   ↓
Group 1
   ↓
...
```

You can observe activity with:

```sql
SELECT group#,
       sequence#,
       status
FROM v$log
ORDER BY group#;
```

A DBA should understand excessive log switching as a potential configuration/performance issue.

---

# 4.31 Database Sessions

Daily administration requires knowing who is connected.

Run:

```sql
SELECT sid,
       serial#,
       username,
       status,
       machine,
       program
FROM v$session
WHERE username IS NOT NULL
ORDER BY username;
```

You may see:

```text
SID
SERIAL#
USERNAME
STATUS
MACHINE
PROGRAM
```

This is extremely useful for troubleshooting.

---

# 4.32 Active Sessions

For active sessions:

```sql
SELECT sid,
       serial#,
       username,
       status,
       event,
       sql_id
FROM v$session
WHERE status = 'ACTIVE';
```

Now you can begin asking:

> What is this session doing?

That leads us toward Lesson 6.

---

# 4.33 Blocking Sessions

A classic production problem:

```text
Session A
    |
    +-- holds lock
            |
            v
        Session B
            |
            +-- waiting
```

A basic query:

```sql
SELECT blocking_session,
       sid,
       serial#,
       username,
       event
FROM v$session
WHERE blocking_session IS NOT NULL;
```

If this returns rows, investigate carefully.

**Never kill a session simply because it appears in a blocking query.**

First determine:

* Who owns the transaction?
* What SQL is running?
* Why is the lock held?
* Is it legitimate?
* What will happen if you terminate it?

---

# 4.34 Killing a Session

In appropriate circumstances:

```sql
ALTER SYSTEM KILL SESSION 'sid,serial#';
```

For example:

```sql
ALTER SYSTEM KILL SESSION '123,4567';
```

The exact action and consequences depend on the situation.

Production rule:

> **Killing a session is a corrective action, not a troubleshooting methodology.**

---

# 4.35 Invalid Objects

After changes, upgrades, imports, or other operations, you may encounter invalid objects.

Check:

```sql
SELECT owner,
       object_type,
       COUNT(*) AS invalid_count
FROM dba_objects
WHERE status = 'INVALID'
GROUP BY owner, object_type
ORDER BY owner, object_type;
```

You can inspect specific objects:

```sql
SELECT owner,
       object_name,
       object_type
FROM dba_objects
WHERE status = 'INVALID'
ORDER BY owner, object_type;
```

We'll deal with recompilation and post-upgrade validation in Lesson 9.

---

# 4.36 Database Health Check

A basic DBA health check should cover:

```text
Database status
        ↓
Instance status
        ↓
PDB status
        ↓
Listener
        ↓
Tablespace usage
        ↓
Filesystem/storage
        ↓
Undo
        ↓
Redo
        ↓
Sessions
        ↓
Blocking
        ↓
Invalid objects
        ↓
Backup status
        ↓
Alert log
```

You should eventually automate these checks.

---

# 4.37 A Basic Health-Check Script

For learning, start with:

```sql
SELECT name,
       open_mode,
       database_role
FROM v$database;
```

```sql
SELECT instance_name,
       status
FROM v$instance;
```

```sql
SELECT tablespace_name,
       used_percent
FROM dba_tablespace_usage_metrics
ORDER BY used_percent DESC;
```

```sql
SELECT username,
       account_status
FROM dba_users
ORDER BY username;
```

```sql
SELECT COUNT(*) AS active_sessions
FROM v$session
WHERE status = 'ACTIVE';
```

```sql
SELECT COUNT(*) AS invalid_objects
FROM dba_objects
WHERE status = 'INVALID';
```

These aren't sufficient for a production monitoring system, but they teach the correct categories.

---

# 4.38 Windows DBA Administration

The database commands remain essentially the same.

The OS layer differs.

For example:

```powershell
Get-Service *Oracle*
```

Check processes:

```powershell
Get-Process | Where-Object {$_.ProcessName -like "*oracle*"}
```

Check disks:

```powershell
Get-Volume
```

Your mental model should be:

```text
Oracle SQL
   +
Windows administration
   =
Windows DBA
```

---

# 4.39 Linux DBA Administration

Linux:

```bash
df -h
```

Disk utilization:

```bash
df -h /u01
```

Processes:

```bash
ps -ef | grep ora_
```

Memory:

```bash
free -h
```

Filesystem:

```bash
lsblk
```

Oracle DBA work requires both:

```text
Linux knowledge
       +
Oracle knowledge
```

---

# 4.40 Production Storage Failure Example

Imagine the application team reports:

> "The database is throwing errors."

You discover:

```text
APP_DATA = 99%
```

But don't immediately add space.

Investigate:

```text
Tablespace
   ↓
Datafiles
   ↓
Filesystem
   ↓
Actual free space
   ↓
Growth rate
   ↓
Largest segments
   ↓
Application behavior
```

For example:

```sql
SELECT owner,
       segment_name,
       segment_type,
       bytes/1024/1024 AS size_mb
FROM dba_segments
ORDER BY bytes DESC
FETCH FIRST 20 ROWS ONLY;
```

Now you can determine what is consuming storage.

---

# 4.41 Lesson 4 Lab

Perform these operations in your lab database.

### Lab A — Create a user

```sql
CREATE USER appuser
IDENTIFIED BY "StrongPassword";
```

```sql
GRANT CREATE SESSION TO appuser;
```

Verify:

```sql
SELECT username,
       account_status,
       default_tablespace,
       temporary_tablespace
FROM dba_users
WHERE username = 'APPUSER';
```

---

### Lab B — Create a role

```sql
CREATE ROLE app_role;
```

```sql
GRANT CREATE SESSION TO app_role;
```

```sql
GRANT app_role TO appuser;
```

Check:

```sql
SELECT *
FROM dba_role_privs
WHERE grantee = 'APPUSER';
```

---

### Lab C — Create a tablespace

Use a path appropriate for your OS:

```sql
CREATE TABLESPACE app_data
DATAFILE '/u02/oradata/ORCL/app_data01.dbf'
SIZE 500M
AUTOEXTEND ON
NEXT 50M
MAXSIZE 2G;
```

**Do not execute this exact path on Windows.** The datafile path must exist and be appropriate for your Windows installation.

---

### Lab D — Assign tablespace

```sql
ALTER USER appuser
DEFAULT TABLESPACE app_data;
```

Then:

```sql
ALTER USER appuser
QUOTA 1G ON app_data;
```

---

### Lab E — Check storage

```sql
SELECT tablespace_name,
       used_percent
FROM dba_tablespace_usage_metrics
ORDER BY used_percent DESC;
```

---

### Lab F — Check datafiles

```sql
SELECT tablespace_name,
       file_name,
       bytes/1024/1024 AS size_mb,
       autoextensible
FROM dba_data_files
ORDER BY tablespace_name;
```

---

### Lab G — Check undo

```sql
SHOW PARAMETER undo;
```

---

### Lab H — Check redo

```sql
SELECT group#,
       thread#,
       sequence#,
       bytes/1024/1024 AS size_mb,
       members,
       status
FROM v$log
ORDER BY group#;
```

---

### Lab I — Check sessions

```sql
SELECT sid,
       serial#,
       username,
       status,
       machine,
       program
FROM v$session
WHERE username IS NOT NULL
ORDER BY username;
```

---

### Lab J — Health check

Run:

```sql
SELECT name, open_mode, database_role
FROM v$database;
```

```sql
SELECT instance_name, status
FROM v$instance;
```

```sql
SELECT tablespace_name, used_percent
FROM dba_tablespace_usage_metrics
ORDER BY used_percent DESC;
```

```sql
SELECT COUNT(*) AS invalid_objects
FROM dba_objects
WHERE status = 'INVALID';
```

---

# 4.42 Lesson 4 Interview Questions

You should be able to answer these before proceeding:

1. What is a database user?
2. What is a schema?
3. What is a role?
4. Difference between system and object privileges?
5. What is least privilege?
6. How do you lock/unlock a user?
7. How do you change a user's password?
8. What is a tablespace?
9. What is a datafile?
10. Difference between tablespace and datafile?
11. What is a temporary tablespace?
12. What is an undo tablespace?
13. What is a quota?
14. What is autoextend?
15. Why can unlimited autoextend be dangerous?
16. How do you find tablespace utilization?
17. How do you find datafile locations?
18. What is undo?
19. What is redo?
20. Difference between redo and undo?
21. What is a redo log group?
22. What is a redo log member?
23. Why multiplex redo logs?
24. How do you find active sessions?
25. How do you identify blocking sessions?
26. Why shouldn't you immediately kill a blocking session?
27. How do you find invalid objects?
28. What should a daily DBA health check contain?

---

# 4.43 Lesson 4 — The Mental Model

By now your Oracle environment should look like:

```text
                    ORACLE DATABASE
                           |
        +------------------+------------------+
        |                  |                  |
      USERS             STORAGE            REDO
        |                  |                  |
     Roles             Tablespaces        Log Groups
     Privileges             |                  |
     Profiles            Datafiles          Members
        |                  |
        v                  v
    SCHEMAS             Segments
                           |
                         Extents
                           |
                         Blocks

                    +
                  UNDO
                    |
              Undo Tablespace

                    +
                SESSIONS
                    |
              Active/Waiting
                    |
                Blocking
```

And the DBA's operational cycle is:

```text
MONITOR
   ↓
IDENTIFY
   ↓
ANALYZE
   ↓
CHANGE
   ↓
VERIFY
   ↓
DOCUMENT
```

That is the foundation of real DBA work.

**Lesson 4/10 complete.**

Next is **Lesson 5/10 — Backup & Recovery with RMAN**, where we'll cover **ARCHIVELOG, RMAN architecture, backup types, backup strategy, restore vs recovery, control-file recovery, datafile recovery, point-in-time recovery, corruption, retention policies, validation, and disaster-recovery scenarios**.
