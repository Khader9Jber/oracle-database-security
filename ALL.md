# Oracle Database Security

## Chapter 1

---

## Chapter 2

---

## Chapter 3

---

## Chapter 4

---

## Chapter 5

> Show all recorded records
>
> > These stored in AUDSYS schema and SYSAUX tablespace

```sql
select * from unified_audit_trail;
```

> Show if XML standard auditing is on:

```sql
select * from v$xml_audit_trail;
```

> Show the location of standard auditing:

```sql
show parameter audit_file_dest;
```

> Enabling Standard Auditing:

```sql
Show parameter audit_trail;

--Auditing is disabled, when audit_trail is set to NONE
  alter system set audit_trail = none  scope= spfile;

--- Either set audit_trail to DB or DB,EXTENDED.
alter system set audit_trail = db  scope= spfile;
alter system set audit_trail = db, extended scope= spfile;

-- Either set audit_trail to xml or XML,EXTENDED
alter system set audit_trail = xml scope= spfile;
alter system set audit_trail = xml, extended scope= spfile;

-- Restart Database
shutdown immediate;
startup;
```

---

## Chapter 6 - Backup and Recovery Using RMAN
