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
> > 1. Connect sys as sysdba => 2. Make sure you are in the root => 3.Do this: ARCHIVE LOG LIST; to see if option is enabled or no => 4. If not then

```sql
Shutdown immediate;
STARTUP MOUNT;
ALTER database archivelog;
ALTER DATABASE OPEN;
ALTER PLUGGABLE DATABASE ALL OPEN;
```
