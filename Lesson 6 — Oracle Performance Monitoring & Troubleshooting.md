# Lesson 6/10 — Oracle Performance Monitoring & Troubleshooting

**Strictly Lesson 6 of 10.**

This lesson focuses on how an Oracle DBA diagnoses a slow or unhealthy database. The concepts and terminology follow Oracle's official documentation for Oracle Database performance tuning and diagnostics.

> **Core DBA rule:**
> **Don't tune what you haven't measured.**
> First identify the symptom → collect evidence → find the bottleneck → make one controlled change → measure again.

---

# 6.1 What Does "Oracle Is Slow" Actually Mean?

When someone says:

> "The database is slow."

That isn't a diagnosis.

You need to determine **what is slow**:

```text
Application
    |
    v
Connection
    |
    v
SQL
    |
    v
Oracle session
    |
    v
CPU / Memory / I/O / Network
    |
    v
Storage / OS
```

Possible causes include:

* CPU saturation
* Excessive physical I/O
* Poor SQL
* Missing or inappropriate indexes
* Blocking sessions
* Lock contention
* Memory pressure
* Excessive parsing
* Network latency
* Storage latency
* Application connection problems
* Bad execution plans
* Configuration problems

---

# 6.2 The Performance Troubleshooting Method

Use this sequence:

```text
1. Define the symptom
        ↓
2. Determine the time period
        ↓
3. Determine affected users
        ↓
4. Check database health
        ↓
5. Check OS resources
        ↓
6. Check sessions
        ↓
7. Check wait events
        ↓
8. Identify expensive SQL
        ↓
9. Examine execution plans
        ↓
10. Correct the root cause
        ↓
11. Measure again
```

Never start with:

```text
"Increase SGA."
```

or:

```text
"Add an index."
```

without evidence.

---

# 6.3 Dynamic Performance Views

You've already used views such as:

```sql
V$DATABASE
V$INSTANCE
V$SESSION
V$LOG
```

Performance troubleshooting heavily uses Oracle's dynamic performance views.

Examples:

```text
V$SESSION
V$SQL
V$SQLAREA
V$SYSTEM_EVENT
V$SESSION_WAIT
V$SYSSTAT
V$SYSTEM_PARAMETER
```

The exact available views and columns vary by Oracle release.

---

# 6.4 V$SESSION

Start with sessions:

```sql
SELECT sid,
       serial#,
       username,
       status,
       event,
       wait_class,
       sql_id,
       machine,
       program
FROM v$session
WHERE username IS NOT NULL
ORDER BY status DESC;
```

Important information:

```text
SID
SERIAL#
USERNAME
STATUS
EVENT
WAIT_CLASS
SQL_ID
MACHINE
PROGRAM
```

---

# 6.5 ACTIVE Sessions

Find active sessions:

```sql
SELECT sid,
       serial#,
       username,
       event,
       wait_class,
       sql_id
FROM v$session
WHERE status = 'ACTIVE'
ORDER BY username;
```

But remember:

> **ACTIVE does not necessarily mean consuming CPU.**

An active session may be waiting for:

* I/O
* Locks
* Network
* CPU scheduling
* Other database resources

---

# 6.6 Wait Events

Oracle sessions spend time either doing work or waiting for something.

Examples of wait categories include:

```text
CPU
User I/O
System I/O
Concurrency
Commit
Network
Configuration
Administrative
Application
Cluster
```

The wait event tells you what Oracle is waiting for.

---

# 6.7 Why Wait Events Matter

Suppose the application reports:

> "The query takes 30 seconds."

You discover:

```text
Database CPU = low
```

That doesn't mean Oracle is healthy.

The session could be waiting on:

```text
I/O
```

or:

```text
Lock
```

or:

```text
Network
```

Therefore:

```text
Low CPU ≠ automatically good performance
```

---

# 6.8 Finding Current Waits

Example:

```sql
SELECT sid,
       serial#,
       username,
       event,
       wait_class,
       seconds_in_wait,
       sql_id
FROM v$session
WHERE status = 'ACTIVE'
ORDER BY seconds_in_wait DESC;
```

Look for patterns rather than reacting to one row.

---

# 6.9 Blocking Sessions

Check:

```sql
SELECT sid,
       serial#,
       username,
       blocking_session,
       event,
       sql_id
FROM v$session
WHERE blocking_session IS NOT NULL;
```

Conceptually:

```text
Session A
   |
   +-- owns lock
   |
   v
Session B
   |
   +-- waits
   |
   v
Application delay
```

Your job is to identify **why A is holding the lock**, not merely kill A.

---

# 6.10 Lock Troubleshooting

A useful first step is:

```sql
SELECT
    sid,
    serial#,
    username,
    blocking_session,
    event,
    sql_id
FROM v$session
WHERE blocking_session IS NOT NULL;
```

Then investigate the blocker.

Questions:

```text
Who is the blocker?
What SQL is it executing?
Is it a legitimate transaction?
How long has it been active?
Is the application holding a transaction open?
Is this recurring?
```

---

# 6.11 V$SQL

`V$SQL` provides information about SQL statements in the shared SQL area.

Example:

```sql
SELECT sql_id,
       executions,
       elapsed_time,
       cpu_time,
       buffer_gets,
       disk_reads,
       rows_processed
FROM v$sql
WHERE executions > 0
ORDER BY elapsed_time DESC
FETCH FIRST 20 ROWS ONLY;
```

This gives you a starting point for finding expensive SQL.

---

# 6.12 What Makes SQL "Expensive"?

There isn't one definition.

A SQL statement might be expensive because of:

```text
High elapsed time
High CPU
High logical reads
High physical reads
Many executions
Large result set
Lock waits
Poor execution plan
```

For example:

```text
SQL A
Execution time = 10 seconds
Executions = 1

SQL B
Execution time = 0.1 seconds
Executions = 1,000,000
```

SQL B can have a much larger overall impact.

---

# 6.13 Total Work Matters

Think:

```text
Total Cost
=
Cost per execution
×
Number of executions
```

Therefore don't only search for:

> "Longest single SQL."

Also look for:

> "SQL generating the most total workload."

---

# 6.14 SQL_ID

Oracle identifies SQL statements using a `SQL_ID`.

Example:

```text
SQL_ID
------
8abc123xyz
```

Once you have the SQL_ID, you can investigate the statement.

For example:

```sql
SELECT sql_id,
       child_number,
       executions,
       elapsed_time,
       cpu_time
FROM v$sql
WHERE sql_id = '8abc123xyz';
```

---

# 6.15 Getting SQL Text

You can retrieve SQL text:

```sql
SELECT sql_id,
       sql_text
FROM v$sql
WHERE sql_id = '8abc123xyz';
```

Depending on the statement length, `SQL_FULLTEXT` or related mechanisms may be more appropriate.

---

# 6.16 Execution Plans

The execution plan describes how Oracle intends to execute SQL.

Conceptually:

```text
SQL
 |
 v
Optimizer
 |
 v
Execution Plan
 |
 +-- Table Access
 +-- Index Access
 +-- Join
 +-- Sort
 +-- Filter
```

A plan might contain operations such as:

```text
TABLE ACCESS FULL
INDEX RANGE SCAN
NESTED LOOPS
HASH JOIN
SORT
```

---

# 6.17 Explain Plan

A common learning command:

```sql
EXPLAIN PLAN FOR
SELECT *
FROM employees
WHERE employee_id = 100;
```

Then:

```sql
SELECT *
FROM TABLE(DBMS_XPLAN.DISPLAY);
```

`DBMS_XPLAN` is an important Oracle utility for displaying execution plans.

---

# 6.18 Actual vs Estimated Rows

One of the most important tuning concepts is:

```text
Estimated rows
        vs
Actual rows
```

If Oracle estimates:

```text
10 rows
```

but reality is:

```text
10,000,000 rows
```

the optimizer may choose a poor execution strategy.

This can lead to major performance problems.

---

# 6.19 Statistics

Oracle's optimizer relies heavily on statistics.

Statistics describe characteristics of database objects such as:

* Tables
* Columns
* Indexes

They help Oracle estimate:

```text
How many rows?
How selective is this predicate?
Which access path is appropriate?
Which join method is appropriate?
```

---

# 6.20 DBMS_STATS

Oracle provides the `DBMS_STATS` package for statistics management.

For example:

```sql
EXEC DBMS_STATS.GATHER_TABLE_STATS(
    ownname => 'APPUSER',
    tabname => 'EMPLOYEES'
);
```

Don't blindly gather statistics on everything every time performance declines.

First determine whether stale or inappropriate statistics are actually contributing to the problem.

---

# 6.21 Indexes

Indexes can improve access to data.

Conceptually:

```text
Table
 |
 +-- Millions of rows
```

Without an appropriate index, Oracle may need to inspect many rows.

With an appropriate index:

```text
Index
 |
 +-- Locate relevant row(s)
          |
          v
        Table
```

But indexes are not free.

They add:

* Storage
* DML maintenance
* Insert/update/delete overhead
* Potential optimizer complexity

---

# 6.22 Don't Add Indexes Blindly

Suppose:

```sql
SELECT *
FROM orders
WHERE customer_id = 1001;
```

You might consider an index:

```sql
CREATE INDEX idx_orders_customer
ON orders(customer_id);
```

But before creating it, ask:

```text
Does the query need an index?
Is the column selective?
Does an appropriate index already exist?
How frequently is the query executed?
What does the current plan show?
Will DML overhead increase?
```

---

# 6.23 CPU Troubleshooting

At the OS level, check CPU.

Linux:

```bash
top
```

or:

```bash
vmstat 1
```

Windows PowerShell:

```powershell
Get-Counter '\Processor(_Total)\% Processor Time'
```

Then correlate OS CPU with Oracle activity.

---

# 6.24 High CPU Scenario

Suppose:

```text
CPU = 98%
```

Don't immediately increase CPU.

Find:

```text
Which process?
      ↓
Oracle?
      ↓
Which sessions?
      ↓
Which SQL?
      ↓
Why is SQL consuming CPU?
```

Oracle query:

```sql
SELECT sql_id,
       executions,
       cpu_time,
       elapsed_time
FROM v$sql
ORDER BY cpu_time DESC
FETCH FIRST 20 ROWS ONLY;
```

---

# 6.25 I/O Troubleshooting

If storage is slow:

```text
SQL
 ↓
Oracle
 ↓
I/O request
 ↓
Storage
 ↓
Response
```

High I/O wait can result from:

* Poor SQL
* Full table scans
* Large sorts
* Insufficient memory
* Storage latency
* Application workload

Don't automatically blame the storage system.

---

# 6.26 Linux I/O

Useful OS tools include:

```bash
iostat
```

and:

```bash
vmstat
```

Depending on the Linux distribution, `iostat` may be provided by the `sysstat` package.

Look at:

```text
I/O utilization
Read/write rates
Latency
Queue depth
```

---

# 6.27 Windows I/O

Windows provides Performance Monitor counters.

For example:

```text
PhysicalDisk
LogicalDisk
Processor
Memory
Network Interface
```

The important DBA skill is **correlation**:

```text
Oracle wait
     +
OS metric
     +
SQL
     =
Likely root cause
```

---

# 6.28 Memory

Oracle memory management can involve:

```text
SGA
PGA
```

### SGA

Shared memory used by the Oracle instance.

Conceptually:

```text
SGA
 |
 +-- Buffer Cache
 +-- Shared Pool
 +-- Redo Buffer
 +-- Other components
```

### PGA

Process-specific memory.

Conceptually:

```text
Server Process
      |
      +-- PGA
```

---

# 6.29 Check Memory Parameters

```sql
SHOW PARAMETER sga;
```

```sql
SHOW PARAMETER pga;
```

Depending on configuration, Oracle may use automatic memory-management or automatic shared-memory-management approaches.

The exact parameters in use must be determined from the database.

---

# 6.30 Don't Tune Memory by Guessing

Bad DBA approach:

```text
"Database is slow."
        ↓
Increase SGA.
```

Better:

```text
Database slow
    ↓
Identify workload
    ↓
Identify waits
    ↓
Check memory pressure
    ↓
Check SQL
    ↓
Check I/O
    ↓
Determine root cause
```

---

# 6.31 AWR

**Automatic Workload Repository (AWR)** stores performance-related historical information.

It allows you to compare database activity over time.

Conceptually:

```text
10:00 Snapshot
      |
      v
11:00 Snapshot
      |
      v
12:00 Snapshot
```

You can investigate what happened during a period.

---

# 6.32 AWR Reports

AWR reports are particularly useful when the problem:

* Happened in the past
* Is intermittent
* Is difficult to reproduce
* Needs historical comparison

Typical workflow:

```text
Problem occurred
      ↓
Identify start/end time
      ↓
Find AWR snapshots
      ↓
Generate report
      ↓
Analyze workload
```

---

# 6.33 AWR Licensing

This is critical.

Oracle performance-management features have licensing implications depending on Oracle Database edition and options.

Do **not** assume that because a command exists in your installation, you are automatically licensed to use every associated feature.

Before using AWR/ASH/ADDM in a production environment, verify your Oracle edition, version, and applicable Oracle licensing terms.

---

# 6.34 ASH

**Active Session History (ASH)** records sampled information about active sessions.

Conceptually:

```text
Session activity
      |
      v
Sampling
      |
      v
Historical performance analysis
```

ASH is particularly useful for answering:

> What were sessions doing during this specific period?

---

# 6.35 ADDM

**Automatic Database Diagnostic Monitor (ADDM)** analyzes database performance data and identifies potential problems.

Conceptually:

```text
AWR data
   ↓
ADDM
   ↓
Find significant performance problems
   ↓
Recommendations
```

Again, verify licensing before using applicable diagnostic/tuning features in production.

---

# 6.36 Alert Log

Every Oracle DBA must know the alert log.

It records important database events such as:

* Startup
* Shutdown
* Errors
* Recovery events
* Checkpoint information
* Administrative operations
* Other significant messages

The exact content varies with the event and Oracle release.

---

# 6.37 ADR

Oracle uses the:

**Automatic Diagnostic Repository (ADR)**

for diagnostic information.

The ADR can contain things such as:

```text
Alert log
Trace files
Incident information
Diagnostic data
```

---

# 6.38 ADRCI

Oracle provides:

```text
adrci
```

Start:

```bash
adrci
```

Then:

```text
adrci> show homes
```

You can inspect diagnostic information from there.

For example:

```text
adrci> show alert -tail 50
```

The exact ADR behavior and available commands depend on your Oracle version.

---

# 6.39 Trace Files

When deeper troubleshooting is required, Oracle can generate trace information.

Conceptually:

```text
Session
   |
   v
Trace
   |
   v
Diagnostic analysis
```

Trace files can help investigate:

* SQL execution
* Errors
* Waits
* Internal processing

Don't randomly enable tracing in production because it can generate significant diagnostic data and overhead.

---

# 6.40 Common Performance Symptoms

### Symptom 1

```text
CPU = 100%
```

Investigate:

```text
Top CPU SQL
Top CPU sessions
Execution count
Plan changes
Application workload
```

---

### Symptom 2

```text
CPU low
Database slow
```

Investigate:

```text
Wait events
I/O
Locks
Network
Concurrency
```

---

### Symptom 3

```text
One application query suddenly became slow
```

Investigate:

```text
SQL_ID
Execution plan
Statistics
Bind behavior
Data distribution
Recent changes
```

---

### Symptom 4

```text
Many sessions waiting
```

Investigate:

```text
Blocking
Locks
I/O
Concurrency
Database-wide waits
```

---

# 6.41 Performance Troubleshooting Example

Application team says:

> "Orders are taking 20 seconds today. Yesterday they took 1 second."

Don't immediately tune.

Start:

```text
1. What SQL?
       ↓
2. What SQL_ID?
       ↓
3. What changed?
       ↓
4. Execution plan changed?
       ↓
5. Statistics changed?
       ↓
6. Data volume changed?
       ↓
7. Wait events changed?
       ↓
8. Storage latency changed?
       ↓
9. Blocking?
       ↓
10. Application changed?
```

This is real DBA troubleshooting.

---

# 6.42 Execution Plan Regression

Suppose yesterday:

```text
INDEX RANGE SCAN
```

Today:

```text
FULL TABLE SCAN
```

Possible reasons include:

* Statistics changed
* Data distribution changed
* Bind variable behavior
* Object changes
* Optimizer environment changes
* SQL/profile/baseline-related behavior
* Different database environment

The DBA must establish **why** the plan changed before choosing the remedy.

---

# 6.43 SQL Tuning Workflow

Use:

```text
SQL_ID
  ↓
SQL text
  ↓
Execution statistics
  ↓
Execution plan
  ↓
Wait events
  ↓
Object statistics
  ↓
Data distribution
  ↓
Root cause
  ↓
Controlled change
  ↓
Measure again
```

---

# 6.44 Don't Confuse Symptom and Cause

Example:

```text
Application slow
      ↓
High DB CPU
      ↓
CPU is the symptom
```

The root cause may be:

```text
One inefficient query
```

So:

```text
High CPU
```

doesn't necessarily mean:

```text
Need more CPU
```

---

# 6.45 Performance Baselines

A good DBA knows what "normal" looks like.

For example:

```text
Normal CPU       = 35–50%
Normal I/O       = X
Normal sessions  = Y
Normal latency   = Z
Normal workload  = N
```

Then compare:

```text
Today
vs
Normal
```

This makes abnormal behavior much easier to identify.

---

# 6.46 Production Performance Checklist

When receiving a performance incident:

```text
[ ] What changed?
[ ] When did it start?
[ ] Who is affected?
[ ] Which database?
[ ] Which instance?
[ ] Which service?
[ ] Current CPU?
[ ] Current memory?
[ ] Current I/O?
[ ] Active sessions?
[ ] Blocking sessions?
[ ] Top waits?
[ ] Top SQL?
[ ] SQL plan?
[ ] Recent deployment?
[ ] Statistics changes?
[ ] Storage problems?
[ ] Network problems?
```

---

# 6.47 Windows Performance Workflow

On Windows:

```text
Application
    ↓
Oracle session
    ↓
V$SESSION
    ↓
V$SQL
    ↓
Oracle wait events
    ↓
Windows Performance Monitor
    ↓
CPU / Memory / Disk / Network
```

Useful tools include:

```text
Performance Monitor
Event Viewer
Task Manager
PowerShell
```

---

# 6.48 Linux Performance Workflow

On Linux:

```text
Application
    ↓
Oracle
    ↓
V$SESSION / V$SQL
    ↓
Wait events
    ↓
Linux tools
```

Common tools:

```bash
top
vmstat
iostat
sar
free
df
```

Depending on your distribution, some tools may require additional packages.

---

# 6.49 Lesson 6 Lab

Start with:

```sql
SELECT sid,
       serial#,
       username,
       status,
       event,
       wait_class,
       sql_id
FROM v$session
WHERE username IS NOT NULL;
```

Then find SQL:

```sql
SELECT sql_id,
       executions,
       elapsed_time,
       cpu_time,
       buffer_gets,
       disk_reads
FROM v$sql
WHERE executions > 0
ORDER BY elapsed_time DESC
FETCH FIRST 20 ROWS ONLY;
```

Find invalid objects:

```sql
SELECT owner,
       object_name,
       object_type
FROM dba_objects
WHERE status = 'INVALID';
```

Find blockers:

```sql
SELECT sid,
       serial#,
       username,
       blocking_session,
       event,
       sql_id
FROM v$session
WHERE blocking_session IS NOT NULL;
```

Check memory:

```sql
SHOW PARAMETER sga;
```

```sql
SHOW PARAMETER pga;
```

Check database:

```sql
SELECT name,
       open_mode,
       database_role
FROM v$database;
```

---

# 6.50 Lesson 6 Practical Exercise

Create a test table:

```sql
CREATE TABLE performance_test AS
SELECT *
FROM all_objects;
```

Check its size:

```sql
SELECT segment_name,
       segment_type,
       bytes/1024/1024 AS size_mb
FROM user_segments
WHERE segment_name = 'PERFORMANCE_TEST';
```

Then inspect a query:

```sql
EXPLAIN PLAN FOR
SELECT *
FROM performance_test
WHERE object_name = 'EMPLOYEES';
```

Display the plan:

```sql
SELECT *
FROM TABLE(DBMS_XPLAN.DISPLAY);
```

Now ask yourself:

```text
What access path is Oracle using?
How many rows does it expect?
Is there an index?
Would an index necessarily help?
How would I verify the actual workload?
```

That reasoning is more important than memorizing an index command.

---

# 6.51 Lesson 6 Interview Questions

You should be able to explain:

1. What is Oracle performance tuning?
2. What is a wait event?
3. What is `V$SESSION`?
4. What is `V$SQL`?
5. What is a SQL_ID?
6. How do you find expensive SQL?
7. What is an execution plan?
8. What is `DBMS_XPLAN`?
9. What are optimizer statistics?
10. What is `DBMS_STATS`?
11. Why shouldn't you blindly create indexes?
12. Why doesn't low CPU necessarily mean good performance?
13. How do you identify blocking sessions?
14. What is a blocking session?
15. What is the difference between blocker and waiter?
16. What is SGA?
17. What is PGA?
18. What is AWR?
19. What is ASH?
20. What is ADDM?
21. What is ADR?
22. What is ADRCI?
23. What is an alert log?
24. How do you investigate high CPU?
25. How do you investigate high I/O?
26. How do you investigate a sudden SQL slowdown?
27. What is an execution-plan regression?
28. What is a performance baseline?
29. What are RPO and RTO? *(Review from Lesson 5.)*
30. Why must performance tuning be evidence-driven?

---

# 6.52 The Golden Performance Method

Memorize this:

```text
             PERFORMANCE PROBLEM
                      |
                      v
                 DEFINE IT
                      |
                      v
                 COLLECT DATA
                      |
          +-----------+-----------+
          |           |           |
         CPU         I/O        WAITS
          |           |           |
          +-----------+-----------+
                      |
                      v
                 FIND SQL
                      |
                      v
                CHECK PLAN
                      |
                      v
              FIND ROOT CAUSE
                      |
                      v
             MAKE ONE CHANGE
                      |
                      v
                  MEASURE
                      |
                      v
                DOCUMENT
```

**Never jump directly from:**

```text
"Database is slow"
```

to:

```text
"Change the SGA."
```

or:

```text
"Create an index."
```

A professional Oracle DBA diagnoses first.

---

## Official Oracle documentation

For the authoritative material behind this lesson, use Oracle's official documentation for your exact database release, particularly the **Database Performance Tuning Guide**, **Database Administrator's Guide**, and **Database SQL Tuning Guide**.

**Lesson 6/10 complete.**

### Course progress

```text
1. Oracle Architecture & DBA Foundations       ✓
2. Installation & Database Creation             ✓
3. Configuration & Oracle Networking            ✓
4. Core DBA Administration                     ✓
5. Backup & Recovery / RMAN                    ✓
6. Performance Monitoring & Troubleshooting   ✓
7. Security & Advanced Administration
8. Patching & Patch Management
9. Migration & Database Upgrades
10. RAC, Data Guard & End-to-End DBA Operations
```

**Next: Lesson 7/10 — Oracle Security & Advanced Administration**, covering **profiles, password policies, roles, privileges, auditing, unified auditing, encryption concepts, TDE, wallets/keystores, database links, directories, jobs, Resource Manager, multitenant administration, and DBA security practices.**
