# Lesson 10/10 — Oracle RAC + ASM + Data Guard + End-to-End DBA Operations

**Lesson 10 of 10.**

This is the final lesson of the course. We will put together the major Oracle DBA technologies into one production-oriented operating model.

I’m basing the technical material below on Oracle's official 19c documentation, including the current Oracle Grid Infrastructure, RAC, ASM, and Data Guard documentation. 

Oracle documents Grid Infrastructure as including **Oracle Clusterware + Oracle ASM** for RAC deployments. ([Oracle Documentation][1])

---

# 10.1 What You Must Know at the End of This Course

Your complete DBA skill map is:

```text
                         ORACLE DBA
                             |
       +---------------------+---------------------+
       |                     |                     |
   DATABASE              STORAGE               NETWORK
       |                     |                     |
  Instance                 ASM                 Listener
  CDB/PDB                 Files                SQL*Net
  Memory                  Diskgroups            Services
  Processes               OMF
       |
       +-------------------+
       |
   ADMINISTRATION
       |
       +-- Users / Roles
       +-- Tablespaces
       +-- Jobs
       +-- Database Links
       +-- Security
       +-- Performance
       +-- Backup / Recovery
       +-- Patching
       +-- Migration
       +-- Upgrade
       |
       +---------------------------+
       |                           |
      RAC                      DATA GUARD
       |                           |
 Clusterware                 Primary
 ASM                         Standby
 Services                    Redo Transport
 Cache Fusion                Redo Apply
       |                     Broker
       |                     Switchover
       |                     Failover
       |
       +------------+
                    |
              PRODUCTION DBA
```

---

# 10.2 RAC — What Is It?

**Oracle Real Application Clusters (RAC)** allows multiple instances on different servers to access the same Oracle database.

Conceptually:

```text
             APPLICATION
                  |
            Load Balancer
                  |
        +---------+---------+
        |                   |
        v                   v
     NODE 1              NODE 2
   Instance 1           Instance 2
        |                   |
        +---------+---------+
                  |
                  v
              Shared Storage
                  |
                  v
               DATABASE
```

The important distinction:

```text
RAC = multiple instances
      |
      +---- one database
```

It is not:

```text
RAC = two independent databases
```

---

# 10.3 RAC Architecture

A typical RAC environment contains:

```text
Node 1
 ├── Oracle Grid Infrastructure
 ├── Clusterware
 ├── ASM instance
 └── Oracle RAC instance

Node 2
 ├── Oracle Grid Infrastructure
 ├── Clusterware
 ├── ASM instance
 └── Oracle RAC instance

Shared Storage
 ├── OCR
 ├── Voting files
 ├── Datafiles
 ├── Redo
 └── FRA
```

Oracle requires shared storage for RAC database files, and Oracle recommends ASM for RAC storage. ([Oracle Documentation][2])

---

# 10.4 Grid Infrastructure

Remember:

```text
Grid Infrastructure
       |
       +-- Oracle Clusterware
       |
       +-- Oracle ASM
```

Oracle's official installation documentation explicitly defines Grid Infrastructure this way. ([Oracle Documentation][1])

Clusterware provides the cluster management layer.

ASM provides storage management.

---

# 10.5 Oracle Clusterware

Clusterware manages resources such as:

```text
Database
Instance
Listener
ASM
Services
VIP
SCAN
```

Conceptually:

```text
             Clusterware
                  |
       +----------+----------+
       |          |          |
     ASM       Database    Service
       |          |          |
     Node 1     Node 1     Node 1
```

If a managed resource fails, Clusterware can restart or relocate resources according to their configuration.

---

# 10.6 RAC Private Interconnect

RAC nodes communicate over the private interconnect.

```text
NODE 1 ================= NODE 2
          PRIVATE
        INTERCONNECT
```

This carries important cluster traffic, including cache-related coordination.

Do not confuse:

```text
Public network
```

with:

```text
Private interconnect
```

---

# 10.7 Cache Fusion

One of the most important RAC concepts.

Suppose:

```text
Instance 1
   |
   | block
   v
Cache
```

Instance 2 needs the same block.

RAC coordinates access between instances through the cluster interconnect rather than requiring every request to go back to disk.

Conceptually:

```text
Instance 1 Cache
       |
       | block coordination
       v
Private Interconnect
       |
       v
Instance 2 Cache
```

This is part of Oracle RAC's cache-coherency architecture.

---

# 10.8 ASM

**Automatic Storage Management (ASM)** is Oracle's storage management technology.

Oracle describes ASM as both a volume manager and file system for Oracle database files. It distributes file contents across disks in a disk group and can provide mirroring through redundancy settings. ([Oracle Documentation][2])

Think:

```text
                ASM
                 |
       +---------+---------+
       |         |         |
      Disk1     Disk2     Disk3
       |         |         |
       +---------+---------+
                 |
             Disk Group
```

---

# 10.9 ASM Disk Groups

Typical:

```text
+DATA
+FRA
```

Example:

```text
+DATA
 ├── datafiles
 ├── controlfiles
 └── online redo

+FRA
 ├── archived logs
 ├── backups
 └── recovery files
```

The exact layout depends on your architecture.

---

# 10.10 ASM Redundancy

ASM can use:

```text
EXTERNAL
NORMAL
HIGH
```

Conceptually:

```text
EXTERNAL
Oracle does not mirror.
Storage subsystem provides redundancy.

NORMAL
Two-way mirroring.

HIGH
Three-way mirroring.
```

Oracle documents normal and high redundancy as two-way and three-way mirroring respectively. ([Oracle Documentation][2])

---

# 10.11 ASM Administration

Important tools:

```text
ASMCA
ASMCMD
SQL*Plus
Enterprise Manager
```

Oracle explicitly documents ASMCA and ASMCMD among the administration interfaces for ASM. ([Oracle Documentation][2])

---

# 10.12 ASM Views

Connect to ASM:

```sql
sqlplus / as sysasm
```

Check disk groups:

```sql
SELECT name,
       state,
       type,
       total_mb,
       free_mb
FROM v$asm_diskgroup;
```

Check disks:

```sql
SELECT name,
       path,
       mount_status,
       header_status,
       mode_status,
       state
FROM v$asm_disk;
```

---

# 10.13 ASMCMD

Typical commands:

```bash
asmcmd
```

Then:

```text
ls
du
cd
pwd
```

You can navigate ASM storage similarly to a filesystem, but ASM is not an ordinary filesystem.

---

# 10.14 Adding an ASM Disk

Conceptually:

```sql
ALTER DISKGROUP DATA
ADD DISK '/path/to/disk';
```

Then ASM rebalances the disk group.

Oracle notes that disks can be added or removed while the database continues accessing the disk group, with ASM redistributing file contents. ([Oracle Documentation][2])

---

# 10.15 ASM Rebalance

Suppose:

```text
Before

Disk1  90%
Disk2  90%
Disk3   0%
```

After adding Disk3:

```text
ASM REBALANCE

Disk1  60%
Disk2  60%
Disk3  60%
```

This is one of the major operational advantages of ASM.

---

# 10.16 RAC Services

Applications should generally connect through **services**, not by hard-coding a particular RAC instance.

Example:

```text
Application
     |
     v
SALES service
     |
 +---+---+
 |       |
Node1   Node2
```

Oracle RAC services provide workload management and can be configured with preferred and available instances. ([Oracle Documentation][3])

---

# 10.17 SRVCTL

One of your most important RAC commands:

```bash
srvctl
```

Examples:

```bash
srvctl status database -db ORCL
```

```bash
srvctl status service -db ORCL
```

```bash
srvctl config service -db ORCL
```

Oracle documents `srvctl` for starting, stopping, enabling, disabling and inspecting RAC services. ([Oracle Documentation][3])

---

# 10.18 Service Management

Start:

```bash
srvctl start service \
-db ORCL \
-service SALES
```

Stop:

```bash
srvctl stop service \
-db ORCL \
-service SALES
```

Status:

```bash
srvctl status service \
-db ORCL \
-service SALES
```

Configuration:

```bash
srvctl config service \
-db ORCL \
-service SALES
```

Oracle documents these service operations directly. ([Oracle Documentation][3])

---

# 10.19 RAC Installation — Linux

The broad production sequence is:

```text
1. Prepare OS
       ↓
2. Configure network
       ↓
3. Configure shared storage
       ↓
4. Install Grid Infrastructure
       ↓
5. Configure Clusterware + ASM
       ↓
6. Verify cluster
       ↓
7. Install Oracle Database software
       ↓
8. Create RAC database
       ↓
9. Configure services
       ↓
10. Validate
```

Oracle's 19c Linux Grid Infrastructure guide provides the official OS, storage and installation procedures. ([Oracle Documentation][4])

---

# 10.20 Linux RAC Prerequisites

Before installation:

```text
[ ] Supported Linux release
[ ] Correct kernel
[ ] Required packages
[ ] OpenSSH
[ ] Oracle users/groups
[ ] DNS/hosts resolution
[ ] Public network
[ ] Private interconnect
[ ] SCAN
[ ] VIP
[ ] Shared storage
[ ] Time synchronization
[ ] Firewall/network requirements
```

Oracle specifically requires consistent OS/kernel/package configuration across cluster nodes and documents the supported operating-system requirements. ([Oracle Documentation][5])

---

# 10.21 RAC on Windows

Oracle also provides a separate Grid Infrastructure installation guide for Microsoft Windows. ([Oracle Documentation][6])

The architecture remains:

```text
Windows Node 1
       |
       +---- Grid Infrastructure
       +---- ASM
       +---- RAC Instance

Windows Node 2
       |
       +---- Grid Infrastructure
       +---- ASM
       +---- RAC Instance
```

But administration differs in areas such as:

```text
Windows services
Windows networking
File paths
Permissions
OS configuration
```

---

# 10.22 RAC Installation — Windows

High-level:

```text
Prepare Windows nodes
        ↓
Configure networking
        ↓
Configure shared storage
        ↓
Install Grid Infrastructure
        ↓
Configure Clusterware + ASM
        ↓
Install Database software
        ↓
Create RAC database
        ↓
Configure services
        ↓
Validate
```

Oracle's Windows documentation provides the corresponding cluster installation procedure. ([Oracle Documentation][7])

---

# 10.23 RAC Validation

After installation:

```bash
crsctl check cluster -all
```

Check resources:

```bash
crsctl status resource -t
```

Database:

```bash
srvctl status database -db ORCL
```

Services:

```bash
srvctl status service -db ORCL
```

ASM:

```bash
srvctl status asm
```

---

# 10.24 Cluster Verification Utility

Oracle provides CVU for checking cluster prerequisites and configuration.

Example:

```bash
cluvfy comp asm -n node1,node2 -verbose
```

Oracle documents CVU as a validation tool for Oracle ASM and cluster configuration. ([Oracle Documentation][2])

---

# 10.25 Now — Data Guard

RAC protects against certain **instance/node failures**.

Data Guard protects against **database/site failures** by maintaining standby databases.

Basic architecture:

```text
             PRIMARY
                |
          Redo Transport
                |
                v
             STANDBY
                |
            Redo Apply
```

---

# 10.26 Physical Standby

Most common architecture:

```text
PRIMARY DATABASE
      |
      | REDO
      v
STANDBY DATABASE
      |
      | REDO APPLY
      v
STANDBY DATAFILES
```

The standby maintains a physical copy of the primary database.

---

# 10.27 Data Guard Components

Know these:

```text
Primary Database
Standby Database
Redo Transport
Standby Redo Logs
Redo Apply
Data Guard Broker
DGMGRL
Protection Modes
Switchover
Failover
Fast-Start Failover
Observer
```

---

# 10.28 Redo Transport

Primary generates redo:

```text
Transaction
    ↓
Redo
    ↓
Redo Transport
    ↓
Standby Redo Logs
```

Then:

```text
Standby Redo Logs
        ↓
     Redo Apply
        ↓
Standby Database
```

---

# 10.29 Standby Redo Logs

A common mistake is forgetting SRLs.

You should configure standby redo logs appropriately on Data Guard members.

Conceptually:

```text
PRIMARY
 Online Redo
      |
      v
Network
      |
      v
STANDBY
Standby Redo Logs
      |
      v
Redo Apply
```

Oracle's Broker documentation specifically notes that standby redo logs should be prepared on the primary so that it can assume a standby role after a switchover. ([Oracle Documentation][8])

---

# 10.30 Data Guard Protection Modes

Know:

```text
Maximum Performance
Maximum Availability
Maximum Protection
```

Simplified:

### Maximum Performance

Prioritizes performance.

```text
Primary
  |
  | asynchronous transport
  v
Standby
```

### Maximum Availability

Balances availability and protection.

### Maximum Protection

Provides the highest level of data protection, subject to the configuration requirements.

Do not memorize these merely as labels; understand the redo transport and commit behavior implications.

---

# 10.31 Data Guard Broker

Oracle provides **Data Guard Broker** to simplify management.

Command-line tool:

```bash
dgmgrl
```

Connect:

```text
DGMGRL> CONNECT /
```

Show configuration:

```text
DGMGRL> SHOW CONFIGURATION;
```

Show database:

```text
DGMGRL> SHOW DATABASE VERBOSE ORCL;
```

Oracle documents DGMGRL as the command-line interface for Data Guard Broker management. ([Oracle Documentation][8])

---

# 10.32 Broker Architecture

```text
              DGMGRL
                 |
                 v
        Data Guard Broker
          /           \
         /             \
   PRIMARY           STANDBY
```

The Broker coordinates role transitions and related configuration operations.

---

# 10.33 Data Guard Health Check

Start with:

```text
DGMGRL> SHOW CONFIGURATION;
```

Then:

```text
DGMGRL> VALIDATE DATABASE VERBOSE ORCL;
```

And:

```text
DGMGRL> VALIDATE NETWORK CONFIGURATION;
```

Oracle specifically documents `VALIDATE DATABASE` and network validation for assessing readiness for role changes. ([Oracle Documentation][8])

---

# 10.34 Switchover

**Switchover = planned role reversal.**

```text
BEFORE

PRIMARY  →  STANDBY
DB1          DB2


AFTER

STANDBY  ←  PRIMARY
DB1          DB2
```

Oracle states that a switchover is a role reversal and is intended for planned operations such as maintenance; a properly performed switchover guarantees no data loss. ([Oracle Documentation][8])

---

# 10.35 Switchover Command

Using Broker:

```text
DGMGRL> SWITCHOVER TO DB2;
```

The Broker coordinates the role transition.

Oracle documents `SWITCHOVER` through DGMGRL and explains the sequence Broker performs. ([Oracle Documentation][8])

---

# 10.36 Failover

**Failover = unplanned primary failure.**

```text
PRIMARY
   X
   |
   v
STANDBY
   |
   v
NEW PRIMARY
```

Unlike a switchover, failover may involve data loss depending on synchronization and protection mode. ([Oracle Documentation][8])

---

# 10.37 Complete vs Immediate Failover

Oracle recommends attempting a **complete failover** first.

Conceptually:

```text
Complete
   |
   +-- Apply available redo
   +-- Then become primary
```

Immediate:

```text
Immediate
   |
   +-- Stop applying
   +-- Become primary faster
   +-- Potentially more data loss
```

Oracle explicitly describes complete failover as the recommended/default option and immediate failover as the faster option that can result in data loss. ([Oracle Documentation][8])

---

# 10.38 Failover Command

Example:

```text
DGMGRL> FAILOVER TO DB2;
```

Immediate, when specifically justified:

```text
DGMGRL> FAILOVER TO DB2 IMMEDIATE;
```

Oracle documents the distinction and warns that immediate failover stops apply without waiting for accumulated redo. ([Oracle Documentation][8])

---

# 10.39 Fast-Start Failover

FSFO allows Broker to automatically initiate failover when configured conditions are met.

Architecture:

```text
             Observer
                |
        +-------+-------+
        |               |
        v               v
     PRIMARY        STANDBY
```

Oracle documents Fast-Start Failover as an automated Broker-managed failover mechanism and provides configurable thresholds and conditions. ([Oracle Documentation][9])

---

# 10.40 Observer

The observer monitors the Data Guard environment.

Conceptually:

```text
                 OBSERVER
                /        \
               /          \
              v            v
          PRIMARY        STANDBY
```

This is particularly important in automated failover designs.

---

# 10.41 Observe-Only Mode

Oracle provides an observe-only mode for testing Fast-Start Failover behavior without actually performing failover.

Example:

```text
DGMGRL> ENABLE FAST_START FAILOVER OBSERVE ONLY;
```

Oracle documents this mode specifically as a way to test FSFO behavior without making actual configuration changes. ([Oracle Documentation][8])

---

# 10.42 RAC + Data Guard Together

Now combine everything.

Production architecture:

```text
             APPLICATION
                  |
              Services
                  |
          +-------+-------+
          |               |
       RAC NODE 1      RAC NODE 2
          |               |
          +-------+-------+
                  |
               PRIMARY
                  |
             REDO TRANSPORT
                  |
                  v
          +---------------+
          | DR SITE       |
          |               |
          | RAC NODE 1    |
          | RAC NODE 2    |
          |               |
          | STANDBY       |
          +---------------+
```

This provides:

```text
RAC
 ↓
instance/node availability

Data Guard
 ↓
database/site disaster recovery
```

---

# 10.43 RAC Does NOT Replace Data Guard

Very important.

RAC protects primarily against:

```text
Instance failure
Node failure
Certain local hardware failures
```

Data Guard protects against:

```text
Database/site disaster
Storage/site-level failure
Logical/operational disaster scenarios
```

Therefore:

```text
RAC ≠ DR
```

and:

```text
Data Guard ≠ RAC
```

They solve different problems.

---

# 10.44 Complete Production Architecture

A mature Oracle environment might look like:

```text
                         USERS
                           |
                     LOAD BALANCER
                           |
                    APPLICATION TIER
                           |
                    ORACLE SERVICES
                           |
              +------------+------------+
              |                         |
           RAC NODE 1               RAC NODE 2
              |                         |
          INSTANCE 1               INSTANCE 2
              |                         |
              +------------+------------+
                           |
                         ASM
                           |
                       +DATA
                       +FRA
                           |
                       PRIMARY
                           |
                    REDO TRANSPORT
                           |
                           v
                 =====================
                       DR SITE
                 =====================
                           |
                       RAC NODE 1
                           |
                       RAC NODE 2
                           |
                        STANDBY
                           |
                      REDO APPLY
                           |
                       BROKER
                           |
                       OBSERVER
```

---

# 10.45 Complete DBA Daily Health Check

A production DBA should routinely inspect:

### Database

```sql
SELECT name,
       open_mode,
       database_role
FROM v$database;
```

### Instance

```sql
SELECT instance_name,
       status,
       version
FROM v$instance;
```

### Invalid objects

```sql
SELECT owner,
       object_name,
       object_type
FROM dba_objects
WHERE status <> 'VALID';
```

### Tablespaces

```sql
SELECT tablespace_name,
       status,
       contents
FROM dba_tablespaces;
```

### Sessions

```sql
SELECT status,
       COUNT(*)
FROM v$session
GROUP BY status;
```

---

# 10.46 RAC Daily Health Check

```bash
crsctl check cluster -all
```

```bash
crsctl status resource -t
```

```bash
srvctl status database -db ORCL
```

```bash
srvctl status service -db ORCL
```

Then inspect:

```text
ASM
Disk groups
Services
Instances
Listeners
SCAN
Cluster resources
Alert logs
Clusterware logs
```

---

# 10.47 ASM Daily Health Check

```sql
SELECT name,
       state,
       type,
       total_mb,
       free_mb
FROM v$asm_diskgroup;
```

Look for:

```text
MOUNTED
NORMAL
Adequate free space
```

Investigate:

```text
DISMOUNTED
OFFLINE
Unexpected rebalance
Low free space
Missing disks
```

---

# 10.48 Data Guard Daily Health Check

Start with:

```text
DGMGRL> SHOW CONFIGURATION;
```

Then:

```text
DGMGRL> VALIDATE DATABASE VERBOSE PRIMARY;
```

```text
DGMGRL> VALIDATE DATABASE VERBOSE STANDBY;
```

Monitor:

```text
Transport lag
Apply lag
Redo gaps
Standby status
Broker status
Archive destination status
```

---

# 10.49 Performance DBA Workflow

When users say:

> "Database is slow."

Do NOT immediately increase memory.

Follow:

```text
Application
    ↓
Database wait
    ↓
SQL
    ↓
Execution plan
    ↓
I/O / CPU / Memory
    ↓
Locks
    ↓
Storage
    ↓
OS
```

---

# 10.50 Blocking Session Workflow

Find blockers:

```sql
SELECT sid,
       serial#,
       blocking_session,
       event,
       status
FROM v$session
WHERE blocking_session IS NOT NULL;
```

Then investigate:

```text
Who is blocking?
What SQL?
Why is the transaction open?
How long?
Business impact?
```

Never blindly kill sessions.

---

# 10.51 Backup DBA Workflow

Your backup hierarchy:

```text
DATABASE
   |
   +-- RMAN Full/Incremental
   |
   +-- Archived Redo
   |
   +-- Control File
   |
   +-- SPFILE
   |
   +-- Recovery Catalog, where used
```

And the most important rule:

```text
BACKUP SUCCESS
       ≠
RECOVERY SUCCESS
```

You must test restoration/recovery.

---

# 10.52 Patch Management Workflow

For every Oracle Home:

```text
Inventory
   ↓
Required patches
   ↓
Download
   ↓
Read README
   ↓
Backup
   ↓
Test
   ↓
Apply
   ↓
Datapatch where required
   ↓
Validate
```

Remember the distinction:

```text
Binary patch
      +
SQL patch
```

Check:

```sql
SELECT patch_id,
       action,
       status,
       action_time,
       description
FROM dba_registry_sqlpatch
ORDER BY action_time DESC;
```

---

# 10.53 Upgrade Workflow

Your final upgrade model:

```text
SOURCE
  |
  v
BACKUP
  |
  v
NEW ORACLE HOME
  |
  v
PATCH NEW HOME
  |
  v
AUTOUPGRADE ANALYZE
  |
  v
FIX PROBLEMS
  |
  v
TEST
  |
  v
DEPLOY
  |
  v
POST-UPGRADE VALIDATION
  |
  v
APPLICATION TEST
```

---

# 10.54 Migration Workflow

```text
SOURCE
  |
  v
DISCOVERY
  |
  v
COMPATIBILITY CHECK
  |
  v
CHOOSE METHOD
  |
  +-- Data Pump
  +-- TTS
  +-- Full Transportable
  +-- RMAN
  +-- Data Guard
  +-- GoldenGate
  |
  v
TEST
  |
  v
FINAL SYNC
  |
  v
CUTOVER
  |
  v
VALIDATION
```

---

# 10.55 Incident Management

When production breaks:

```text
1. Detect
2. Confirm
3. Assess impact
4. Stabilize
5. Diagnose
6. Recover
7. Validate
8. Communicate
9. Document
10. Prevent recurrence
```

Don't start randomly changing parameters.

---

# 10.56 Oracle DBA Emergency Decision Tree

```text
                  PRODUCTION ISSUE
                         |
                         v
                   Is DB reachable?
                    /          \
                  NO            YES
                  |              |
             Connectivity      Slow?
                  |            /    \
               Listener      YES     NO
               Network       |       |
               Instance    Performance  Functional
                            |           |
                           SQL        Objects/
                           Waits      Application
```

---

# 10.57 If Database Is Down

Check:

```bash
ps -ef | grep pmon
```

Then:

```sql
sqlplus / as sysdba
```

```sql
SELECT status
FROM v$instance;
```

If necessary:

```sql
STARTUP;
```

But in RAC:

> Do not treat a RAC database like a standalone database.

Use Clusterware/SRVCTL-aware administration.

---

# 10.58 If RAC Node Fails

Conceptually:

```text
NODE 1
  X

NODE 2
  |
  +-- surviving instance
  |
  +-- services
```

Your DBA checks:

```text
Clusterware
Database instances
Services
Application connections
ASM
Alert logs
Cluster logs
```

Then determine whether the failure is:

```text
OS
Network
Storage
Clusterware
Database
Application
```

---

# 10.59 If Primary Database Fails

Data Guard decision:

```text
Primary failure
      |
      v
Can primary recover quickly?
      |
    YES ──> Recover primary
      |
     NO
      |
      v
Choose standby
      |
      v
VALIDATE
      |
      v
FAILOVER
      |
      v
New Primary
```

Oracle explicitly says failover should be performed when the original primary cannot be recovered in a timely manner. ([Oracle Documentation][8])

---

# 10.60 After Data Guard Failover

Your work isn't finished.

You must:

```text
1. Validate new primary
2. Validate applications
3. Check services
4. Re-establish redo transport
5. Rebuild/reinstate old primary where appropriate
6. Validate Broker
7. Restore DR protection
```

Oracle notes that after failover, some standby databases may need to be reinstated or recreated to become viable standbys for the new primary. ([Oracle Documentation][8])

---

# 10.61 DBA Security Checklist

Every production DBA should understand:

```text
Users
Roles
Privileges
Profiles
Password policies
Auditing
TDE
Wallet/keystore
Network encryption
Database links
OS permissions
Listener security
SYS/SYSTEM usage
```

Never make:

```text
GRANT DBA TO application_user;
```

your default solution.

---

# 10.62 DBA Documentation

Maintain:

```text
Database inventory
Oracle Home inventory
Patch inventory
Architecture diagram
Storage map
RAC topology
Data Guard topology
Backup policy
Recovery procedures
Migration runbook
Upgrade runbook
Security procedures
Monitoring procedures
Incident history
```

---

# 10.63 Your Production Oracle DBA Folder

A useful structure:

```text
oracle-dba/
|
+-- architecture/
|
+-- installation/
|
+-- configuration/
|
+-- networking/
|
+-- security/
|
+-- backup/
|
+-- recovery/
|
+-- performance/
|
+-- patching/
|
+-- migration/
|
+-- upgrade/
|
+-- rac/
|
+-- asm/
|
+-- dataguard/
|
+-- incidents/
|
+-- runbooks/
|
+-- scripts/
```

---

# 10.64 The 10-Lesson Course — COMPLETE

You now have the complete roadmap:

```text
LESSON 1
Oracle Architecture & DBA Foundations
             ↓
LESSON 2
Installation & Database Creation
             ↓
LESSON 3
Configuration & Oracle Networking
             ↓
LESSON 4
Core DBA Administration
             ↓
LESSON 5
RMAN Backup & Recovery
             ↓
LESSON 6
Performance & Troubleshooting
             ↓
LESSON 7
Security & Advanced Administration
             ↓
LESSON 8
Patching & Patch Management
             ↓
LESSON 9
Migration & Upgrading
             ↓
LESSON 10
RAC + ASM + Data Guard
             ↓
      PRODUCTION DBA
```

---

# 10.65 Final DBA Skill Matrix

| Area              | Beginner           | Production DBA                    |
| ----------------- | ------------------ | --------------------------------- |
| Installation      | Install DB         | Design/build environments         |
| Configuration     | Change parameters  | Capacity/configuration management |
| SQL               | Basic SQL          | DBA diagnostic SQL                |
| Storage           | Tablespaces        | ASM/storage architecture          |
| Backup            | Run RMAN           | Design/test recovery              |
| Recovery          | Restore DB         | RTO/RPO-driven recovery           |
| Performance       | Check CPU          | SQL/wait/I/O diagnosis            |
| Security          | Create users       | Enterprise security               |
| Patching          | Apply patch        | Patch lifecycle                   |
| Upgrade           | Run utility        | Plan/test/rollback                |
| Migration         | Data Pump          | Platform migration architecture   |
| RAC               | Understand nodes   | Cluster operations                |
| Data Guard        | Understand standby | DR operations/failover            |
| Automation        | Scripts            | Repeatable runbooks               |
| Incident response | Fix issue          | Stabilize → recover → RCA         |

---

# 10.66 The Most Important DBA Principle

A senior DBA isn't simply someone who knows hundreds of commands.

A senior DBA thinks:

```text
                 BUSINESS
                    |
                    v
              Availability
                    |
                    v
              Data Integrity
                    |
                    v
                Security
                    |
                    v
              Performance
                    |
                    v
              Recoverability
                    |
                    v
             OPERABILITY
```

Every technical decision should ultimately answer:

> **Can the business safely run, recover, and continue operating?**

---

# Official Oracle Documentation

The primary Oracle sources for this final lesson are:

* [Oracle Database 19c — Install and Upgrade Documentation](https://docs.oracle.com/en/database/oracle/oracle-database/19/install-and-upgrade.html)
* [Oracle Grid Infrastructure 19c for Linux](https://docs.oracle.com/en/database/oracle/oracle-database/19/cwlin/)
* [Oracle Grid Infrastructure 19c for Windows](https://docs.oracle.com/en/database/oracle/oracle-database/19/cwwin/)
* [Oracle RAC 19c — Administering Storage](https://docs.oracle.com/en/database/oracle/oracle-database/19/racad/administering-storage-in-oracle-rac.html)
* [Oracle Data Guard Broker 19c — Switchover and Failover](https://docs.oracle.com/en/database/oracle/oracle-database/19/dgbkr/using-data-guard-broker-to-manage-switchovers-failovers.html)

---

## 🎓 COURSE STATUS: 10/10 COMPLETE

You now have the **end-to-end Oracle DBA syllabus** covering:

**Installation → Configuration → Networking → Administration → Backup → Recovery → Performance → Security → Patching → Migration → Upgrade → RAC → ASM → Data Guard → Switchover → Failover → Production Operations.**

The next logical step is **not Lesson 11**, because you asked for **STRICTLY 10 lessons**. The best continuation is a **hands-on Oracle DBA lab/practical track**, where each of these 10 lessons is turned into real Windows + Linux exercises and production-style scenarios.

[1]: https://docs.oracle.com/en/database/oracle/oracle-database/19/cwwin/installing-oracle-grid-infrastructure-for-a-cluster.html "Installing Oracle Grid Infrastructure for a Cluster"
[2]: https://docs.oracle.com/en/database/oracle/oracle-database/19/racad/administering-storage-in-oracle-rac.html "Administering Storage in Oracle RAC"
[3]: https://docs.oracle.com/en/database/oracle/oracle-database/19/racad/workload-management-with-dynamic-database-services.html "Workload Management with Dynamic Database Services"
[4]: https://docs.oracle.com/en/database/oracle/oracle-database/19/cwlin/ "Oracle Grid Infrastructure Grid Infrastructure Installation and Upgrade Guide, 19c for Linux"
[5]: https://docs.oracle.com/en/database/oracle/oracle-database/19/cwlin/operating-system-checklist-for-oracle-grid-infrastructure-and-oracle-rac.html?source=%3Aow%3Alp%3Acpo%3A%3A%3A%3ARC_CORP240617P00044%3ADMO400365213&utm_source=chatgpt.com "Operating System Checklist for Oracle Grid Infrastructure and Oracle RAC"
[6]: https://docs.oracle.com/en/database/oracle/oracle-database/19/cwwin/ "Oracle Grid Infrastructure Installation and Upgrade Guide, 19c for Microsoft Windows"
[7]: https://docs.oracle.com/en/database/oracle/oracle-database/19/cwwin/installing-oracle-grid-infrastructure-for-a-new-cluster.html "Installing Oracle Grid Infrastructure for a New Cluster"
[8]: https://docs.oracle.com/en/database/oracle/oracle-database/19/dgbkr/using-data-guard-broker-to-manage-switchovers-failovers.html?source=%3Aso%3Ach%3Aor%3Aawr%3A%3A%3A%3APrivateEndpointsPlaybook&utm_source=chatgpt.com "Switchover and Failover Operations"
[9]: https://docs.oracle.com/en/database/oracle/oracle-database/19/dgbkr/managing-oracle-data-guard-broker-configuration-members.html "Managing the Members of a Broker Configuration"
