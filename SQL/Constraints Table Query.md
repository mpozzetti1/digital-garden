#sql #oracle

```sql
SELECT 'ALTER TABLE ' || a.owner || '.' || a.table_name || 
       ' ENABLE CONSTRAINT ' || a.constraint_name || ';' as enable_cmd
FROM all_constraints a
WHERE a.r_constraint_name IN
    (SELECT constraint_name
     FROM all_constraints
     WHERE owner IN ('ADMREGBASILICATA', 'ADMASP6PALERMO')
     AND table_name IN ('WRM', 'WRS', 'BL_BCK20191211', 'BL_BCK20200109')
     AND constraint_type IN ('P','U'))
AND a.constraint_type = 'R'
ORDER BY a.owner, a.table_name;
```

