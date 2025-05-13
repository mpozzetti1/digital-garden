#sql #oracle 

```sql
set long 20000
set longchunksize 20000
set pagesize 0
set linesize 1000
set trimspool on
set column ddl format a1000
set feedback off
set verify off
--Add a semicolon at the end of each statement
execute dbms_METADATA.SET_TRANSFORM_PARAM(DBMS_METADATA.SESSION_TRANSFORM,'SQLTERMINATOR',true);
--Generate the DDL for User you enter
select dbms_metadata.get_ddl('USER', u.username) AS ddl
from dba_users u
where u.username = 'C##ZABBIX'
union all
select dbms_metadata.get_granted_ddl('TABLESPACE_QUOTA', tq.username) AS ddl
from dba_ts_quotas tq
where tq.username = 'C##ZABBIX' and rownum = 1
union all
select dbms_metadata.get_granted_ddl('ROLE_GRANT', rp.grantee) AS ddl
from dba_role_privs rp
where rp.grantee = 'C##ZABBIX'
and rownum = 1
union all
select dbms_metadata.get_granted_ddl('SYSTEM_GRANT', sp.grantee) AS ddl
from dba_sys_privs sp
where sp.grantee = 'C##ZABBIX'
and rownum = 1
union all
select dbms_metadata.get_granted_ddl('OBJECT_GRANT', tp.grantee) AS ddl
from dba_tab_privs tp
where tp.grantee = 'C##ZABBIX'
and rownum = 1
union all
select dbms_metadata.get_granted_ddl('DEFAULT_ROLE', rp.grantee) AS ddl
from dba_role_privs rp
where rp.grantee = 'C##ZABBIX'
and rp.default_role = 'YES'
and rownum = 1
union all
select to_clob('/* Start profile creation script in case they are missing') AS ddl
from dba_users u
where u.username = 'C##ZABBIX'
and u.profile='DEFAULT'
and rownum = 1
union all
select dbms_metadata.get_ddl('PROFILE', u.profile) AS ddl
from dba_users u
where u.username = 'C##ZABBIX'
and u.profile='DEFAULT'
union all
select to_clob('End profile creation script */') AS ddl
from dba_users u
where u.username = 'C##ZABBIX'
and u.profile='DEFAULT'
and rownum = 1
/
```