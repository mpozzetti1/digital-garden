#sql #oracle 

```sql
set pages 200
select
'===========================================================' || chr(10) ||
'Total Database Physical Size = ' || round(redolog_size_gb+dbfiles_size_gb+tempfiles_size_gb+ctlfiles_size_gb,2) || ' GB' || chr(10) ||
'===========================================================' || chr(10) ||
' Redo Logs Size : ' || round(redolog_size_gb,3) || ' GB' || chr(10) ||
' Data Files Size : ' || round(dbfiles_size_gb,3) || ' GB' || chr(10) ||
' Temp Files Size : ' || round(tempfiles_size_gb,3) || ' GB' || chr(10) ||
' Archive Log Size - Approx only : ' || round(archlog_size_gb,3) || ' GB' || chr(10) ||
' Control Files Size : ' || round(ctlfiles_size_gb,3) || ' GB' || chr(10) ||
'===========================================================' || chr(10) ||
' Used Database Size : ' || used_db_size_gb || ' GB' || chr(10) ||
' Free Database Size : ' || free_db_size_gb || ' GB' ||chr(10) ||
' Data Pump Directory Size : ' || dpump_db_size_gb || ' GB' || chr(10) ||
' BDUMP Directory Size : ' || bdump_db_size_gb || ' GB' || chr(10) ||
' ADUMP Directory Size : ' || adump_db_size_gb || ' GB' || chr(10) ||
'===========================================================' || chr(10) ||
'Total Size (including Dump and Log Files) = ' || round(round(redolog_size_gb,2) +round(dbfiles_size_gb,2)+round(tempfiles_size_gb,2)+round(ctlfiles_size_gb,2) +round(adump_db_size_gb,2) +round(dpump_db_size_gb,2)+round(bdump_db_size_gb,2),2) || ' GB' || chr(10) ||
'===========================================================' as summary
FROM (SELECT sys_context('USERENV', 'DB_NAME')
db_name,
(SELECT SUM(bytes) / 1024 / 1024 / 1024 redo_size
FROM v$log)
redolog_size_gb,
(SELECT SUM(bytes) / 1024 / 1024 / 1024 data_size
FROM dba_data_files)
dbfiles_size_gb,
(SELECT nvl(SUM(bytes), 0) / 1024 / 1024 / 1024 temp_size
FROM dba_temp_files)
tempfiles_size_gb,
(SELECT SUM(blocks * block_size / 1024 / 1024 / 1024) size_gb
FROM v$archived_log
WHERE first_time >= SYSDATE - (
(SELECT value
FROM rdsadmin.rds_configuration
WHERE name =
'archivelog retention hours') /
24 ))
archlog_size_gb,
(SELECT SUM(block_size * file_size_blks) / 1024 / 1024 / 1024
controlfile_size
FROM v$controlfile)
ctlfiles_size_gb,
round(SUM(used.bytes) / 1024 / 1024 / 1024, 3)
db_size_gb,
round(SUM(used.bytes) / 1024 / 1024 / 1024, 3) - round(
free.f / 1024 / 1024 / 1024)
used_db_size_gb,
round(free.f / 1024 / 1024 / 1024, 3)
free_db_size_gb,
(SELECT round(SUM(filesize) / 1024 / 1024 / 1024, 3)
FROM TABLE(rdsadmin.rds_file_util.listdir('BDUMP')))
bdump_db_size_gb,
(SELECT round(SUM(filesize) / 1024 / 1024 / 1024, 3)
FROM TABLE(rdsadmin.rds_file_util.listdir('ADUMP')))
adump_db_size_gb,
(SELECT round(SUM(filesize) / 1024 / 1024 / 1024, 3)
FROM TABLE(rdsadmin.rds_file_util.listdir('DATA_PUMP_DIR')))
dpump_db_size_gb
FROM (SELECT bytes
FROM v$datafile
UNION ALL
SELECT bytes
FROM v$tempfile) used,
(SELECT SUM(bytes) AS f
FROM dba_free_space) free
GROUP BY free.f);
```