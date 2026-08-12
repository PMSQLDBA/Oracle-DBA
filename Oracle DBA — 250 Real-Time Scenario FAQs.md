# Oracle DBA — 250 Real-Time Scenario FAQs

## Section 1 — Installation & Initial Configuration

**FAQs 1–25**

### 1. Database installation failed halfway. What will you do?

1. Check the Oracle Universal Installer logs and exact error message.
2. Verify OS prerequisites, disk space, permissions, users/groups, and required packages.
3. Check whether the Oracle Home is partially installed.
4. Correct the underlying problem before rerunning the installer.
5. Remove only incomplete components according to Oracle's documented procedure.
6. Retry installation and validate the Oracle Home afterward.

---

### 2. Oracle database software is installed but the database is not starting. What do you check?

1. Confirm `ORACLE_HOME` and `ORACLE_SID`.
2. Check whether the listener and database services/processes are running.
3. Connect using `sqlplus / as sysdba`.
4. Check the instance status with `SELECT status FROM v$instance`.
5. Review the alert log for startup errors.
6. Check parameter files, control files, redo logs, and datafile availability.

---

### 3. `ORA-01034: ORACLE not available` appears. What do you do?

1. Verify that the instance is actually running.
2. Check the correct `ORACLE_SID`.
3. Check the PMON process on Linux/Unix or Oracle service on Windows.
4. Connect as SYSDBA locally.
5. Review the alert log if startup failed.
6. Start the instance if appropriate.

---

### 4. You installed Oracle on Windows but the database service is missing. What do you check?

1. Verify that Oracle Database software installation completed successfully.
2. Check Windows Services.
3. Verify the Oracle Home and database SID.
4. Check whether the database was actually created.
5. Use Oracle's supported configuration utilities where appropriate.
6. Review installation/configuration logs for errors.

---

### 5. Linux Oracle installation says required packages are missing. What is your approach?

1. Record the exact missing packages.
2. Confirm the Linux distribution/version is supported.
3. Install the required packages using the OS package manager.
4. Re-run prerequisite validation.
5. Check kernel, limits, users, groups, and filesystem requirements.
6. Continue only after the prerequisite checks are clean.

---

### 6. Oracle database is installed but SQL*Plus cannot be found.

1. Check `ORACLE_HOME`.
2. Check whether `$ORACLE_HOME/bin/sqlplus` exists.
3. Add `$ORACLE_HOME/bin` to `PATH`.
4. Confirm `ORACLE_SID`.
5. Check that the Oracle user has access to the Oracle Home.
6. Test with `sqlplus / as sysdba`.

---

### 7. Database is running but remote users cannot connect.

1. Check whether the listener is running.
2. Run `lsnrctl status`.
3. Verify the listener port.
4. Check service registration.
5. Test name resolution and network connectivity.
6. Review `listener.ora`, `tnsnames.ora`, `sqlnet.ora`, and firewall rules.

---

### 8. Listener is running but database service is not displayed.

1. Run `lsnrctl status`.
2. Check `SERVICE_NAMES`/service configuration as appropriate.
3. Check local listener configuration.
4. Confirm the database instance is open.
5. Force registration if appropriate using `ALTER SYSTEM REGISTER`.
6. In RAC, check service configuration through Clusterware/SRVCTL rather than treating it as a standalone instance.

---

### 9. Users receive `ORA-12514`. What does it mean?

1. The listener does not know the requested service.
2. Verify the service name in the client's connect descriptor.
3. Run `lsnrctl status`.
4. Check whether the database/service is registered.
5. Check listener and database configuration.
6. Correct the service/connect-string mismatch.

---

### 10. Users receive `ORA-12541: TNS:no listener`.

1. Confirm the listener process/service is running.
2. Check the hostname and port.
3. Test network connectivity to the listener port.
4. Check firewall rules.
5. Run `lsnrctl status`.
6. Start/restart the listener only after identifying the cause.

---

### 11. Database opens but tablespaces are offline.

1. Query `DBA_TABLESPACES`.
2. Determine why the tablespace is offline.
3. Check whether its datafiles are available.
4. Review the alert log.
5. If the files were lost, determine the required recovery procedure.
6. Bring the tablespace online only after verifying consistency.

---

### 12. Database reports `ORA-01157` during startup.

1. Identify the datafile mentioned by the error.
2. Check whether the file exists at the expected path.
3. Check OS permissions.
4. Determine whether the file was accidentally moved/deleted.
5. Restore/recover it if necessary.
6. Retry startup after resolving the underlying file problem.

---

### 13. A datafile was accidentally deleted from the OS. What do you do?

1. Do **not** create an arbitrary empty file with the same name.
2. Identify the affected tablespace/datafile.
3. Determine whether the database is in ARCHIVELOG mode.
4. Check available RMAN backups.
5. Restore and recover the affected file.
6. Validate the database and affected objects afterward.

---

### 14. TEMP tablespace is full. What will you check?

1. Check TEMP usage.
2. Identify sessions consuming excessive temporary space.
3. Determine whether a query is creating a large sort/hash operation.
4. Check TEMPFILE capacity.
5. Add/resize tempfile capacity if justified.
6. Investigate the SQL causing abnormal TEMP consumption.

---

### 15. SYSAUX is rapidly growing. What is your approach?

1. Check SYSAUX space usage by component.
2. Identify the largest occupants.
3. Review Oracle-provided space-management information.
4. Determine whether historical data, statistics, AWR, audit data, or another component is responsible.
5. Apply the appropriate Oracle-supported cleanup/retention procedure.
6. Do not blindly delete rows from SYS-owned tables.

---

### 16. SYSTEM tablespace is almost full. What do you do?

1. Check datafile usage.
2. Identify which objects are consuming SYSTEM.
3. Determine whether the growth is expected.
4. Extend/add a datafile if appropriate.
5. Investigate abnormal SYSTEM growth.
6. Avoid moving/deleting internal objects without Oracle-supported procedures.

---

### 17. Database is running out of filesystem space.

1. Identify the filesystem consuming space.
2. Check datafiles, archive logs, trace files, audit files, backups, and diagnostic logs.
3. Determine whether archive logs can safely be backed up/deleted.
4. Check FRA usage if applicable.
5. Expand storage or remove eligible files according to policy.
6. Fix the root cause to prevent recurrence.

---

### 18. FRA is 100% full. What do you do first?

1. Check `V$RECOVERY_FILE_DEST`.
2. Determine which files consume the FRA.
3. Check whether archived logs have been backed up.
4. Use RMAN to manage obsolete/eligible backups and archived logs according to your retention policy.
5. Increase FRA capacity if necessary.
6. Do not manually delete files from FRA as a first response.

---

### 19. Database is in `MOUNTED` state but won't open.

1. Check the alert log.
2. Check datafile and redo-log availability.
3. Look for recovery requirements.
4. Check for control-file or datafile inconsistencies.
5. Perform only the recovery required by the actual error.
6. Open the database after recovery completes successfully.

---

### 20. Database is stuck during startup. What will you investigate?

1. Check the alert log.
2. Identify the stage at which startup is waiting.
3. Check storage responsiveness.
4. Check recovery activity.
5. Check instance/background processes.
6. Determine whether the issue is database, storage, OS, or network related.

---

### 21. Oracle database suddenly shuts down.

1. Check the alert log immediately.
2. Determine whether the shutdown was manual, automatic, or crash-related.
3. Check OS logs.
4. Check memory, CPU, storage, and filesystem conditions.
5. Determine whether Oracle encountered an internal error.
6. Restart only after understanding whether the underlying condition remains.

---

### 22. A database server rebooted unexpectedly.

1. Confirm when the reboot occurred.
2. Check OS/system logs.
3. Check database alert logs.
4. Check storage and filesystem health.
5. Verify database startup and application connectivity.
6. Perform a post-incident root-cause analysis.

---

### 23. Database installation succeeded but DBCA database creation failed.

1. Review DBCA logs.
2. Check disk space and filesystem permissions.
3. Check memory and OS prerequisites.
4. Verify database naming and storage configuration.
5. Determine whether partial database files were created.
6. Correct the problem and rerun DBCA appropriately.

---

### 24. Oracle Home has multiple databases. One database works and another doesn't.

1. Verify the correct `ORACLE_SID`.
2. Check each database's parameter/control/datafile configuration.
3. Check listener/service registration.
4. Check environment variables.
5. Review the failing database's alert log.
6. Treat each database as a separate diagnostic case.

---

### 25. How do you perform an initial production readiness check?

1. Validate database status, version, role, and open mode.
2. Check tablespaces, FRA, archive generation, and filesystem capacity.
3. Verify RMAN backups and recovery capability.
4. Check listener, services, and connectivity.
5. Check invalid objects, database jobs, and major alerts.
6. Document the baseline before handing the database to the application team.

---

# Section 2 — Users, Security & Database Administration

**FAQs 26–50**

### 26. A user cannot log in although the password is correct.

1. Check whether the account is locked.
2. Check whether the account is expired.
3. Verify the username/container.
4. Check profile password policies.
5. Check authentication method.
6. Unlock/reset only after confirming the appropriate security procedure.

---

### 27. User receives `ORA-28000: account is locked`.

1. Confirm the account status.
2. Determine why it was locked.
3. Check failed-login activity where auditing/logging permits.
4. Unlock the account if authorized.
5. Reset the password if required.
6. Investigate the application causing repeated failed logins.

---

### 28. User receives `ORA-01031: insufficient privileges`.

1. Identify the exact SQL operation.
2. Determine which object privilege is required.
3. Check existing privileges.
4. Grant the minimum required privilege.
5. Avoid granting powerful roles such as DBA unnecessarily.
6. Retest with the application user's actual account.

---

### 29. Application requests DBA privilege for its schema.

1. Do not grant DBA automatically.
2. Identify the exact operations the application performs.
3. Create/grant only required object/system privileges.
4. Test functionality.
5. Document the security exception if one is genuinely necessary.
6. Review the privileges periodically.

---

### 30. A user can access another schema's table unexpectedly.

1. Query the user's privileges.
2. Check direct grants.
3. Check role-based privileges.
4. Check synonyms and views.
5. Remove unnecessary privileges.
6. Retest access using the affected user.

---

### 31. A user was dropped accidentally.

1. Determine whether the user owned objects.
2. Check backup/export availability.
3. Identify the business impact.
4. Recreate only if appropriate.
5. Restore/import required objects and privileges.
6. Investigate how the unauthorized/destructive action occurred.

---

### 32. Password is expiring frequently.

1. Check the user's profile.
2. Review password lifetime settings.
3. Determine whether the requirement is security policy or application behavior.
4. Avoid disabling expiration globally.
5. Coordinate with security/application teams.
6. Apply the least disruptive approved solution.

---

### 33. How do you identify privileged users?

1. Review DBA and powerful system privileges.
2. Check role memberships.
3. Check direct grants.
4. Review administrative accounts.
5. Compare against approved access lists.
6. Remove excessive privileges through the change process.

---

### 34. How do you investigate suspicious database activity?

1. Identify the account, timestamp, SQL, and source.
2. Review available audit records.
3. Check application and OS logs.
4. Determine whether the activity was authorized.
5. Preserve evidence according to security procedures.
6. Escalate confirmed security incidents appropriately.

---

### 35. Application user owns objects in SYSTEM. What do you do?

1. Confirm the object's ownership.
2. Determine how it happened.
3. Avoid manually modifying SYS-owned metadata.
4. Move application objects into an appropriate application schema/tablespace using supported procedures.
5. Review deployment scripts.
6. Prevent recurrence through schema controls.

---

### 36. A database link stopped working.

1. Test the database link.
2. Check target connectivity.
3. Check credentials and authentication.
4. Check listener/service availability on the remote database.
5. Check network and naming configuration.
6. Coordinate with the remote DBA if the target is unavailable.

---

### 37. A scheduled DBMS_SCHEDULER job isn't running.

1. Check job state.
2. Check `last_start_date` and `next_run_date`.
3. Check job logs/run details.
4. Check credentials and privileges.
5. Check whether the job is disabled.
6. Investigate the job action itself if scheduling is correct.

---

### 38. A scheduled job is running continuously.

1. Identify the job session.
2. Determine what SQL/action it is executing.
3. Check waits and resource consumption.
4. Determine whether it is blocked.
5. Check historical execution duration.
6. Stop/reschedule it only after assessing business impact.

---

### 39. Database links cause slow queries.

1. Determine whether the bottleneck is local or remote.
2. Check execution plans.
3. Check network latency.
4. Check data volume transferred between databases.
5. Reduce unnecessary remote data movement.
6. Optimize SQL and network topology where possible.

---

### 40. Audit tables are growing rapidly.

1. Identify which auditing mechanism is producing the data.
2. Determine the retention requirement.
3. Check audit trail growth.
4. Implement Oracle-supported retention/purge procedures.
5. Move/partition audit data according to architecture where appropriate.
6. Monitor growth afterward.

---

### 41. A developer asks for `SELECT ANY TABLE`.

1. Determine why the privilege is required.
2. Check whether specific object grants solve the requirement.
3. Prefer least privilege.
4. Use roles where appropriate.
5. Document elevated access.
6. Review/remove temporary privileges after the task.

---

### 42. How do you handle emergency SYS access?

1. Confirm the incident and authorization.
2. Use SYSDBA only for administrative operations requiring it.
3. Record the activity.
4. Avoid application work using SYS.
5. Revoke temporary access if another privileged account was created.
6. Document the incident.

---

### 43. A user says their schema disappeared.

1. Verify the user exists.
2. Check current container.
3. Check whether objects were dropped rather than the user.
4. Review audit/history if available.
5. Check whether the user was renamed or recreated.
6. Restore/recover only after determining exactly what disappeared.

---

### 44. A table was dropped accidentally.

1. Stop unnecessary changes.
2. Determine whether Flashback Drop is available.
3. Check recycle bin where applicable.
4. If not recoverable there, use appropriate backup/recovery techniques.
5. Validate recovered data.
6. Investigate and prevent recurrence.

---

### 45. A developer wants to disable auditing for performance.

1. Measure whether auditing is actually causing the problem.
2. Identify the specific audit workload.
3. Review compliance/security requirements.
4. Tune or reduce unnecessary auditing only within policy.
5. Do not disable required auditing blindly.
6. Validate performance after any change.

---

### 46. How do you safely grant a privilege temporarily?

1. Define the exact task.
2. Grant the minimum privilege.
3. Record who authorized it.
4. Set an expiration/removal plan.
5. Test the task.
6. Revoke it immediately after completion.

---

### 47. A schema has thousands of invalid objects.

1. Determine when they became invalid.
2. Identify dependencies.
3. Check whether an upgrade/patch caused the issue.
4. Recompile using appropriate Oracle-supported mechanisms.
5. Investigate objects that remain invalid.
6. Validate application functionality afterward.

---

### 48. User can connect but cannot execute a package.

1. Check `EXECUTE` privilege.
2. Check package validity.
3. Check package dependencies.
4. Check definer/invoker rights behavior.
5. Verify the object owner and synonym.
6. Grant only the required privilege.

---

### 49. A user can select a table but cannot update it.

1. Check SELECT privilege.
2. Check UPDATE privilege.
3. Check whether UPDATE is being attempted through a view.
4. Check triggers and constraints.
5. Check application transaction behavior.
6. Grant the appropriate privilege only if authorized.

---

### 50. Security team asks for a database privilege report.

1. Extract users and account status.
2. Extract role memberships.
3. Extract system privileges.
4. Extract object privileges.
5. Identify highly privileged accounts.
6. Compare results with the approved access matrix.

---

# Section 3 — Tablespaces, Storage & ASM

**FAQs 51–75**

### 51. A tablespace is 95% full. What do you do?

1. Confirm actual used/free space.
2. Determine growth rate.
3. Identify largest segments.
4. Check whether autoextend is enabled.
5. Resize/add a datafile where appropriate.
6. Investigate the business reason for abnormal growth.

---

### 52. Autoextend is enabled but the tablespace is still full.

1. Check maximum datafile size.
2. Check filesystem/ASM free space.
3. Check whether the datafile reached `MAXSIZE`.
4. Check whether autoextend increment is appropriate.
5. Add another datafile if architecture permits.
6. Monitor storage capacity after the change.

---

### 53. A datafile cannot grow because the filesystem is full.

1. Confirm filesystem usage.
2. Identify non-database files consuming space.
3. Check whether eligible archive logs/backups can be removed safely.
4. Expand the filesystem if necessary.
5. Resize/add the datafile.
6. Implement capacity monitoring.

---

### 54. ASM disk group is 90% full.

1. Check `V$ASM_DISKGROUP`.
2. Identify which database files consume space.
3. Check whether space is needed immediately.
4. Add storage if required.
5. Allow ASM to rebalance.
6. Review capacity planning thresholds.

---

### 55. An ASM disk went offline.

1. Check `V$ASM_DISK`.
2. Identify the disk and disk group.
3. Check OS/storage visibility.
4. Determine whether the disk is physically or logically unavailable.
5. Follow the appropriate ASM/storage recovery procedure.
6. Monitor rebalancing and redundancy afterward.

---

### 56. ASM disk group is dismounted.

1. Determine which instances are affected.
2. Check the ASM alert log.
3. Check storage accessibility.
4. Check Clusterware if RAC/Oracle Restart is involved.
5. Restore storage connectivity.
6. Mount/recover the disk group only after the underlying issue is resolved.

---

### 57. ASM rebalance is taking too long.

1. Check rebalance operation status.
2. Check ASM power/rebalance settings.
3. Check storage performance.
4. Check whether the rebalance is competing with production workload.
5. Adjust only after evaluating business impact.
6. Allow sufficient time for large disk groups.

---

### 58. A new disk was added to ASM but free space isn't balanced yet.

1. Confirm the disk was added correctly.
2. Check rebalance status.
3. Understand that ASM must redistribute existing extents.
4. Monitor the rebalance.
5. Check diskgroup state and redundancy.
6. Do not manually move Oracle files between ASM disks.

---

### 59. You need to remove an ASM disk.

1. Verify redundancy and disk-group health.
2. Check whether the disk is actually intended for removal.
3. Drop the disk using ASM-supported commands.
4. Monitor the rebalance.
5. Verify the disk is no longer required.
6. Only then remove it from the underlying storage configuration.

---

### 60. ASM reports insufficient redundancy.

1. Check diskgroup redundancy type.
2. Check disk status.
3. Identify failed/missing disks.
4. Restore the failed storage where possible.
5. Add replacement capacity if necessary.
6. Verify redundancy after rebalancing.

---

### 61. Database files in ASM are difficult to locate.

1. Use Oracle ASM metadata views.
2. Use `ASMCMD`.
3. Identify the disk group.
4. Identify file type and database.
5. Avoid treating ASM paths as normal OS filesystem paths.
6. Use Oracle-supported ASM tools for file operations.

---

### 62. FRA is growing because of archived logs.

1. Check archive generation rate.
2. Check RMAN backup status.
3. Check Data Guard destinations.
4. Determine whether logs are eligible for deletion.
5. Back up/delete according to the configured recovery policy.
6. Increase FRA only if capacity requirements justify it.

---

### 63. Archive destination is full.

1. Check `V$ARCHIVE_DEST`.
2. Determine why archived logs cannot be written.
3. Check filesystem/FRA space.
4. Check standby transport status if applicable.
5. Restore archive destination capacity.
6. Ensure redo generation can continue safely.

---

### 64. A datafile was accidentally placed in the wrong filesystem.

1. Confirm the datafile path.
2. Check whether the database is online and using the file.
3. Plan the move using supported Oracle commands.
4. Ensure sufficient target storage.
5. Validate after relocation.
6. Update operational documentation.

---

### 65. A tablespace is read-only and application writes are failing.

1. Confirm tablespace status.
2. Determine why it was made read-only.
3. Check whether the application requires writes.
4. Change status only if authorized and appropriate.
5. Validate application behavior.
6. Document the change.

---

### 66. TEMPFILE is missing.

1. Check `DBA_TEMP_FILES`.
2. Confirm the missing tempfile path.
3. Determine whether TEMP can operate with remaining tempfiles.
4. Add/recreate the tempfile appropriately.
5. Validate temporary operations.
6. Monitor TEMP usage.

---

### 67. Undo tablespace is filling rapidly.

1. Check undo usage.
2. Identify long-running transactions.
3. Check transaction duration and SQL.
4. Determine whether workload increased.
5. Add/resize undo capacity if required.
6. Investigate abnormal transaction behavior.

---

### 68. Long-running query receives snapshot-too-old errors.

1. Identify the SQL and transaction pattern.
2. Check undo retention and undo usage.
3. Determine whether undo is being overwritten before the query completes.
4. Investigate long-running queries/transactions.
5. Adjust undo sizing/retention based on requirements.
6. Optimize the query where possible.

---

### 69. A large table needs more storage.

1. Check current tablespace capacity.
2. Estimate growth.
3. Check segment/storage characteristics.
4. Extend the tablespace if necessary.
5. Verify ASM/filesystem capacity.
6. Add monitoring for future growth.

---

### 70. Database storage is growing unexpectedly.

1. Compare current and historical growth.
2. Identify largest growing segments.
3. Check application changes.
4. Check audit, logging, staging, and temporary objects.
5. Determine whether growth is legitimate.
6. Implement cleanup/retention or capacity expansion as appropriate.

---

### 71. How do you identify the largest segments?

1. Query `DBA_SEGMENTS`.
2. Group by owner and segment type.
3. Sort by size.
4. Compare against historical growth.
5. Identify objects causing unexpected storage usage.
6. Coordinate with application owners before deleting data.

---

### 72. A partitioned table is consuming huge space.

1. Identify partitions with `DBA_SEGMENTS`.
2. Determine which partitions are growing.
3. Check retention policy.
4. Archive/drop old partitions only after approval.
5. Review partitioning strategy.
6. Monitor future partition growth.

---

### 73. Database storage is full during peak hours.

1. Stabilize the immediate capacity problem.
2. Determine whether database or non-database files caused it.
3. Expand capacity if required.
4. Protect database availability.
5. Investigate why capacity monitoring failed.
6. Implement proactive thresholds and alerts.

---

### 74. ASM is healthy but the application reports I/O slowness.

1. Check database wait events.
2. Check I/O statistics.
3. Check ASM diskgroup and disk performance.
4. Check OS/storage latency.
5. Compare current versus historical performance.
6. Escalate to storage infrastructure with evidence.

---

### 75. Storage team replaced a failed disk. What do you verify?

1. Confirm the replacement disk is visible to the OS/ASM.
2. Check ASM disk header/state.
3. Confirm disk group redundancy.
4. Add/replace the disk using the supported ASM procedure.
5. Monitor rebalance.
6. Verify normal redundancy after completion.

---

# Section 4 — Backup & Recovery / RMAN

**FAQs 76–100**

Oracle identifies backup/recovery as the mechanism for protecting the database against data loss and reconstructing the database after loss. ([Oracle Documentation][2])

### 76. RMAN backup completed successfully. Is the database protected?

1. Not necessarily.
2. Verify the backup status and backup pieces.
3. Verify archived redo availability.
4. Test restoration.
5. Test recovery procedures.
6. Confirm the backup meets the required RPO/RTO.

---

### 77. RMAN backup failed overnight. What do you do?

1. Identify the exact RMAN error.
2. Check storage destination.
3. Check database/archive-log availability.
4. Determine whether the failure affected the entire backup or only specific files.
5. Correct the underlying problem.
6. Rerun the required backup and verify completion.

---

### 78. RMAN reports `ORA-19504` while creating a backup piece.

1. Check the target backup path.
2. Check filesystem availability.
3. Check OS permissions.
4. Check storage capacity.
5. Check whether the destination is mounted.
6. Correct the destination issue and rerun the backup.

---

### 79. You lost one datafile. Can you restore only that file?

1. Yes, when the recovery scenario and backup allow it.
2. Identify the datafile.
3. Restore the file using RMAN.
4. Recover it using available archived redo.
5. Validate the datafile/tablespace.
6. Confirm database consistency afterward.

---

### 80. Control file was lost. What is your recovery approach?

1. Determine whether multiplexed control files remain.
2. If not, identify RMAN/control-file backups.
3. Restore an appropriate control file.
4. Mount the database.
5. Recover according to the recovery scenario.
6. Open the database using the appropriate procedure.

---

### 81. SPFILE was accidentally deleted.

1. Check whether another copy/multiplexed configuration exists.
2. Check RMAN backups.
3. Check whether an initialization parameter file can be recreated.
4. Restore/recreate the SPFILE appropriately.
5. Validate required parameters.
6. Restart and verify database behavior.

---

### 82. Archived logs were accidentally deleted.

1. Determine whether the logs are still needed for recovery.
2. Check RMAN backup status.
3. Check standby requirements.
4. Restore missing archived logs from backup if needed.
5. Validate recovery/transport continuity.
6. Review the deletion process.

---

### 83. Database must be recovered to yesterday 10 AM.

1. Identify the required recovery target.
2. Confirm backups and archived redo cover the target.
3. Choose the appropriate recovery method.
4. Restore required files.
5. Perform point-in-time recovery.
6. Validate the recovered database before handing it over.

---

### 84. Someone accidentally deleted important rows. What recovery options exist?

1. Determine when the deletion occurred.
2. Check whether Flashback Query/Table can help.
3. Check transaction/audit information.
4. If necessary, use backup/PITR techniques in an isolated environment.
5. Recover only the required data if practical.
6. Validate before applying recovered data to production.

---

### 85. Database corruption is detected.

1. Identify the corrupted file/block/object.
2. Use Oracle-supported diagnostic tools.
3. Check RMAN validation/backup status.
4. Determine whether corruption is physical or logical.
5. Restore/recover affected blocks/files when appropriate.
6. Investigate the underlying storage/hardware issue.

---

### 86. RMAN says a backup is obsolete.

1. Understand the configured retention policy.
2. Determine whether the backup is still required for recovery.
3. Check Data Guard dependencies.
4. Run RMAN's supported deletion process.
5. Do not delete backup files manually.
6. Monitor backup retention afterward.

---

### 87. RMAN backup repository is inconsistent with actual backup files.

1. Cross-check RMAN metadata.
2. Determine whether files were moved/deleted externally.
3. Use RMAN `CROSSCHECK`.
4. Identify expired versus available backups.
5. Resynchronize/catalog only when appropriate.
6. Correct operational procedures that allowed manual file changes.

---

### 88. RMAN cannot find an archived log needed for recovery.

1. Identify the missing sequence.
2. Check whether it exists on another archive destination.
3. Check RMAN backups.
4. Restore the archived log if available.
5. Check Data Guard/other copies.
6. Determine whether recovery can proceed without it.

---

### 89. Backup destination is filling rapidly.

1. Check backup frequency and size.
2. Check retention policy.
3. Check obsolete backups.
4. Check duplicate/full backup jobs.
5. Increase capacity or adjust policy based on requirements.
6. Implement capacity alerts.

---

### 90. RMAN backup is extremely slow.

1. Check I/O throughput.
2. Check CPU/compression overhead.
3. Check backup destination.
4. Check concurrent jobs.
5. Check RMAN channels/configuration.
6. Compare performance against previous backups before tuning.

---

### 91. Restore is slower than expected.

1. Check backup media throughput.
2. Check target storage performance.
3. Check RMAN channels.
4. Check CPU if decompression is involved.
5. Estimate remaining recovery time.
6. Prioritize actions according to RTO.

---

### 92. You need to prove backups are recoverable.

1. Select representative backups.
2. Restore in a test environment.
3. Recover the database.
4. Open/validate the recovered database.
5. Test application data.
6. Record restoration time and results.

---

### 93. Business asks for RPO = 5 minutes.

1. Understand the actual business requirement.
2. Design redo/backup/replication architecture accordingly.
3. Check archive generation and transport.
4. Consider Data Guard where appropriate.
5. Test actual achievable recovery point.
6. Document limitations.

---

### 94. Business asks for RTO = 15 minutes.

1. Identify the recovery scenario.
2. Measure current restoration/recovery duration.
3. Optimize backup and recovery architecture.
4. Consider standby-based recovery where appropriate.
5. Test failover/recovery timing.
6. Document actual versus required RTO.

---

### 95. RMAN backup works but restore fails.

1. Treat this as a critical backup failure.
2. Determine why restore cannot locate/read the backup.
3. Check backup-piece integrity and availability.
4. Test alternate backup copies.
5. Fix the backup process.
6. Do not consider the backup strategy healthy until restore succeeds.

---

### 96. You need a backup before a major upgrade.

1. Take a validated backup.
2. Verify backup completion.
3. Ensure required archived redo is available.
4. Test restoration if the upgrade is high-risk.
5. Preserve the backup according to rollback policy.
6. Proceed only after recovery capability is confirmed.

---

### 97. How do you verify RMAN backup status?

1. Query RMAN metadata.
2. Check recent backup jobs.
3. Review backup duration and sizes.
4. Check backup pieces.
5. Check archived-log backups.
6. Confirm there are no unexplained failed jobs.

---

### 98. Database is in NOARCHIVELOG mode and a datafile is lost.

1. Determine whether a valid offline backup exists.
2. Understand that media recovery options are limited.
3. Restore according to the available backup strategy.
4. Assess potential data loss.
5. Validate the database.
6. Recommend ARCHIVELOG mode for appropriate production environments.

---

### 99. You need to recover only one table.

1. Determine whether logical recovery is possible.
2. Consider Flashback features.
3. If backup recovery is required, recover into an auxiliary environment where appropriate.
4. Extract the required data.
5. Apply it carefully to production.
6. Validate dependencies and application consistency.

---

### 100. What is the first question during a recovery incident?

1. **What is the business impact?**
2. What data was lost or corrupted?
3. When did the incident occur?
4. What recovery point is required?
5. What backups/standbys are available?
6. What is the safest recovery path within the required RTO?

---

# Section 5 — Performance & Troubleshooting

**FAQs 101–125**

### 101. Users report the database is suddenly slow.

1. Confirm whether the issue affects all users or specific workloads.
2. Check database wait events and active sessions.
3. Check CPU, memory, and I/O.
4. Identify top SQL.
5. Check blocking/locking.
6. Compare against the normal baseline.

---

### 102. CPU is 100% on the database server.

1. Identify Oracle versus non-Oracle CPU consumers.
2. Find top database sessions/SQL.
3. Check execution plans.
4. Determine whether workload or plan changed.
5. Stabilize the highest-impact SQL.
6. Investigate OS/application causes as well.

---

### 103. One SQL suddenly became very slow.

1. Capture the SQL ID.
2. Compare current and previous execution plans.
3. Check statistics.
4. Check bind-variable behavior.
5. Check waits and I/O.
6. Apply the appropriate SQL/performance remedy after evidence is collected.

---

### 104. SQL execution plan suddenly changed.

1. Compare old and new plans.
2. Check object statistics.
3. Check optimizer parameters.
4. Check data distribution changes.
5. Check SQL plan management where applicable.
6. Validate the replacement plan before forcing anything.

---

### 105. Users report blocking.

1. Identify blocking and waiting sessions.
2. Find the blocker SQL.
3. Determine transaction duration.
4. Contact the application owner if needed.
5. Kill a session only when authorized and necessary.
6. Fix the transaction/application root cause.

---

### 106. A transaction has been open for several hours.

1. Identify the session.
2. Identify the SQL and application module.
3. Check transaction start time.
4. Determine whether it is blocking others.
5. Coordinate with the application owner.
6. Resolve/terminate only after impact assessment.

---

### 107. Database has high I/O waits.

1. Identify the dominant wait events.
2. Find SQL generating the I/O.
3. Check datafile/storage latency.
4. Check execution plans.
5. Determine whether excessive physical reads are expected.
6. Optimize SQL or storage based on evidence.

---

### 108. TEMP usage suddenly reaches 90%.

1. Identify sessions using TEMP.
2. Identify SQL causing large sorts/hashes.
3. Check execution plans.
4. Check whether workload changed.
5. Increase TEMP only if required.
6. Correct the query/workload causing abnormal consumption.

---

### 109. Sessions are waiting on locks.

1. Identify blocker/waiter relationships.
2. Find the locked object.
3. Find the blocking transaction.
4. Determine why the transaction remains open.
5. Resolve through application/business coordination.
6. Kill the blocker only as a controlled emergency measure.

---

### 110. Database connections suddenly increase from 500 to 5000.

1. Check application connection pools.
2. Check session states.
3. Check whether connections are leaked.
4. Check listener/database limits.
5. Identify the application causing the spike.
6. Stabilize connection demand before increasing limits blindly.

---

### 111. Connection pool is exhausted.

1. Determine whether sessions are active, idle, or blocked.
2. Check application pool configuration.
3. Check database response time.
4. Check long-running SQL.
5. Identify connection leaks.
6. Coordinate with application owners.

---

### 112. Database is slow after an application release.

1. Compare workload before and after deployment.
2. Identify new SQL.
3. Check SQL execution plans.
4. Check new indexes/schema changes.
5. Compare CPU/I/O/waits.
6. Roll back the application change only through the appropriate release process if it is proven causal.

---

### 113. A batch job consumes all database resources.

1. Identify the batch session and SQL.
2. Determine resource impact.
3. Check whether the workload can be rescheduled.
4. Optimize SQL/batch design.
5. Consider Resource Manager where appropriate.
6. Establish operational controls for future runs.

---

### 114. A query is blocked by another session.

1. Identify blocker and waiter.
2. Identify transaction owner.
3. Determine whether the blocker is legitimate.
4. Contact the owning application/team.
5. Resolve safely.
6. Investigate why blocking occurred.

---

### 115. Deadlocks occur repeatedly.

1. Capture deadlock information.
2. Identify involved sessions and SQL.
3. Analyze lock acquisition order.
4. Coordinate with application developers.
5. Correct transaction ordering/design.
6. Monitor after the fix.

---

### 116. Database memory usage is high.

1. Determine which Oracle memory areas consume it.
2. Check SGA/PGA configuration.
3. Check OS memory pressure.
4. Check large SQL/PGA consumers.
5. Compare against workload requirements.
6. Tune based on evidence rather than simply increasing memory.

---

### 117. PGA usage is unexpectedly high.

1. Identify sessions consuming PGA.
2. Check sort/hash operations.
3. Identify large SQL.
4. Check workarea behavior.
5. Determine whether workload increased.
6. Tune SQL/workload and memory configuration appropriately.

---

### 118. Shared pool is frequently exhausted.

1. Check library cache/shared pool symptoms.
2. Look for excessive hard parsing.
3. Check application SQL reuse.
4. Check shared pool sizing.
5. Identify problematic SQL patterns.
6. Fix application/database causes before simply increasing memory.

---

### 119. Database experiences frequent log switches.

1. Check redo generation rate.
2. Check redo log size.
3. Determine whether frequent switches are expected.
4. Check checkpoint behavior.
5. Size redo logs appropriately for workload/recovery requirements.
6. Monitor after changes.

---

### 120. Checkpoint-related waits increase.

1. Check redo generation.
2. Check checkpoint frequency.
3. Check redo log configuration.
4. Check I/O throughput.
5. Determine whether database configuration is causing excessive checkpoint activity.
6. Tune based on workload and Oracle recommendations.

---

### 121. AWR shows high DB time.

1. Identify top wait classes.
2. Identify top SQL.
3. Identify top services/users.
4. Compare with previous periods.
5. Determine the dominant bottleneck.
6. Fix the root cause rather than treating DB Time itself.

---

### 122. Performance degraded only during month-end.

1. Compare workload to normal days.
2. Identify month-end batch SQL.
3. Check CPU/I/O/locks/TEMP.
4. Check data volume changes.
5. Optimize or schedule heavy workloads.
6. Capacity-plan specifically for peak periods.

---

### 123. Database performance is fine but application is slow.

1. Verify database response times.
2. Check network latency.
3. Check application server CPU/memory.
4. Check connection pool behavior.
5. Trace representative requests.
6. Avoid blaming the database without evidence.

---

### 124. Network latency increased between application and database.

1. Test network connectivity/latency.
2. Compare with historical values.
3. Check packet loss.
4. Check listener/database response separately.
5. Coordinate with network infrastructure.
6. Validate application performance after the network issue is fixed.

---

### 125. How do you troubleshoot any Oracle performance issue?

1. Define the exact symptom.
2. Establish the time window.
3. Identify the dominant resource/wait.
4. Find the responsible SQL/session/application.
5. Apply the smallest evidence-based change.
6. Measure the result.

---

# Section 6 — Patching & Maintenance

**FAQs 126–150**

### 126. Production needs an Oracle patch. What is your process?

1. Confirm the required patch and affected Oracle Homes.
2. Read the patch README and prerequisites.
3. Check current inventory/version.
4. Test in lower environments.
5. Back up before the maintenance.
6. Apply, validate, and document the patch.

---

### 127. Patch was applied to the Oracle Home but SQL patch status is missing.

1. Understand that binary patching and database SQL patching are distinct activities.
2. Check `DBA_REGISTRY_SQLPATCH`.
3. Review OPatch/patch logs.
4. Run the appropriate Oracle-supported SQL patching step when required.
5. Validate status.
6. Record the patch inventory.

---

### 128. OPatch says a prerequisite patch is missing.

1. Read the patch README.
2. Check current OPatch inventory.
3. Identify the prerequisite patch.
4. Confirm patch compatibility.
5. Apply prerequisites in the required order.
6. Re-run prerequisite validation.

---

### 129. OPatch reports a conflict.

1. Identify the conflicting patch.
2. Check `opatch lsinventory`.
3. Read the patch conflict instructions.
4. Determine whether a merge/superseding patch is required.
5. Do not remove patches blindly.
6. Follow Oracle's documented patching path.

---

### 130. Patch installation fails halfway.

1. Preserve the logs.
2. Identify the exact failure stage.
3. Check filesystem, permissions, inventory, and conflicts.
4. Do not randomly rerun or remove files.
5. Follow the patch README/recovery procedure.
6. Validate Oracle Home integrity before proceeding.

---

### 131. How do you patch RAC?

1. Review RAC patching prerequisites.
2. Back up the database and Oracle installation as appropriate.
3. Validate cluster readiness.
4. Plan node-by-node or rolling/nonrolling maintenance according to patch support.
5. Apply and validate each node.
6. Confirm cluster/database/service health afterward.

Oracle explicitly recommends backing up the Oracle RAC installation before software changes and provides CVU-based readiness checks. ([Oracle Documentation][3])

---

### 132. One RAC node was patched but another wasn't.

1. Check cluster/Oracle Home versions.
2. Determine whether the patch is rolling or nonrolling.
3. Check patch inventory on every node.
4. Complete the approved patching sequence.
5. Validate Clusterware, ASM, database, and services.
6. Document node consistency.

---

### 133. Database is down after patching.

1. Check alert log.
2. Check Oracle Home and environment.
3. Check listener/services.
4. Check SQL patch status if applicable.
5. Check database startup errors.
6. Follow the documented rollback/recovery procedure if the patch caused the failure.

---

### 134. Patch works in test but fails in production.

1. Compare environments.
2. Check Oracle Home inventory.
3. Compare OS/packages/configuration.
4. Check custom configuration.
5. Review patch logs.
6. Do not assume production failure is caused by the same factor as test.

---

### 135. How do you validate a patch after installation?

1. Check OPatch inventory.
2. Check database SQL patch registry.
3. Start/validate database.
4. Validate listener/services.
5. Run application smoke tests.
6. Monitor alert and diagnostic logs.

---

### 136. Patch caused invalid objects.

1. Identify invalid objects.
2. Determine whether they are expected post-patch.
3. Recompile using supported methods.
4. Investigate objects that remain invalid.
5. Test application functionality.
6. Document exceptions.

---

### 137. You need to patch a Data Guard environment.

1. Determine the supported Data Guard patching strategy.
2. Check primary/standby synchronization.
3. Validate backups.
4. Plan role transitions where appropriate.
5. Patch and validate each environment carefully.
6. Verify redo transport/apply afterward.

---

### 138. Can you patch and upgrade in the same operation?

1. Distinguish **patching** from **upgrading**.
2. Verify the supported tool/workflow.
3. For AutoUpgrade, Oracle documents patching and upgrade operations as distinct configuration operation types. ([Oracle Documentation][4])
4. Plan the required sequence.
5. Test the exact combination.
6. Do not assume every patch/upgrade combination is supported.

---

### 139. Patch inventory differs across servers.

1. Run inventory on each Oracle Home.
2. Compare versions and patch IDs.
3. Determine whether differences are intentional.
4. Correct inconsistent nodes where required.
5. Validate applications.
6. Maintain an Oracle Home inventory document.

---

### 140. OPatch version is too old.

1. Check the patch README requirement.
2. Verify current OPatch version.
3. Obtain the supported OPatch version from Oracle resources.
4. Back up the Oracle Home as appropriate.
5. Update OPatch.
6. Re-run patch prerequisite checks.

---

### 141. Patch requires downtime. How do you plan it?

1. Determine exact outage requirements.
2. Validate backups.
3. Prepare rollback/recovery steps.
4. Test the procedure.
5. Communicate downtime and validation criteria.
6. Execute through a controlled change window.

---

### 142. Business says downtime cannot exceed 10 minutes.

1. Determine whether rolling maintenance is supported.
2. Consider RAC/Data Guard architecture.
3. Measure actual maintenance time.
4. Prepare role/service transition procedures if applicable.
5. Test the complete runbook.
6. Do not promise the 10-minute target without testing it.

---

### 143. Patch was installed but database version output hasn't changed as expected.

1. Determine whether you applied a Release Update or another patch type.
2. Check Oracle Home inventory.
3. Check database banner/version information.
4. Check SQL patch registry.
5. Verify the database is using the intended Oracle Home.
6. Restart/use the correct home if required.

---

### 144. Oracle Home is corrupted after maintenance.

1. Stop affected Oracle services/processes.
2. Preserve logs.
3. Determine whether another validated Oracle Home exists.
4. Restore/reinstall according to supported procedures.
5. Point the database to the validated home.
6. Test all services and applications.

---

### 145. Patch caused a regression.

1. Capture evidence.
2. Compare SQL/performance before and after.
3. Determine whether the issue is patch-related.
4. Search Oracle support documentation/known issues.
5. Apply a supported fix or rollback strategy.
6. Validate before returning to normal operations.

---

### 146. A patch must be urgently applied because of a security vulnerability.

1. Assess exposure and business risk.
2. Identify the exact Oracle security patch/release update.
3. Test urgently in a representative environment.
4. Prepare backup and rollback/recovery procedures.
5. Patch according to Oracle instructions.
6. Validate security and application functionality.

---

### 147. How do you maintain patch compliance?

1. Maintain Oracle Home inventory.
2. Track required security/RU levels.
3. Compare production against standards.
4. Monitor missing patches.
5. Schedule remediation.
6. Report compliance regularly.

---

### 148. Patch validation shows a listener problem.

1. Check listener status.
2. Check listener configuration.
3. Verify Oracle Home used by the listener.
4. Check service registration.
5. Review listener logs.
6. Correct configuration and retest connectivity.

---

### 149. After patching, application cannot connect.

1. Test local database connectivity.
2. Test listener connectivity.
3. Verify service name.
4. Check client/server compatibility and configuration.
5. Check application connection strings.
6. Determine whether the problem is database, listener, network, or client-side.

---

### 150. What is the biggest patching mistake?

1. Patching without reading the README.
2. Not backing up.
3. Not testing.
4. Not checking prerequisites.
5. Not validating after patching.
6. Not having a recovery/rollback plan.

---

# Section 7 — Migration & Upgrade

**FAQs 151–175**

Oracle documents multiple upgrade/migration approaches, including AutoUpgrade, DBUA, manual upgrade, and Data Pump-based migration. Oracle recommends AutoUpgrade for many upgrade scenarios. ([Oracle Documentation][5])

### 151. You need to upgrade 19c to a newer supported release. What is your first step?

1. Confirm the supported upgrade path.
2. Inventory the source database.
3. Install/prepare the target Oracle Home.
4. Run AutoUpgrade Analyze.
5. Fix reported issues.
6. Build and test the complete upgrade plan.

---

### 152. What does AutoUpgrade Analyze do?

1. It checks upgrade readiness.
2. It identifies potential problems before deployment.
3. Analyze mode is read-only against the database.
4. It generates reports/logs.
5. It can be run during normal operations.
6. Fix reported issues before production deployment.

Oracle explicitly documents Analyze as a read-only readiness assessment. ([Oracle Documentation][6])

---

### 153. AutoUpgrade Analyze reports errors. Can you proceed?

1. Read every reported error/warning.
2. Separate automatic fixups from manual actions.
3. Resolve required manual issues.
4. Rerun Analyze.
5. Continue until the upgrade is ready.
6. Never ignore critical upgrade checks.

---

### 154. Upgrade failed halfway.

1. Stop and preserve AutoUpgrade logs.
2. Identify the failed stage.
3. Determine whether the failure is recoverable.
4. Use AutoUpgrade's supported recovery/restore options where applicable.
5. Do not manually modify upgrade metadata.
6. Escalate with logs if necessary.

---

### 155. Can AutoUpgrade automatically restore a failed upgrade?

1. AutoUpgrade provides `restore_on_fail`.
2. It can restore failed jobs in supported scenarios.
3. Understand exactly what the option covers.
4. Ensure the required recovery mechanism is available.
5. Do not treat it as a replacement for backups.
6. Test recovery behavior before production.

Oracle documents `restore_on_fail` and the `-restore` functionality. ([Oracle Documentation][7])

---

### 156. You need to migrate a database to a new Linux server with the same release.

1. Define whether this is migration or upgrade.
2. Inventory source configuration.
3. Select the migration method.
4. Validate storage/platform compatibility.
5. Perform the migration and final synchronization.
6. Test application connectivity and functionality.

Oracle provides specific documented migration scenarios for new hardware with the same OS/release. ([Oracle Documentation][1])

---

### 157. You need to migrate from Windows to Linux.

1. Confirm endian/platform compatibility and supported migration path.
2. Choose an appropriate method such as Data Pump or another supported transport/migration mechanism.
3. Validate database/application dependencies.
4. Build the target environment.
5. Test migration thoroughly.
6. Plan final cutover and rollback.

---

### 158. Can Data Pump be used for migration?

1. Yes, Data Pump can export/import database objects and data.
2. It is especially useful for logical migration.
3. Validate object compatibility.
4. Consider data volume and migration duration.
5. Test application behavior after import.
6. Plan final synchronization/cutover.

Oracle explicitly documents Export/Import using Data Pump as an upgrade/migration method. ([Oracle Documentation][5])

---

### 159. Data Pump import is slower than expected.

1. Check dump-file storage.
2. Check target database I/O.
3. Check parallelism.
4. Check object types being imported.
5. Check network/storage if files are remote.
6. Compare actual workload against expected migration duration.

---

### 160. Data Pump import reports object creation errors.

1. Identify the exact object and error.
2. Check dependency/order issues.
3. Check target privileges/tablespaces.
4. Review import logs.
5. Correct the underlying issue.
6. Re-import/reprocess only what is required.

---

### 161. Upgrade is required but application downtime must be minimal.

1. Determine whether online/minimal-downtime migration is supported.
2. Evaluate Data Guard/standby-based approaches.
3. Evaluate logical migration technologies where appropriate.
4. Test the entire cutover.
5. Define rollback criteria.
6. Measure actual downtime before committing to the business target.

---

### 162. You need to migrate a very large database.

1. Estimate data volume and transfer time.
2. Evaluate physical versus logical migration.
3. Consider transport/standby-based approaches.
4. Plan network/storage throughput.
5. Perform multiple rehearsals.
6. Optimize the final synchronization window.

---

### 163. Upgrade works in development but fails in production.

1. Compare versions and patch levels.
2. Compare schemas and application versions.
3. Compare configuration.
4. Compare data volume and object counts.
5. Reproduce the issue in a production-like environment.
6. Fix and rerun before production.

---

### 164. Invalid objects appear after upgrade.

1. List invalid objects.
2. Determine whether they are Oracle-owned or application-owned.
3. Recompile using supported mechanisms.
4. Check dependency failures.
5. Investigate objects that remain invalid.
6. Run application validation.

---

### 165. Time zone file needs attention after upgrade.

1. Check the source/target time zone file versions.
2. Review Oracle's post-upgrade requirements.
3. Determine whether the application uses timestamp/time-zone data.
4. Plan the time-zone upgrade separately where required.
5. Test application date/time behavior.
6. Validate after upgrade.

Oracle includes time-zone file upgrade among post-upgrade tasks. ([Oracle Documentation][8])

---

### 166. How do you upgrade a non-CDB to a PDB?

1. Prepare the target CDB.
2. Run AutoUpgrade Analyze.
3. Configure the non-CDB-to-PDB conversion.
4. Back up the database.
5. Execute the supported conversion/upgrade workflow.
6. Validate the resulting PDB and application.

Oracle documents AutoUpgrade workflows for non-CDB to PDB conversion. ([Oracle Documentation][9])

---

### 167. You want to unplug a PDB and plug it into a newer CDB.

1. Confirm compatibility requirements.
2. Prepare the target CDB.
3. Back up the source.
4. Use the supported AutoUpgrade unplug/plug workflow.
5. Run required upgrade/fixup steps.
6. Validate PDB services, listeners, and application connectivity.

Oracle documents unplug/plug PDB upgrades through AutoUpgrade. ([Oracle Documentation][9])

---

### 168. Can you rely on Flashback as the only fallback for unplug/plug PDB upgrade?

1. No.
2. Oracle specifically warns that Flashback Database does not work across the PDB conversion in this scenario.
3. A reliable backup is therefore essential.
4. Understand the exact rollback limitations.
5. Test the recovery strategy.
6. Do not start the conversion without a viable fallback.

Oracle explicitly states that backups are the fallback for the documented unplug/plug conversion scenario. ([Oracle Documentation][9])

---

### 169. Target Oracle Home is not yet installed. Can you run AutoUpgrade Analyze?

1. Yes, Oracle documents Analyze scenarios where the target home is not yet available.
2. Configure the source/target parameters appropriately.
3. Run Analyze.
4. Review the generated findings.
5. Install the target home afterward.
6. Revalidate before deployment.

([Oracle Documentation][6])

---

### 170. Upgrade requires a new Oracle Home. Should you overwrite the old home?

1. Generally use a separate target Oracle Home for upgrade planning.
2. Keep the old home available for controlled fallback where supported.
3. Install and patch the new home.
4. Validate the new home.
5. Point the upgraded database to it.
6. Retire the old home only after successful validation and change approval.

---

### 171. How do you upgrade multiple databases?

1. Inventory all source databases.
2. Group compatible databases.
3. Prepare target Oracle Homes.
4. Build AutoUpgrade configuration for multiple databases.
5. Run Analyze before deployment.
6. Deploy in controlled batches.

Oracle documents AutoUpgrade's ability to handle multiple database deployments using a configuration file. ([Oracle Documentation][6])

---

### 172. Upgrade is complete but application performance changed.

1. Compare SQL performance before/after.
2. Check optimizer/statistics changes.
3. Check execution plans.
4. Check database parameters.
5. Compare waits and resource consumption.
6. Tune based on evidence.

---

### 173. After upgrade, listener uses the old Oracle Home.

1. Check listener configuration.
2. Check which Oracle Home owns the listener.
3. Update service/startup configuration as appropriate.
4. Validate listener status.
5. Validate database service registration.
6. Test client connectivity.

---

### 174. Upgrade is complete but Data Guard is not synchronized.

1. Check Broker configuration.
2. Check redo transport.
3. Check standby apply.
4. Check for archive gaps.
5. Follow Oracle's supported post-upgrade Data Guard procedure.
6. Validate synchronization before declaring DR healthy.

---

### 175. What is the most important upgrade rule?

1. Inventory first.
2. Back up first.
3. Analyze first.
4. Test first.
5. Upgrade using a documented runbook.
6. Validate both database and application afterward.

Oracle's upgrade documentation emphasizes preparation, analysis, testing, backup, and post-upgrade tasks. ([Oracle Documentation][10])

---

# Section 8 — RAC & ASM

**FAQs 176–200**

### 176. One RAC node is down. What do you check?

1. Check Clusterware status.
2. Check the failed node's OS.
3. Check ASM/storage.
4. Check surviving database instance.
5. Check application services.
6. Determine whether automatic failover/restart occurred.

---

### 177. RAC database instance crashed on one node.

1. Check the instance alert log.
2. Check Clusterware resource status.
3. Check whether the instance restarted automatically.
4. Check surviving instances/services.
5. Investigate the crash cause.
6. Validate application connections.

---

### 178. RAC service is running only on one node.

1. Check `srvctl status service`.
2. Check service configuration.
3. Check preferred/available instances.
4. Check Clusterware resource status.
5. Determine whether the behavior is intentional.
6. Correct service configuration if required.

---

### 179. Application connects to a specific RAC instance.

1. Identify the connection string.
2. Determine whether it bypasses services.
3. Configure/use the appropriate RAC service.
4. Validate service placement.
5. Test failover/load balancing.
6. Update application configuration.

---

### 180. RAC service fails over unexpectedly.

1. Check Clusterware events.
2. Check database instance status.
3. Check node/network health.
4. Check service configuration.
5. Review application connection behavior.
6. Identify the trigger before changing service policy.

---

### 181. `srvctl` reports a resource is offline.

1. Identify the resource.
2. Check dependent resources.
3. Run `crsctl status resource -t`.
4. Check Clusterware logs.
5. Determine why the resource stopped.
6. Start it only after understanding the dependency failure.

---

### 182. ASM instance is not running on one RAC node.

1. Check `srvctl status asm`.
2. Check Clusterware.
3. Check ASM alert log.
4. Check storage visibility.
5. Check voting/OCR/Clusterware dependencies.
6. Restart/recover according to the cluster architecture.

---

### 183. RAC private interconnect is unstable.

1. Check cluster interconnect configuration.
2. Check network interfaces.
3. Check packet loss/latency.
4. Check OS/network errors.
5. Determine whether the issue affects cache coherency/cluster health.
6. Escalate urgently to network infrastructure.

---

### 184. RAC nodes cannot communicate.

1. Check public and private networks separately.
2. Verify name resolution.
3. Check network interfaces.
4. Check firewall rules.
5. Check Clusterware status.
6. Resolve connectivity before restarting cluster resources repeatedly.

---

### 185. SCAN connection fails.

1. Check SCAN listeners.
2. Check DNS/name resolution.
3. Check SCAN VIP resources.
4. Check service registration.
5. Test connectivity from the application server.
6. Correct DNS/network/listener configuration.

---

### 186. VIP fails over unexpectedly.

1. Check node network health.
2. Check Clusterware logs.
3. Check VIP resource status.
4. Check public interface availability.
5. Determine whether the node was considered unavailable.
6. Fix the network/node issue rather than repeatedly moving the VIP.

---

### 187. RAC database is healthy but application cannot connect.

1. Check service status.
2. Check SCAN/listener.
3. Check client connect descriptor.
4. Check DNS/network.
5. Test direct service connectivity.
6. Determine whether the failure is application-side or RAC-side.

---

### 188. ASM disk group won't mount after reboot.

1. Check storage visibility.
2. Check ASM disk discovery.
3. Check Clusterware resource dependencies.
4. Check ASM alert log.
5. Check disk headers/status.
6. Correct storage/discovery configuration before mounting.

---

### 189. ASM disk discovery fails after server reboot.

1. Check device visibility.
2. Check permissions/ownership.
3. Check ASM discovery string.
4. Check multipath configuration if applicable.
5. Verify persistent device configuration.
6. Restart/reload ASM only after storage is correctly visible.

---

### 190. RAC database reports high global cache waits.

1. Identify affected SQL.
2. Check interconnect health.
3. Check hot blocks/data access patterns.
4. Check SQL concurrency.
5. Check workload distribution across instances.
6. Tune application/data access patterns where necessary.

---

### 191. One RAC instance is much busier than another.

1. Check service placement.
2. Check connection distribution.
3. Check application routing.
4. Check workload characteristics.
5. Adjust service configuration/load balancing if appropriate.
6. Monitor after the change.

---

### 192. Need to add an ASM disk during production.

1. Verify storage has been provisioned.
2. Verify disk visibility and identity.
3. Confirm disk group capacity need.
4. Add the disk using supported ASM commands.
5. Monitor rebalance.
6. Validate diskgroup health.

---

### 193. ASM diskgroup is unbalanced after disk addition.

1. Check rebalance status.
2. Confirm the new disk is online.
3. Allow rebalance to complete.
4. Check storage performance.
5. Verify free-space distribution afterward.
6. Do not manually redistribute ASM extents.

---

### 194. RAC service is not starting automatically.

1. Check service configuration.
2. Check Clusterware resource status.
3. Check database availability.
4. Check service dependencies.
5. Correct the configuration.
6. Test restart/failover behavior.

---

### 195. A RAC node needs planned maintenance.

1. Check service placement.
2. Relocate/drain services where appropriate.
3. Confirm another node can handle workload.
4. Stop the node using supported cluster procedures.
5. Perform OS/maintenance work.
6. Validate node, database, ASM, and services afterward.

---

### 196. One RAC node has a different Oracle Home patch level.

1. Confirm exact patch inventory.
2. Determine supported cluster patching state.
3. Patch/align the node as required.
4. Validate Clusterware and database compatibility.
5. Check services.
6. Document the final cluster state.

---

### 197. RAC database won't start after storage maintenance.

1. Check ASM.
2. Check disk groups.
3. Check Clusterware dependencies.
4. Check database alert log.
5. Check datafile/control-file accessibility.
6. Restore storage functionality before repeatedly restarting the database.

---

### 198. ASM reports a disk in `FORMER` state.

1. Understand the disk's previous role.
2. Verify whether it belongs to another disk group.
3. Confirm storage identity.
4. Do not add it blindly.
5. Clear/reuse it only through an approved procedure.
6. Validate the disk group afterward.

---

### 199. RAC maintenance caused unexpected application disconnects.

1. Identify which service moved.
2. Check connection pooling.
3. Check application failover configuration.
4. Check service relocation timing.
5. Determine whether connections were expected to drop.
6. Improve service/application failover behavior if required.

---

### 200. What is the most important RAC troubleshooting rule?

1. Check Clusterware first.
2. Check ASM/storage.
3. Check database instances.
4. Check services.
5. Check network/interconnect.
6. Diagnose dependencies rather than restarting everything.

---

# Section 9 — Data Guard / DR

**FAQs 201–225**

### 201. Data Guard standby is lagging.

1. Check transport lag.
2. Check apply lag.
3. Check redo generation rate.
4. Check network transport.
5. Check standby I/O/apply performance.
6. Determine whether the bottleneck is primary, network, or standby.

---

### 202. Data Guard redo transport stopped.

1. Check the archive destination status.
2. Check network connectivity.
3. Check standby availability.
4. Check Broker status.
5. Check destination errors.
6. Correct the underlying issue and verify transport resumes.

---

### 203. Standby is receiving redo but not applying it.

1. Check managed recovery/apply status.
2. Check standby database state.
3. Check for errors in the alert log.
4. Check Broker status.
5. Check standby resources.
6. Resume apply only after understanding why it stopped.

---

### 204. Data Guard reports an archive gap.

1. Identify missing redo sequences.
2. Check whether the primary still has them.
3. Check archived-log destinations/backups.
4. Transfer/register the missing logs using supported procedures.
5. Resume apply.
6. Verify the gap is resolved.

---

### 205. Broker reports `WARNING`.

1. Run `SHOW CONFIGURATION`.
2. Identify the affected database/property.
3. Run validation.
4. Check transport/apply status.
5. Correct the underlying issue.
6. Recheck Broker until healthy.

---

### 206. Broker reports `ERROR`.

1. Do not ignore it.
2. Run `SHOW CONFIGURATION`.
3. Identify the exact database/resource.
4. Run `VALIDATE DATABASE`.
5. Check alert/Broker logs.
6. Resolve and revalidate.

---

### 207. Primary database fails. Should you immediately fail over?

1. First assess whether the primary can recover within the required RTO.
2. Determine standby synchronization status.
3. Determine business impact.
4. Confirm the appropriate failover target.
5. Perform failover according to the DR runbook if required.
6. Validate the new primary afterward.

---

### 208. Switchover is planned for maintenance. What should you do?

1. Verify Data Guard health.
2. Validate both primary and standby.
3. Check transport/apply lag.
4. Validate Broker configuration.
5. Perform switchover using the supported process.
6. Validate services and applications after role reversal.

---

### 209. Switchover failed.

1. Capture Broker output.
2. Identify which stage failed.
3. Check database roles.
4. Check redo transport/apply.
5. Resolve the reported condition.
6. Revalidate before retrying.

---

### 210. Failover completed but applications cannot connect.

1. Confirm new primary role.
2. Check services.
3. Check listener/SCAN/DNS.
4. Check application connect strings.
5. Start/relocate required services.
6. Perform application smoke testing.

---

### 211. After failover, old primary must become standby.

1. Determine whether reinstatement is supported.
2. Check Flashback/Data Guard configuration.
3. Follow Broker-supported reinstatement procedures.
4. Reestablish redo transport.
5. Validate apply.
6. Confirm DR protection is restored.

---

### 212. Standby disk space is filling.

1. Check datafiles.
2. Check archived logs.
3. Check FRA.
4. Check apply status.
5. Check whether logs are accumulating because apply is stopped.
6. Resolve the apply/storage issue before deleting anything.

---

### 213. Standby apply is slower than redo generation.

1. Check standby I/O.
2. Check CPU.
3. Check redo apply processes.
4. Check storage latency.
5. Check workload characteristics.
6. Scale/tune standby resources as appropriate.

---

### 214. Network between primary and standby is unstable.

1. Check packet loss and latency.
2. Check redo transport errors.
3. Check network interfaces.
4. Check firewall/security devices.
5. Determine impact on RPO.
6. Restore stable transport and validate synchronization.

---

### 215. Data Guard protection mode changed unexpectedly.

1. Check Broker configuration.
2. Check database properties.
3. Review recent changes.
4. Verify primary/standby availability.
5. Restore the intended configuration if authorized.
6. Document the change.

---

### 216. Standby database is accidentally opened incorrectly.

1. Determine its current role/state.
2. Stop inappropriate application access.
3. Check Data Guard apply status.
4. Follow the supported procedure for returning it to the required state.
5. Revalidate synchronization.
6. Investigate why the incorrect state occurred.

---

### 217. Data Guard observer is down.

1. Check observer host/process.
2. Check network connectivity.
3. Determine whether FSFO depends on that observer.
4. Restore observer availability.
5. Verify Broker/FSFO status.
6. Confirm automated protection is operating as designed.

---

### 218. Fast-Start Failover is enabled. How do you test safely?

1. Understand the configured thresholds and conditions.
2. Validate the configuration.
3. Use Oracle-supported testing/observe-only capabilities where appropriate.
4. Do not simulate failures blindly in production.
5. Monitor Broker and observer behavior.
6. Document test results.

---

### 219. Standby is significantly behind after maintenance.

1. Check how much redo accumulated.
2. Check transport.
3. Check apply.
4. Check standby storage/performance.
5. Estimate catch-up time.
6. Monitor until synchronization is restored.

---

### 220. Primary redo generation suddenly doubles.

1. Check application workload.
2. Identify top redo-generating SQL/transactions.
3. Check batch jobs.
4. Check bulk operations.
5. Determine impact on Data Guard transport/apply.
6. Capacity-plan if the increase is permanent.

---

### 221. Data Guard was healthy yesterday but unhealthy today.

1. Compare Broker configuration.
2. Check recent maintenance.
3. Check network/storage changes.
4. Check archive/redo status.
5. Review alert/Broker logs.
6. Identify the change that preceded the failure.

---

### 222. Business asks whether Data Guard guarantees zero data loss.

1. It depends on the protection mode and configuration.
2. Maximum Protection is designed for zero data loss under its requirements.
3. Maximum Availability can provide strong protection with different behavior.
4. Maximum Performance uses asynchronous transport.
5. Validate actual configuration rather than relying on terminology.
6. Test DR behavior.

---

### 223. Data Guard standby is unavailable during a primary outage.

1. Determine whether another standby exists.
2. Assess primary recovery options.
3. Determine business RTO/RPO.
4. Use the available DR/recovery mechanism.
5. Communicate the increased risk.
6. Restore standby protection afterward.

---

### 224. How often should Data Guard failover be tested?

1. According to business DR policy.
2. Test regularly enough to maintain operational readiness.
3. Include planned and realistic failure scenarios.
4. Measure RTO/RPO.
5. Document issues discovered.
6. Update the runbook after each exercise.

---

### 225. What is the most important Data Guard rule?

1. Monitor transport.
2. Monitor apply.
3. Monitor Broker.
4. Test switchover.
5. Test failover.
6. Never assume DR works merely because the standby is running.

---

# Section 10 — Production Incidents / Senior DBA Scenarios

**FAQs 226–250**

### 226. Production database is completely unavailable. What do you do first?

1. Confirm the outage from multiple sources.
2. Determine whether the database/server/network is responsible.
3. Check database/instance status.
4. Check listener and application connectivity.
5. Stabilize service using the safest available recovery path.
6. Communicate status while troubleshooting.

---

### 227. Production database is down and management asks, "When will it be back?"

1. Determine the actual failure first.
2. Identify the recovery path.
3. Estimate using known recovery/test timings.
4. Do not invent an ETA.
5. Communicate assumptions and risks.
6. Update the ETA as evidence changes.

---

### 228. Production storage suddenly disappears.

1. Stop destructive actions.
2. Determine affected disk/filesystems/ASM diskgroups.
3. Check storage infrastructure.
4. Determine database impact.
5. Use redundancy/recovery mechanisms where available.
6. Coordinate with storage and infrastructure teams.

---

### 229. Database is corrupted after a storage failure.

1. Identify corruption scope.
2. Check RMAN validation/backups.
3. Determine whether ASM/storage redundancy can recover it.
4. Restore/recover affected files/blocks as appropriate.
5. Validate database consistency.
6. Investigate storage root cause.

---

### 230. Application team says "Oracle is slow," but Oracle metrics look normal.

1. Establish the exact request/transaction that is slow.
2. Measure database response time.
3. Check network latency.
4. Check application server performance.
5. Trace the transaction across tiers.
6. Provide evidence identifying the actual bottleneck.

---

### 231. CPU, memory, and I/O are all high.

1. Identify which resource is the primary bottleneck.
2. Identify top Oracle sessions/SQL.
3. Check OS processes.
4. Check application workload.
5. Stabilize the highest-impact workload.
6. Perform root-cause analysis after stabilization.

---

### 232. Database is slow after a server reboot.

1. Check database initialization/configuration.
2. Check filesystem and storage mounts.
3. Check Oracle services.
4. Check statistics/cache warm-up.
5. Compare performance with pre-reboot baseline.
6. Investigate configuration drift introduced during reboot.

---

### 233. Database startup after reboot takes 30 minutes.

1. Check alert log startup stages.
2. Determine whether recovery is occurring.
3. Check storage performance.
4. Check redo volume.
5. Determine whether this is expected recovery behavior.
6. Improve recovery/RTO architecture if the startup time is unacceptable.

---

### 234. A critical production schema was accidentally dropped.

1. Stop further changes.
2. Determine exact time of deletion.
3. Check Flashback capabilities.
4. If necessary, recover into an auxiliary environment.
5. Restore only the required schema/data where practical.
6. Validate application consistency before reopening access.

---

### 235. Someone accidentally ran DELETE without WHERE.

1. Stop/limit further changes.
2. Determine whether the transaction is still uncommitted.
3. If uncommitted, coordinate immediate rollback.
4. If committed, determine Flashback/recovery options.
5. Recover affected data carefully.
6. Implement safeguards against recurrence.

---

### 236. A deployment changed thousands of rows incorrectly.

1. Stop the deployment/application process.
2. Determine transaction boundaries and commit status.
3. Identify affected data.
4. Use Flashback/recovery techniques where appropriate.
5. Validate the corrected dataset.
6. Perform application/deployment root-cause analysis.

---

### 237. Database is filling with trace files.

1. Check diagnostic directories.
2. Identify which process generates the traces.
3. Determine whether there is an underlying Oracle/application error.
4. Remove/archive files according to operational policy.
5. Fix the source of repeated errors.
6. Monitor filesystem usage.

---

### 238. Alert log is filling with errors.

1. Identify the repeated error code.
2. Determine whether it is harmless or critical.
3. Check frequency and timestamps.
4. Correlate with application/OS events.
5. Fix the root cause.
6. Continue monitoring after remediation.

---

### 239. Database has thousands of sessions after a release.

1. Identify the source application.
2. Check session states.
3. Check connection pool configuration.
4. Check for connection leaks.
5. Protect database resources.
6. Coordinate an application-side fix.

---

### 240. Production database is under attack from a runaway query.

1. Identify the SQL/session.
2. Determine resource impact.
3. Contact the responsible application team.
4. Stop the query/session only under emergency/change procedures.
5. Stabilize the database.
6. Fix the SQL/application issue.

---

### 241. A DBA accidentally dropped a critical tablespace.

1. Stop further changes.
2. Determine whether the operation committed.
3. Identify datafile availability.
4. Use appropriate recovery mechanisms.
5. Restore/recover affected files/tablespace.
6. Validate the database and application.

---

### 242. A database upgrade fails immediately after starting.

1. Stop and capture logs.
2. Run/inspect AutoUpgrade Analyze output.
3. Check prerequisites and source/target compatibility.
4. Correct the identified issue.
5. Re-run the supported workflow.
6. Do not manually bypass upgrade checks.

---

### 243. Production upgrade succeeded but application doesn't work.

1. Verify database status.
2. Check invalid objects.
3. Check services/listeners.
4. Check application connection configuration.
5. Check changed SQL/performance behavior.
6. Work with application owners on functional validation.

---

### 244. Production patch succeeded but performance degraded.

1. Establish exact before/after timing.
2. Identify affected SQL.
3. Compare execution plans.
4. Check waits/resources.
5. Determine whether the patch is causal.
6. Apply an Oracle-supported remediation after evidence confirms the cause.

---

### 245. Management asks for a DR readiness report.

1. Report primary/standby status.
2. Report transport/apply lag.
3. Report Broker health.
4. Report backup/recovery status.
5. Report last successful DR test.
6. Clearly state any gaps against RPO/RTO.

---

### 246. Management asks, "Are we protected if the entire data center fails?"

1. Check whether a geographically separate standby exists.
2. Check Data Guard configuration.
3. Check storage/network independence.
4. Check application failover architecture.
5. Check documented RTO/RPO.
6. Only answer "yes" if the complete recovery path has been tested.

---

### 247. You inherit an Oracle production database from another DBA.

1. Inventory Oracle version/homes.
2. Inventory database/RAC/ASM/Data Guard architecture.
3. Review backups and recovery tests.
4. Review patch level and security.
5. Review monitoring/runbooks.
6. Perform a controlled health assessment and document gaps.

---

### 248. You join a company with no DBA documentation.

1. Do not immediately change production.
2. Build a database inventory.
3. Document architecture and dependencies.
4. Identify backup/recovery procedures.
5. Identify RAC/Data Guard topology.
6. Create runbooks and validate them with controlled tests.

---

### 249. What should a senior DBA do during a major production incident?

1. **Stabilize** the environment.
2. **Diagnose** using evidence, not assumptions.
3. **Communicate** clear status and impact.
4. **Recover** using the safest tested procedure.
5. **Validate** database and application functionality.
6. **Perform RCA** and implement preventive actions.

---

### 250. You are the production DBA and everything is working. What should you do?

1. Monitor instead of assuming everything is healthy.
2. Check backups and recovery readiness.
3. Check storage/capacity trends.
4. Check patch/security compliance.
5. Test DR and recovery procedures regularly.
6. Continuously improve automation, monitoring, documentation, and runbooks.

---

# 🎯 Final 250-Scenario DBA Map

```text
001–025   Installation & Configuration
026–050   Users & Security
051–075   Storage & ASM
076–100   RMAN Backup & Recovery
101–125   Performance & Troubleshooting
126–150   Patching & Maintenance
151–175   Migration & Upgrade
176–200   RAC & ASM
201–225   Data Guard / DR
226–250   Production Incidents / Senior DBA
```

These scenarios are designed around the actual operational areas Oracle documents for database administration, backup/recovery, RAC, Data Guard, patching, migration, and upgrading. 

Oracle's upgrade documentation specifically covers AutoUpgrade Analyze/Fixup/Deploy/Upgrade workflows, multiple migration paths, and post-upgrade validation; 

its backup documentation covers RMAN and recovery strategy. ([Oracle Documentation][11])

**This gives you the requested 250 real-time FAQs, with each answer structured as 6 practical points.**

[1]: https://docs.oracle.com/en/database/oracle/oracle-database/19/install-and-upgrade.html "Oracle Database 19c - Install and Upgrade"
[2]: https://docs.oracle.com/en/database/oracle/oracle-database/19/bradv/introduction-backup-recovery.html "Introduction to Backup and Recovery"
[3]: https://docs.oracle.com/en/database/oracle/oracle-database/19/rilin/preparing-to-upgrade-an-existing-oracle-rac-database.html "Preparing to Upgrade an Existing Oracle RAC Database"
[4]: https://docs.oracle.com/en/database/oracle/oracle-database/19/upgrd/upgrade-parameters-autoupgrade-config-file.html "Upgrade Parameters for the AutoUpgrade Configuration File"
[5]: https://docs.oracle.com/en/database/oracle/oracle-database/19/upgrd/choose-an-upgrade-method-for-oracle-database.html "Choose an Upgrade Method for Oracle Database"
[6]: https://docs.oracle.com/en/database/oracle/oracle-database/19/spuss/preparing-to-upgrade.html "Preparing to Upgrade Oracle Database"
[7]: https://docs.oracle.com/en/database/oracle/oracle-database/19/upgrd/autoupgrade-command-line-parameters.html "AutoUpgrade Command-Line Parameters and Options"
[8]: https://docs.oracle.com/en/database/oracle/oracle-database/19/upgrd/required-tasks-complete-upgrading-oracle-database.html "Required Tasks to Complete After Upgrading Oracle Database"
[9]: https://docs.oracle.com/en/database/oracle/oracle-database/19/spucs/preparing-to-upgrade.html "Preparing to Upgrade Oracle Database"
[10]: https://docs.oracle.com/en/database/oracle/oracle-database/19/upgrd/tasks-prepare-upgrade-oracle-database.html "Preparing to Upgrade Oracle Database"
[11]: https://docs.oracle.com/en/database/oracle/oracle-database/19/upgrd/using-autoupgrade-oracle-database-upgrades.html "Using AutoUpgrade for Oracle Database Upgrades"
