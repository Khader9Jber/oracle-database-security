# Chapter 6 - Backup and Recovery Using RMAN

##

> Check of Archiving Mode is Enabled:

```sql
ARCHIVE LOG LIST;
 -- OR
 SELECT dbid, name, created, log_mode FROM v$database;
```
