# Chapter 6 - Backup and Recovery

## Prerequisites

### 1. Archiving Mode is ON

> Check if Archiving Mode is Enabled:

```sql
ARCHIVE LOG LIST;
-- OR
SELECT dbid, name, created, log_mode FROM v$database;
```

> Enabling Archiving Mode
>
> > 1. Connect sys as sysdba => 2. Make sure you are in the root => 3. Do this: ARCHIVE LOG LIST; to see if option is enabled or not => 4. If not, then:

```sql
Shutdown immediate;
STARTUP MOUNT;
ALTER database archivelog;
ALTER DATABASE OPEN;
ALTER PLUGGABLE DATABASE ALL OPEN;
```

---

### 2. Flash Recovery Area is ON

> Check if Flash Recovery Area is Enabled:

```sql
show parameter db_recovery_file;
-- OR
show parameter db_recovery_file_dest;
-- OR
show parameter db_recovery_file_dest_size;
-- OR
select * from V$RECOVERY_FILE_DEST;
-- OR
select * from v$flash_recovery_area_usage;
```

> Enabling Flash Recovery Area
>
> > 1. Create a Directory => 2. Make sure you are in the root => 3. Do this: SHOW PARAMETER db_recovery_file; to see if option is enabled or not => 4. If not, then:

```sql

alter system set db_recovery_file_dest_size=2G scope=both sid='*';


ALTER SYSTEM SET DB_RECOVERY_FILE_DEST =' C:\app\Administrator\FRA\ ' scope=both sid='*';

alter system set log_archive_dest_1=' LOCATION=C:\app\Administrator\FRA1\ ' scope=both;


-- After enabled Flash Recovery area check the status
select * from v$flash_recovery_area_usage;

-- Switch Archivelog from database level and double check the archivelog
alter system switch logfile;
select dest_name,status,error from v$archive_dest_status;
```

---

## Important Notes

- Inconsistent Backup = Hot Backup = Online Backup
- Consistent Backup = Cold Backup = Offline Backup
- U can perform both (Hot and Cold) if U enabled ARCHIVELOG mode. but if not, U can do Only Cold Backup.
- RMAN is the primary component of the Oracle database used to perform backup and recovery operations.
- You can use RMAN to back up all types: whole or partial databases, full or incremental, and consistent or inconsistent.
- Oracle RMAN (Oracle Recovery Manager) is a utility built into Oracle databases to automate backup and recovery.
- How RMAN works
  - The RMAN environment must include a target database and the RMAN client.
  - It performs backing up, recovering and restoring the database files.
- To perform backup and recovery tasks with RMAN, you must connect to the database as a user with the **SYSDBA** or **SYSBACKUP** privilege.

> Backing Up the Control File
>
> > Before backup you should making multiplexing

```sql
Rman target /
-- RMAN>
BACKUP CURRENT CONTROLFILE;
```

> Backing Up the Database

```sql
Rman target /
RMAN> BACKUP CURRENT CONTROLFILE;
```

---

#### Performing User-Managed Cold Backups

> Cold backups are performed after shutting down the database.

```sql
SHUTDOWN IMMEDIATE or SHUTDOWN TRANSACTIONAL
```

> Copy all control files and data files to another location or to your tape management system using OS commands. You also need to back up the parameter file (init file or spfile) and password file.

```sql
select * from V$CONTROLFILE;
```

---

#### Performing User-Managed Hot Backups

> Before starting to copy the data files belonging to a tablespace, you must place the tablespace in backup mode using the BEGIN BACKUP clause.

```sql
ALTER TABLESPACE users BEGIN BACKUP;
```

> When a tablespace is in backup mode, use OS utilities to copy the data files belonging to the tablespace to another location or to the tape management system.

```sql
ALTER TABLESPACE users END BACKUP;
```
