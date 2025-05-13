#oracle #rds 

## Basic Archive Destination Queries

```sql
-- List valid archive destinations
SELECT dest_name, destination 
FROM v$archive_dest 
WHERE status = 'VALID';

-- View all archive destinations (including invalid)
SELECT * FROM v$archive_dest;

-- View archived log metadata
SELECT * FROM v$archived_log;
```

## Storage Usage Analysis

```sql
-- Calculate GB of archived logs in DEST_ID=1 (not deleted)
SELECT nvl(sum(BLOCKS * BLOCK_SIZE),0)/1024/1024/1024 GB
FROM V$ARCHIVED_LOG
WHERE DEST_ID=1 AND ARCHIVED='YES' AND DELETED='NO';

-- List files in LOG_ARCHIVE_DEST_1 directory (AWS RDS specific)
SELECT filename, filesize
FROM table(rdsadmin.rds_file_util.listdir('LOG_ARCHIVE_DEST_1'))
ORDER BY mtime;
```

## Database Configuration Checks

```sql
-- Verify database archiving mode
SELECT log_mode FROM v$database;

-- View alert log messages (AWS RDS)
SELECT message_text FROM alertlog;
```

## RDS-Specific Utilities

```sql
-- Show RDS configuration (PL/SQL block)
SET SERVEROUTPUT ON
EXEC rdsadmin.rdsadmin_util.show_configuration;

-- Crosscheck archivelogs without deleting (PL/SQL block)
SET SERVEROUTPUT ON
BEGIN
    rdsadmin.rdsadmin_rman_util.crosscheck_archivelog(
        p_delete_expired => FALSE,
        p_rman_to_dbms_output => FALSE);
END;
/
```