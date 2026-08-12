# Lesson 2/10 — Oracle Database Installation

**Scope: STRICTLY Lesson 2.**

We will cover the installation lifecycle across **Windows, Linux, and other Oracle-supported operating systems**, but we will use **Oracle Database 19c** for the hands-on examples because Oracle provides detailed current 19c installation documentation for these platforms. Oracle's 19c installation documentation was updated in 2026 for Linux and in November 2025 for Windows. ([Oracle Documentation][1])

> **Important:** Exact OS versions, packages, kernel requirements, and supported configurations are release/update specific. Always verify the current Oracle certification and the installation guide for your exact RU/platform before installing production systems.

---

## 2.1 The DBA Installation Lifecycle

Don't think of installation as simply running `setup.exe`.

A DBA should think:

```text
1. Plan
   ↓
2. Check OS
   ↓
3. Check hardware
   ↓
4. Plan storage
   ↓
5. Prepare OS
   ↓
6. Obtain Oracle software
   ↓
7. Install Oracle Home
   ↓
8. Create database
   ↓
9. Configure networking
   ↓
10. Verify
```

Oracle itself recommends using installation checklists covering hardware, OS, server configuration, storage, users/environment, and installer planning. ([Oracle Documentation][2])

---

# 2.2 Oracle Home vs Oracle Base

These two terms must become automatic for you.

### ORACLE_BASE

The top-level directory for Oracle software installations and related files.

Example Linux:

```text
/u01/app/oracle
```

Example Windows:

```text
C:\app\oracle
```

### ORACLE_HOME

The specific directory containing one Oracle software installation.

Example:

```text
/u01/app/oracle/product/19.0.0/dbhome_1
```

or Windows:

```text
C:\app\oracle\product\19.0.0\dbhome_1
```

Conceptually:

```text
ORACLE_BASE
   |
   +-- product
   |      |
   |      +-- 19.0.0
   |             |
   |             +-- dbhome_1     <-- ORACLE_HOME
   |
   +-- oraInventory
   |
   +-- other Oracle files
```

On Windows, Oracle documents the Oracle Base as typically `DRIVE_LETTER:\app\username` for a default installation, with important security considerations around the Windows Oracle Home User and sharing Oracle Base directories. ([Oracle Documentation][3])

---

# 2.3 Oracle Inventory

On Linux/UNIX, you will also encounter:

```text
/u01/app/oraInventory
```

The Oracle inventory keeps information about Oracle installations and their components.

Conceptually:

```text
Oracle Inventory
       |
       +-- Oracle Homes
       +-- Components
       +-- Installation information
```

Don't casually delete or modify it.

Oracle's Linux installation procedure explicitly creates an Oracle inventory directory and assigns appropriate ownership and permissions. ([Oracle Documentation][4])

---

# 2.4 Installation Planning

Before installing, answer these questions.

### Hardware

* How much RAM?
* How many CPUs?
* How much storage?
* What storage technology?
* What filesystem?
* What network interfaces?

### OS

* Which OS?
* Which exact release?
* Is that release certified for the Oracle Database release/update?
* Are required packages installed?
* Are kernel parameters/resource limits appropriate?

### Storage

Decide where:

```text
Oracle software
Database datafiles
Redo logs
Archive logs
Backups
```

will live.

For a simple Linux lab:

```text
/u01/app/oracle
/u01/app/oraInventory
/u02/oradata
/u03/fast_recovery_area
```

This is a **lab layout**, not a universal production storage design.

---

# 2.5 Linux Installation

Oracle's 19c Linux documentation supports image-based installation, GUI installation, silent installation, and RPM-based installation. ([Oracle Documentation][5])

For learning DBA administration, we'll primarily understand the **traditional image-based installation** because it exposes the Oracle software architecture clearly.

---

## 2.5.1 Linux OS preparation

Typical Oracle software owner:

```text
oracle
```

Typical groups include:

```text
oinstall
dba
```

The exact groups depend on whether you use standard or role-separated privileges.

Oracle's preinstallation RPM can automate much of the prerequisite configuration on Oracle Linux. Oracle documents:

```bash
dnf install oracle-database-preinstall-19c
```

for Oracle Linux 8/9. ([Oracle Documentation][6])

On Oracle Linux 7, Oracle documents `yum` instead. ([Oracle Documentation][6])

---

# 2.5.2 Linux directories

For our lab:

```bash
mkdir -p /u01/app/oracle
mkdir -p /u01/app/oraInventory
```

Then ownership:

```bash
chown -R oracle:oinstall /u01/app/oracle
chown -R oracle:oinstall /u01/app/oraInventory
```

Oracle's documented installation procedure similarly creates Oracle Base and inventory directories and assigns the Oracle installation owner/group. ([Oracle Documentation][4])

---

# 2.5.3 Linux environment

Switch to the Oracle software owner:

```bash
su - oracle
```

Set:

```bash
export ORACLE_BASE=/u01/app/oracle
export ORACLE_HOME=$ORACLE_BASE/product/19.0.0/dbhome_1
export ORACLE_SID=ORCL
export PATH=$ORACLE_HOME/bin:$PATH
```

Check:

```bash
echo $ORACLE_BASE
echo $ORACLE_HOME
echo $ORACLE_SID
```

And:

```bash
which sqlplus
```

Expected concept:

```text
/u01/app/oracle/product/19.0.0/dbhome_1/bin/sqlplus
```

---

# 2.6 Installing the Linux Oracle Software

Oracle's image-based installation process extracts the Oracle Database image into the Oracle Home and runs `runInstaller`. ([Oracle Documentation][5])

Conceptually:

```text
Download Oracle image
        ↓
Extract into ORACLE_HOME
        ↓
runInstaller
        ↓
Install Oracle software
        ↓
Run required root scripts
        ↓
Oracle Home ready
```

Example:

```bash
cd $ORACLE_HOME
```

Then:

```bash
./runInstaller
```

Oracle specifically documents starting the installation from the extracted database image using `runInstaller`. ([Oracle Documentation][4])

---

# 2.7 Software-Only Installation

This is an extremely important DBA concept.

You can install:

**Oracle software first**

and create the database afterward.

```text
Oracle Software
      ↓
Oracle Home
      ↓
Database creation
      ↓
DBCA
```

Why is this useful?

Because in enterprise environments you may want to:

* Prepare an Oracle Home
* Patch it
* Test it
* Create multiple databases from it
* Upgrade databases into it
* Maintain separate Oracle Homes

This becomes very important in **Lesson 7 — Patching** and **Lesson 9 — Upgrading**.

---

# 2.8 Creating the Database

Installing Oracle software does **not necessarily mean you have created the database you want**.

You can create/configure a database using **DBCA — Database Configuration Assistant**.

Conceptually:

```text
Oracle Home
     |
     v
   DBCA
     |
     +-- Database name
     +-- CDB/PDB configuration
     +-- Storage
     +-- Memory
     +-- Character set
     +-- Administrative passwords
     +-- Listener/service configuration
     |
     v
Oracle Database
```

For our lab:

```text
Database name: ORCL
```

Potential architecture:

```text
ORCL
 |
 +-- CDB$ROOT
 |
 +-- PDB$SEED
 |
 +-- APPPDB
```

---

# 2.9 Windows Installation

Now the process changes because Windows has its own Oracle-specific integration.

Oracle's current 19c Windows documentation covers Windows installation, Oracle Home User, Oracle Base, Windows services, networking, and silent/response-file installation. ([Oracle Documentation][7])

---

## 2.9.1 Windows prerequisites

Oracle's current 19c Windows checklist includes supported Windows client/server versions and specifies that some newer Windows versions require particular 19c Release Updates. For example, Windows Server 2025 support starts with 19c RU 19.27 or later; Windows 11 and Windows Server 2022 support starts with 19c RU 19.13 or later. ([Oracle Documentation][8])

**This is why you should never memorize "Oracle 19c supports Windows X" without checking the RU.**

The combination matters:

```text
Oracle Release
+
Release Update
+
OS version
=
Supported configuration
```

---

# 2.9.2 Windows Administrator

Oracle requires appropriate Administrator privileges for Windows installation. ([Oracle Documentation][9])

Open an elevated command prompt when required:

```cmd
cmd
```

Run it as:

```text
Administrator
```

---

# 2.9.3 Windows Oracle Base

Example:

```text
C:\app\oracle
```

Oracle documents this as a normal default-style Oracle Base layout. ([Oracle Documentation][3])

Then:

```text
C:\app\oracle
   |
   +-- product
         |
         +-- 19.0.0
                |
                +-- dbhome_1
```

---

# 2.9.4 Windows Oracle Home User

Windows has an important concept that Linux does not use in the same way:

**Oracle Home User**

This Windows account is used for Oracle services and software-related access.

Oracle documents security considerations around the Oracle Home User and Oracle Base, including restrictions on sharing an Oracle Base between different Windows Oracle Home Users. ([Oracle Documentation][3])

For DBA interviews, remember:

> **Windows Oracle installation involves Windows services and an Oracle Home User, while Linux traditionally uses the Oracle software owner and OS groups.**

---

# 2.10 Windows Software Installation

Oracle's documented 19c Windows process is essentially:

```text
Download db_home.zip
        ↓
Create Oracle Home
        ↓
Extract image
        ↓
Run setup.exe
        ↓
Choose installation configuration
        ↓
Install software
```

Oracle specifically instructs you to run `setup.exe` from the Oracle Home directory, not the copy under another Oracle installer path. ([Oracle Documentation][7])

Example:

```cmd
C:\> md C:\app\oracle\product\19.0.0\dbhome_1
```

Extract the Oracle image into that directory.

Then:

```cmd
C:\app\oracle\product\19.0.0\dbhome_1>setup.exe
```

Oracle's documented Windows workflow allows **Set Up Software Only**, followed by a single-instance installation selection. ([Oracle Documentation][7])

---

# 2.11 Windows Services

This is a major difference from Linux.

After database creation, Windows exposes Oracle components through Windows services.

You can inspect services with:

```powershell
Get-Service
```

or:

```cmd
services.msc
```

You may encounter services with names similar to:

```text
OracleServiceORCL
OracleOraDB19Home1TNSListener
```

The exact service names depend on the Oracle Home/database configuration.

As a DBA, you need to understand:

```text
Windows Service
       ↓
Oracle executable/process
       ↓
Oracle Instance
       ↓
Database
```

---

# 2.12 Linux vs Windows — Important Comparison

| Area                | Linux                                      | Windows                                      |
| ------------------- | ------------------------------------------ | -------------------------------------------- |
| Software owner      | `oracle` commonly                          | Windows Oracle Home User / installer account |
| Groups              | `oinstall`, `dba`, etc.                    | Windows groups/permissions                   |
| Installer           | `runInstaller`                             | `setup.exe`                                  |
| Environment         | Shell variables                            | Registry/system environment                  |
| Services            | systemd/scripts depending on configuration | Windows Services                             |
| Oracle Home         | `/u01/.../dbhome_1`                        | `C:\app\...\dbhome_1`                        |
| Listener            | `lsnrctl`                                  | `lsnrctl` + Windows service                  |
| SQL*Plus            | Same                                       | Same                                         |
| DBCA                | Yes                                        | Yes                                          |
| Silent installation | Yes                                        | Yes                                          |
| ASM                 | Grid Infrastructure required               | Grid Infrastructure required                 |
| OS administration   | Linux commands                             | PowerShell/Windows tools                     |

Oracle's official documentation maintains separate installation guides because these platform-specific differences matter. Oracle's 19c documentation set includes dedicated guides for Windows, Linux, Solaris, AIX, HP-UX, and other supported platforms. ([Oracle Documentation][10])

---

# 2.13 Other UNIX/Linux Platforms

The DBA principles remain the same:

```text
OS
 ↓
Oracle software
 ↓
Oracle Home
 ↓
Instance
 ↓
Database
```

But OS-specific procedures differ.

Oracle's documentation set includes platform-specific installation and administration documentation for platforms such as:

* Linux
* Microsoft Windows
* Solaris
* IBM AIX
* HP-UX Itanium
* Fujitsu BS2000/OSD

rather than assuming one installation procedure works everywhere. ([Oracle Documentation][10])

So our rule is:

> **Never copy a Linux installation procedure blindly onto AIX, Solaris, HP-UX, or Windows.**

---

# 2.14 Silent Installation

Enterprise DBAs often don't want to manually click through an installer.

Oracle supports silent/response-file installation on both Linux and Windows. ([Oracle Documentation][7])

Conceptually:

```text
Response file
     |
     v
Installer
     |
     v
Repeatable installation
```

This is useful for:

* Multiple servers
* Automated deployments
* Standardized environments
* Data centers
* Testing
* Dev/Test/QA/Production consistency

We will return to automation later, but **silent installation belongs to Lesson 2** because it's part of software deployment.

---

# 2.15 ASM — Important Installation Boundary

Here's an important distinction.

If you want Oracle Database files stored on **ASM**, Oracle requires **Oracle Grid Infrastructure for a standalone server** first. Grid Infrastructure includes Oracle ASM and Oracle Restart. ([Oracle Documentation][5])

Architecture:

```text
Grid Infrastructure
       |
       +-- Oracle ASM
       |
       +-- Oracle Restart
       |
       v
Oracle Database
```

For a simple filesystem lab:

```text
Oracle Database
       |
       v
Filesystem
```

For ASM:

```text
Grid Infrastructure
       |
       v
ASM
       |
       v
Oracle Database
```

**Do not install ASM as though it were simply another database option.**

---

# 2.16 Installation Verification

After installation, don't assume success because the installer displayed "completed."

Verify.

### Check SQL*Plus

Linux:

```bash
which sqlplus
```

Windows:

```cmd
where sqlplus
```

---

### Check Oracle version

```sql
SELECT banner_full
FROM v$version;
```

---

### Check instance

```sql
SELECT instance_name,
       status
FROM v$instance;
```

---

### Check database

```sql
SELECT name,
       open_mode,
       database_role
FROM v$database;
```

---

### Check PDBs

```sql
SHOW PDBS;
```

---

### Check listener

Linux/Windows:

```bash
lsnrctl status
```

---

### Check database services

```sql
SELECT name
FROM v$services;
```

---

# 2.17 Installation Troubleshooting

A DBA should immediately divide installation problems into categories.

### Category 1 — OS

```text
Unsupported OS
Missing packages
Permissions
Kernel configuration
Resource limits
Administrator privileges
```

### Category 2 — Storage

```text
Insufficient space
Wrong ownership
Wrong permissions
Filesystem problems
```

### Category 3 — Network

```text
Hostname
DNS
/etc/hosts
Firewall
Listener
```

### Category 4 — Oracle software

```text
Wrong Oracle Home
Corrupt installation image
Incorrect environment
Inventory problems
Existing Oracle installation conflicts
```

### Category 5 — Database creation

```text
DBCA failure
Invalid storage path
Memory configuration
PDB creation failure
Password/configuration issues
```

---

# 2.18 A Critical DBA Rule

Suppose installation fails.

**Don't immediately rerun the installer.**

First determine:

```text
What failed?
     ↓
Where did it fail?
     ↓
What error was generated?
     ↓
What log contains the evidence?
     ↓
What prerequisite was missing?
     ↓
Fix the root cause
     ↓
Retry
```

This mindset will become extremely important when we reach **patching and upgrades**.

---

# 2.19 Your Lesson 2 Lab

If you have a Linux machine available, your target lab architecture is:

```text
Linux Server
     |
     +-- Oracle User
     |
     +-- /u01/app/oracle
     |
     +-- /u01/app/oraInventory
     |
     +-- Oracle Home
     |
     +-- ORCL Database
     |
     +-- Listener
     |
     +-- CDB
           |
           +-- PDB$SEED
           |
           +-- APPPDB
```

For Windows:

```text
Windows Server
     |
     +-- Oracle Base
     |
     +-- Oracle Home
     |
     +-- Oracle Home User
     |
     +-- Oracle Database
     |
     +-- Windows Oracle Services
     |
     +-- Listener
     |
     +-- CDB
           |
           +-- PDB$SEED
           |
           +-- APPPDB
```

---

# 2.20 Lesson 2 — DBA Questions

Before moving to Lesson 3, you should be able to answer:

1. What is `ORACLE_BASE`?
2. What is `ORACLE_HOME`?
3. What is Oracle Inventory?
4. What is the purpose of the `oracle` OS user?
5. What are `oinstall` and `dba`?
6. Why are OS prerequisites important?
7. What is image-based installation?
8. What does `runInstaller` do?
9. What does `setup.exe` do on Windows?
10. What is software-only installation?
11. What is DBCA?
12. What is the difference between installing Oracle software and creating a database?
13. What is an Oracle Home User on Windows?
14. Why does Windows use Oracle services?
15. How do you check the Oracle version?
16. How do you check whether the instance is running?
17. How do you check whether the database is open?
18. How do you check the listener?
19. Why is ASM different from filesystem storage?
20. Why do Oracle installation procedures differ by OS?

---

## Lesson 2 — What you should remember

The most important architecture is:

```text
                OPERATING SYSTEM
                       |
             +---------+---------+
             |                   |
        ORACLE SOFTWARE      STORAGE
             |                   |
        ORACLE_HOME        Data/Redo/etc.
             |
          INSTANCE
             |
          DATABASE
             |
          CDB / PDB
```

And the most important installation distinction is:

```text
INSTALL ORACLE SOFTWARE
          ≠
CREATE ORACLE DATABASE
```

You can install the Oracle Home first and create/configure databases afterward.

**Oracle sources used for this lesson:** Oracle Database 19c Installation Guide for Linux, Oracle Database 19c Installation Guide for Microsoft Windows, and Oracle's official installation checklists and platform documentation. ([Oracle Documentation][5])

**Lesson 2/10 complete.**

Next is **Lesson 3/10 — Database Configuration & Networking**, where we'll go deep into **PFILE/SPFILE, initialization parameters, memory configuration, listener, `listener.ora`, `tnsnames.ora`, `sqlnet.ora`, services, CDB/PDB configuration, and Oracle Net troubleshooting**.

[1]: https://docs.oracle.com/en/database/oracle/oracle-database/19/ladbi/index.html?utm_source=chatgpt.com "Oracle Database Database Installation Guide, 19c for Linux"
[2]: https://docs.oracle.com/en/database/oracle/oracle-database/19/ntdbi/oracle-database-installation-checklist.html?utm_source=chatgpt.com "Oracle Database Installation Checklist"
[3]: https://docs.oracle.com/en/database/oracle/oracle-database/19/ntdbi/oracle-base-directory.html?utm_source=chatgpt.com "Oracle Base Directory"
[4]: https://docs.oracle.com/en/database/oracle/oracle-database/19/ladbi/running-oracle-universal-installer-to-install-oracle-database.html?source=%3Aso%3Afb%3Aor%3Aawr%3Aosec%3A&utm_source=chatgpt.com "Running Oracle Database Setup Wizard to Install Oracle Database"
[5]: https://docs.oracle.com/en/database/oracle/oracle-database/19/ladbi/installing-oracle-database.html?utm_source=chatgpt.com "Installing Oracle Database"
[6]: https://docs.oracle.com/en/database/oracle/oracle-database/19/ladbi/installing-oracle-linux-with-public-yum-repository-support.html?utm_source=chatgpt.com "Installing Oracle Oracle Database Preinstallation RPM Using the Oracle Linux yum Server"
[7]: https://docs.oracle.com/en/database/oracle/oracle-database/19/ntdbi/install-oracle-database.html?utm_source=chatgpt.com "Installing the Oracle Database Software"
[8]: https://docs.oracle.com/en/database/oracle/oracle-database/19/ntdbi/operating-system-checklist-oracle-database-installation-microsoft-windows.html?utm_source=chatgpt.com "Operating System Checklist for Oracle Database Installation on Microsoft Windows"
[9]: https://docs.oracle.com/en/database/oracle/oracle-database/19/ntdbi/preinstallation-considerations-for-installing-oracle-database.html?utm_source=chatgpt.com "Preinstallation Considerations for Installing Oracle Database"
[10]: https://docs.oracle.com/en/database/oracle/oracle-database/19/install-and-upgrade.html?utm_source=chatgpt.com "Oracle Database 19c - Install and Upgrade"
