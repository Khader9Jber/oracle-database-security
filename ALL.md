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

> The main show command

```sql
SQL> sho parameter audit;
-- NAME                                 TYPE        VALUE
-- ------------------------------------ ----------- ------------------------------
-- audit_file_dest                      string      C:\APP\ADMINISTRATOR\ADMIN\SEC
--                                                  \ADUMP
-- audit_sys_operations                 boolean     TRUE
-- audit_trail                          string      NONE
-- unified_audit_systemlog              boolean     FALSE
```

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

> Enabling and Showing if the additional actions for SYSDBA and SYSOPER is audited

```sql
sho parameter audit_sys_operations;
-- Enables additional auditing of the SYSDBA or SYSOPER actions.
 alter system set audit_sys_operations = true scope=spfile;
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
[NO]AUDIT operation [BY user] [BY ACCESS|SESSION] [WHENEVER [NOT] SUCCESSFUL];

AUDIT table;
NOAUDIT table BY hr;
AUDIT table BY hr WHENEVER NOT SUCCESSFUL;
AUDIT table BY hr WHENEVER SUCCESSFUL;
AUDIT INSERT TABLE BY hr BY ACCESS;
```

![I1](.Images/i1.jpg)
![I2](.Images/i2.jpg)
![I3](.Images/i3.jpg)

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

> Purging the auditing trail

```sql
-- Deleting auditing in the last 90 days
DELETE FROM sys.aud$ WHERE timestamp# < SYSDATE -90;
```

### Value-Based Auditing

> Done by developers Using Triggers
>
> > The addition value of this type that U can save old and new values for any edit

```sql
CREATE OR REPLACE TRIGGER system.hrsalary_audit
AFTER UPDATE OF salary
ON hr.employees
REFERENCING NEW AS NEW OLD AS OLD
FOR EACH ROW
BEGIN
      IF :old.salary != :new.salary THEN
              INSERT INTO system.audit_employees
             VALUES (sys_context('userenv','os_user'), sysdate,
             sys_context('userenv','ip_address’),
             :new.employee_id ||' salary changed from
              '||:old.salary|| ' to '||:new.salary);
END IF;
END;
/
```

### Fine-Grained Auditing (FGA)

> Auditing According to specific conditions (e.g if someone made a query on the manager salary)
>
> > - Working on SELECT + DML (INSERT, UPDATE, DELETE)
> > - On operations happening on table or column inside table
> > - Fist we need to create a policy, then we need to enable it.
> > - U can control policies via package located in SYS schema

```sql
-- To create a new FGA policy, use the packaged procedure DBMS_FGA.ADD_POLICY
-- This procedure has the following parameters:

-- INPUT PARAMETERS
--   object_schema   - schema owning the table/view, current user if NULL
--   object_name     - name of table or view
--   policy_name     - name of policy to be added
--   audit_column    - column to be audited
--   audit_condition - predicates for this policy
--   handler_schema  - schema where the event handler procedure is
--   handler_module  - name of the event handler
--   enable          - policy is enabled by DEFAULT
--   statement_type  - statement type a policy applies to (default SELECT)
--   audit_trail     - Write sqltext and sqlbind into audit trail as well
--                     as destination of audit trail (default DB_EXTENDED)
--   audit_column_options - option of using 'Any' or 'All' on audit columns
--                          for the policy (default ANY)
--   policy_owner    - Owner of FGA policy to be added (default NULL)
```

> Creating a new disabled policy

```sql
DBMS_FGA.ADD_POLICY(object_schema=>'HR',
object_name=>'EMPLOYEES',
policy_name=>'COMPENSATION_AUD',
audit_column=>'SALARY, COMMISSION_PCT',
enable=>FALSE,
statement_types=>'SELECT');

-- Check if it works (Must be not because it is disabled)
SELECT * FROM employees;
```

> Enabling & Disabling disabled policy

```sql
-- Enabling
DBMS_FGA.ENABLE_POLICY(object_schema=>'HR',
object_name=>'EMPLOYEES',
policy_name=>'COMPENSATION_AUD');

-- Check if it works (Must be working because it is now enabled)
SELECT * FROM employees;

-- Disabling
DBMS_FGA.DISABLE_POLICY(object_schema=>'HR',
object_name=>'EMPLOYEES',
policy_name=>'COMPENSATION_AUD');

-- Check if it works (Must be not working because it is now disabled)
SELECT * FROM employees;
```

> Dropping FGA policy

```sql
DBMS_FGA.DROP_POLICY(object_schema=>'HR',
object_name=>'EMPLOYEES',
policy_name=>'COMPENSATION_AUD');
```

> Viewing FGA policies and related info

```sql
-- View all policies
SELECT * FROM dba_audit_policies;

-- Reporting on FGA audit-trail entries
SELECT *
FROM dba_fga_audit_trail
WHERE policy_name='COMPENSATION_AUD’
```

### Unified Auditing

> Enabling & Disabling Unified Auditing

```sql
-- 1- Stopping Lister
lsnrctl stop

-- 2- Stopping DB
shutdown immediate

-- 3- Stopping DB Service (Append UR CDB name)
net stop OracleService~~sec~~

-- 4- Relink Oracle Home Binary:
-- Unix Systems
cd $ORACLE_HOME/rdbms/lib
make -f ins_rdbms.mk uniaud_on ioracle ORACLE_HOME=$ORACLE_HOME
-- Windows Systems
C:\app\db_home\bin\orauniaud.dll.dbl -- Delete .dbl

-- 5- Startup the DB
startup

-- 6- Startup DB Service (Append UR CDB name)
net start OracleService~~sec~~

-- 7- Startup Lister
lsnrctl start

-- 8- Check if the Unified Auditing is enabled
select * from v$option where parameter='Unified Auditing';

-- 9- Disable other auditing mechanisms
alter system set audit_trail = none  scope= spfile;


--- NOTE: Now there are some actions will be audited mandatory without any policy
-- CREATE AUDIT POLICY
-- ALTER AUDIT POLICY
-- DROP AUDIT POLICY
-- AUDIT
-- NOAUDIT
-- EXECUTE of DBMS_FGA
-- EXECUTE of DBMS_AUDIT_MGMT
```

> Showing audited records
>
> > AUD$ and FGA$ tables replaced with one single audit trail table (UNIFIED_AUDIT_TRAIL).

```sql
select * from unified_audit_trail;
```

> Creating audit policy
>
> > U must have AUDIT SYSTEM (system privilege) or AUDIT ADMIN (role)

```sql
CREATE AUDIT POLICY policy_name audit_clauses
[WHEN 'audit_condition'
EVALUATE PER [STATEMENT | SESSION | INSTANCE]]

-- The audit_clauses have different options and syntax based on whether privilege, statement, or role is audited.
-- The audit_condition specifies a condition that determines if the unified audit policy is enforced


CREATE AUDIT POLICY unified_syspriv_1
PRIVILEGES SELECT ANY TABLE, ALTER SESSION;

-- Policy to audit all system and object privileges granted directly to the role resource
CREATE AUDIT POLICY unified_role_audit_1
ROLES resource;


-- Create Multi purpose Policy
CREATE AUDIT POLICY unified_multi_1
PRIVILEGES SELECT ANY TABLE, ALTER SESSION
ACTIONS execute, alter database link
ROLES dba, resource;

CREATE AUDIT POLICY unified_obj1_hr
ACTIONS delete, insert ON hr.departments;

-- Audit datapump export actions
-- RMAN backup, restore, and recover operations are audited by default by the unified audit
CREATE AUDIT POLICY datapump_exp_aud
ACTIONS COMPONENT=datapump export;
```

> Enabling and Disabling Audit Policies

```sql
AUDIT POLICY policy
[ { BY user [, user]... } | { EXCEPT user [, user]... } ]
[ WHENEVER [ NOT ] SUCCESSFUL ]


-- Enabling
AUDIT POLICY  unified_obj1_hr

AUDIT POLICY unified_syspriv_1
BY nael, ahmad WHENEVER NOT SUCCESSFUL;

-- Disabling
NOAUDIT POLICY unified_obj1_hr
```

> Policies Query

```
-- All Predefined Policies
select *
from AUDIT_UNIFIED_POLICIES;

-- Enabled policies
select * from audit_unified_enabled_audit;
```

> Purging Unified Audit Records
>
> > - Unlike SYS.AUD$, UNIFIED_AUDIT_TRAIL cannot be deleted using DML statements.
> > - delete from unified_audit_trail; (Cause ERROR)
> > - Done using using the DBMS_AUDIT_MGMT.CLEAN_AUDIT_TRAIL procedure

```
begin
dbms_audit_mgmt.clean_audit_trail(
audit_trail_type => dbms_audit_mgmt.AUDIT_TRAIL_UNIFIED,
use_last_arch_timestamp => false);
end;
```

> Full Example

```sql
-- Create
CREATE AUDIT POLICY unified_obj1_hr
ACTIONS delete, insert ON hr.departments;

-- Enable
audit policy unified_obj1_hr;

-- Test
Insert into HR.DEPARTMENTS (DEPARTMENT_ID,DEPARTMENT_NAME,MANAGER_ID,LOCATION_ID) values (2000,'Test',null,1700);


-- Check
select *
from unified_audit_trail
WHERE  UNIFIED_AUDIT_POLICIES like '%HR%'


-- Clear

begin
dbms_audit_mgmt.clean_audit_trail(
audit_trail_type => dbms_audit_mgmt.AUDIT_TRAIL_UNIFIED,
use_last_arch_timestamp => false);
end;

-- Disable
noaudit policy unified_obj1_hr;

-- Test and Check again!
```

> Query RMAN operations

```
select *
from UNIFIED_AUDIT_TRAIL
where RMAN_OPERATION is not null;
```

---

## Chapter 6 - Backup and Recovery
