# Lesson 7/10 — Oracle Security & Advanced Administration

**Strictly Lesson 7 of 10.**

This lesson follows Oracle's official 19c security documentation, including authorization, unified auditing, administrative privileges, and multitenant security. Oracle's Security documentation separates these topics into access management, auditing, encryption, centralized user management, and related security features. ([Oracle Documentation][1])

---

# 7.1 Oracle Database Security Model

Think about security in layers:

```text
                 DATABASE SECURITY
                        |
        +---------------+---------------+
        |               |               |
   Authentication   Authorization     Auditing
        |               |               |
      Who?          What can they do?  What happened?
        |               |               |
     Passwords       Roles            Audit trail
     OS auth         Privileges        Policies
     External auth   Object access     Monitoring
```

A secure DBA must control all three:

```text
WHO
 ↓
WHAT
 ↓
WHAT HAPPENED
```

Oracle explicitly describes privileges and roles as mechanisms for controlling what users can access and what actions they can perform, with least privilege as a fundamental principle. ([Oracle Documentation][2])

---

# 7.2 Authentication vs Authorization

This is an essential interview question.

### Authentication

> **Who are you?**

Examples:

```text
Username/password
Operating-system authentication
External authentication
Enterprise authentication
```

### Authorization

> **What are you allowed to do?**

Examples:

```text
SELECT
INSERT
UPDATE
DELETE
CREATE TABLE
ALTER USER
```

So:

```text
Authentication
      ↓
Identity established
      ↓
Authorization
      ↓
Privileges checked
      ↓
Operation allowed/denied
```

---

# 7.3 Users

Create a user:

```sql
CREATE USER appuser
IDENTIFIED BY "StrongPassword";
```

Allow connection:

```sql
GRANT CREATE SESSION TO appuser;
```

Check:

```sql
SELECT username,
       account_status,
       default_tablespace,
       temporary_tablespace
FROM dba_users
WHERE username = 'APPUSER';
```

---

# 7.4 Password Management

A DBA must understand:

```text
Password lifetime
Failed-login limits
Account locking
Password reuse
Password complexity
```

These are commonly controlled through **profiles**.

---

# 7.5 Profiles

A profile can define resource and password-related limits.

Example:

```sql
CREATE PROFILE app_profile
LIMIT
    FAILED_LOGIN_ATTEMPTS 5
    PASSWORD_LIFE_TIME 90
    PASSWORD_REUSE_TIME 365;
```

Then assign:

```sql
ALTER USER appuser
PROFILE app_profile;
```

Check:

```sql
SELECT username,
       profile
FROM dba_users
WHERE username = 'APPUSER';
```

---

# 7.6 Why Profiles Matter

Imagine an account with unlimited password lifetime and unlimited failed attempts.

An attacker could repeatedly attempt authentication.

A properly designed profile can provide controls such as:

```text
Failed attempts
      ↓
Account locked
      ↓
Administrative review
```

The exact security policy should be determined by your organization's requirements.

---

# 7.7 Account Locking

Lock:

```sql
ALTER USER appuser ACCOUNT LOCK;
```

Unlock:

```sql
ALTER USER appuser ACCOUNT UNLOCK;
```

Check:

```sql
SELECT username,
       account_status
FROM dba_users
WHERE username = 'APPUSER';
```

---

# 7.8 Password Expiration

You may encounter:

```text
OPEN
EXPIRED
LOCKED
EXPIRED & LOCKED
```

The DBA should determine why the account entered that state rather than simply changing passwords without understanding the underlying policy.

---

# 7.9 Roles

A role is a named collection of privileges.

Oracle's documentation describes roles as groups of privileges used to enable users to perform their required tasks. ([Oracle Documentation][3])

Create:

```sql
CREATE ROLE app_read_role;
```

Grant:

```sql
GRANT SELECT ON hr.employees
TO app_read_role;
```

Then:

```sql
GRANT app_read_role
TO appuser;
```

Conceptually:

```text
                  APP_READ_ROLE
                       |
              +--------+--------+
              |                 |
        SELECT HR.EMPLOYEES   ...
              |
              v
           APPUSER
```

---

# 7.10 Role Hierarchies

Roles can contain other roles.

For example:

```sql
CREATE ROLE app_rw_role;

GRANT app_read_role
TO app_rw_role;
```

Then:

```sql
GRANT app_rw_role
TO appuser;
```

Conceptually:

```text
APP_RW_ROLE
     |
     +-- APP_READ_ROLE
             |
             +-- SELECT
```

This can simplify administration.

---

# 7.11 System Privileges

Examples:

```sql
GRANT CREATE SESSION TO appuser;
```

```sql
GRANT CREATE TABLE TO appuser;
```

```sql
GRANT CREATE VIEW TO appuser;
```

These authorize classes of operations.

---

# 7.12 Object Privileges

Example:

```sql
GRANT SELECT
ON hr.employees
TO appuser;
```

Another:

```sql
GRANT INSERT, UPDATE
ON hr.employees
TO appuser;
```

You can revoke:

```sql
REVOKE UPDATE
ON hr.employees
FROM appuser;
```

---

# 7.13 ANY Privileges

Be extremely careful with privileges such as:

```text
SELECT ANY TABLE
UPDATE ANY TABLE
DELETE ANY TABLE
CREATE ANY TABLE
DROP ANY TABLE
```

They can grant broad authority.

For example:

```sql
GRANT SELECT ANY TABLE TO appuser;
```

is dramatically broader than:

```sql
GRANT SELECT
ON hr.employees
TO appuser;
```

Prefer narrowly scoped privileges wherever possible.

---

# 7.14 Least Privilege

Your security architecture should resemble:

```text
Application
    |
    v
Application Role
    |
    +-- Required privileges only
```

Not:

```text
Application
    |
    v
DBA
```

Oracle specifically recommends granting privileges according to required job tasks. ([Oracle Documentation][2])

---

# 7.15 Administrative Privileges

Oracle provides administrative privileges for specialized operations, including:

```text
SYSDBA
SYSOPER
SYSBACKUP
SYSDG
SYSKM
SYSRAC
```

Oracle documents these as more task-specific administrative privileges intended to support separation of duties. ([Oracle Documentation][4])

For example:

```text
SYSBACKUP
    ↓
Backup/recovery administration

SYSDG
    ↓
Data Guard operations

SYSKM
    ↓
TDE key-management operations

SYSRAC
    ↓
RAC-related administration
```

---

# 7.16 SYSDBA

`SYSDBA` is extremely powerful.

It is not simply:

> "A normal DBA role."

It is an administrative privilege with special capabilities.

Therefore:

```text
SYSDBA
```

should be tightly controlled.

Oracle recommends using more specialized administrative privileges where appropriate for separation of duties, while `SYSDBA` remains available for compatibility and broad administration. ([Oracle Documentation][4])

---

# 7.17 Password File

Oracle can use a password file for administrative authentication.

You can inspect administrative users with:

```sql
SELECT username,
       sysdba,
       sysoper,
       sysbackup,
       sysdg,
       syskm,
       sysrac
FROM v$pwfile_users;
```

This is especially important when administering databases remotely.

---

# 7.18 Granting SYSBACKUP

For an appropriate administrative account:

```sql
GRANT SYSBACKUP TO backup_admin;
```

This is an example of separation of duties.

Instead of giving the backup administrator:

```text
SYSDBA
```

you can use the task-specific administrative privilege where appropriate.

---

# 7.19 Auditing

Now we move from:

```text
Who can do something?
```

to:

```text
Who actually did something?
```

Oracle Unified Auditing provides a unified framework for auditing user activity, roles, privileges, object actions, and other database activities. ([Oracle Documentation][5])

---

# 7.20 Unified Auditing

Key concepts:

```text
Audit Policy
      ↓
Enable Policy
      ↓
User performs activity
      ↓
Audit Record
      ↓
UNIFIED_AUDIT_TRAIL
```

Check available policies:

```sql
SELECT policy_name
FROM audit_unified_policies
ORDER BY policy_name;
```

---

# 7.21 Create an Audit Policy

Example:

```sql
CREATE AUDIT POLICY app_user_audit_pol
ACTIONS SELECT, INSERT, UPDATE, DELETE
ON hr.employees;
```

Then enable:

```sql
AUDIT POLICY app_user_audit_pol
BY appuser;
```

Oracle documents `AUDIT POLICY` as the mechanism for enabling unified audit policies, including policies applied to specific users or roles. ([Oracle Documentation][6])

---

# 7.22 Query Audit Records

Use:

```sql
SELECT event_timestamp,
       dbusername,
       action_name,
       object_schema,
       object_name,
       return_code
FROM unified_audit_trail
ORDER BY event_timestamp DESC;
```

This can help answer:

```text
Who?
When?
What operation?
Which object?
Succeeded or failed?
```

---

# 7.23 Audit Failed Attempts

You can configure policies to capture unsuccessful activity:

```sql
AUDIT POLICY app_user_audit_pol
BY appuser
WHENEVER NOT SUCCESSFUL;
```

Oracle supports `WHENEVER SUCCESSFUL` and `WHENEVER NOT SUCCESSFUL` for unified auditing. ([Oracle Documentation][6])

This is useful for investigating:

```text
Repeated failed operations
Unauthorized access attempts
Potential misuse
Application errors
```

---

# 7.24 Audit Roles

Oracle can audit role usage as well.

For example:

```sql
CREATE AUDIT POLICY audit_role_pol
ROLES DBA;
```

Then:

```sql
AUDIT POLICY audit_role_pol;
```

Oracle documents the `ROLES` clause specifically for creating unified audit policies that audit role use. ([Oracle Documentation][7])

---

# 7.25 Audit System Privileges

Oracle also supports policies for auditing system privilege use.

Example:

```sql
CREATE AUDIT POLICY audit_any_table_pol
PRIVILEGES SELECT ANY TABLE;
```

Then:

```sql
AUDIT POLICY audit_any_table_pol;
```

Oracle documents the `PRIVILEGES` clause for this purpose. ([Oracle Documentation][8])

---

# 7.26 Don't Audit Everything Blindly

More auditing isn't automatically better.

Excessive auditing can create:

```text
Large audit volume
      ↓
Storage growth
      ↓
Management overhead
      ↓
Potential performance/operational impact
```

Instead:

```text
Security requirement
       ↓
What must be audited?
       ↓
Who?
       ↓
Which objects?
       ↓
Which actions?
       ↓
How long retained?
```

---

# 7.27 Audit Retention

Audit records themselves need lifecycle management.

You must plan:

```text
Collection
   ↓
Storage
   ↓
Retention
   ↓
Archiving
   ↓
Purging
```

For production, coordinate retention with:

* Compliance
* Security
* Legal requirements
* Storage capacity

---

# 7.28 TDE — Transparent Data Encryption

Now encryption.

Oracle Advanced Security includes **Transparent Data Encryption (TDE)**. Oracle's official documentation describes TDE as a mechanism for encrypting database data. ([Oracle Documentation][1])

Conceptually:

```text
Application
     |
     v
Oracle Database
     |
     v
Encrypted Data
     |
     v
Disk
```

The application generally does not need to manually encrypt/decrypt each database value when TDE is configured appropriately.

---

# 7.29 Encryption at Rest

TDE primarily addresses:

> **Data at rest**

For example:

```text
Database files
Backup-related data
Storage
```

It should not be confused with network encryption.

---

# 7.30 Encryption in Transit

For network communication, Oracle supports technologies such as:

```text
TLS / TCPS
Native network encryption
```

Conceptually:

```text
Client
   |
   | encrypted connection
   v
Network
   |
   v
Oracle Listener
   |
   v
Database
```

So remember:

```text
TDE
 ↓
Data at rest

TLS/TCPS/network encryption
 ↓
Data in transit
```

---

# 7.31 TDE Keystore

TDE relies on key-management infrastructure.

Conceptually:

```text
TDE
 |
 +-- Keystore
       |
       +-- Encryption keys
```

The keystore must be managed securely.

This is why Oracle provides the `SYSKM` administrative privilege for TDE wallet/key-management operations. ([Oracle Documentation][4])

---

# 7.32 Wallet / Keystore

You will encounter terminology such as:

```text
Keystore
Wallet
TDE master key
```

Don't treat these as merely "password files."

They are part of the cryptographic key-management architecture.

Production procedures must protect:

```text
Keys
Keystore
Passwords/secrets
Backups
Recovery procedures
```

---

# 7.33 Data Redaction

Oracle Advanced Security also includes data redaction capabilities. ([Oracle Documentation][1])

Conceptually:

```text
Database value:
4111111111111111

Application sees:
XXXXXXXXXXXX1111
```

The underlying data may remain unchanged while presentation to a user is controlled.

This is different from:

```text
Encryption
```

and:

```text
Data masking
```

---

# 7.34 Database Links

A database link allows SQL to access objects in another database.

Conceptually:

```text
DATABASE A
    |
    | DB LINK
    v
DATABASE B
```

Example:

```sql
SELECT *
FROM employees@remote_db;
```

Database links can be useful, but they introduce security and operational considerations.

---

# 7.35 Database Link Security

Consider:

```text
Credentials
Network security
Remote privileges
Encryption
Password storage
Dependency on remote DB
```

Avoid creating broad database links without understanding the trust relationship.

---

# 7.36 DIRECTORY Objects

Oracle directory objects provide database references to OS directories.

Example:

```sql
CREATE DIRECTORY dp_dir
AS '/u01/app/oracle/dpump';
```

Then:

```sql
GRANT READ, WRITE
ON DIRECTORY dp_dir
TO appuser;
```

Important:

> A DIRECTORY object does not itself create or secure an OS directory.

The operating-system directory and OS permissions must exist separately.

---

# 7.37 Scheduler

Oracle Scheduler can automate jobs.

For example:

```text
Daily task
     |
     v
Oracle Scheduler
     |
     v
Procedure/script/job
```

Check jobs:

```sql
SELECT owner,
       job_name,
       enabled,
       state
FROM dba_scheduler_jobs
ORDER BY owner, job_name;
```

---

# 7.38 Scheduler Security

A DBA must know:

```text
Who owns the job?
Who can execute it?
What does it run?
What privileges does it have?
Where does it write?
How often does it run?
```

A scheduled job running with excessive privileges can become a security risk.

---

# 7.39 Resource Manager

Oracle Database Resource Manager can help control resource usage among workloads.

Conceptually:

```text
DATABASE
   |
   v
RESOURCE MANAGER
   |
   +-- OLTP
   +-- Reporting
   +-- Batch
   +-- Maintenance
```

You can design resource plans to control allocation among competing workloads.

---

# 7.40 Why Resource Manager Matters

Suppose:

```text
09:00
```

A reporting query starts consuming enormous resources.

Meanwhile:

```text
OLTP application
```

needs predictable response times.

Resource management can help prioritize workloads according to business requirements.

---

# 7.41 Multitenant Security

Modern Oracle Database environments commonly use:

```text
CDB
 |
 +-- PDB1
 +-- PDB2
 +-- PDB3
```

Privileges can be:

```text
Common
```

or:

```text
Local
```

Oracle documents that in a multitenant environment privileges can be granted commonly across a CDB/application container or locally within a PDB. ([Oracle Documentation][2])

---

# 7.42 Common vs Local User

Conceptually:

```text
CDB$ROOT
   |
   +-- Common user
   |
   +-- PDB1
   +-- PDB2
   +-- PDB3
```

A local user belongs to a particular PDB.

A common user is designed for administration across the container environment, subject to Oracle's multitenant rules.

---

# 7.43 Check Current Container

Always know where you are.

```sql
SHOW CON_NAME;
```

Or:

```sql
SELECT sys_context(
    'USERENV',
    'CON_NAME'
)
FROM dual;
```

This is **extremely important** in a multitenant database.

Before executing:

```sql
CREATE USER
GRANT
ALTER SYSTEM
AUDIT
```

ask:

> **Am I in CDB$ROOT or the intended PDB?**

---

# 7.44 Container Switching

Depending on your privileges and environment:

```sql
ALTER SESSION SET CONTAINER = PDB1;
```

Then:

```sql
SHOW CON_NAME;
```

Verify before performing administrative operations.

A large number of multitenant administration mistakes come from performing an otherwise valid command in the wrong container.

---

# 7.45 Common Administrative Mistake

Imagine:

```text
CDB$ROOT
```

You intend to modify:

```text
PDB1
```

but forget:

```sql
ALTER SESSION SET CONTAINER = PDB1;
```

You may accidentally apply a common/root-level operation rather than the intended PDB-local operation.

Therefore:

```text
CONNECT
   ↓
SHOW CON_NAME
   ↓
VERIFY
   ↓
ADMINISTER
```

---

# 7.46 Security Views You Must Know

Start memorizing these:

```text
DBA_USERS
DBA_ROLES
DBA_SYS_PRIVS
DBA_TAB_PRIVS
DBA_ROLE_PRIVS
ROLE_SYS_PRIVS
ROLE_TAB_PRIVS
DBA_PROFILES
V$PWFILE_USERS
UNIFIED_AUDIT_TRAIL
AUDIT_UNIFIED_POLICIES
AUDIT_UNIFIED_ENABLED_POLICIES
```

These are fundamental DBA/security views.

Oracle documents the privilege/role dictionary views as the primary way to inspect role and privilege domains. ([Oracle Documentation][9])

---

# 7.47 Security Audit Query

A useful starting point:

```sql
SELECT dbusername,
       action_name,
       object_schema,
       object_name,
       event_timestamp,
       return_code
FROM unified_audit_trail
ORDER BY event_timestamp DESC
FETCH FIRST 50 ROWS ONLY;
```

This gives you a basic view of recent audited activity.

---

# 7.48 Security Review Query

Users:

```sql
SELECT username,
       account_status,
       profile
FROM dba_users
ORDER BY username;
```

Roles:

```sql
SELECT grantee,
       granted_role
FROM dba_role_privs
ORDER BY grantee;
```

System privileges:

```sql
SELECT grantee,
       privilege
FROM dba_sys_privs
ORDER BY grantee, privilege;
```

Object privileges:

```sql
SELECT grantee,
       owner,
       table_name,
       privilege
FROM dba_tab_privs
ORDER BY grantee, owner, table_name;
```

---

# 7.49 Security Hardening Checklist

A DBA should periodically review:

```text
[ ] Unused users
[ ] Locked users
[ ] Expired users
[ ] Default accounts
[ ] Excessive privileges
[ ] ANY privileges
[ ] DBA role assignments
[ ] Administrative privileges
[ ] Password profiles
[ ] Audit policies
[ ] Audit retention
[ ] TDE/keystore status
[ ] Network encryption
[ ] Database links
[ ] DIRECTORY objects
[ ] Scheduler jobs
[ ] OS permissions
[ ] Listener configuration
[ ] Multitenant common privileges
```

---

# 7.50 Windows Security Considerations

On Windows, Oracle security also interacts with:

```text
Windows accounts
Oracle services
File-system ACLs
Windows Event Viewer
Credential management
```

For example, database files must be protected by appropriate Windows permissions.

Don't rely solely on Oracle privileges.

Security is:

```text
Oracle
 +
Operating System
 +
Network
 +
Storage
 +
People/process
```

---

# 7.51 Linux Security Considerations

On Linux:

```bash
ls -l
```

can help inspect file ownership and permissions.

For example:

```bash
ls -l /u01/app/oracle
```

Check:

```text
Owner
Group
Permissions
```

Oracle's OS account should not have unnecessarily broad access.

---

# 7.52 Lesson 7 Lab — Create a Secure Application User

Create:

```sql
CREATE PROFILE app_profile
LIMIT
    FAILED_LOGIN_ATTEMPTS 5
    PASSWORD_LIFE_TIME 90;
```

Create user:

```sql
CREATE USER secure_app
IDENTIFIED BY "StrongPassword"
PROFILE app_profile;
```

Allow connection:

```sql
GRANT CREATE SESSION TO secure_app;
```

Check:

```sql
SELECT username,
       account_status,
       profile
FROM dba_users
WHERE username = 'SECURE_APP';
```

---

# 7.53 Lesson 7 Lab — Role-Based Security

Create:

```sql
CREATE ROLE secure_app_role;
```

Grant only what is required:

```sql
GRANT SELECT
ON hr.employees
TO secure_app_role;
```

Then:

```sql
GRANT secure_app_role
TO secure_app;
```

Verify:

```sql
SELECT grantee,
       granted_role
FROM dba_role_privs
WHERE grantee = 'SECURE_APP';
```

---

# 7.54 Lesson 7 Lab — Audit

Create a lab policy:

```sql
CREATE AUDIT POLICY secure_app_audit
ACTIONS SELECT ON hr.employees;
```

Enable it:

```sql
AUDIT POLICY secure_app_audit
BY secure_app;
```

Then perform an appropriate test operation using `SECURE_APP`.

Query:

```sql
SELECT dbusername,
       action_name,
       object_schema,
       object_name,
       event_timestamp
FROM unified_audit_trail
WHERE dbusername = 'SECURE_APP'
ORDER BY event_timestamp DESC;
```

Remember that Oracle notes that an enabled unified audit policy takes effect for subsequent sessions; users who were already logged in may need to reconnect before the policy applies to their session. ([Oracle Documentation][6])

---

# 7.55 Lesson 7 Lab — Multitenant

If your database is multitenant:

```sql
SHOW CON_NAME;
```

Then:

```sql
SELECT name,
       open_mode
FROM v$pdbs;
```

Switch to your lab PDB:

```sql
ALTER SESSION SET CONTAINER = PDB1;
```

Verify:

```sql
SHOW CON_NAME;
```

Then perform only **PDB-local lab operations**.

Return:

```sql
ALTER SESSION SET CONTAINER = CDB$ROOT;
```

Verify again.

---

# 7.56 Lesson 7 Interview Questions

You should be able to answer:

1. Authentication vs authorization?
2. What is a profile?
3. Why use password profiles?
4. What is least privilege?
5. System privilege vs object privilege?
6. Role vs privilege?
7. Why avoid excessive `ANY` privileges?
8. What is `SYSDBA`?
9. What is `SYSBACKUP`?
10. What is `SYSDG`?
11. What is `SYSKM`?
12. What is a password file?
13. What is Unified Auditing?
14. What is an audit policy?
15. How do you enable a unified audit policy?
16. Where do you query unified audit records?
17. What is `AUDIT_UNIFIED_POLICIES`?
18. What is `UNIFIED_AUDIT_TRAIL`?
19. How do you audit system privilege usage?
20. How do you audit role usage?
21. What is TDE?
22. What does TDE protect?
23. TDE vs network encryption?
24. What is a keystore?
25. What is a database link?
26. What is a DIRECTORY object?
27. What is Oracle Scheduler?
28. What is Resource Manager?
29. Common user vs local user?
30. Common privilege vs local privilege?
31. Why is `SHOW CON_NAME` important?
32. How does Oracle security interact with Windows/Linux security?

---

# 7.57 The Security Mental Model

Memorize this:

```text
                    SECURITY
                       |
        +--------------+--------------+
        |              |              |
 AUTHENTICATION   AUTHORIZATION    AUDITING
        |              |              |
       WHO?          WHAT?         WHAT HAPPENED?
        |              |              |
    Passwords       Roles          Policies
    OS Auth         Privileges     Audit trail
    Admin Auth      Objects        Monitoring
                       |
                  Least Privilege
                       |
              +--------+--------+
              |                 |
            TDE             Network Security
              |                 |
        Data at Rest       Data in Transit
```

And for multitenant:

```text
                    CDB
                     |
                 CDB$ROOT
                     |
          +----------+----------+
          |          |          |
         PDB1       PDB2       PDB3
          |
      Local Users
      Local Roles
      Local Objects
```

**Always know which container you're administering.**

---

## Official Oracle sources

Oracle's official 19c security documentation covers the security areas in this lesson: ([Oracle Documentation][1])

* **Database Security Guide** — authorization, roles, privileges, auditing, multitenant security. ([Oracle Documentation][2])
* **Advanced Security Guide** — TDE and data redaction. ([Oracle Documentation][1])
* **Administrative Privileges** — `SYSDBA`, `SYSBACKUP`, `SYSDG`, `SYSKM`, `SYSRAC`, and separation of duties. ([Oracle Documentation][4])
* **Unified Auditing** — audit policies, users/roles, and audit trail. ([Oracle Documentation][6])

**Lesson 7/10 complete.**

### Course progress

```text
1. Oracle Architecture & DBA Foundations       ✓
2. Installation & Database Creation             ✓
3. Configuration & Oracle Networking            ✓
4. Core DBA Administration                     ✓
5. Backup & Recovery / RMAN                    ✓
6. Performance Monitoring & Troubleshooting   ✓
7. Security & Advanced Administration         ✓
8. Patching & Patch Management
9. Migration & Database Upgrades
10. RAC, Data Guard & End-to-End DBA Operations
```

**Next: Lesson 8/10 — Oracle Patching & Patch Management**, covering **Oracle Release Updates (RUs), Release Update Revisions (RURs), CPUs/PSUs terminology, OPatch, OPatchAuto, inventory, patch prerequisites, conflict checks, rollback, Grid Infrastructure patching, Windows vs Linux patching, out-of-place patching, and a production patching runbook.**

[1]: https://docs.oracle.com/en/database/oracle/oracle-database/19/security.html "Oracle Database 19c - Security"
[2]: https://docs.oracle.com/en/database/oracle/oracle-database/19/dbseg/configuring-privilege-and-role-authorization.htmlm "Configuring Privilege and Role Authorization"
[3]: https://docs.oracle.com/en/database/oracle/oracle-database/19/adfns/security.html "Security"
[4]: https://docs.oracle.com/en/database/oracle/oracle-database/19/dbseg/managing-administrative-privileges.html "Managing Administrative Privileges"
[5]: https://docs.oracle.com/en/database/oracle/oracle-database/19/dbseg/custom-unified-audit-policies.html "About Auditing Activities with Unified Audit Policies and AUDIT"
[6]: https://docs.oracle.com/en/database/oracle/oracle-database/19/dbseg/enabling-and-applying-unified-audit-policies-users-and-roles.html "Enabling and Applying Unified Audit Policies to Users and Roles"
[7]: https://docs.oracle.com/en/database/oracle/oracle-database/19/dbseg/auditing-roles.html "Auditing Roles"
[8]: https://docs.oracle.com/en/database/oracle/oracle-database/19/dbseg/auditing-system-privileges.html "Auditing System Privileges"
[9]: https://docs.oracle.com/en/database/oracle/oracle-database/19/dbseg/user-privilege-and-role-data-dictionary-views.html "User Privilege and Role Data Dictionary Views"
