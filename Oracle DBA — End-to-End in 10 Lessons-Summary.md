
I’ll base the technical direction on **Oracle’s official documentation**, not third-party tutorials. 

Oracle’s current documentation includes Oracle AI Database 26ai as well as the widely used 19c documentation, and Oracle provides platform-specific installation/upgrade guides for Windows, Linux, Solaris, AIX, HP-UX, and other platforms. ([Oracle Documentation][1])

# Oracle DBA — End-to-End in 10 Lessons

### Course philosophy

By the end of the 10 lessons, you should be able to understand this lifecycle:

**Plan → Install → Configure → Administer → Secure → Backup → Recover → Patch → Migrate → Upgrade → Operate**

We will use **Oracle Database 19c concepts extensively for hands-on administration**, while pointing out where newer Oracle AI Database 26ai procedures differ. 

Oracle's documentation explicitly covers installation, upgrade, migration, patching, RAC, Grid Infrastructure, Windows and Linux separately. ([Oracle Documentation][1])

---

## Lesson 1 — Oracle Database Architecture & DBA Fundamentals

**Goal:** Understand what you are administering before touching an installation.

### 1.1 Oracle architecture

Learn:

* Oracle Database vs Oracle Instance
* Instance

  * SGA
  * PGA
  * Background processes
* Database

  * Data files
  * Control files
  * Online redo logs
  * Temp files
* Logical storage

  * Tablespaces
  * Segments
  * Extents
  * Blocks
* Database schemas
* Users and privileges
* CDB / PDB architecture
* Multitenant architecture
* Non-CDB vs CDB/PDB
* `SYS`, `SYSTEM`, `SYSDBA`
* Oracle processes
* Memory architecture
* Database startup/shutdown

### 1.2 Important DBA commands

You will learn:

```sql
SELECT * FROM v$instance;

SELECT * FROM v$database;

SELECT name, open_mode, database_role
FROM v$database;

SELECT banner_full
FROM v$version;
```

Oracle documents `V$VERSION`/`BANNER_FULL` as a way to identify the installed database release and update level. ([Oracle Documentation][2])

### 1.3 Startup and shutdown

Master:

```sql
STARTUP;
STARTUP NOMOUNT;
STARTUP MOUNT;
STARTUP OPEN;

SHUTDOWN IMMEDIATE;
SHUTDOWN TRANSACTIONAL;
SHUTDOWN NORMAL;
SHUTDOWN ABORT;
```

### 1.4 DBA operating-system knowledge

Linux:

```bash
ps
top
df -h
free -h
mount
lsblk
```

Windows:

```powershell
Get-Service
Get-Process
Get-Volume
```

### Practical outcome

You should be able to look at an Oracle server and explain:

> **What is running, where the database files are, what instance is attached to the database, which containers exist, and what state the database is in.**

---

# Lesson 2 — Oracle Database Installation: Windows + Linux + Other OS

**Goal:** Install Oracle Database correctly instead of simply clicking "Next."

Oracle's installation documentation covers both GUI and silent/response-file installation. Oracle Universal Installer (OUI) is used for installation, while DBCA can create/configure the database. ([Oracle Documentation][3])

### 2.1 Installation planning

Learn:

* Hardware requirements
* OS requirements
* Kernel/package prerequisites
* Memory
* Swap
* Disk layout
* File systems
* Oracle Base
* Oracle Home
* Oracle Inventory
* Database files
* Listener
* OS groups/users
* Environment variables
* Hostname
* DNS
* `/etc/hosts`
* Windows services
* Windows Oracle Home User

### 2.2 Linux installation

Understand:

```text
Oracle Base
    |
    +-- Oracle Home
    |
    +-- oraInventory
    |
    +-- database files
```

Typical environment:

```bash
export ORACLE_BASE=/u01/app/oracle
export ORACLE_HOME=$ORACLE_BASE/product/19c/dbhome_1
export ORACLE_SID=ORCL
export PATH=$ORACLE_HOME/bin:$PATH
```

Learn:

* Installing prerequisites
* Creating `oracle` user
* Creating `oinstall` / `dba`
* Kernel parameters
* Resource limits
* Installing software
* `runInstaller`
* Root scripts
* Listener creation
* DBCA
* Verification

Oracle specifically documents software-only installation followed by DBCA, and also documents silent installation. ([Oracle Documentation][3])

### 2.3 Windows installation

Learn:

* Oracle Home
* Oracle Base
* Oracle Home User
* Windows services
* Listener
* `ORACLE_SID`
* Registry considerations
* DBCA
* SQL*Plus
* Windows service startup/shutdown

Typical verification:

```cmd
sqlplus / as sysdba
```

Then:

```sql
SELECT instance_name, status
FROM v$instance;
```

### 2.4 Other operating systems

We will understand the differences for:

* Linux
* Windows
* Solaris
* IBM AIX
* HP-UX
* Other Oracle-supported UNIX platforms

Oracle publishes separate platform-specific installation guides rather than treating every OS as identical. ([Oracle Documentation][1])

### Practical outcome

You can install a **clean Oracle environment on Windows and Linux**, understand the corresponding OS differences, and verify the installation.

---

# Lesson 3 — Database Configuration & Networking

**Goal:** Turn an installed Oracle Database into a properly configured database service.

### 3.1 Database configuration

Learn:

* Initialization parameters
* SPFILE
* PFILE
* `COMPATIBLE`
* Memory parameters
* Processes/sessions
* DB files
* Control files
* Redo logs
* Undo
* Temporary tablespace
* Archive logging

Commands:

```sql
SHOW PARAMETER;

SHOW PARAMETER spfile;

SHOW PARAMETER compatible;

SHOW PARAMETER memory;
```

### 3.2 Listener

Understand:

```text
Client
   |
   v
Listener
   |
   v
Database Service
   |
   v
Instance
```

Learn:

* `listener.ora`
* `tnsnames.ora`
* `sqlnet.ora`
* Static registration
* Dynamic registration
* Service names
* Listener status

Commands:

```bash
lsnrctl status
lsnrctl start
lsnrctl stop
lsnrctl reload
```

### 3.3 Connectivity troubleshooting

Master:

```bash
tnsping ORCL
```

and:

```sql
SELECT instance_name, status
FROM v$instance;
```

Learn to diagnose:

* ORA-12514
* ORA-12541
* ORA-12154
* ORA-01017
* Listener unavailable
* Service not registered
* DNS/hostname problems
* Firewall problems

### 3.4 CDB/PDB administration

Learn:

```sql
SHOW CON_NAME;

SHOW PDBS;

ALTER PLUGGABLE DATABASE pdb1 OPEN;

ALTER PLUGGABLE DATABASE pdb1 SAVE STATE;
```

### Practical outcome

You can take an installed database and make it **usable by applications and DBAs**.

---

# Lesson 4 — Core DBA Administration: Users, Storage, Tablespaces & Daily Operations

**Goal:** Perform everyday DBA work.

### 4.1 User administration

Learn:

```sql
CREATE USER appuser IDENTIFIED BY password;

ALTER USER appuser ACCOUNT UNLOCK;

DROP USER appuser CASCADE;
```

Privileges:

```sql
GRANT CREATE SESSION TO appuser;

GRANT SELECT ON hr.employees TO appuser;

REVOKE CREATE SESSION FROM appuser;
```

Roles:

```sql
CREATE ROLE app_role;

GRANT CREATE SESSION TO app_role;

GRANT app_role TO appuser;
```

### 4.2 Tablespaces

Learn:

* Permanent tablespaces
* Temporary tablespaces
* Undo tablespaces
* Locally managed tablespaces
* Automatic segment space management
* Datafiles
* Autoextend
* Bigfile tablespaces

Example:

```sql
CREATE TABLESPACE app_data
DATAFILE '/u01/oradata/ORCL/app_data01.dbf'
SIZE 1G
AUTOEXTEND ON
NEXT 100M
MAXSIZE 10G;
```

### 4.3 Storage administration

Learn:

```sql
SELECT tablespace_name,
       status,
       contents
FROM dba_tablespaces;
```

and:

```sql
SELECT file_name,
       bytes/1024/1024 AS mb,
       autoextensible
FROM dba_data_files;
```

### 4.4 Redo and undo

Master:

* Online redo logs
* Redo members
* Log groups
* Log switches
* Archived redo
* Undo
* Undo retention

### 4.5 Daily DBA health checks

You will create your own DBA health-check routine:

```text
Database status
Listener status
Tablespace usage
Filesystem usage
Alert log
Invalid objects
Blocking sessions
Long-running sessions
Archive destination
Backup status
Database errors
```

### Practical outcome

You can perform the majority of **normal day-to-day Oracle DBA operations**.

---

# Lesson 5 — Backup, Recovery, RMAN & Disaster Recovery

**Goal:** Learn the most important DBA responsibility: **recovering the database when something goes wrong.**

### 5.1 Backup concepts

Learn:

* Physical backup
* Logical backup
* Full backup
* Incremental backup
* Archive logs
* Online backup
* Offline backup
* Consistent backup
* Recovery
* Restore
* Point-in-time recovery

### 5.2 RMAN

Master:

```bash
rman target /
```

Then:

```rman
BACKUP DATABASE;
```

Archive logs:

```rman
BACKUP ARCHIVELOG ALL;
```

Backup database plus archive logs:

```rman
BACKUP DATABASE PLUS ARCHIVELOG;
```

### 5.3 Restore vs recovery

You must clearly understand:

**RESTORE**

> Bring database files back from backup.

**RECOVER**

> Apply redo/archive information to make those files current.

### 5.4 Recovery scenarios

We will practice:

* Lost datafile
* Lost control file
* Lost redo log
* Deleted table
* Corrupted block
* Complete database failure
* Point-in-time recovery
* Incomplete recovery

### 5.5 RMAN validation

Learn:

```rman
LIST BACKUP;

LIST BACKUP SUMMARY;

REPORT OBSOLETE;

CROSSCHECK BACKUP;

VALIDATE DATABASE;
```

### Practical outcome

You should be able to answer the most important production question:

> **"The database is down/corrupted. What exactly do I do?"**

---

# Lesson 6 — Performance, Monitoring, Security & Troubleshooting

**Goal:** Move from "DBA who manages databases" to **production DBA**.

### 6.1 Performance fundamentals

Learn:

* CPU
* Memory
* I/O
* Wait events
* Sessions
* Locks
* Blocking
* Latches
* Enqueues
* SQL performance
* Execution plans
* AWR
* ASH
* ADDM

### 6.2 Session monitoring

```sql
SELECT sid,
       serial#,
       username,
       status,
       event
FROM v$session
WHERE username IS NOT NULL;
```

### 6.3 Blocking sessions

Understand:

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

Learn how to identify and resolve blocking safely.

### 6.4 Security

Learn:

* Authentication
* Authorization
* Privileges
* Roles
* Profiles
* Password policies
* Auditing
* Least privilege
* `SYSDBA`
* Network security
* Encryption concepts

### 6.5 Troubleshooting methodology

We will use:

```text
Symptom
   ↓
Collect evidence
   ↓
Identify affected component
   ↓
Check alert log
   ↓
Check database views
   ↓
Check OS
   ↓
Check listener/network
   ↓
Determine root cause
   ↓
Correct
   ↓
Validate
   ↓
Document
```

### Practical outcome

You can troubleshoot real production problems rather than simply restart the database.

---

# Lesson 7 — Oracle Patching: RU, MRP, OPatch, OPatchAuto & Production Strategy

**Goal:** Learn how Oracle software is maintained safely.

Oracle states that since October 2022 it delivers quarterly **Release Updates (RUs)** and **Monthly Recommended Patches (MRPs)** and no longer releases patch sets. ([Oracle Documentation][2])

For Oracle AI Database 26ai, Oracle recommends **out-of-place patching with Gold Images** for quarterly RUs, particularly for advanced deployments such as RAC and Data Guard. ([Oracle Documentation][4])

### 7.1 Understand patch terminology

Learn:

* RU
* MRP
* One-off patch
* Security fixes
* Bug fixes
* Patch conflicts
* OPatch
* OPatchAuto
* Datapatch
* Inventory
* Oracle Home
* Out-of-place patching
* In-place patching
* Gold images

### 7.2 Pre-patching process

Learn:

```text
Check current version
        ↓
Check supported target
        ↓
Read Oracle README
        ↓
Check MOS certification
        ↓
Check conflicts
        ↓
Backup
        ↓
Test
        ↓
Patch
        ↓
Datapatch
        ↓
Validate
```

Oracle's documentation specifically directs DBAs to obtain the patch and read the patch README because it contains the installation procedure and requirements for that patch. ([Oracle Documentation][5])

### 7.3 OPatch concepts

Typical inspection:

```bash
$ORACLE_HOME/OPatch/opatch version

$ORACLE_HOME/OPatch/opatch lsinventory
```

Patch verification:

```bash
$ORACLE_HOME/OPatch/opatch lspatches
```

### 7.4 SQL patch verification

Learn:

```sql
SELECT patch_id,
       action,
       status,
       action_time,
       description
FROM dba_registry_sqlpatch
ORDER BY action_time;
```

### 7.5 Windows patching

Understand:

* Oracle Home
* Windows services
* Stop database/listener
* Patch Oracle Home
* Start services
* Run SQL patching
* Verify inventory
* Verify database

### 7.6 Linux patching

Understand:

* Oracle Home ownership
* Environment
* `opatch`
* `datapatch`
* Listener
* Database services
* systemd/Oracle restart considerations
* GI/RAC considerations

### Practical outcome

You can build a **production patch plan**, execute it, validate it, and understand how Windows and Linux differ.

---

# Lesson 8 — Database Migration: Same OS, Cross-Platform & Data Pump

**Goal:** Move an Oracle database safely from one environment to another.

Oracle documents multiple migration approaches, including Data Pump, transportable tablespaces, full transportable export/import, physical file movement, and cross-platform conversion. ([Oracle Documentation][6])

### 8.1 Migration scenarios

We will cover:

**Scenario A**

```text
Oracle 19c Linux
      ↓
Oracle 19c Linux
```

**Scenario B**

```text
Oracle 19c Windows
      ↓
Oracle 19c Windows
```

**Scenario C**

```text
Oracle Windows
      ↓
Oracle Linux
```

**Scenario D**

```text
Older Oracle release
      ↓
New Oracle release
```

**Scenario E**

```text
Different endian platforms
      ↓
Different endian platforms
```

### 8.2 Data Pump

Master:

```bash
expdp
impdp
```

Learn:

* Schema export/import
* Table export/import
* Full database export/import
* Network import
* `DIRECTORY`
* `DUMPFILE`
* `LOGFILE`
* `REMAP_SCHEMA`
* `REMAP_TABLESPACE`
* `EXCLUDE`
* `INCLUDE`
* `PARALLEL`
* `VERSION`

Example:

```bash
expdp system/password \
  schemas=HR \
  directory=DATA_PUMP_DIR \
  dumpfile=hr.dmp \
  logfile=hr_export.log
```

### 8.3 Transportable Tablespaces

Understand:

```text
Source
  |
  +-- Datafiles
  |
  +-- Metadata via Data Pump
          |
          v
       Target
```

Oracle notes that transportable operations can be much faster than conventional export/import because actual datafiles are copied while Data Pump transfers the metadata. ([Oracle Documentation][7])

### 8.4 Cross-platform migration

Critical concept:

**Endian format**

Oracle documents that when source and target platforms have different endian formats, data must be converted; Oracle provides `DBMS_FILE_TRANSFER` and RMAN `CONVERT` for this purpose. ([Oracle Documentation][7])

### Practical outcome

You can design and execute a migration rather than treating migration as simply "take an export and import it."

---

# Lesson 9 — Oracle Database Upgrades: 11g/12c/18c/19c → New Release

**Goal:** Learn the difference between **patching, upgrading and migrating**.

This distinction is critical.

### Patching

```text
19c RU
   ↓
19c newer RU
```

Same major database release.

### Upgrade

```text
19c
 ↓
new Oracle Database release
```

Database version changes.

### Migration

```text
Server A
   ↓
Server B
```

The location/platform/environment changes.

They can also occur together:

```text
Old DB
   ↓
New OS + New Server + New Oracle Release
```

### 9.1 Oracle upgrade methods

Oracle documents:

* AutoUpgrade
* DBUA
* Manual command-line upgrade
* Export/Import
* Other migration/upgrade scenarios

Oracle currently identifies **AutoUpgrade as the preferred upgrade method**, while DBUA remains available for certain CDB/PDB upgrade scenarios. ([Oracle Documentation][8])

### 9.2 AutoUpgrade

Learn:

```text
Analyze
   ↓
Fix prerequisites
   ↓
Deploy
   ↓
Post-upgrade
   ↓
Validate
```

AutoUpgrade can perform pre-upgrade analysis, deployment and post-upgrade processing. ([Oracle Documentation][8])

### 9.3 Out-of-place upgrade

Oracle recommends installing the new Oracle software in a **new Oracle Home** for out-of-place upgrades/patching. This preserves the old Oracle Home and provides a cleaner rollback/testing strategy. ([Oracle Documentation][9])

Concept:

```text
OLD ORACLE_HOME
       |
       | database currently running
       |
       +-------------------+
                           |
NEW ORACLE_HOME            |
       |                   |
       | upgrade           |
       +-------------------+
```

### 9.4 Upgrade preparation

Learn:

* Supported upgrade paths
* Certification
* Compatibility
* Pre-upgrade checks
* Invalid objects
* Deprecated parameters
* Obsolete parameters
* Time zone considerations
* Statistics
* Backup
* Application compatibility
* Downtime planning
* Rollback planning

### 9.5 Post-upgrade

Learn:

```sql
SELECT * FROM dba_registry;

SELECT object_type,
       COUNT(*)
FROM dba_objects
WHERE status = 'INVALID'
GROUP BY object_type;
```

Oracle specifically requires post-upgrade tasks and recommends recompiling invalid objects after installation, patching or upgrading. ([Oracle Documentation][10])

### 9.6 Windows upgrade

We will cover:

```text
Old Oracle Home
       ↓
Install New Oracle Home
       ↓
Pre-upgrade checks
       ↓
Upgrade
       ↓
Update Windows services/environment
       ↓
Validate
```

### 9.7 Linux upgrade

We will cover:

```text
Old ORACLE_HOME
       ↓
New ORACLE_HOME
       ↓
AutoUpgrade
       ↓
Environment changes
       ↓
Listener/service configuration
       ↓
Validation
```

### Practical outcome

You will understand how to perform a **controlled Oracle version upgrade**, not confuse it with patching.

---

# Lesson 10 — Production DBA: High Availability, Data Guard, RAC & Complete DBA Lifecycle

**Goal:** Put everything together into production-grade administration.

### 10.1 Oracle Data Guard

Learn:

```text
              Primary
                 |
        Redo Transport
                 |
                 v
             Standby
```

Understand:

* Primary database
* Physical standby
* Redo transport
* Redo apply
* Switchover
* Failover
* Broker
* Protection modes
* Data Guard monitoring

### 10.2 Oracle RAC

Learn the architecture:

```text
          Shared Storage
               |
       +-------+-------+
       |               |
    Node 1           Node 2
       |               |
   Instance 1       Instance 2
       |               |
       +-------+-------+
               |
          One Database
```

Learn:

* Grid Infrastructure
* Clusterware
* ASM
* RAC instances
* Services
* SCAN
* Voting files
* OCR
* Node management

Oracle provides separate RAC installation and upgrade documentation for Linux/UNIX and Windows. ([Oracle Documentation][1])

### 10.3 ASM

Learn:

* Disk groups
* ASM instances
* Failure groups
* Redundancy
* ASM disks
* Rebalancing
* Storage administration

### 10.4 Oracle Restart

Understand when Oracle Restart is appropriate for a standalone server and how it relates to Grid Infrastructure.

### 10.5 Production DBA lifecycle

Finally, combine everything:

```text
                DESIGN
                   ↓
             OS PREPARATION
                   ↓
               INSTALL
                   ↓
             CONFIGURATION
                   ↓
           DATABASE CREATION
                   ↓
          NETWORK CONFIGURATION
                   ↓
             SECURITY SETUP
                   ↓
          BACKUP/RECOVERY SETUP
                   ↓
            MONITORING
                   ↓
             PATCHING
                   ↓
             MIGRATION
                   ↓
              UPGRADE
                   ↓
          HIGH AVAILABILITY
                   ↓
          DISASTER RECOVERY
                   ↓
          DAILY OPERATIONS
                   ↓
             DECOMMISSION
```

---

# The Strict 10-Lesson Map

| #      | Lesson                      | Main DBA Skill                   |
| ------ | --------------------------- | -------------------------------- |
| **1**  | Architecture & Fundamentals | Understand Oracle                |
| **2**  | Installation                | Windows + Linux + other OS       |
| **3**  | Configuration & Networking  | Listener, TNS, CDB/PDB           |
| **4**  | Core Administration         | Users, storage, tablespaces      |
| **5**  | Backup & Recovery           | RMAN + disaster recovery         |
| **6**  | Performance & Security      | Production troubleshooting       |
| **7**  | Patching                    | RU/MRP, OPatch, Datapatch        |
| **8**  | Migration                   | Data Pump, TTS, cross-platform   |
| **9**  | Upgrading                   | AutoUpgrade, DBUA, Windows/Linux |
| **10** | Production DBA              | RAC, ASM, Data Guard, HA/DR      |

## Oracle-specific references we'll use

The primary references for this course will be Oracle's own documentation:

* [Oracle Database 19c — Install and Upgrade documentation](https://docs.oracle.com/en/database/oracle/oracle-database/19/install-and-upgrade.html)
* [Oracle AI Database 26ai documentation](https://docs.oracle.com/en/database/oracle/oracle-database/26/)
* [Oracle Database Installation Guide for Linux](https://docs.oracle.com/en/database/oracle/oracle-database/19/ladbi/)
* [Oracle Database Upgrade Guide](https://docs.oracle.com/en/database/oracle/oracle-database/19/upgrd/)
* [Oracle Database Patching documentation](https://docs.oracle.com/en/database/oracle/oracle-database/26/patching.html)
* [Oracle Data Pump documentation](https://docs.oracle.com/en/database/oracle/oracle-database/19/sutil/oracle-data-pump-overview.html)

**Important:** Oracle's procedures are release- and platform-specific. For actual production patching/upgrades, the applicable Oracle release documentation, certification information, and the specific patch README must take precedence over generic commands. Oracle explicitly directs administrators to check the patch README and supported platform/product information before applying patches. ([Oracle Documentation][5])

If we follow these **10 lessons**, each lesson can then be taught as a hands-on DBA class with **concepts → Oracle commands → Windows procedure → Linux procedure → lab → troubleshooting → interview questions → production scenario**, without adding an 11th lesson.

[1]: https://docs.oracle.com/en/database/oracle/oracle-database/19/install-and-upgrade.html?utm_source=chatgpt.com "Oracle Database 19c - Install and Upgrade"
[2]: https://docs.oracle.com/en/database/oracle/oracle-database/26/upgrd/oracle-database-release-numbers.html?utm_source=chatgpt.com "About Oracle Database Release Numbers"
[3]: https://docs.oracle.com/en/database/oracle/oracle-database/19/admqs/installing-oracle-database-creating-database.html?utm_source=chatgpt.com "Installing Oracle Database and Creating a Database"
[4]: https://docs.oracle.com/en/database/oracle/oracle-database/26/patching.html?utm_source=chatgpt.com "Oracle AI Database 26ai - Patching"
[5]: https://docs.oracle.com/en/database/oracle/oracle-database/19/ladbi/downloading-and-installing-patch-updates.html?utm_source=chatgpt.com "Downloading Release Update Patches"
[6]: https://docs.oracle.com/en/database/oracle/oracle-database/19/sutil/oracle-data-pump-overview.html?utm_source=chatgpt.com "Overview of Oracle Data Pump"
[7]: https://docs.oracle.com/en/database/oracle/oracle-database/19/admin/transporting-data.html?utm_source=chatgpt.com "Transporting Data"
[8]: https://docs.oracle.com/en/database/oracle/oracle-database/19/upgrd/choose-an-upgrade-method-for-oracle-database.html?utm_source=chatgpt.com "Choose an Upgrade Method for Oracle Database"
[9]: https://docs.oracle.com/en/database/oracle/oracle-database/19/spucs/installing-oracle-software-in-a-new-oracle-home.html?utm_source=chatgpt.com "Installing Oracle Software in a New Oracle Home"
[10]: https://docs.oracle.com/en/database/oracle/oracle-database/19/upgrd/required-tasks-complete-upgrading-oracle-database.html?utm_source=chatgpt.com "Required Tasks to Complete After Upgrading Oracle Database"
