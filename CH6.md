# Chapter 6 - Backup and Recovery Using RMAN

##

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


ALTER SYSTEM SET DB_RECOVERY_FILE_DEST ='C:\app\Administrator\FRA\' scope=both sid='*';

alter system set log_archive_dest_1='LOCATION=C:\app\Administrator\FRA1\' scope=both;


-- After enabled Flash Recovery area check the status
select * from v$flash_recovery_area_usage;


-- Switch Archivelog from database level and double check the archivelog
alter system switch logfile;
select dest_name,status,error from v$archive_dest_status;
```
