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

### Mandatory Auditing

> Show all recorded records
>
> > These stored in AUDSYS schema and SYSAUX tablespace

```sql
select * from unified_audit_trail;
```

> Show the physical location where the audit stored

```sql
sho parameter audit_file_dest;
```

### Standard Auditing

> Show The standard XML auditing records:

```sql
select * from v$xml_audit_trail;
```

> Disabling AND Enabling Standard Auditing
>
> > The records are sorted in SYS.AUD$

```sql
Show parameter audit_trail;

-- Disable Auditing
  alter system set audit_trail = none  scope= spfile;

--- Enabling either DB or XML auditing.
-- EXTENDED (To records SQL statements and bind variables).
alter system set audit_trail = [db|xml][, extended]  scope= spfile;
-- Restart Database
shutdown immediate;
startup;

-- Sho recoded records
select * from sys.aud$;
```

> Enabling and Disabling statement auditing
>
> > Always BY ACCESS

```sql
[NO]AUDIT operation [BY user] [ON object] [WHENEVER [NOT] SUCCESSFUL];

AUDIT table;
NOAUDIT table BY hr;
AUDIT table BY hr WHENEVER NOT SUCCESSFUL;
AUDIT table BY hr WHENEVER SUCCESSFUL;
AUDIT INSERT TABLE BY hr BY ACCESS;
```

> Show records happening because specific user

```sql
SELECT username, timestamp, action_name
FROM dba_audit_trail
WHERE username = 'HR';
```

> Show enabled statement audit options

```sql
select * from dba_stmt_audit_opts;
```

> Show enabled privilege audit options

```sql
select * from dba_priv_audit_opts;
```

---

## Chapter 6 - Backup and Recovery Using RMAN
