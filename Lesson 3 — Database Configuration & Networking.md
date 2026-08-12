# Lesson 3/10 — Database Configuration & Networking

**Strictly Lesson 3 of 10.**

This lesson takes the Oracle software/database from **"installed" → "properly configured and reachable."**

Oracle's official 19c documentation treats initialization parameters, networking, listeners, services, and multitenant configuration as separate administration areas. We'll connect them into one DBA workflow.

---

## 3.1 The Configuration Architecture

Keep this picture in your head:

```text
                    CLIENT
                       |
                       v
                Oracle Net / TNS
                       |
                       v
                  LISTENER
                       |
                 Database Service
                       |
                       v
                Oracle Instance
                       |
                 Oracle Database
                       |
                CDB / PDB
```

There are therefore **two different configuration worlds**:

### Database configuration

```text
SPFILE / PFILE
Memory
Processes
Control files
Redo
Undo
Database parameters
CDB/PDB parameters
```

### Network configuration

```text
listener.ora
tnsnames.ora
sqlnet.ora
Listener
Services
DNS/hosts
Firewall
Ports
```

A DBA must understand both.

---

# 3.2 Initialization Parameters

Oracle Database has initialization parameters that control how an instance operates.

Examples:

```text
DB_NAME
COMPATIBLE
MEMORY_TARGET
SGA_TARGET
PGA_AGGREGATE_TARGET
PROCESSES
CONTROL_FILES
UNDO_TABLESPACE
LOCAL_LISTENER
```

You can inspect them with:

```sql
SHOW PARAMETER;
```

Or search for a specific parameter:

```sql
SHOW PARAMETER memory;
```

```sql
SHOW PARAMETER processes;
```

```sql
SHOW PARAMETER control_files;
```

---

# 3.3 PFILE vs SPFILE

This is a critical DBA topic.

## PFILE

A **PFILE** is a text-based initialization parameter file.

Conceptually:

```text
initORCL.ora
```

Example:

```text
db_name='ORCL'
control_files='/u02/oradata/ORCL/control01.ctl'
processes=1000
```

You can read/edit it as text.

---

## SPFILE

An **SPFILE** is a server parameter file maintained by Oracle.

Typical Linux location:

```text
$ORACLE_HOME/dbs/spfileORCL.ora
```

or commonly in an Oracle-managed environment:

```text
$ORACLE_HOME/dbs/spfileORCL.ora
```

You can determine whether the running instance is using one:

```sql
SHOW PARAMETER spfile;
```

If the value is populated, the instance is using an SPFILE.

---

# 3.4 Why SPFILE Matters

With an SPFILE you can make parameter changes through SQL.

For example:

```sql
ALTER SYSTEM SET processes=1000 SCOPE=SPFILE;
```

The important part is:

```text
SCOPE=SPFILE
```

The parameter is saved for future startup but does not necessarily change the currently running instance.

---

# 3.5 SCOPE

You need to understand three values.

### MEMORY

Change the currently running instance.

```sql
ALTER SYSTEM SET parameter=value SCOPE=MEMORY;
```

The change is not retained after restart.

---

### SPFILE

Change the SPFILE.

```sql
ALTER SYSTEM SET parameter=value SCOPE=SPFILE;
```

The change takes effect after the appropriate restart.

---

### BOTH

Change the running instance and SPFILE.

```sql
ALTER SYSTEM SET parameter=value SCOPE=BOTH;
```

Whether a particular parameter permits `MEMORY`, `SPFILE`, or `BOTH` depends on the parameter.

Always check the parameter documentation before changing production settings.

---

# 3.6 Example: Changing a Parameter

First:

```sql
SHOW PARAMETER processes;
```

Then, for a parameter that requires restart:

```sql
ALTER SYSTEM SET processes=1000 SCOPE=SPFILE;
```

Then:

```sql
SHUTDOWN IMMEDIATE;
```

And:

```sql
STARTUP;
```

Verify:

```sql
SHOW PARAMETER processes;
```

**Production rule:** never change a parameter simply because you found a value in a blog. Determine why the parameter needs changing and verify Oracle's documentation for your release.

---

# 3.7 Memory Configuration

Oracle memory management can be approached in different ways depending on platform, release, and deployment.

Important parameters include:

```text
MEMORY_TARGET
MEMORY_MAX_TARGET
SGA_TARGET
SGA_MAX_SIZE
PGA_AGGREGATE_TARGET
```

For example:

```sql
SHOW PARAMETER sga;
```

```sql
SHOW PARAMETER pga;
```

You should understand:

```text
Total Oracle memory
       |
       +---- SGA
       |
       +---- PGA
```

Do not randomly set memory parameters.

Memory configuration should be based on:

* Physical RAM
* Operating system requirements
* Other applications
* Number of databases
* Workload
* Oracle memory architecture
* Production requirements

---

# 3.8 Processes and Sessions

Two parameters you will encounter frequently:

```sql
SHOW PARAMETER processes;
```

and:

```sql
SHOW PARAMETER sessions;
```

Conceptually:

```text
Client
  |
  v
Session
  |
  v
Oracle Process
```

A production DBA needs to understand the distinction between:

**Session**

A logical user/application connection.

**Process**

An operating-system or Oracle process associated with database work.

---

# 3.9 Database Identity

Check:

```sql
SELECT name,
       db_unique_name,
       open_mode,
       database_role
FROM v$database;
```

You may see:

```text
NAME        ORCL
DB_UNIQUE_NAME ORCL
OPEN_MODE   READ WRITE
DATABASE_ROLE PRIMARY
```

These values become particularly important when you learn **Data Guard in Lesson 10**.

---

# 3.10 Database Startup Configuration

Remember Lesson 1:

```text
NOMOUNT
   ↓
MOUNT
   ↓
OPEN
```

Now connect that to configuration.

### NOMOUNT

Oracle needs initialization information to start the instance.

```text
PFILE/SPFILE
     ↓
Instance
     ↓
NOMOUNT
```

### MOUNT

Oracle reads the control files.

```text
Instance
   +
Control Files
   ↓
MOUNT
```

### OPEN

Oracle opens the database files for normal access.

```text
MOUNT
  ↓
OPEN
```

This relationship becomes extremely important during recovery and troubleshooting.

---

# 3.11 Oracle Net Architecture

Now we move into networking.

A typical connection looks like:

```text
Application
    |
    | TCP/IP
    v
Oracle Listener
    |
    | Service
    v
Database Instance
```

The listener is **not the database itself**.

The listener is a network process that receives incoming Oracle Net connection requests.

---

# 3.12 Default Listener

A common listener is named:

```text
LISTENER
```

A typical Oracle listener uses TCP port:

```text
1521
```

But **1521 is a convention/default, not something you should blindly assume**.

Check the listener:

```bash
lsnrctl status
```

You may see something similar to:

```text
Listener Parameter File
...
Listening Endpoints Summary
...
Services Summary
...
```

---

# 3.13 Listener Commands

Start:

```bash
lsnrctl start
```

Stop:

```bash
lsnrctl stop
```

Status:

```bash
lsnrctl status
```

Reload:

```bash
lsnrctl reload
```

Version:

```bash
lsnrctl version
```

The same `lsnrctl` utility is available on Windows, although the surrounding service/OS administration differs.

---

# 3.14 listener.ora

The listener configuration file is commonly:

Linux:

```text
$ORACLE_HOME/network/admin/listener.ora
```

Windows:

```text
%ORACLE_HOME%\network\admin\listener.ora
```

A conceptual configuration looks like:

```text
LISTENER =
  (DESCRIPTION_LIST =
    (DESCRIPTION =
      (ADDRESS =
        (PROTOCOL=TCP)
        (HOST=dbserver)
        (PORT=1521)
      )
    )
  )
```

Don't memorize syntax yet.

Understand the information:

```text
Listener name
      +
Protocol
      +
Host
      +
Port
```

---

# 3.15 Dynamic Service Registration

This is one of the most important networking concepts.

Oracle instances can dynamically register their services with the listener.

Conceptually:

```text
Oracle Instance
      |
      | Service registration
      v
   Listener
      |
      +-- ORCL
      +-- APPPDB
```

Check listener services:

```bash
lsnrctl status
```

Look for:

```text
Service "ORCL" ...
```

or:

```text
Service "APPPDB" ...
```

---

# 3.16 LOCAL_LISTENER

Oracle uses the `LOCAL_LISTENER` parameter to identify a local listener for registration when appropriate.

Check:

```sql
SHOW PARAMETER local_listener;
```

You may see:

```text
local_listener
```

If the listener is not correctly configured/registered, applications can experience errors such as:

```text
ORA-12514
```

---

# 3.17 ORA-12514

A classic DBA problem:

> Listener does not currently know of service requested in connect descriptor.

Think:

```text
Client
  |
  v
Listener
  |
  X
Service not registered
```

Troubleshooting sequence:

```bash
lsnrctl status
```

Then:

```sql
SHOW PARAMETER local_listener;
```

Then inspect services:

```sql
SELECT name
FROM v$services;
```

Then consider registration.

You may use:

```sql
ALTER SYSTEM REGISTER;
```

Then:

```bash
lsnrctl status
```

Again, don't treat `ALTER SYSTEM REGISTER` as a magic fix. Determine **why registration failed**.

---

# 3.18 tnsnames.ora

`tnsnames.ora` provides client-side network aliases.

Example:

```text
ORCL =
  (DESCRIPTION =
    (ADDRESS =
      (PROTOCOL = TCP)
      (HOST = dbserver)
      (PORT = 1521)
    )
    (CONNECT_DATA =
      (SERVICE_NAME = ORCL)
    )
  )
```

Then a client can use:

```bash
sqlplus username/password@ORCL
```

The important relationship is:

```text
ORCL
 ↓
HOST=dbserver
 ↓
PORT=1521
 ↓
SERVICE_NAME=ORCL
```

---

# 3.19 Service Name vs SID

Don't confuse:

**SID**

with:

**SERVICE_NAME**

Historically, the SID identifies an Oracle instance.

A service represents a logical service to which clients connect.

For modern applications, especially multitenant and RAC environments, **services are extremely important**.

Think:

```text
Instance
   |
   +---- Service A
   |
   +---- Service B
```

An application can connect to a service rather than thinking directly in terms of an instance.

This becomes essential when we eventually discuss RAC.

---

# 3.20 sqlnet.ora

Another important Oracle Net file:

```text
sqlnet.ora
```

Typical location:

```text
$ORACLE_HOME/network/admin/sqlnet.ora
```

It can control Oracle Net behavior such as:

* Naming methods
* Authentication-related settings
* Encryption/integrity configuration
* Connection behavior

Don't blindly copy `sqlnet.ora` from another server.

---

# 3.21 Naming Methods

Oracle Net can use different naming methods.

Important examples include:

```text
Easy Connect
Local naming
Directory naming
```

### Easy Connect

For example:

```text
username/password@//dbserver:1521/ORCL
```

This can be useful because it doesn't require a `tnsnames.ora` alias.

---

### Local naming

Uses:

```text
tnsnames.ora
```

Example:

```text
@ORCL
```

where `ORCL` is an alias defined in the file.

---

# 3.22 Testing Connectivity

Never jump directly to SQL*Plus when troubleshooting.

Use layers.

### Layer 1 — DNS/hostname

Linux:

```bash
getent hosts dbserver
```

Windows:

```powershell
Resolve-DnsName dbserver
```

Depending on your environment, `ping` can also be useful, but ICMP being blocked doesn't necessarily mean Oracle TCP connectivity is broken.

---

### Layer 2 — TCP port

Linux:

```bash
nc -vz dbserver 1521
```

Windows PowerShell:

```powershell
Test-NetConnection dbserver -Port 1521
```

---

### Layer 3 — Oracle Net

```bash
tnsping ORCL
```

---

### Layer 4 — Database login

```bash
sqlplus username/password@ORCL
```

This gives you a powerful troubleshooting hierarchy:

```text
DNS
 ↓
TCP
 ↓
Listener
 ↓
Oracle Service
 ↓
Authentication
 ↓
Database session
```

---

# 3.23 Common Oracle Networking Errors

Memorize the **meaning**, not just the error number.

### ORA-12154

```text
Could not resolve the connect identifier specified
```

Think:

```text
tnsnames.ora
Naming method
Alias
TNS_ADMIN
```

---

### ORA-12514

```text
Listener does not currently know of service requested
```

Think:

```text
Service registration
Listener
SERVICE_NAME
```

---

### ORA-12541

```text
No listener
```

Think:

```text
Listener stopped
Wrong host/port
Firewall/network issue
```

---

### ORA-01017

```text
Invalid username/password
```

Think:

```text
Authentication
Credentials
Account state
```

---

# 3.24 TNS_ADMIN

A useful environment variable is:

```text
TNS_ADMIN
```

It can tell Oracle Net where to find network configuration files.

For example:

Linux:

```bash
export TNS_ADMIN=/etc/oracle/network/admin
```

Windows:

```cmd
set TNS_ADMIN=C:\oracle\network\admin
```

Then Oracle Net can locate files such as:

```text
tnsnames.ora
sqlnet.ora
```

Be careful with environment configuration—different users, services, Oracle Homes, and applications can end up using different network configuration locations.

---

# 3.25 CDB/PDB Configuration

Now connect networking to multitenant.

Check current container:

```sql
SHOW CON_NAME;
```

List containers:

```sql
SHOW PDBS;
```

Example:

```text
CON_ID  CON_NAME     OPEN MODE
------  -----------  ----------
2       PDB$SEED     READ ONLY
3       APPPDB       READ WRITE
```

Open a PDB:

```sql
ALTER PLUGGABLE DATABASE APPPDB OPEN;
```

Check again:

```sql
SHOW PDBS;
```

---

# 3.26 PDB Services

This is where configuration and networking meet.

A PDB can have services associated with it.

Check:

```sql
SELECT name
FROM v$services;
```

Conceptually:

```text
Listener
   |
   +-- ORCL
   |
   +-- APPPDB service
             |
             v
           APPPDB
```

This is one reason modern Oracle networking is more than:

```text
hostname + port + SID
```

---

# 3.27 Saving PDB State

If a PDB should automatically reopen after a CDB restart, you may save its state:

```sql
ALTER PLUGGABLE DATABASE APPPDB SAVE STATE;
```

Then inspect:

```sql
SELECT con_name,
       state
FROM dba_pdb_saved_states;
```

This is an important operational configuration.

---

# 3.28 Windows Networking Differences

On Windows, Oracle Net concepts remain the same:

```text
listener.ora
tnsnames.ora
sqlnet.ora
lsnrctl
services
```

But Windows adds:

```text
Windows Services
Registry/environment configuration
Windows Firewall
PowerShell
```

Useful command:

```powershell
Test-NetConnection dbserver -Port 1521
```

And:

```cmd
lsnrctl status
```

You may also inspect Oracle services:

```powershell
Get-Service *Oracle*
```

---

# 3.29 Linux Networking Differences

Linux troubleshooting commonly involves:

```bash
ss -lntp
```

to inspect listening TCP ports.

You may also inspect:

```bash
hostname
getent hosts dbserver
```

and:

```bash
lsnrctl status
```

Then:

```bash
tnsping ORCL
```

The Oracle layer is the same; the OS troubleshooting layer differs.

---

# 3.30 Configuration Change Methodology

This is an important professional DBA habit.

**Never do this:**

```text
Problem
 ↓
Change random parameter
 ↓
Restart
 ↓
Hope
```

Instead:

```text
Problem
   ↓
Gather evidence
   ↓
Identify parameter/configuration
   ↓
Check Oracle documentation
   ↓
Determine scope
   ↓
Change
   ↓
Verify
   ↓
Document
```

For production databases:

**Always know how to reverse the change.**

---

# 3.31 Lesson 3 Hands-On Lab

Connect as SYSDBA.

### Step 1 — Database identity

```sql
SELECT name,
       db_unique_name,
       open_mode,
       database_role
FROM v$database;
```

### Step 2 — Instance

```sql
SELECT instance_name,
       host_name,
       status
FROM v$instance;
```

### Step 3 — SPFILE

```sql
SHOW PARAMETER spfile;
```

### Step 4 — Control files

```sql
SHOW PARAMETER control_files;
```

### Step 5 — Memory

```sql
SHOW PARAMETER sga;
```

```sql
SHOW PARAMETER pga;
```

### Step 6 — Processes

```sql
SHOW PARAMETER processes;
```

### Step 7 — Listener registration

```sql
SHOW PARAMETER local_listener;
```

Then:

```sql
ALTER SYSTEM REGISTER;
```

### Step 8 — Services

```sql
SELECT name
FROM v$services
ORDER BY name;
```

### Step 9 — Containers

```sql
SHOW CON_NAME;
```

```sql
SHOW PDBS;
```

### Step 10 — Listener

From the OS:

```bash
lsnrctl status
```

### Step 11 — Oracle Net

```bash
tnsping ORCL
```

---

# 3.32 Troubleshooting Exercise

Imagine the application team says:

> "The database server is up, but our application cannot connect."

Don't immediately restart Oracle.

Follow this:

```text
1. Is the server reachable?
       ↓
2. Is port 1521 reachable?
       ↓
3. Is listener running?
       ↓
4. Does listener know the requested service?
       ↓
5. Is the PDB open?
       ↓
6. Is the connect descriptor correct?
       ↓
7. Does authentication work?
```

For example:

```bash
lsnrctl status
```

then:

```bash
tnsping ORCL
```

then:

```bash
sqlplus user/password@ORCL
```

This layered troubleshooting approach is far more useful than memorizing ORA codes.

---

# 3.33 Lesson 3 Interview Questions

You should be able to explain:

1. What is an SPFILE?
2. What is a PFILE?
3. Difference between PFILE and SPFILE?
4. What does `SCOPE=MEMORY` mean?
5. What does `SCOPE=SPFILE` mean?
6. What does `SCOPE=BOTH` mean?
7. What is a listener?
8. Is the listener part of the database instance?
9. What is `listener.ora`?
10. What is `tnsnames.ora`?
11. What is `sqlnet.ora`?
12. What is a service?
13. Difference between SID and SERVICE_NAME?
14. What does `lsnrctl status` show?
15. What does `tnsping` test?
16. What does ORA-12154 usually indicate?
17. What does ORA-12514 indicate?
18. What does ORA-12541 indicate?
19. What is `LOCAL_LISTENER`?
20. Why would you use `ALTER SYSTEM REGISTER`?
21. How do you check PDBs?
22. How do you open a PDB?
23. How do you save PDB state?
24. How would you troubleshoot an application connection failure?
25. What is the difference between database configuration and network configuration?

---

## Lesson 3 — The DBA Mental Model

You should now be able to visualize:

```text
                       APPLICATION
                            |
                            v
                    Oracle Net Client
                            |
                +-----------+-----------+
                |                       |
          tnsnames.ora             Easy Connect
                |                       |
                +-----------+-----------+
                            |
                            v
                       TCP/IP
                            |
                            v
                       LISTENER
                            |
                    Service Registration
                            |
                            v
                       DB INSTANCE
                            |
                   +--------+--------+
                   |                 |
                 CDB              Services
                   |
             +-----+-----+
             |           |
         CDB$ROOT      PDB
                         |
                         v
                     Application
                     Objects/Data
```

And:

```text
                    INSTANCE
                       |
             +---------+---------+
             |                   |
            SGA                 PGA
             |
       Initialization
       Parameters
             |
        PFILE/SPFILE
```

**Lesson 3/10 complete.**

The next lesson is **Lesson 4/10 — Core DBA Administration**, covering **users, roles, privileges, profiles, tablespaces, datafiles, temporary tablespaces, undo, redo logs, storage management, quotas, and daily DBA health checks**.
