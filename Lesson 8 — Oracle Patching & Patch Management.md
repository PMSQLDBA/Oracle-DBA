# Lesson 8/10 — Oracle Patching & Patch Management

**Strictly Lesson 8 of 10.**

This is one of the most important DBA lessons because production Oracle databases must be maintained without introducing unnecessary outage or risk.

Oracle's current documentation describes **Release Updates (RUs)** as the primary proactive quarterly update mechanism for Oracle Database 19c and later, while one-off patches are used for specific problems. Oracle also documents platform-specific patching methods, including OPatch/OPatchAuto and other lifecycle tools. ([Oracle Documentation][1])

---

# 8.1 What Is Oracle Patching?

Patching means applying Oracle software fixes to an existing Oracle installation.

Think:

```text
Oracle Home
     |
     +-- Oracle binaries
     +-- Libraries
     +-- Executables
     +-- Components
     |
     v
Patch
     |
     v
Updated Oracle Home
```

A patch may address:

```text
Security vulnerabilities
Bug fixes
Regression fixes
Optimizer issues
Functional problems
Platform-specific issues
```

---

# 8.2 Patch vs Upgrade

This distinction is critical.

### Patching

Generally:

```text
19c
 ↓
19c + current RU
```

The database remains within the same major release family.

### Upgrade

For example:

```text
19c
 ↓
26ai
```

That is a release upgrade, with different planning and tooling.

Oracle's upgrade documentation specifically recommends updating the **new Oracle home with the latest applicable RU before starting an upgrade**. ([Oracle Documentation][2])

---

# 8.3 Oracle Patch Terminology

You will encounter:

```text
RU
RUR
CPU
PSU
One-off patch
Bundle Patch
MRP
OPatch
OPatchAuto
```

Some terminology has changed over Oracle releases.

For modern Oracle Database 19c administration, focus heavily on:

```text
RU
MRP
One-off
OPatch
OPatchAuto
```

Oracle states that it no longer releases traditional patch sets and provides quarterly RUs; MRPs are available for 19c on Linux x86-64. ([Oracle Documentation][3])

---

# 8.4 Release Update — RU

**RU = Release Update**

Oracle describes RUs as cumulative bundles containing fixes such as:

```text
Security fixes
Regression fixes
Optimizer fixes
Functional fixes
```

They are released quarterly. ([Oracle Documentation][1])

Conceptually:

```text
19c base
   |
   +-- RU 1
   |
   +-- RU 2
   |
   +-- RU 3
   |
   +-- Current RU
```

Because RUs are cumulative, you generally don't need to install every previous RU individually.

---

# 8.5 Monthly Recommended Patches — MRP

For Oracle Database 19c on **Linux x86-64**, Oracle provides Monthly Recommended Patches.

MRPs:

```text
Monthly
Cumulative
Linux x86-64
19c
```

They don't change the release number. ([Oracle Documentation][1])

Important:

> Do not assume MRP availability on Windows or every other platform.

Oracle's documentation identifies MRPs as a 19c Linux x86-64 mechanism. ([Oracle Documentation][1])

---

# 8.6 Critical Patch Updates — CPU

You will frequently hear:

> "Oracle CPU."

Historically, Oracle's **Critical Patch Updates** are security-fix mechanisms.

Modern Oracle documentation describes security fixes as being delivered through the proactive patching mechanisms such as RUs/MRPs. ([Oracle Documentation][1])

Therefore don't confuse:

```text
CPU = simply another patch command
```

with:

```text
Security fixes delivered through Oracle's current patching mechanisms.
```

---

# 8.7 One-Off Patch

A one-off patch addresses a particular issue.

Example:

```text
Bug XYZ
   ↓
Oracle provides one-off patch
```

You might need one because:

```text
Your application hits a known bug
```

or:

```text
Oracle Support recommends it
```

Never install a random one-off patch just because its description sounds useful.

---

# 8.8 Where Do You Get Oracle Patches?

Oracle patches are obtained through:

**My Oracle Support (MOS).**

Oracle's official installation documentation instructs administrators to use My Oracle Support's **Patches and Updates** functionality and to read the patch README before installation. ([Oracle Documentation][3])

The basic process:

```text
My Oracle Support
       ↓
Patches & Updates
       ↓
Product
       ↓
Release
       ↓
Platform
       ↓
Patch
       ↓
README
       ↓
Download
```

---

# 8.9 The README Is Mandatory

This is a professional DBA rule:

> **Never patch Oracle without reading the patch README.**

The README tells you things such as:

```text
Prerequisites
Conflicts
Required OPatch version
Shutdown requirements
SQL steps
Post-installation steps
Rollback instructions
Known issues
Platform-specific instructions
```

Oracle explicitly instructs administrators to review the README before applying the patch. ([Oracle Documentation][3])

---

# 8.10 OPatch

**OPatch** is Oracle's patching utility for Oracle software homes.

Typical location:

```text
$ORACLE_HOME/OPatch
```

Check version:

```bash
$ORACLE_HOME/OPatch/opatch version
```

Windows:

```cmd
%ORACLE_HOME%\OPatch\opatch.bat version
```

---

# 8.11 Check Existing Patches

Linux:

```bash
$ORACLE_HOME/OPatch/opatch lsinventory
```

Windows:

```cmd
%ORACLE_HOME%\OPatch\opatch.bat lsinventory
```

This is one of the most important DBA commands.

It tells you about patches installed in the Oracle Home.

---

# 8.12 Patch Inventory

Think:

```text
Oracle Home
     |
     v
OPatch Inventory
     |
     +-- Patch A
     +-- Patch B
     +-- Patch C
     +-- RU
     +-- One-offs
```

Before patching:

```bash
opatch lsinventory
```

After patching:

```bash
opatch lsinventory
```

Then compare.

---

# 8.13 Database SQL Patch Inventory

There's another important distinction.

Oracle documentation identifies:

```text
DBA_REGISTRY_SQLPATCH
```

as the view that tells you which SQL patches have been applied to the database. ([Oracle Documentation][1])

Query:

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

---

# 8.14 Binary vs SQL Patching

This distinction is extremely important.

```text
Oracle Home
     |
     +-- Binary patching
     |
     v
Database
     |
     +-- SQL patching
```

You can therefore have:

```text
Oracle binaries patched
```

but still need:

```text
Datapatch SQL actions
```

depending on the patch.

---

# 8.15 Datapatch

`datapatch` performs required SQL changes associated with database patches.

Typical command:

```bash
$ORACLE_HOME/OPatch/datapatch -verbose
```

Run according to the patch README and Oracle's documented procedure.

Then verify:

```sql
SELECT patch_id,
       action,
       status,
       action_time
FROM dba_registry_sqlpatch
ORDER BY action_time DESC;
```

You want successful status.

---

# 8.16 OPatchAuto

**OPatchAuto** automates patching across applicable Oracle software components, especially in environments involving Grid Infrastructure.

Typical form:

```bash
opatchauto apply /path/to/patch
```

Oracle documents OPatchAuto for Grid Infrastructure and related patching scenarios. ([Oracle Documentation][4])

---

# 8.17 OPatch vs OPatchAuto

Remember:

```text
OPatch
  ↓
Oracle Home patching

OPatchAuto
  ↓
Automated patching of applicable Oracle stack components
```

Do not substitute one for the other blindly.

Always follow the specific patch README.

---

# 8.18 Pre-Patching Checklist

Before touching production:

```text
[ ] Identify database version
[ ] Identify OS/platform
[ ] Identify Oracle Home
[ ] Identify Grid Home if applicable
[ ] Check current RU
[ ] Check OPatch version
[ ] Review patch README
[ ] Check prerequisites
[ ] Check conflicts
[ ] Check disk space
[ ] Check database health
[ ] Check backups
[ ] Check Data Guard
[ ] Check RAC topology
[ ] Check application dependencies
[ ] Prepare rollback plan
[ ] Prepare outage/rolling plan
```

---

# 8.19 Determine Version

```sql
SELECT banner_full
FROM v$version;
```

Also:

```sql
SELECT *
FROM product_component_version;
```

For the Oracle Home itself:

```bash
$ORACLE_HOME/OPatch/opatch lsinventory
```

---

# 8.20 Check OPatch Version

Linux:

```bash
$ORACLE_HOME/OPatch/opatch version
```

Windows:

```cmd
%ORACLE_HOME%\OPatch\opatch.bat version
```

The patch README may require a minimum OPatch version.

If so:

```text
Old OPatch
    ↓
Upgrade OPatch
    ↓
Apply database patch
```

---

# 8.21 Patch Conflicts

A patch can conflict with an existing one-off.

Conceptually:

```text
Existing One-Off
       +
New RU
       |
       v
   Conflict?
```

Oracle documents patch conflict resolution as a specific part of patch maintenance. ([Oracle Documentation][1])

Do not ignore conflict messages.

---

# 8.22 Conflict Checking

Use the mechanisms specified in the patch README/MOS documentation.

For OPatch environments, commands such as:

```bash
opatch prereq CheckConflictAgainstOHWithDetail -phBaseDir /path/to/patch
```

may be used where applicable.

The exact prerequisite command depends on the patch type and README.

---

# 8.23 Dry Run / Prerequisites

Before applying:

```text id="d5ct3u"
Patch
 ↓
Prerequisite checks
 ↓
Conflict checks
 ↓
Space checks
 ↓
Apply
```

Never treat prerequisite checking as optional.

---

# 8.24 Backup Before Patching

You need a recovery strategy.

At minimum determine:

```text
Database backup
Oracle Home backup/clone strategy
Configuration backup
Wallet/keystore backup if applicable
Listener/network configuration
Cluster configuration
```

The exact backup method depends on your architecture.

---

# 8.25 Why Back Up the Oracle Home?

Suppose:

```text
Oracle Home
    |
    v
Patch fails
```

You may need to restore or switch to another Oracle Home.

This is why **out-of-place patching** is attractive.

---

# 8.26 In-Place Patching

In-place:

```text
Old Oracle Home
      |
      +-- Patch directly
      |
      v
Same Oracle Home
```

Advantages:

```text
Simple
Less additional disk space
Traditional workflow
```

Disadvantages:

```text
Rollback can be more complicated
Changes existing home
Potential operational risk
```

---

# 8.27 Out-of-Place Patching

Out-of-place:

```text
OLD HOME
   |
   | remains intact
   |
   +----------------+
                    |
NEW HOME            |
   |                |
   +-- patched      |
                    |
Database ----------> NEW HOME
```

The new Oracle Home is prepared separately.

Then the database is moved to it.

This provides a cleaner rollback strategy.

---

# 8.28 Out-of-Place Example

Suppose:

```text
/u01/app/oracle/product/19.0.0/dbhome_1
```

is current.

Create:

```text
/u01/app/oracle/product/19.0.0/dbhome_2
```

Install/copy the required Oracle software and apply the RU.

Then:

```text
dbhome_1
   |
   | old
   |
dbhome_2
   |
   | patched
   v
Database
```

If something goes wrong, the old home may remain available for rollback according to Oracle's supported procedure.

---

# 8.29 Windows Patching

Windows uses Windows-specific Oracle patch bundles and procedures.

Typical environment:

```text
C:\app\oracle\product\19.0.0\dbhome_1
```

Check:

```cmd
%ORACLE_HOME%\OPatch\opatch.bat lsinventory
```

The exact patch package and README must correspond to:

```text
Oracle release
Windows architecture
Database/Grid component
```

Never apply a Linux patch to a Windows Oracle Home.

---

# 8.30 Linux Patching

Linux:

```bash
$ORACLE_HOME/OPatch/opatch lsinventory
```

Patch extraction:

```bash
unzip patch.zip
```

Then follow the README.

Typical flow:

```text
Download
   ↓
Unzip
   ↓
Read README
   ↓
Prerequisite check
   ↓
Conflict check
   ↓
Stop services if required
   ↓
Apply
   ↓
Datapatch if required
   ↓
Verify
```

---

# 8.31 RAC Patching

With Oracle RAC:

```text
Node 1
  |
  +-- Grid Infrastructure
  +-- Database instance

Node 2
  |
  +-- Grid Infrastructure
  +-- Database instance
```

Patching may be performed in a **rolling** manner depending on patch/environment support.

Conceptually:

```text
Node 1 → patch
Node 2 → serving workload

Node 2 → patch
Node 1 → serving workload
```

This can reduce application downtime.

Oracle documents rolling Grid Infrastructure patching procedures and node-by-node application. ([Oracle Documentation][4])

---

# 8.32 Rolling Patching

Rolling means:

```text
Not all nodes are unavailable simultaneously.
```

But:

> **Not every patch supports every rolling scenario.**

Always verify the patch documentation.

---

# 8.33 Oracle Grid Infrastructure

Grid Infrastructure contains components such as:

```text
Clusterware
ASM
Other Grid components
```

Therefore patching GI is more sensitive than patching a standalone database home.

Oracle has separate Grid Infrastructure patching procedures. ([Oracle Documentation][5])

---

# 8.34 Grid Infrastructure + Database

Conceptually:

```text
Grid Home
    |
    +-- ASM
    +-- Clusterware
    |
Database Home
    |
    +-- Oracle Database binaries
```

You must determine:

```text
What is being patched?
GI?
DB?
Both?
```

Never assume a database RU automatically means the entire Oracle stack has been patched.

---

# 8.35 Oracle Restart

Oracle Restart is the single-server high-availability stack.

Oracle documents patching Oracle Restart using OPatchAuto in applicable scenarios. ([Oracle Documentation][6])

Conceptually:

```text
Standalone Server
       |
 Oracle Restart
       |
   Database
       |
      ASM
```

---

# 8.36 Zero-Downtime Grid Patching

Oracle 19c documentation describes **Zero-Downtime Oracle Grid Infrastructure Patching** for supported environments.

Oracle states this capability is available with Fleet Patching and Provisioning for applicable 19c configurations and out-of-place patching. ([Oracle Documentation][7])

Don't interpret "zero downtime" as:

> Every Oracle environment can be patched with zero downtime.

Eligibility depends on:

```text
Oracle version
GI RU
RAC/RAC One
Database version
Patching method
Architecture
```

---

# 8.37 Patch Order

A simplified conceptual model:

```text
        PATCH PLANNING
              |
              v
        Grid Infrastructure
              |
              v
          Database Home
              |
              v
           Datapatch
              |
              v
           Validation
```

The exact sequence depends on your environment and the patch README.

---

# 8.38 Datapatch Verification

After patching:

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

Look for:

```text
STATUS = SUCCESS
```

Investigate anything else.

---

# 8.39 Invalid Objects

After maintenance:

```sql
SELECT owner,
       object_name,
       object_type,
       status
FROM dba_objects
WHERE status <> 'VALID'
ORDER BY owner, object_type, object_name;
```

If invalid objects exist:

```text
Investigate
   ↓
Determine cause
   ↓
Compile/rebuild where appropriate
   ↓
Validate
```

Don't blindly recompile everything without understanding why objects became invalid.

---

# 8.40 Post-Patch Validation

Check:

```text
[ ] Database opens
[ ] Services start
[ ] Listener works
[ ] Applications connect
[ ] SQL works
[ ] Jobs run
[ ] Database links work
[ ] Backup works
[ ] Monitoring works
[ ] Data Guard works
[ ] RAC instances healthy
[ ] ASM healthy
[ ] No unexpected errors
[ ] SQL patch status SUCCESS
```

---

# 8.41 Check Database

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

---

# 8.42 Check Services

```bash
lsnrctl status
```

Windows:

```cmd
lsnrctl status
```

Check that the expected services are registered.

---

# 8.43 Check Alert Log

After patching, inspect the alert log.

You're looking for:

```text
ORA- errors
Startup errors
ASM errors
Listener-related problems
Unexpected warnings
Recovery problems
```

Don't stop at:

```text
"Database opened successfully."
```

A database can open while an application dependency remains broken.

---

# 8.44 Check SQL Patch History

```sql
SELECT patch_id,
       action,
       status,
       action_time
FROM dba_registry_sqlpatch
ORDER BY action_time DESC;
```

This is part of your permanent patch verification.

---

# 8.45 Patch Rollback

If a patch causes a problem, rollback may be possible depending on the patch and procedure.

Typical OPatch syntax:

```bash
opatch rollback -id PATCH_ID
```

But:

> **Never invent rollback steps from memory.**

Use the patch README and Oracle-supported rollback procedure.

Some patch types have special rollback requirements.

---

# 8.46 Rollback Strategy

Before patching:

```text
              PATCH
                |
        +-------+-------+
        |               |
     SUCCESS           FAILURE
        |               |
      Verify          Diagnose
                        |
                        v
                     Rollback
                        |
                        v
                     Verify
```

A rollback plan must exist **before** production patching begins.

---

# 8.47 Production Patching Runbook

Here's a professional runbook structure.

### Phase 1 — Planning

```text
1. Identify target databases
2. Identify versions
3. Identify platforms
4. Identify Oracle Homes
5. Identify GI/RAC/Data Guard
6. Select RU
7. Review MOS
8. Review README
9. Identify conflicts
10. Define outage/rolling strategy
```

---

### Phase 2 — Preparation

```text
1. Backup
2. Download patch
3. Update OPatch if required
4. Verify disk space
5. Test patch in non-production
6. Validate application compatibility
7. Prepare rollback
```

---

### Phase 3 — Precheck

```text
1. Database health
2. Listener health
3. ASM health
4. RAC health
5. Data Guard health
6. Invalid objects
7. Existing patch inventory
8. SQL patch inventory
```

---

### Phase 4 — Apply

```text
1. Stop required services
2. Apply patch
3. Start required services
4. Run datapatch if required
```

---

### Phase 5 — Validate

```text
1. Check OPatch inventory
2. Check DBA_REGISTRY_SQLPATCH
3. Check database
4. Check listener
5. Check ASM
6. Check RAC
7. Check application
8. Check alert log
9. Check monitoring
```

---

### Phase 6 — Close

```text
1. Document patch ID
2. Document date/time
3. Document Oracle Home
4. Document OPatch version
5. Document SQL patch status
6. Document issues
7. Update CMDB/change record
```

---

# 8.48 Patching Mistakes You Must Avoid

### Mistake 1

```text
Download patch
↓
Immediately run opatch
```

Wrong.

First:

```text
README
Prerequisites
Conflicts
Backup
Plan
```

---

### Mistake 2

Using the wrong platform patch.

```text
Linux patch → Windows
```

Never.

---

### Mistake 3

Ignoring OPatch version requirements.

```text
Patch README
      |
      +-- Requires newer OPatch
```

Upgrade OPatch first when required.

---

### Mistake 4

Forgetting datapatch.

```text
Binary patch
    ≠
Always complete database patching
```

Check the patch documentation and `DBA_REGISTRY_SQLPATCH`.

---

### Mistake 5

No rollback plan.

Never patch production with:

> "We'll figure it out if it fails."

---

### Mistake 6

Patching production first.

Professional flow:

```text
DEV
 ↓
TEST
 ↓
UAT
 ↓
PRODUCTION
```

---

# 8.49 Patch Lab — Linux

Assuming a lab Oracle Home:

```bash
echo $ORACLE_HOME
```

Check version:

```bash
$ORACLE_HOME/OPatch/opatch version
```

Inventory:

```bash
$ORACLE_HOME/OPatch/opatch lsinventory
```

Database:

```sql
SELECT banner_full
FROM v$version;
```

SQL patch history:

```sql
SELECT patch_id,
       action,
       status,
       action_time
FROM dba_registry_sqlpatch
ORDER BY action_time DESC;
```

---

# 8.50 Patch Lab — Windows

Check:

```cmd
echo %ORACLE_HOME%
```

OPatch:

```cmd
%ORACLE_HOME%\OPatch\opatch.bat version
```

Inventory:

```cmd
%ORACLE_HOME%\OPatch\opatch.bat lsinventory
```

Database:

```sql
SELECT banner_full
FROM v$version;
```

SQL patch history:

```sql
SELECT patch_id,
       action,
       status,
       action_time
FROM dba_registry_sqlpatch
ORDER BY action_time DESC;
```

---

# 8.51 Your Patch Decision Tree

Memorize this:

```text
NEED ORACLE PATCH?
       |
       v
What version?
       |
       v
What OS/platform?
       |
       v
Single instance / RAC / Restart?
       |
       v
Which RU / MRP / one-off?
       |
       v
Read README
       |
       v
Check OPatch
       |
       v
Check conflicts
       |
       v
Backup
       |
       v
Test
       |
       v
Patch
       |
       v
Datapatch if required
       |
       v
Verify
       |
       v
Document
```

---

# 8.52 The DBA Patch Command Set

Know these:

```bash
opatch version
```

```bash
opatch lsinventory
```

```bash
opatch prereq ...
```

```bash
opatch apply
```

```bash
opatch rollback
```

```bash
opatchauto apply
```

And:

```bash
datapatch -verbose
```

But remember:

> **Commands are not the procedure.**

The **patch README + Oracle documentation + your environment's architecture** determine the correct procedure.

---

# 8.53 Current Oracle Direction

Oracle's 19c patch-maintenance documentation emphasizes proactive maintenance with RUs and provides multiple software-maintenance tools.

Oracle documents:

```text
OPatch / OPatchAuto
DBCA
Fleet Patching and Provisioning
Enterprise Manager
```

with different capabilities depending on the environment. ([Oracle Documentation][8])

For large RAC/Data Guard fleets, Oracle recommends **Fleet Patching and Provisioning (FPP)** as a maintenance approach. ([Oracle Documentation][1])

For this course, however, you must understand **OPatch and OPatchAuto first**, because they're fundamental DBA skills.

---

# 8.54 Lesson 8 Interview Questions

You should now be able to answer:

1. What is an Oracle RU?
2. What is an MRP?
3. What is a one-off patch?
4. What is OPatch?
5. What is OPatchAuto?
6. What is `opatch lsinventory`?
7. How do you check the OPatch version?
8. What is `datapatch`?
9. What is `DBA_REGISTRY_SQLPATCH`?
10. Binary patch vs SQL patch?
11. What is a patch conflict?
12. What is a prerequisite check?
13. What is in-place patching?
14. What is out-of-place patching?
15. Advantages of out-of-place patching?
16. How do you patch Oracle on Windows?
17. How do you patch Oracle on Linux?
18. How does RAC patching differ from single-instance patching?
19. What is rolling patching?
20. What is Oracle Grid Infrastructure patching?
21. What is Oracle Restart patching?
22. How do you verify a patch?
23. How do you rollback a patch?
24. What should you check before patching?
25. What should you check after patching?
26. Why must you read the patch README?
27. Why shouldn't you install a random one-off?
28. What is the difference between patching and upgrading?
29. What is FPP?
30. Why is a rollback plan required?

---

# 8.55 The Golden Patching Formula

Memorize this:

```text
           PATCHING
              |
          PLAN FIRST
              |
              v
        READ README
              |
              v
       CHECK INVENTORY
              |
              v
     CHECK PREREQUISITES
              |
              v
       CHECK CONFLICTS
              |
              v
          BACKUP
              |
              v
          TEST PATCH
              |
              v
           APPLY
              |
              v
        DATAPATCH
       (if required)
              |
              v
          VERIFY
              |
              v
         DOCUMENT
```

That's the DBA mindset.

---

## Official Oracle references

* **Oracle Database 19c Patch Maintenance** — RUs, MRPs, one-off patches, conflict resolution and patching methods. [Oracle Database 19c Patch Maintenance](https://docs.oracle.com/en/database/oracle/oracle-database/19/dbptc/index.html)
* **Oracle Database 19c Installation Guide — Downloading Release Update Patches** — MOS download procedure and README requirements. [Oracle Database 19c Release Update Patches](https://docs.oracle.com/en/database/oracle/oracle-database/19/ladbi/downloading-and-installing-patch-updates.html)
* **Oracle Grid Infrastructure 19c Patching** — Grid Infrastructure patching procedures. [Oracle Grid Infrastructure Patching](https://docs.oracle.com/en/database/oracle/oracle-database/19/cwlin/patching-oracle-grid-infrastructure.html)
* **Oracle 19c Software Maintenance Tools** — capabilities of OPatch/OPatchAuto, DBCA, FPP and other tools. [Oracle Software Maintenance Tools](https://docs.oracle.com/en/database/oracle/oracle-database/19/haovw/software-maintenance-tools-and-capabilities.html)

**Lesson 8/10 complete.**

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
9. Migration & Database Upgrades
10. RAC, Data Guard & End-to-End DBA Operations
```

**Next: Lesson 9/10 — Oracle Database Migration & Upgrades**, including **Windows ↔ Linux considerations, same-platform and cross-platform migration, Data Pump, RMAN conversion, Transportable Tablespaces, Full Transportable Export/Import, unplug/plug PDB migration, AutoUpgrade, pre-upgrade checks, compatibility, rollback strategy, post-upgrade validation, and a complete production migration runbook.**

[1]: https://docs.oracle.com/en/database/oracle/oracle-database/19/dbptc/index.html "Oracle Database Oracle Database Patch Maintenance, Release 19c and Later Releases"
[2]: https://docs.oracle.com/en/database/oracle/oracle-database/19/spuss/patch-set-updates-and-requirements.html "Release Updates and Requirements for Upgrading Oracle Database"
[3]: https://docs.oracle.com/en/database/oracle/oracle-database/19/ladbi/downloading-and-installing-patch-updates.html "Downloading Release Update Patches"
[4]: https://docs.oracle.com/en/database/oracle/oracle-database/19/cwlin/applying-patching-after-grid-install-or-upgrade.html "Applying Patches After an Oracle Grid Infrastructure Installation or Upgrade"
[5]: https://docs.oracle.com/en/database/oracle/oracle-database/19/cwlin/patching-oracle-grid-infrastructure.html "Patching Oracle Grid Infrastructure"
[6]: https://docs.oracle.com/en/database/oracle/oracle-database/19/upgor/patching-oracle-restart.html "Patching Oracle Restart"
[7]: https://docs.oracle.com/en/database/oracle/oracle-database/19/fppad/zero-downtime-grid-patching.html "Zero-Downtime Oracle Grid Infrastructure Patching"
[8]: https://docs.oracle.com/en/database/oracle/oracle-database/19/haovw/software-maintenance-tools-and-capabilities.html "Software Maintenance Tools and Capabilities"
