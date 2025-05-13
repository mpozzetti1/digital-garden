#sql #oracle 

## List Files in Directory

```sql
SELECT filename, filesize FROM table(rdsadmin.rds_file_util.listdir('DATA_PUMP_DIR')) order by mtime;
```

## Remove file from Directory

```sql
EXEC UTL_FILE.FREMOVE('DATA_PUMP_DIR','ULSLAST.dmp');
```

## Move File to another DB using DBLINK

```sql
BEGIN
DBMS_FILE_TRANSFER.PUT_FILE(
  source_directory_object => 'EXP_FOR_RDS',
  source_file_name => 'base392.dmp',
  destination_directory_object => 'DATA_PUMP_DIR',
  destination_file_name => 'base392.dmp',
  destination_database => 'marcos_fmport'
);
END;
/
```