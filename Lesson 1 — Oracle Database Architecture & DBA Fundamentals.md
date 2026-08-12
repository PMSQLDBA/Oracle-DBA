## Lesson 1 — Oracle Database Architecture & DBA Fundamentals

**Strictly Lesson 1 of 10.**

The objective of this lesson is to understand **what an Oracle Database actually consists of, how the instance works, how users connect, and how a DBA observes and controls it**.

---

### 1. Oracle Database vs Oracle Instance

This distinction is fundamental.

**Oracle Database** = physical files stored on disk.

A database consists primarily of:

* Data files
* Control files
* Online redo log files
* Temp files
* Archived redo logs, when archiving is enabled

**Oracle Instance** = memory + background processes that operate on the database.

Conceptually:

```text
                 ORACLE INSTANCE
        +-----------------------------+
        |          Memory             |
        |                             |
        |     SGA          PGA        |
        |                             |
        +-----------------------------+
        |     Background Processes    |
        +-----------------------------+
                    |
                    |
                    v
             ORACLE DATABASE
        +-----------------------------+
        | Data Files                  |
        | Control Files               |
        | Online Redo Logs            |
        | Temp Files                  |
        +-----------------------------+
```

A useful interview answer:

> **An Oracle instance is the combination of SGA and background processes. An Oracle database is the collection of physical files stored on disk.**

---

# 2. The SGA

**SGA = System Global Area**

It is shared memory associated with an Oracle instance.

Important components include:

### Database Buffer Cache

Caches data blocks read from data files.

```text
Data File
   ↓
Database Buffer Cache
   ↓
SQL operation
```

If a required block is already in the buffer cache, Oracle may not need to read it from disk.

---

### Shared Pool

The Shared Pool contains important structures used for SQL and PL/SQL processing.

Major areas include:

* Library cache
* Data dictionary cache

For example, when Oracle receives:

```sql
SELECT * FROM employees;
```

Oracle needs to parse and process the statement and determine how it should execute.

---

### Redo Log Buffer

Stores redo information before it is written to the online redo logs.

Conceptually:

```text
SQL change
   ↓
Redo information
   ↓
Redo Log Buffer
   ↓
LGWR
   ↓
Online Redo Log
```

This is essential to Oracle recovery.

---

# 3. PGA

**PGA = Program Global Area**

Unlike the SGA, PGA memory is associated with an individual server/background process.

It can contain things such as:

* Session-related information
* Sort-related memory
* Hash-related memory
* Private SQL areas

Conceptually:

```text
                 SGA
        +-------------------+
        | Shared Memory     |
        +-------------------+

       Process 1    Process 2
       +------+     +------+
       | PGA  |     | PGA  |
       +------+     +------+
```

Remember:

> **SGA = shared instance memory**
> **PGA = process-specific memory**

---

# 4. Oracle Background Processes

Oracle uses background processes to perform important database work.

Some important processes you should know as a DBA:

### DBWn — Database Writer

Writes modified database blocks from memory to data files.

```text
Buffer Cache
     |
     | dirty blocks
     v
   DBWn
     |
     v
Data Files
```

---

### LGWR — Log Writer

Writes redo information to online redo log files.

```text
Redo Log Buffer
      |
      v
     LGWR
      |
      v
Online Redo Logs
```

---

### CKPT — Checkpoint Process

Coordinates checkpoint-related activity and updates checkpoint information.

---

### SMON — System Monitor

Performs system-level recovery and other instance maintenance tasks.

---

### PMON — Process Monitor

Handles cleanup/recovery associated with failed processes and performs other process-monitoring functions.

---

### ARCn — Archiver

When the database is in `ARCHIVELOG` mode, archiver processes copy filled online redo logs to archived redo destinations.

```text
Online Redo Log
       |
       v
      ARCn
       |
       v
Archived Redo Log
```

These processes become particularly important in **Lesson 5 — Backup & Recovery**.

---

# 5. The Database Physical Files

A DBA must know the major physical files.

## Data Files

Data files contain database data.

For example:

```text
SYSTEM01.DBF
SYSAUX01.DBF
UNDOTBS01.DBF
USERS01.DBF
APP_DATA01.DBF
```

They belong to tablespaces.

---

## Control Files

Control files contain important structural information about the database.

They include information such as:

* Database name
* Database structure
* Data file information
* Redo log information
* Checkpoint information

You can see their locations:

```sql
SHOW PARAMETER control_files;
```

---

## Online Redo Logs

Redo logs record changes made to the database.

Example:

```text
Redo Group 1
   ├── Member 1
   └── Member 2

Redo Group 2
   ├── Member 1
   └── Member 2

Redo Group 3
   ├── Member 1
   └── Member 2
```

Multiple members provide redundancy.

---

# 6. Tablespaces

Oracle logically organizes database storage using **tablespaces**.

Think of the hierarchy as:

```text
Database
   |
   +-- Tablespaces
          |
          +-- Segments
                 |
                 +-- Extents
                        |
                        +-- Blocks
```

For example:

```text
USERS tablespace
      |
      +-- EMPLOYEES segment
      |
      +-- CUSTOMERS segment
```

A tablespace is associated with one or more data files.

```text
TABLESPACE
    |
    +------ DATAFILE
    |
    +------ DATAFILE
```

You'll work extensively with tablespaces in **Lesson 4**.

---

# 7. Oracle Blocks

The Oracle block is the basic unit of database storage.

Conceptually:

```text
Data File
+--------------------------------+
| Block | Block | Block | Block |
+--------------------------------+
```

Tables and indexes occupy Oracle blocks.

This becomes important later when learning:

* I/O
* Buffer cache
* Row storage
* Block corruption
* Performance
* RMAN block recovery

---

# 8. Schema

A **schema** is associated with a database user and contains that user's database objects.

For example:

```text
HR
 |
 +-- EMPLOYEES
 +-- DEPARTMENTS
 +-- JOBS
 +-- EMPLOYEES_IDX
 +-- procedures
 +-- views
```

You might create:

```sql
CREATE USER appuser IDENTIFIED BY "Password";
```

The user/schema relationship is important:

> A schema is essentially the collection of objects owned by a database user.

---

# 9. CDB and PDB

Modern Oracle Database administration requires understanding **multitenant architecture**.

Oracle's multitenant architecture uses:

**CDB = Container Database**

and

**PDB = Pluggable Database**

Conceptually:

```text
                 CDB
                  |
       +----------+----------+
       |          |          |
      CDB$ROOT   PDB1       PDB2
                  |          |
               Application Application
```

The root container is:

```text
CDB$ROOT
```

The PDBs contain application data and objects.

You can check the current container:

```sql
SHOW CON_NAME;
```

And list PDBs:

```sql
SHOW PDBS;
```

You'll spend considerably more time on PDB administration in **Lesson 3**.

---

# 10. Connect to Oracle

For a local administrative connection on Linux or Windows, one common method is:

```bash
sqlplus / as sysdba
```

You may see:

```text
SQL*Plus: Release ...
Connected to:
Oracle Database ...
```

`SYSDBA` is a powerful administrative privilege.

**Do not use `SYS`/`SYSDBA` for ordinary application activity.**

---

# 11. First DBA Commands

Once connected:

```sql
SELECT * FROM v$instance;
```

This provides instance information.

Try:

```sql
SELECT name, open_mode, database_role
FROM v$database;
```

You can check the Oracle release:

```sql
SELECT banner_full
FROM v$version;
```

And:

```sql
SHOW PARAMETER spfile;
```

---

# 12. Understand Database State

One of the most important DBA concepts is the database startup sequence.

```text
STARTUP NOMOUNT
      ↓
    MOUNT
      ↓
     OPEN
```

### NOMOUNT

Oracle starts the instance but does not mount the database.

Conceptually:

```text
Instance started
     ↓
SGA + processes
```

The control files are not yet used to mount the database.

---

### MOUNT

Oracle reads the control files and mounts the database.

```text
Instance
   +
Control Files
   ↓
MOUNT
```

The database is not yet open for normal user access.

---

### OPEN

Oracle opens the database for normal access.

```text
NOMOUNT
   ↓
MOUNT
   ↓
OPEN
```

Normal startup:

```sql
STARTUP;
```

---

# 13. Shutdown Modes

You should know the major shutdown modes.

### SHUTDOWN NORMAL

Waits for users to disconnect.

### SHUTDOWN IMMEDIATE

Terminates active transactions and disconnects sessions without waiting for users to voluntarily disconnect.

This is commonly used for controlled administrative shutdown.

```sql
SHUTDOWN IMMEDIATE;
```

### SHUTDOWN TRANSACTIONAL

Allows active transactions to complete before shutdown proceeds.

### SHUTDOWN ABORT

Immediately terminates the instance.

```sql
SHUTDOWN ABORT;
```

It should not be your normal shutdown method.

---

# 14. DBA Mental Model

At this point, memorize this architecture:

```text
                    CLIENT
                      |
                      v
                  LISTENER
                      |
                      v
                  DATABASE
                      |
             +--------+--------+
             |                 |
          INSTANCE          FILES
             |                 |
       +-----+-----+      +----+----+
       |           |      |         |
      SGA         PGA   Data      Control
       |                  Files     Files
       |
   +---+-------------------+
   |   |   |   |   |       |
 DBWn LGWR CKPT SMON PMON ARCn
```

This diagram is the foundation for the rest of the course.

---

# 15. Lesson 1 Practical Lab

If you already have Oracle installed, connect as SYSDBA and run these **in order**:

```sql
SELECT * FROM v$instance;
```

```sql
SELECT name,
       open_mode,
       database_role
FROM v$database;
```

```sql
SELECT banner_full
FROM v$version;
```

```sql
SHOW PARAMETER spfile;
```

```sql
SHOW PARAMETER control_files;
```

```sql
SHOW PARAMETER db_name;
```

```sql
SHOW PARAMETER compatible;
```

Then:

```sql
SHOW CON_NAME;
```

And:

```sql
SHOW PDBS;
```

Finally:

```sql
SELECT tablespace_name,
       status,
       contents
FROM dba_tablespaces;
```

---

# 16. What You Must Be Able to Explain

Before moving to Lesson 2, make sure you can answer these without memorizing blindly:

1. What is an Oracle instance?
2. What is an Oracle database?
3. What is the difference between SGA and PGA?
4. What does DBWn do?
5. What does LGWR do?
6. What is the purpose of redo?
7. What is a control file?
8. What is a data file?
9. What is a tablespace?
10. What is a schema?
11. What is CDB?
12. What is PDB?
13. What happens during `STARTUP NOMOUNT`?
14. What happens during `STARTUP MOUNT`?
15. What happens during `STARTUP OPEN`?
16. Difference between `SHUTDOWN IMMEDIATE` and `SHUTDOWN ABORT`.
17. How do you find the Oracle version?
18. How do you determine whether the database is open?
19. How do you find the control-file locations?
20. How do you list PDBs?

### Lesson 1 completion target

You should now have the mental model:

**Instance = memory + processes**

**Database = physical files**

**Tablespace → Segment → Extent → Block**

**CDB → PDB**

**NOMOUNT → MOUNT → OPEN**

**SQL → Instance → Database files**

That completes **Lesson 1/10**.

**Lesson 2/10 will be Oracle Database Installation — Windows + Linux + other supported OS**, including prerequisites, Oracle Home/Base, OS users/groups, software-only installation, DBCA, listener creation, silent installation, environment configuration, and post-installation verification.
