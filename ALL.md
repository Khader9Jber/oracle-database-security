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

#### Statement Level

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

#### Privilege Level

> Enabling and Disabling privilege auditing
>
> > Defaults: SYSTEM privilege - By Access

```sql
AUDIT privilege_name [BY username] [by SESSION | ACCESS] [WHENEVER SUCCESSFUL | NOT SUCCESSFUL]

NOAUDIT CREATE SESSION ;
NOAUDIT CREATE SESSION BY HR BY ACCESS;
NOAUDIT UPDATE ANY TABLE  BY HR WHENEVER  NOT SUCCESSFUL;
AUDIT DELETE ANY TABLE BY HR BY ACCESS;
```

> Show enabled privilege audit options

```sql
select * from dba_priv_audit_opts;
```

#### Object Level

> Enabling and Disabling object auditing
>
> > Defaults: SYSTEM privilege - By Access

```sql
AUDIT object_priv_name ON object_name [BY ACCESS | SESSION][WHENEVER SUCCESSFUL|NOT SUCCESSFUL]

AUDIT all ON employees;
AUDIT DELETE ON hr.employees;
AUDIT DELETE ON hr.employees by access;
```

> Show enabled privilege audit options
>
> > - "-" audit option is not set. "S" BY SESSION. "A" BY ACCESS.
> > - Audit option either WHENEVER SUCCESSFUL and WHENEVER NOT SUCCESSFUL

```sql
select * from dba_obj_audit_opts;
```

> Showing auditing records recorded according to specific statement or object or both

```sql
select * from dba_audit_trail WHERE obj_name='EMPLOYEES' AND action_name='DELETE';
```

---

## Chapter 6 - Backup and Recovery
