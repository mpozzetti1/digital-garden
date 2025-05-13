#sql #oracle 

```sql
set lines 256
set trimout on
set tab off

SELECT 
    owner, 
    ROUND(SUM(bytes)/1024/1024) AS size_mb, 
    (SELECT COUNT(*) FROM all_objects ao WHERE ao.owner = a.owner AND ao.object_type = 'TABLE') AS table_count,
    (SELECT COUNT(*) FROM all_objects ao WHERE ao.owner = a.owner AND ao.object_type = 'TRIGGER') AS trigger_count,
    (SELECT COUNT(*) FROM all_objects ao WHERE ao.owner = a.owner AND ao.object_type = 'VIEW') AS view_count,
    (SELECT COUNT(*) FROM all_objects ao WHERE ao.owner = a.owner AND ao.object_type = 'PROCEDURE') AS procedure_count,
    (SELECT COUNT(*) FROM all_objects ao WHERE ao.owner = a.owner AND ao.object_type = 'FUNCTION') AS function_count,
    (SELECT COUNT(*) FROM all_objects ao WHERE ao.owner = a.owner AND ao.object_type = 'PACKAGE') AS package_count,
    (SELECT COUNT(*) FROM all_objects ao WHERE ao.owner = a.owner AND ao.object_type = 'SEQUENCE') AS sequence_count,
    (SELECT COUNT(*) FROM all_objects ao WHERE ao.owner = a.owner AND ao.object_type = 'SYNONYM') AS synonym_count,
    (SELECT COUNT(*) FROM all_objects ao WHERE ao.owner = a.owner AND ao.object_type = 'INDEX') AS index_count,
    (SELECT COUNT(*) FROM all_objects ao WHERE ao.owner = a.owner AND ao.status = 'INVALID') AS invalid_object_count
FROM 
    dba_segments a
GROUP BY 
    owner
ORDER BY 
    size_mb DESC;
```