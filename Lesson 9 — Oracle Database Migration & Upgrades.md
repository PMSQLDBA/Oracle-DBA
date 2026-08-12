# Lesson 9/10 — Oracle Database Migration & Upgrades

**Strictly Lesson 9 of 10.**

This lesson covers **migration and upgrading across Windows, Linux and other supported platforms**, including Data Pump, Transportable Tablespaces, RMAN conversion, PDB migration, AutoUpgrade, pre-upgrade checks, rollback planning, and production runbooks.

Oracle provides several upgrade approaches, including **AutoUpgrade, DBUA, manual upgrade, Data Pump migration, and transport-based methods**. Oracle currently recommends AutoUpgrade as the primary automated upgrade method. ([Oracle Documentation][1])

---

# 9.1 Migration vs Upgrade

First, never confuse these.

### Migration

Moving the database/data to another environment:

```text
Server A
   |
   | Migration
   v
Server B
```

Examples:

```text
Windows → Linux
Linux → Linux
Physical server → new physical server
On-premises → cloud
Non-CDB → PDB
```

### Upgrade

Moving the database to a newer Oracle Database release:

```text
Oracle 19c
    |
    | Upgrade
    v
Newer Oracle release
```

### Migration + Upgrade

You can combine them:

```text
Old server
   |
   | migrate
   v
New server
   |
   | upgrade
   v
New Oracle release
```

Or use a method that accomplishes both as part of the same overall project.

---

# 9.2 The Five Major Migration/Upgrade Methods

You need to know these:

```text
1. Data Pump
2. Transportable Tablespaces
3. Full Transportable Export/Import
4. RMAN-based migration/conversion
5. AutoUpgrade / upgrade utilities
```

And for advanced availability:

```text
6. Data Guard
7. GoldenGate
8. PDB unplug/plug
```

Oracle explicitly documents Data Pump, transportable technologies, Data Guard, GoldenGate, and AutoUpgrade as possible approaches depending on the migration/upgrade scenario. ([Oracle Documentation][2])

---

# 9.3 Choosing the Method

Use this mental model:

```text
                         MIGRATION
                             |
             +---------------+---------------+
             |               |               |
          Logical         Physical       Multitenant
             |               |               |
         Data Pump          RMAN          PDB
             |               |
      TTS / Full TTS      Conversion
```

Then ask:

```text
How much data?
Downtime allowed?
Same platform?
Different platform?
Same Oracle release?
Different release?
CDB/PDB?
Need near-zero downtime?
```

---

# 9.4 Data Pump

Oracle Data Pump consists primarily of:

```text
expdp
impdp
```

Export:

```bash
expdp
```

Import:

```bash
impdp
```

It is a logical migration mechanism.

Conceptually:

```text
SOURCE
  |
  | expdp
  v
Dump files
  |
  | transfer
  v
TARGET
  |
  | impdp
  v
Database objects/data
```

Oracle documents Data Pump as the standard utility for moving database data and metadata through export/import operations. ([Oracle Documentation][3])

---

# 9.5 Data Pump — Basic Schema Migration

Create a directory:

```sql
CREATE DIRECTORY dp_dir
AS '/u01/app/oracle/dpump';
```

Grant:

```sql
GRANT READ, WRITE
ON DIRECTORY dp_dir
TO system;
```

Export:

```bash
expdp system/password \
schemas=HR \
directory=DP_DIR \
dumpfile=hr.dmp \
logfile=hr_export.log
```

Import:

```bash
impdp system/password \
schemas=HR \
directory=DP_DIR \
dumpfile=hr.dmp \
logfile=hr_import.log
```

In production, use secure credential handling rather than putting passwords directly into shell history/scripts.

---

# 9.6 Full Database Data Pump

Export:

```bash
expdp system/password \
full=y \
directory=DP_DIR \
dumpfile=full.dmp \
logfile=full_export.log
```

Import:

```bash
impdp system/password \
full=y \
directory=DP_DIR \
dumpfile=full.dmp \
logfile=full_import.log
```

But don't automatically assume:

> Full Data Pump = best migration method.

For very large databases, physical/transport-based methods may be much faster.

---

# 9.7 Why Data Pump Can Be Slow

Traditional logical migration can involve:

```text
Read rows
   ↓
Convert to export representation
   ↓
Write dump
   ↓
Transfer dump
   ↓
Read dump
   ↓
Create objects
   ↓
Insert rows
   ↓
Build indexes
```

Large databases can therefore require significant time.

---

# 9.8 Transportable Tablespaces

Transportable Tablespaces (TTS) works differently.

Instead of unloading/reloading all table data:

```text
SOURCE DATAFILES
      |
      | Copy
      v
TARGET DATAFILES

Metadata
      |
      | Data Pump
      v
TARGET
```

Oracle states that transporting data can be much faster than conventional export/import because the actual data files are copied and Data Pump primarily transfers metadata. ([Oracle Documentation][2])

---

# 9.9 TTS Workflow

The basic process is:

```text
1. Select self-contained tablespaces
2. Make them READ ONLY
3. Generate transport metadata
4. Copy dump
5. Copy datafiles
6. Import metadata
7. Make tablespaces READ WRITE
8. Validate
```

Oracle documents this sequence for transportable tablespaces. ([Oracle Documentation][4])

---

# 9.10 Self-Contained Tablespace Set

This is important.

Suppose:

```text
TABLE_A
   |
   +-- TABLESPACE_A

INDEX_A
   |
   +-- TABLESPACE_B
```

If you transport only:

```text
TABLESPACE_A
```

but the object has required dependencies in:

```text
TABLESPACE_B
```

you may not have a valid self-contained set.

Oracle requires the transportable set to be self-contained. ([Oracle Documentation][5])

---

# 9.11 Check Transportable Set

Oracle provides:

```sql
DBMS_TTS
```

for checking transportable tablespace sets.

The exact procedure depends on your scenario and Oracle release.

Conceptually:

```text
Tablespaces selected
       |
       v
Check dependencies
       |
       +---- FAIL → add required tablespaces
       |
       +---- PASS → transport
```

---

# 9.12 TTS and READ ONLY

Traditional TTS workflows generally require the relevant user-defined tablespaces to be placed into:

```sql
ALTER TABLESPACE sales READ ONLY;
```

Then generate the transport set.

Oracle's documented full transportable export/import procedure similarly requires user-defined tablespaces to be read-only until export completion. ([Oracle Documentation][2])

---

# 9.13 Cross-Platform Migration

Now the important part.

Suppose:

```text
Windows
   |
   v
Linux
```

You cannot blindly copy Oracle datafiles if the platforms use incompatible endian formats.

Oracle documents cross-platform transport and provides mechanisms including:

```text
DBMS_FILE_TRANSFER
RMAN CONVERT
```

for platform conversion where required. ([Oracle Documentation][2])

---

# 9.14 Endianness

Think:

```text
Platform A
   |
   | Big Endian / Little Endian
   |
   v
Platform B
```

Check supported transport platforms:

```sql
SELECT platform_name,
       endian_format
FROM v$transportable_platform
ORDER BY platform_name;
```

Oracle specifically recommends checking endian format when transporting tablespaces between different platforms. ([Oracle Documentation][4])

---

# 9.15 RMAN CONVERT

For applicable cross-platform datafile migration, RMAN can convert datafiles.

Conceptually:

```text
Source datafile
       |
       v
RMAN CONVERT
       |
       v
Target-compatible datafile
```

Example structure:

```rman
CONVERT DATAFILE
  '/source/oradata/users01.dbf'
TO PLATFORM
  'Linux x86 64-bit'
FROM PLATFORM
  'Microsoft Windows x86 64-bit';
```

**Do not copy this example blindly into production.**

The exact syntax, platform names, file handling and workflow must match the source/target architecture and Oracle documentation.

---

# 9.16 Important Cross-Platform Rule

Different platforms can have different:

```text
Endian format
Filesystem paths
Oracle Home layout
OS users/groups
Permissions
Network configuration
Service management
```

Therefore:

```text
Windows → Linux
```

is not simply:

```text
copy C:\oracle → /u01
```

---

# 9.17 Windows → Linux Migration

Typical architecture:

```text
WINDOWS
Oracle DB
   |
   | migration
   v
LINUX
New Oracle Home
New database environment
```

Possible methods:

```text
Data Pump
Transportable Tablespaces
Full Transportable Export/Import
RMAN cross-platform methods
```

The best method depends on:

```text
Database size
Downtime
Platform compatibility
Object types
Application dependencies
```

Oracle explicitly documents transporting data across platforms and the associated compatibility considerations. ([Oracle Documentation][2])

---

# 9.18 Linux → Windows

The same principles apply:

```text
Linux
   |
   v
Windows
```

But you must account for:

```text
Filesystem paths
Windows services
Oracle Home
ACLs
Listener
Scheduled jobs
Scripts
Shell differences
```

A migration is not complete just because the database opens.

---

# 9.19 Full Transportable Export/Import

Full transportable export/import can move an entire database using a combination of:

```text
Datafiles
+
Data Pump metadata
```

Oracle documents it as a migration method for moving an entire database and as an upgrade method for supported source/target releases. ([Oracle Documentation][2])

---

# 9.20 Full Transportable Concept

```text
SOURCE DATABASE
       |
       +---- User datafiles
       |
       +---- Data Pump metadata
                    |
                    v
              TARGET DATABASE
```

This can be substantially faster than traditional logical export/import for large databases.

---

# 9.21 Full Transportable Example

Oracle's documented pattern uses:

```bash
expdp user_name \
full=y \
dumpfile=expdat.dmp \
directory=data_pump_dir \
transportable=always \
logfile=export.log
```

The important parameter is:

```text
TRANSPORTABLE=ALWAYS
```

for the documented full transportable export workflow. ([Oracle Documentation][2])

---

# 9.22 Compatibility

Migration isn't only about:

```text
Oracle version
```

You must consider:

```text
COMPATIBLE
Character set
Time zone file
Endian format
Block size
Object compatibility
Platform
```

Oracle calculates compatibility requirements for transport operations and documents specific source/target requirements. ([Oracle Documentation][6])

---

# 9.23 Character Set

Before transport/migration:

```sql
SELECT parameter,
       value
FROM nls_database_parameters
WHERE parameter IN
      ('NLS_CHARACTERSET',
       'NLS_NCHAR_CHARACTERSET');
```

Compare source and target.

For some transport scenarios, Oracle requires identical character sets and time-zone settings. ([Oracle Documentation][7])

---

# 9.24 Time Zone File

Check:

```sql
SELECT * FROM v$timezone_file;
```

Why care?

Because different time-zone file versions can affect:

```text
TIMESTAMP WITH TIME ZONE
```

and transport/upgrade compatibility.

Oracle specifically calls out time-zone considerations for transportable migration and AutoUpgrade. ([Oracle Documentation][7])

---

# 9.25 Database Upgrade

Now let's focus on an actual Oracle upgrade.

Example:

```text
Oracle 12c
    |
    v
Oracle 19c
```

or:

```text
Oracle 18c
    |
    v
Oracle 19c
```

Oracle 19c documentation provides dedicated upgrade paths and procedures. ([Oracle Documentation][8])

---

# 9.26 Oracle Homes

Modern upgrade strategy strongly favors separate Oracle Homes.

Example:

```text
OLD HOME
/u01/app/oracle/product/12.2/dbhome_1

NEW HOME
/u01/app/oracle/product/19.0/dbhome_1
```

Conceptually:

```text
OLD DB
  |
  +---- OLD HOME

              NEW HOME
                 |
                 v
             NEW DB
```

This is cleaner than destroying the old environment first.

---

# 9.27 Why Separate Homes?

Advantages:

```text
Old home remains available
New home can be patched independently
Rollback is easier
Testing is cleaner
Multiple versions can coexist
```

---

# 9.28 Pre-Upgrade Analysis

Before upgrading:

```text
DO NOT:
Install new Oracle Home
↓
Immediately upgrade database
```

Instead:

```text
Install new home
      ↓
Patch new home
      ↓
Analyze source database
      ↓
Fix issues
      ↓
Upgrade
```

Oracle recommends AutoUpgrade's **Analyze** mode for pre-upgrade checking. ([Oracle Documentation][9])

---

# 9.29 AutoUpgrade

This is a critical DBA skill.

Oracle describes AutoUpgrade as a tool that automates:

```text
Pre-upgrade checks
Fixups
Upgrade
Post-upgrade checks
Configuration migration
```

and can upgrade multiple databases using one configuration file. ([Oracle Documentation][10])

---

# 9.30 AutoUpgrade Modes

Know these four:

```text
ANALYZE
FIXUPS
DEPLOY
UPGRADE
```

Oracle documents these as AutoUpgrade processing modes. ([Oracle Documentation][10])

Think:

```text
ANALYZE
   ↓
FIXUP
   ↓
DEPLOY
   ↓
VALIDATE
```

---

# 9.31 Analyze Mode

Analyze means:

> "Is my database ready?"

Example:

```bash
java -jar autoupgrade.jar \
-config config.cfg \
-mode analyze
```

Oracle states that Analyze mode reads information from the database without performing database updates. ([Oracle Documentation][9])

---

# 9.32 AutoUpgrade Configuration

Conceptually:

```text
config.cfg

global.autoupg_log_dir=/u01/autoupgrade/logs

upg1.sid=ORCL
upg1.source_home=/u01/app/oracle/product/19.0.0/dbhome_old
upg1.target_home=/u01/app/oracle/product/19.0.0/dbhome_new
upg1.target_version=19
```

**Important:** The exact parameters depend on your source/target release and upgrade scenario. Use Oracle's current AutoUpgrade documentation and the current AutoUpgrade version rather than copying a generic configuration into production. ([Oracle Documentation][10])

---

# 9.33 AutoUpgrade Deploy

After successful analysis and remediation:

```bash
java -jar autoupgrade.jar \
-config config.cfg \
-mode deploy
```

AutoUpgrade then performs the upgrade workflow.

Oracle describes Deploy as performing the required upgrade processing, followed by post-upgrade checks and actions. ([Oracle Documentation][10])

---

# 9.34 AutoUpgrade Workflow

Memorize:

```text
                 AUTOUPGRADE
                      |
                      v
                  ANALYZE
                      |
                      v
                  FIX ISSUES
                      |
                      v
                  FIXUPS
                      |
                      v
                  DEPLOY
                      |
                      v
                   UPGRADE
                      |
                      v
              POST-UPGRADE CHECKS
                      |
                      v
                  VALIDATION
```

---

# 9.35 AutoUpgrade on Windows

The concept is the same.

Example:

```cmd
java -jar C:\oracle\autoupgrade\autoupgrade.jar ^
-config C:\oracle\autoupgrade\config.cfg ^
-mode analyze
```

Then:

```cmd
java -jar C:\oracle\autoupgrade\autoupgrade.jar ^
-config C:\oracle\autoupgrade\config.cfg ^
-mode deploy
```

Windows-specific issues include:

```text
Oracle service names
Windows paths
File permissions
Environment variables
Java availability
Listener/service configuration
```

Oracle's upgrade process itself is cross-platform in concept, but the OS implementation details differ.

---

# 9.36 AutoUpgrade on Linux

Typical:

```bash
java -jar /u01/autoupgrade/autoupgrade.jar \
-config /u01/autoupgrade/config.cfg \
-mode analyze
```

Then:

```bash
java -jar /u01/autoupgrade/autoupgrade.jar \
-config /u01/autoupgrade/config.cfg \
-mode deploy
```

Again:

> Use the latest AutoUpgrade version recommended by Oracle.

Oracle strongly recommends downloading the latest AutoUpgrade utility from MOS; current 19c homes include an AutoUpgrade version, but the latest version may contain important fixes and enhancements. ([Oracle Documentation][9])

---

# 9.37 Non-CDB → PDB

This is extremely important for Oracle 19c.

Oracle 19c is the **terminal release for non-CDB architecture upgrades to non-CDB architecture**. Oracle provides supported paths for moving non-CDB databases into the multitenant architecture. ([Oracle Documentation][11])

Conceptually:

```text
OLD NON-CDB
     |
     v
TARGET CDB
     |
     v
PDB
```

---

# 9.38 Two Major Non-CDB → PDB Approaches

Oracle documents two approaches:

### Approach A

```text
Non-CDB
   ↓
Convert to PDB
   ↓
Upgrade CDB/PDB
```

### Approach B

```text
Non-CDB
   ↓
Plug into target CDB
   ↓
Upgrade
   ↓
Finish conversion
```

Oracle documents both approaches and the related AutoUpgrade methods. ([Oracle Documentation][11])

---

# 9.39 PDB Unplug/Plug Upgrade

Another powerful method:

```text
SOURCE CDB
    |
    +-- PDB
          |
          | UNPLUG
          v
      PDB XML/metadata
          |
          | PLUG
          v
TARGET CDB
    |
    +-- PDB
          |
          v
       Upgrade
```

Oracle's AutoUpgrade documentation specifically supports unplug/plug upgrades. ([Oracle Documentation][10])

---

# 9.40 Migration Using PDB Technology

For example:

```text
CDB19
 |
 +-- SALES
 +-- HR
```

You can move:

```text
SALES
```

to another appropriate CDB.

This can be much more targeted than migrating an entire database.

---

# 9.41 RMAN-Based Migration

RMAN is primarily a physical backup/recovery technology, but physical backups can be used as the foundation of migration workflows.

Conceptually:

```text
SOURCE
   |
   | RMAN backup
   v
Backup
   |
   | restore
   v
TARGET
```

For cross-platform scenarios, RMAN conversion may be involved.

---

# 9.42 Data Guard Migration

For environments requiring low downtime, Data Guard can be used as part of a migration/upgrade strategy.

Conceptually:

```text
SOURCE PRIMARY
       |
       | redo
       v
STANDBY
       |
       v
Prepare new environment
       |
       v
Switch/Roll transition
```

Oracle documents Data Guard transient standby as one possible upgrade technology. ([Oracle Documentation][11])

---

# 9.43 GoldenGate Migration

For more advanced near-zero-downtime scenarios:

```text
SOURCE
   |
   | GoldenGate replication
   v
TARGET
```

Data changes continue flowing while the target environment is prepared.

Then:

```text
SOURCE
   |
   | final synchronization
   v
TARGET
   |
   v
Application cutover
```

Oracle lists GoldenGate-based synchronization as a technology for low-downtime upgrade/migration scenarios. ([Oracle Documentation][11])

---

# 9.44 Choosing a Migration Method

Use this table mentally:

| Requirement                       | Likely approach                             |
| --------------------------------- | ------------------------------------------- |
| Small database                    | Data Pump                                   |
| Schema migration                  | Data Pump                                   |
| Large database                    | TTS / Full Transportable / physical methods |
| Same datafiles can be transported | TTS                                         |
| Entire database migration         | Full Transportable                          |
| Cross-platform                    | TTS + conversion where required             |
| New Oracle release                | AutoUpgrade / supported upgrade method      |
| Non-CDB → PDB                     | AutoUpgrade / plug-convert methods          |
| Very low downtime                 | Data Guard / GoldenGate architecture        |
| Multiple databases                | AutoUpgrade / fleet tooling                 |

The actual method must be validated against the source/target versions and platform combination.

---

# 9.45 Migration Precheck

Before migration:

```text
[ ] Source version
[ ] Target version
[ ] Source OS
[ ] Target OS
[ ] Endian format
[ ] Character set
[ ] Time zone
[ ] COMPATIBLE
[ ] Database size
[ ] Tablespace list
[ ] Object inventory
[ ] Database links
[ ] External directories
[ ] External jobs/scripts
[ ] Application dependencies
[ ] Encryption/TDE
[ ] Wallet/keystore
[ ] Backup
[ ] Rollback plan
```

---

# 9.46 Upgrade Precheck

Before upgrade:

```text
[ ] Backup verified
[ ] New Oracle Home installed
[ ] New Oracle Home patched
[ ] Latest appropriate AutoUpgrade obtained
[ ] Analyze completed
[ ] Blocking issues fixed
[ ] Invalid objects reviewed
[ ] Time zone checked
[ ] Character set checked
[ ] Application compatibility checked
[ ] Space checked
[ ] Listener checked
[ ] Database links tested
[ ] Jobs reviewed
[ ] Recovery strategy tested
```

---

# 9.47 Upgrade Does Not Mean "Change COMPATIBLE Immediately"

This is a common beginner mistake.

There is a difference between:

```text
Oracle software version
```

and:

```text
COMPATIBLE initialization parameter
```

Do not change `COMPATIBLE` casually as part of an upgrade.

Understand the implications and follow the target release's upgrade documentation.

---

# 9.48 Post-Upgrade Checks

After upgrade:

```sql
SELECT banner_full
FROM v$version;
```

Database:

```sql
SELECT name,
       open_mode,
       database_role
FROM v$database;
```

Instance:

```sql
SELECT instance_name,
       status,
       version
FROM v$instance;
```

Invalid objects:

```sql
SELECT owner,
       object_name,
       object_type
FROM dba_objects
WHERE status <> 'VALID';
```

---

# 9.49 Component Status

Check:

```sql
SELECT comp_id,
       comp_name,
       version,
       status
FROM dba_registry
ORDER BY comp_id;
```

You want to investigate components that aren't in the expected state.

---

# 9.50 SQL Patch Verification After Upgrade

```sql
SELECT patch_id,
       patch_type,
       action,
       status,
       action_time,
       description
FROM dba_registry_sqlpatch
ORDER BY action_time DESC;
```

This should be part of your post-upgrade checklist.

---

# 9.51 Application Validation

DBA validation is not enough.

Test:

```text
Application login
     ↓
Connection pool
     ↓
SELECT
     ↓
INSERT
     ↓
UPDATE
     ↓
Stored procedures
     ↓
Reports
     ↓
Batch jobs
```

A database can report:

```text
OPEN
```

while the application is broken.

---

# 9.52 Database Links

Check:

```sql
SELECT owner,
       db_link,
       username,
       host
FROM dba_db_links;
```

Test critical links after migration.

Why?

Because:

```text
Old hostname
Old service
Old listener
Old wallet
Old credentials
```

may no longer work.

---

# 9.53 DIRECTORY Objects

Check:

```sql
SELECT owner,
       directory_name,
       directory_path
FROM dba_directories
ORDER BY owner, directory_name;
```

Windows might have:

```text
C:\oracle\dump
```

while Linux requires:

```text
/u01/app/oracle/dump
```

The database object may need to be recreated/adjusted on the target.

---

# 9.54 External Jobs

Migration can break:

```text
Shell scripts
Batch files
Cron
Windows Task Scheduler
External tables
External procedures
File-system integrations
```

Therefore:

```text
DATABASE MIGRATION
       ≠
DATABASE FILE COPY ONLY
```

---

# 9.55 Migration Runbook

### Phase 1 — Discovery

```text
Source inventory
Target inventory
Application inventory
Dependencies
```

### Phase 2 — Design

```text
Migration method
Downtime
Cutover
Rollback
```

### Phase 3 — Build

```text
Target OS
Oracle Home
Patches
Database
Networking
Storage
Security
```

### Phase 4 — Test

```text
Dry run
Performance test
Application test
Recovery test
```

### Phase 5 — Migration

```text
Final backup
Final sync
Data migration
Metadata migration
Validation
```

### Phase 6 — Cutover

```text
Stop source
Final synchronization
Start target
Application points to target
Validation
```

### Phase 7 — Post-Migration

```text
Monitor
Backup
Performance
Application
Security
Documentation
```

---

# 9.56 Upgrade Runbook

```text
SOURCE DATABASE
      |
      v
BACKUP
      |
      v
INSTALL NEW ORACLE HOME
      |
      v
PATCH NEW HOME
      |
      v
AUTOUPGRADE ANALYZE
      |
      v
FIX BLOCKERS
      |
      v
TEST
      |
      v
UPGRADE
      |
      v
POST-UPGRADE FIXUPS
      |
      v
VALIDATE
      |
      v
APPLICATION TEST
      |
      v
PRODUCTION COMPLETE
```

---

# 9.57 Rollback Strategy

This must be decided **before** migration.

For an upgrade:

```text
OLD DATABASE
      |
      v
NEW DATABASE
```

If the upgrade fails:

```text
Can we restore?
Can we switch back?
Can we revert application connections?
Can we restore old Oracle Home?
Can we recover source database?
```

The answer depends heavily on the chosen upgrade technology.

Do not promise:

> "We can always downgrade."

A database upgrade is not automatically reversible.

---

# 9.58 Why Out-of-Place Is Valuable

For example:

```text
OLD HOME
/u01/oracle/dbhome_old

NEW HOME
/u01/oracle/dbhome_new
```

Then:

```text
             +----------------+
             | OLD HOME       |
             | Keep intact    |
             +----------------+
                     |
                     |
Database ------------+-----> NEW HOME
                              |
                              v
                         New release
```

This gives you a much cleaner operational separation.

---

# 9.59 Windows vs Linux Migration Checklist

### Windows

```text
Oracle Services
Windows paths
ACLs
Event Viewer
Scheduled Tasks
PowerShell/batch scripts
Drive letters
Registry/configuration dependencies
```

### Linux

```text
oracle user
OS groups
File permissions
mount points
/etc/fstab
systemd
cron
shell scripts
SELinux where applicable
```

---

# 9.60 Migration Example — Windows → Linux

Suppose:

```text
SOURCE
Windows
Oracle 19c
DB = SALES
Size = 2 TB
```

Target:

```text
Linux
Oracle 19c
```

Possible approach:

```text
1. Compare platforms
2. Check endian format
3. Choose TTS/full transportable or another suitable method
4. Check character set/time zone
5. Prepare target
6. Generate transport metadata
7. Convert datafiles if required
8. Copy datafiles
9. Import metadata
10. Validate
11. Cut over
```

Oracle specifically supports transporting data across platforms, subject to compatibility and platform limitations. ([Oracle Documentation][2])

---

# 9.61 Example — 19c Upgrade Using AutoUpgrade

```text
OLD HOME
   |
   | source database
   v
ORCL
   |
   +----------------------+
                          |
                    NEW HOME
                          |
                       19c RU
                          |
                          v
                    AutoUpgrade
                          |
                          v
                     ANALYZE
                          |
                          v
                       DEPLOY
                          |
                          v
                     ORCL 19c
```

Oracle's documented AutoUpgrade workflow is specifically designed to automate pre-upgrade, upgrade, and post-upgrade operations. ([Oracle Documentation][10])

---

# 9.62 Lesson 9 Practical Lab

Create this lab architecture:

```text
SERVER
 |
 +-- OLD_ORACLE_HOME
 |
 +-- NEW_ORACLE_HOME
 |
 +-- TESTDB
```

### Step 1

Check source:

```sql
SELECT banner_full
FROM v$version;
```

### Step 2

Check database:

```sql
SELECT name,
       open_mode
FROM v$database;
```

### Step 3

Check components:

```sql
SELECT comp_id,
       comp_name,
       version,
       status
FROM dba_registry;
```

### Step 4

Run AutoUpgrade Analyze:

```bash
java -jar autoupgrade.jar \
-config config.cfg \
-mode analyze
```

### Step 5

Read the generated report.

### Step 6

Fix blocking issues.

### Step 7

Run Analyze again.

### Step 8

Only after successful testing:

```bash
java -jar autoupgrade.jar \
-config config.cfg \
-mode deploy
```

### Step 9

Verify:

```sql
SELECT banner_full
FROM v$version;
```

### Step 10

Check:

```sql
SELECT comp_id,
       status
FROM dba_registry;
```

---

# 9.63 Lesson 9 Interview Questions

You should be able to answer all of these:

1. Migration vs upgrade?
2. Logical vs physical migration?
3. What is Data Pump?
4. `expdp` vs `impdp`?
5. Full Data Pump vs schema Data Pump?
6. What is Transportable Tablespaces?
7. Why is TTS faster?
8. What is a self-contained tablespace set?
9. What is Full Transportable Export/Import?
10. What is cross-platform migration?
11. What is endian format?
12. How do you check platform endian format?
13. What is `V$TRANSPORTABLE_PLATFORM`?
14. What is RMAN `CONVERT`?
15. What is AutoUpgrade?
16. What is AutoUpgrade Analyze mode?
17. What is AutoUpgrade Deploy mode?
18. Why use a new Oracle Home?
19. What is non-CDB → PDB migration?
20. What is unplug/plug?
21. How can Data Guard assist upgrades?
22. How can GoldenGate assist migration?
23. What should be checked before migration?
24. What should be checked after migration?
25. How do Windows and Linux migrations differ?
26. Why are character sets important?
27. Why are time-zone files important?
28. What is `COMPATIBLE`?
29. What is your rollback plan?
30. How do you validate an upgraded database?

---

# 9.64 The DBA Migration Decision Tree

Memorize this:

```text
                 NEED MIGRATION?
                       |
                       v
                SAME PLATFORM?
                 /           \
               YES            NO
                |              |
                v              v
          Logical/Physical   Check endian
                |              |
                v              v
            Database size   TTS/RMAN/etc.
                |
        +-------+-------+
        |               |
      Small            Large
        |               |
    Data Pump       TTS/Physical
```

For upgrades:

```text
              NEED UPGRADE?
                    |
                    v
             NEW ORACLE HOME
                    |
                    v
              PATCH NEW HOME
                    |
                    v
             AUTOUPGRADE
                    |
                    v
                 ANALYZE
                    |
                    v
              FIX BLOCKERS
                    |
                    v
                 TEST
                    |
                    v
                 DEPLOY
                    |
                    v
               VALIDATE
```

---

# 9.65 Golden Rules of Migration & Upgrade

### Rule 1

**Never start production migration without a tested rollback plan.**

### Rule 2

**Never assume a database being OPEN means the migration succeeded.**

### Rule 3

**Always compare source and target compatibility requirements.**

### Rule 4

**For cross-platform transport, check endian format.**

### Rule 5

**For transportable methods, verify character set and time-zone requirements.**

### Rule 6

**Use a separate/new Oracle Home for major upgrades whenever the supported procedure allows it.**

### Rule 7

**Use AutoUpgrade Analyze before an AutoUpgrade deployment.**

### Rule 8

**Test the application, not just the database.**

### Rule 9

**Keep the source available until business validation and rollback criteria are satisfied.**

### Rule 10

**Document every migration.**

---

## Official Oracle sources

* [Oracle Database 19c — Transporting Data](https://docs.oracle.com/en/database/oracle/oracle-database/19/admin/transporting-data.html) — Oracle's official documentation for transportable tablespaces, full transportable export/import and cross-platform transport. ([Oracle Documentation][2])
* [Oracle Database 19c — Using AutoUpgrade](https://docs.oracle.com/en/database/oracle/oracle-database/19/upgrd/using-autoupgrade-oracle-database-upgrades.html) — AutoUpgrade modes, workflow and configuration. ([Oracle Documentation][10])
* [Oracle Database 19c — Choosing an Upgrade Method](https://docs.oracle.com/en/database/oracle/oracle-database/19/upgrd/choose-an-upgrade-method-for-oracle-database.html) — Oracle's documented upgrade options. ([Oracle Documentation][1])
* [Oracle Database 19c — Preparing to Upgrade](https://docs.oracle.com/en/database/oracle/oracle-database/19/spuss/preparing-to-upgrade.html) — Pre-upgrade analysis and AutoUpgrade recommendations. ([Oracle Documentation][9])
* [Oracle Database 19c — Migrating Oracle Database](https://docs.oracle.com/en/database/oracle/oracle-database/19/spmdu/migrating-oracle-database.html) — Oracle's migration procedures and transportable technologies. ([Oracle Documentation][5])

**Lesson 9/10 complete.**

### Course progress

```text
1. Oracle Architecture & DBA Foundations       ✓
2. Installation & Database Creation             ✓
3. Configuration & Oracle Networking            ✓
4. Core DBA Administration                     ✓
5. Backup & Recovery / RMAN                    ✓
6. Performance Monitoring & Troubleshooting   ✓
7. Security & Advanced Administration         ✓
8. Patching & Patch Management                ✓
9. Migration & Database Upgrades              ✓
10. RAC, Data Guard & End-to-End DBA Operations
```

**Next: Lesson 10/10 — RAC + Data Guard + End-to-End Oracle DBA Operations**, including **RAC architecture, Grid Infrastructure, ASM, services, RAC administration, Data Guard architecture, physical standby, redo transport, switchover/failover, Broker, troubleshooting, Windows/Linux considerations, and a complete production DBA operating model.**

[1]: https://docs.oracle.com/en/database/oracle/oracle-database/19/upgrd/choose-an-upgrade-method-for-oracle-database.html "Choose an Upgrade Method for Oracle Database"
[2]: https://docs.oracle.com/en/database/oracle/oracle-database/19/admin/transporting-data.html "Transporting Data"
[3]: https://docs.oracle.com/en/database/oracle/oracle-database/19/sutil/oracle-data-pump-overview.html "Overview of Oracle Data Pump"
[4]: https://docs.oracle.com/en/database/oracle/oracle-database/19/spmdu/transporting-tablespaces-between-databases.html "Transporting Tablespaces Between Databases"
[5]: https://docs.oracle.com/en/database/oracle/oracle-database/19/spmdu/migrating-oracle-database.html "Migrating Oracle Database"
[6]: https://docs.oracle.com/en/database/oracle/oracle-database/19/spmds/compatibility-considerations-for-transporting-data.html "Compatibility Considerations for Transporting Data"
[7]: https://docs.oracle.com/en/database/oracle/oracle-database/19/upgrd/using-preupgrade-information-tool-for-oracle-database.html?source=%3Aow%3Apf%3Acta%3A%3A%3A%3ARC_WWSA220204P00012%3AEM22_TEC_EV_CO65_M4401_SO016YO25_DO1044_AO01_RO001&utm_source=chatgpt.com "Pre-Upgrade Information Tool and AutoUpgrade Preupgrade"
[8]: https://docs.oracle.com/en/database/oracle/oracle-database/19/spuss/upgrading-db.html "Upgrading Oracle Database"
[9]: https://docs.oracle.com/en/database/oracle/oracle-database/19/spuss/preparing-to-upgrade.html "Preparing to Upgrade Oracle Database"
[10]: https://docs.oracle.com/en/database/oracle/oracle-database/19/upgrd/using-autoupgrade-oracle-database-upgrades.html "Using AutoUpgrade for Oracle Database Upgrades"
[11]: https://docs.oracle.com/en/database/oracle/oracle-database/19/upgrd/overview-conveting-databases-during-upgrade.html "Overview of Converting Databases During Upgrades"
