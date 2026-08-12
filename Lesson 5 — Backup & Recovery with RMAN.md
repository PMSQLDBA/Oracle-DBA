# Lesson 5/10 — Backup & Recovery with RMAN

**Strictly Lesson 5 of 10.**

This lesson is based on Oracle's official RMAN/backup-and-recovery documentation. The goal is to understand **how Oracle protects data, how RMAN works, and how a DBA restores and recovers a database after failures**.

> **Critical distinction:**
> **Backup** = a copy of database-related data.
> **Restore** = putting a backup copy back onto storage.
> **Recovery** = applying redo/undo information so the database reaches a consistent state.

---

# 5.1 Why Backup & Recovery Is the DBA's Most Important Responsibility

Imagine:

```text
01:00  Database healthy
02:00  Application writes data
03:00  More transactions
04:00  Datafile accidentally deleted
05:00  DBA discovers problem
```

The question is not:

> "Can we restart Oracle?"

The real question is:

> **"Can we restore the database and recover the committed transactions?"**

That's what RMAN and Oracle recovery architecture are designed to handle.

---

# 5.2 The Recovery Architecture

Think of Oracle recovery as:

```text
                  DATABASE
                     |
          +----------+----------+
          |                     |
       DATAFILES              REDO
          |                     |
          |              Online Redo Logs
          |                     |
          |              Archived Redo
          |                     |
          +----------+----------+
                     |
                  RMAN
                     |
                  BACKUPS
```

RMAN can back up and restore database files and related recovery components.

---

# 5.3 ARCHIVELOG vs NOARCHIVELOG

This is one of the first things you must determine.

Check:

```sql
ARCHIVE LOG LIST;
```

You may see:

```text
Database log mode              ARCHIVELOG
Automatic archival             ENABLED
Archive destination            ...
```

---

# 5.4 NOARCHIVELOG Mode

In `NOARCHIVELOG` mode, Oracle does not archive filled online redo logs for recovery purposes.

The recovery possibilities are therefore much more limited.

Conceptually:

```text
Database
   |
Online Redo
   |
   X
No archived redo
```

This is generally unsuitable for databases requiring robust point-in-time recovery.

---

# 5.5 ARCHIVELOG Mode

In `ARCHIVELOG` mode:

```text
Online Redo Log
       |
       | fills
       v
Archive
       |
       v
Archived Redo Log
```

Archived redo allows Oracle to apply changes after restoring an older backup.

This is fundamental for:

* Point-in-time recovery
* Media recovery
* Many production backup strategies
* Data Guard

---

# 5.6 Checking ARCHIVELOG

Run:

```sql
ARCHIVE LOG LIST;
```

Or:

```sql
SELECT log_mode
FROM v$database;
```

Example:

```text
LOG_MODE
--------
ARCHIVELOG
```

---

# 5.7 Enabling ARCHIVELOG

This is an administrative operation and requires the database to be mounted rather than open.

Typical sequence:

```sql
SHUTDOWN IMMEDIATE;
```

Then:

```sql
STARTUP MOUNT;
```

Then:

```sql
ALTER DATABASE ARCHIVELOG;
```

Then:

```sql
ALTER DATABASE OPEN;
```

Verify:

```sql
ARCHIVE LOG LIST;
```

**Do this only in a lab unless you have an approved production procedure.**

---

# 5.8 What Is RMAN?

RMAN means:

**Recovery Manager**

It is Oracle's database backup and recovery utility.

You normally invoke it from the operating system:

```bash
rman target /
```

For a remote connection:

```bash
rman target sys@ORCL
```

The exact authentication method should follow your organization's security policy.

---

# 5.9 RMAN Architecture

Conceptually:

```text
                  RMAN CLIENT
                       |
                       v
                Target Database
                       |
            +----------+----------+
            |                     |
        Control File        Recovery Catalog
        metadata             (optional)
```

The **target database** is the database being backed up or recovered.

A **recovery catalog** is an optional separate repository for RMAN metadata.

---

# 5.10 RMAN Without a Recovery Catalog

RMAN can use the target database's control file to maintain backup metadata.

Conceptually:

```text
RMAN
 |
 +---- Target Database
          |
          +---- Control File
                    |
                    +---- RMAN metadata
```

This is common in many environments.

---

# 5.11 Recovery Catalog

A recovery catalog is a separate database repository containing RMAN metadata.

Conceptually:

```text
                  RMAN
                   |
        +----------+----------+
        |                     |
     TARGET               CATALOG
     DATABASE             DATABASE
```

The catalog can be particularly useful for managing backups across multiple databases and maintaining RMAN metadata beyond the lifecycle of an individual target control file.

---

# 5.12 Start RMAN

From Linux:

```bash
rman target /
```

On Windows:

```cmd
rman target /
```

You should see an RMAN prompt:

```text
RMAN>
```

Try:

```text
RMAN> SHOW ALL;
```

This displays configured RMAN settings.

---

# 5.13 RMAN Configuration

Run:

```text
RMAN> SHOW ALL;
```

You'll see settings relating to things such as:

```text
Retention policy
Device type
Backup optimization
Controlfile autobackup
Channels
Compression
```

These settings influence how RMAN operates.

---

# 5.14 Retention Policy

A retention policy tells RMAN which backups it needs to retain according to your recovery objective.

Two major concepts are:

```text
REDUNDANCY
```

and:

```text
RECOVERY WINDOW
```

---

# 5.15 Redundancy

Example:

```text
CONFIGURE RETENTION POLICY TO REDUNDANCY 2;
```

Conceptually:

```text
Keep enough backups to maintain
2 backup copies according to RMAN's
redundancy policy.
```

Don't interpret this as:

> "Keep exactly two physical files."

RMAN's retention semantics are based on recoverability and backup metadata.

---

# 5.16 Recovery Window

Example:

```text
CONFIGURE RETENTION POLICY
TO RECOVERY WINDOW OF 7 DAYS;
```

Conceptually:

```text
Today
 |
 +--------------------+
 |                    |
7 days               Older
recoverability       backups
window
```

The goal is to maintain the ability to recover to a point within the defined window.

---

# 5.17 Backup Types

You should know:

```text
Full backup
Incremental backup
Image copy
Backup set
```

---

# 5.18 Backup Set

RMAN commonly creates backup sets.

Conceptually:

```text
Database files
     |
     v
    RMAN
     |
     v
Backup pieces
     |
     v
Backup set
```

The physical storage representation can differ from the logical database structure.

---

# 5.19 Image Copy

An image copy is essentially a copy of a database file.

Conceptually:

```text
datafile
   |
   +---- RMAN image copy
```

Image copies have different operational characteristics from backup sets and can be useful in certain recovery strategies.

---

# 5.20 Full Database Backup

A typical RMAN command:

```text
RMAN> BACKUP DATABASE;
```

This backs up the database according to the configured RMAN behavior.

For a more complete production backup strategy, you generally also consider:

* Archived redo
* Control file
* SPFILE
* Backup validation
* Retention
* Off-host/off-site copies

---

# 5.21 Backup Database Plus Archive Logs

A common conceptual backup operation is:

```text
RMAN> BACKUP DATABASE PLUS ARCHIVELOG;
```

This combines database backup with archived redo.

Why?

Because restoring a database backup without the required archived redo may leave you unable to recover to the desired point.

---

# 5.22 Control File Backup

The control file is critical.

You can enable automatic control-file/SPFILE backups:

```text
RMAN> CONFIGURE CONTROLFILE AUTOBACKUP ON;
```

Then:

```text
RMAN> SHOW ALL;
```

Verify the setting.

---

# 5.23 Why Control File Matters

The control file contains important structural metadata about the database, including information required for database operation and recovery.

Think:

```text
CONTROL FILE
    |
    +-- Database structure
    +-- Datafile information
    +-- Redo information
    +-- Checkpoint information
    +-- Backup/recovery metadata
```

Therefore:

> Losing the control file is not equivalent to merely losing a normal application table.

---

# 5.24 SPFILE Backup

RMAN can back up the SPFILE.

For example:

```text
RMAN> BACKUP SPFILE;
```

You should understand why this matters.

Suppose you lose:

```text
SPFILE
```

You may have the database files but still need the initialization configuration to start the instance correctly.

---

# 5.25 Backup Validation

A backup that was never tested is not something you should blindly trust.

RMAN provides validation mechanisms.

For example:

```text
RMAN> VALIDATE DATABASE;
```

You can also validate backup files.

The exact validation command should be selected based on what you're trying to verify.

---

# 5.26 LIST BACKUP

To inspect RMAN backups:

```text
RMAN> LIST BACKUP;
```

You can also inspect:

```text
RMAN> LIST BACKUP SUMMARY;
```

This is an essential DBA command.

---

# 5.27 CROSSCHECK

RMAN's metadata can become inconsistent with actual storage.

For example:

```text
RMAN thinks backup exists
          |
          v
Backup file was deleted externally
```

Use:

```text
RMAN> CROSSCHECK BACKUP;
```

This checks RMAN records against the physical backup availability.

---

# 5.28 DELETE EXPIRED

After crosschecking:

```text
RMAN> DELETE EXPIRED BACKUP;
```

Important:

**EXPIRED** does not simply mean "old."

It generally means RMAN expected the backup to exist but could not find it during crosscheck.

This distinction is extremely important.

---

# 5.29 OBSOLETE vs EXPIRED

### EXPIRED

```text
RMAN metadata says:
"I expect this backup to exist."

Crosscheck says:
"I can't find it."
```

### OBSOLETE

```text
Backup exists,
but according to the retention policy,
it is no longer required.
```

Therefore:

```text
EXPIRED ≠ OBSOLETE
```

Memorize this.

---

# 5.30 Delete Obsolete Backups

After establishing an appropriate retention policy:

```text
RMAN> DELETE OBSOLETE;
```

RMAN evaluates configured retention rules.

Never use deletion commands casually in production.

---

# 5.31 Restore vs Recovery

This is probably the most important conceptual distinction in this lesson.

### RESTORE

Get files back.

```text
Backup
  ↓
Restore
  ↓
Datafile
```

### RECOVER

Apply changes after restoration.

```text
Restored Datafile
       +
Archived Redo
       +
Online Redo
       |
       v
Recovered Datafile
```

---

# 5.32 Example: Lost Datafile

Suppose:

```text
/u02/oradata/ORCL/app_data01.dbf
```

is accidentally deleted.

The database may report an error indicating the file is missing.

Your conceptual procedure is:

```text
1. Identify missing file
2. Determine backup availability
3. Restore file
4. Recover file
5. Verify database
```

Not:

```text
restart database
```

---

# 5.33 RMAN Restore

A typical RMAN restore:

```text
RMAN> RESTORE DATAFILE 7;
```

Then recovery:

```text
RMAN> RECOVER DATAFILE 7;
```

The actual datafile number must be determined from your database.

Find it:

```sql
SELECT file_id,
       file_name,
       tablespace_name
FROM dba_data_files
ORDER BY file_id;
```

---

# 5.34 Whole Database Restore

Conceptually:

```text
RMAN> RESTORE DATABASE;
```

This restores database files from backup.

It does **not automatically mean the database has been recovered to the desired point**.

That is where:

```text
RECOVER DATABASE;
```

comes in.

---

# 5.35 Complete Recovery

Complete recovery aims to bring the database forward using available redo.

Conceptually:

```text
Backup
  |
  v
Restore
  |
  +---- Archived Redo
  |
  +---- Online Redo
  |
  v
Current/desired consistent state
```

This is why archived redo is so important.

---

# 5.36 Point-in-Time Recovery

Suppose:

```text
10:00  Database healthy
11:00  Bad deployment
11:15  Bad data committed
11:20  DBA discovers problem
```

You may need to recover to a point before the bad change.

Conceptually:

```text
10:00 ---- 11:00 ---- 11:15 ---- 11:20
             |
             |
       Desired recovery
           point
```

This is called:

**Point-in-Time Recovery (PITR).**

The exact RMAN syntax depends on the recovery scenario and Oracle release/configuration, so don't memorize a single command without understanding the recovery procedure.

---

# 5.37 Why PITR Is Dangerous

Point-in-time recovery can mean losing legitimate transactions that occurred after the chosen recovery point.

For example:

```text
11:01 Valid transaction
11:05 Valid transaction
11:10 Bad transaction
11:11 Valid transaction
```

Recovering to:

```text
11:09
```

would remove:

```text
11:10
11:11
```

and everything after that point.

Therefore PITR requires:

* Business approval
* Correct target time/SCN
* Recovery planning
* Validation
* Post-recovery application checks

---

# 5.38 SCN

Oracle uses a:

**System Change Number (SCN)**

to identify points in database change history.

Conceptually:

```text
SCN 100
SCN 200
SCN 300
SCN 400
...
```

Recovery can be targeted using time or SCN depending on the scenario.

SCNs become particularly important in advanced recovery and Data Guard operations.

---

# 5.39 Recovery to a Different Time

A recovery scenario might look like:

```text
Backup
  ↓
Restore
  ↓
Apply redo
  ↓
Stop at target time
  ↓
Open database with appropriate recovery procedure
```

Do **not** experiment with incomplete recovery commands on a production database.

Practice them in a disposable lab.

---

# 5.40 Database Corruption

Corruption can be:

```text
Physical corruption
Logical corruption
```

Physical corruption involves problems with the physical contents of database blocks.

Logical corruption involves blocks that are physically readable but contain logically invalid information.

RMAN can help detect certain types of corruption.

For example:

```text
RMAN> VALIDATE DATABASE;
```

You should know how to distinguish:

```text
Backup problem
Storage problem
Oracle block corruption
Application/data problem
```

---

# 5.41 RMAN Validation

Examples include:

```text
RMAN> VALIDATE DATABASE;
```

and:

```text
RMAN> VALIDATE BACKUPSET <backupset_number>;
```

The appropriate command depends on the object you want to validate.

Validation should be part of a serious backup strategy.

---

# 5.42 Backup Strategy

A production backup strategy should answer:

```text
What?
When?
Where?
How often?
How long retained?
How quickly restored?
How often tested?
```

For example:

```text
Database
   |
   +-- Daily backup
   |
   +-- Archived redo
   |
   +-- Control file
   |
   +-- SPFILE
   |
   +-- Offsite copy
   |
   +-- Retention policy
   |
   +-- Restore testing
```

---

# 5.43 RPO and RTO

You must understand these two terms.

### RPO

**Recovery Point Objective**

How much data loss is acceptable?

Example:

```text
RPO = 15 minutes
```

means the business may tolerate losing approximately 15 minutes of changes, depending on the recovery architecture.

---

### RTO

**Recovery Time Objective**

How quickly must service be restored?

Example:

```text
RTO = 1 hour
```

means the business requires service restoration within approximately one hour.

---

# 5.44 Backup Strategy Must Follow Business Requirements

Don't start with:

> "Oracle recommends this backup schedule."

Start with:

```text
Business requirement
       ↓
RPO
RTO
       ↓
Recovery architecture
       ↓
Backup strategy
       ↓
Testing
```

A backup that takes 12 hours to restore isn't useful to a business requiring a 30-minute RTO.

---

# 5.45 FRA — Fast Recovery Area

Oracle provides the concept of the:

**Fast Recovery Area (FRA)**

It is an Oracle-managed location for recovery-related files.

Check:

```sql
SHOW PARAMETER db_recovery_file_dest;
```

And:

```sql
SHOW PARAMETER db_recovery_file_dest_size;
```

Conceptually:

```text
                 FRA
                  |
       +----------+----------+
       |          |          |
   Archived    Backups    Flashback
     Logs                  Files
```

The actual contents depend on configuration and Oracle features in use.

---

# 5.46 FRA Monitoring

Check:

```sql
SELECT name,
       space_limit,
       space_used,
       space_reclaimable
FROM v$recovery_file_dest;
```

This is an important daily monitoring query.

If FRA fills up, archived log generation and database operations can be affected depending on the situation.

---

# 5.47 FRA Full Scenario

Imagine:

```text
FRA = 100% full
```

Don't simply:

```text
rm *.arc
```

from the operating system.

That can create RMAN metadata problems and potentially remove recovery data that is still needed.

Instead investigate:

```text
Why is FRA full?
       |
       +-- Archived logs?
       +-- Backups?
       +-- Flashback logs?
       +-- Retention?
       +-- Backup failure?
       +-- Data Guard lag?
```

Then use Oracle-supported recovery-space management procedures.

---

# 5.48 Archived Log Management

Check archived logs:

```sql
SELECT sequence#,
       first_time,
       next_time,
       applied,
       deleted
FROM v$archived_log
ORDER BY sequence# DESC;
```

For RMAN:

```text
RMAN> LIST ARCHIVELOG ALL;
```

The exact columns and availability depend on database configuration and version.

---

# 5.49 Backup Monitoring

A DBA shouldn't merely run:

```text
BACKUP DATABASE;
```

and walk away.

You need to know:

```text
Did it start?
Did it finish?
Did it fail?
How long did it take?
How much data was backed up?
Where is the backup?
Can it be restored?
```

RMAN output and backup metadata are part of operational monitoring.

---

# 5.50 Disaster Scenario

Imagine the following:

```text
Server
 |
 +-- Database
      |
      +-- Datafiles
      +-- Control files
      +-- Redo
```

Then:

```text
SERVER LOST
```

A serious recovery plan needs:

```text
New server
    ↓
Oracle software
    ↓
Configuration
    ↓
Backup access
    ↓
Control file / metadata
    ↓
Restore database
    ↓
Recover database
    ↓
Validate
    ↓
Application reconnect
```

This is why **backup alone isn't disaster recovery**.

---

# 5.51 Backup vs High Availability

Don't confuse:

```text
Backup
```

with:

```text
High Availability
```

Backup helps you recover from data loss.

High availability aims to keep service available despite failures.

Examples of Oracle technologies/architectures include:

```text
RMAN
Data Guard
RAC
```

They solve different problems.

We'll cover Data Guard later in **Lesson 10**.

---

# 5.52 Windows RMAN

RMAN works on Windows too.

Open Command Prompt:

```cmd
rman target /
```

Or use the appropriate Oracle environment.

Check Oracle services:

```powershell
Get-Service *Oracle*
```

Check disk availability:

```powershell
Get-Volume
```

The RMAN commands themselves remain Oracle commands.

---

# 5.53 Linux RMAN

Linux example:

```bash
export ORACLE_SID=ORCL
```

Then:

```bash
rman target /
```

Check storage:

```bash
df -h
```

Check Oracle environment:

```bash
echo $ORACLE_HOME
echo $ORACLE_SID
```

Remember:

```text
Oracle recovery
      +
OS storage
      +
OS permissions
```

must all work together.

---

# 5.54 Lesson 5 Lab — RMAN Basics

**Use a disposable lab database.**

Start:

```bash
rman target /
```

Then:

```text
RMAN> SHOW ALL;
```

Check database:

```text
RMAN> REPORT SCHEMA;
```

List backups:

```text
RMAN> LIST BACKUP SUMMARY;
```

Crosscheck:

```text
RMAN> CROSSCHECK BACKUP;
```

Validate:

```text
RMAN> VALIDATE DATABASE;
```

---

# 5.55 Lab — Control File Autobackup

Configure:

```text
RMAN> CONFIGURE CONTROLFILE AUTOBACKUP ON;
```

Then:

```text
RMAN> SHOW ALL;
```

Verify that the setting is enabled.

---

# 5.56 Lab — Backup

In a lab:

```text
RMAN> BACKUP DATABASE;
```

Then:

```text
RMAN> LIST BACKUP SUMMARY;
```

You should now be able to see the backup registered with RMAN.

---

# 5.57 Lab — Backup Plus Archived Logs

If your database is in `ARCHIVELOG` mode:

```text
RMAN> BACKUP DATABASE PLUS ARCHIVELOG;
```

Then:

```text
RMAN> LIST BACKUP SUMMARY;
```

---

# 5.58 Lab — Inspect FRA

From SQL*Plus:

```sql
SHOW PARAMETER db_recovery_file_dest;
```

```sql
SHOW PARAMETER db_recovery_file_dest_size;
```

Then:

```sql
SELECT name,
       space_limit,
       space_used,
       space_reclaimable
FROM v$recovery_file_dest;
```

---

# 5.59 Lab — Simulate a Datafile Recovery

**Do not perform this against a database you care about.**

First identify a suitable lab datafile:

```sql
SELECT file_id,
       file_name,
       tablespace_name
FROM dba_data_files;
```

Then learn the recovery sequence conceptually:

```text
Identify file
    ↓
Take appropriate action based on file/database state
    ↓
RESTORE DATAFILE
    ↓
RECOVER DATAFILE
    ↓
Verify
```

For example:

```text
RMAN> RESTORE DATAFILE 7;
RMAN> RECOVER DATAFILE 7;
```

Don't substitute `7` unless your lab database actually has that file number.

---

# 5.60 Lesson 5 Interview Questions

Before moving on, you should be able to answer:

1. What is RMAN?
2. What is a target database?
3. What is a recovery catalog?
4. Can RMAN work without a recovery catalog?
5. What is ARCHIVELOG mode?
6. Why is ARCHIVELOG important?
7. What is a backup set?
8. What is an image copy?
9. Difference between backup and restore?
10. Difference between restore and recovery?
11. What is complete recovery?
12. What is incomplete recovery?
13. What is point-in-time recovery?
14. What is an SCN?
15. What is a recovery window?
16. What is redundancy?
17. Difference between obsolete and expired backups?
18. What does `CROSSCHECK BACKUP` do?
19. What does `DELETE EXPIRED BACKUP` do?
20. What does `DELETE OBSOLETE` do?
21. Why back up the control file?
22. Why back up the SPFILE?
23. What is control-file autobackup?
24. What is the FRA?
25. How do you monitor FRA usage?
26. What happens if FRA becomes full?
27. What is physical corruption?
28. What is logical corruption?
29. Why validate backups?
30. What is RPO?
31. What is RTO?
32. Why isn't a backup alone a disaster-recovery strategy?
33. Difference between backup and high availability?
34. How would you recover a missing datafile?
35. How would you recover from accidental bad data?

---

# 5.61 The DBA Recovery Mental Model

Memorize this:

```text
                 FAILURE
                    |
        +-----------+-----------+
        |                       |
   User error              Media failure
        |                       |
        v                       v
     PITR                  RMAN Restore
        |                       |
        +-----------+-----------+
                    |
                    v
                 RECOVERY
                    |
              Apply Redo
                    |
                    v
              VALIDATION
                    |
                    v
               APPLICATION
                    |
                    v
                 RESUME
```

And the most important distinction:

```text
BACKUP
  |
  v
COPY

RESTORE
  |
  v
PUT COPY BACK

RECOVER
  |
  v
APPLY CHANGES
```

**Lesson 5/10 complete.**

### Remaining course

```text
1. Oracle Architecture & DBA Foundations       ✓
2. Installation & Database Creation             ✓
3. Configuration & Oracle Networking            ✓
4. Core DBA Administration                     ✓
5. Backup & Recovery / RMAN                    ✓
6. Performance Monitoring & Troubleshooting
7. Security & Advanced Administration
8. Patching & Patch Management
9. Migration & Database Upgrades
10. RAC, Data Guard & End-to-End DBA Operations
```

**Next: Lesson 6/10 — Oracle Performance Monitoring & Troubleshooting**, including **AWR, ADDM, ASH, wait events, execution plans, SQL tuning, locks/blocking, CPU/memory/I/O diagnosis, alert logs, ADRCI, and a systematic production troubleshooting methodology**.
