---
layout: post
nav: blog
title: Get foreign key constraints
---
When attempting to archive a PostgreSQL database, I encountered an issue while trying to delete data from the orders table.

So I have to identify all the foreign key constraints of the orders table before taking any action such as removing foreign key constraints or deleting records from related tables.

Here's the query to get all foreign key constraints of the orders table:

```
SELECT (select  r.relname from pg_class r where r.oid = c.confrelid) as base_table,
       a.attname as base_col,
       (select r.relname from pg_class r where r.oid = c.conrelid) as referencing_table,
       UNNEST((select array_agg(attname) from pg_attribute where attrelid = c.conrelid and array[attnum::int] <@ c.conkey)) as referencing_col,
       pg_get_constraintdef(c.oid) contraint_sql
  FROM pg_constraint c join pg_attribute a on c.confrelid=a.attrelid and a.attnum = ANY(confkey)
 WHERE c.confrelid = (select oid from pg_class where relname = 'orders')
   AND c.confrelid!=c.conrelid;
```

Execute the query, and it will return a result set containing the foreign key constraint name, the table containing the constraint, the column name in the order table, the referenced table, and the referenced column.

By analyzing the output of this query, I had a clear understanding of the foreign key constraints associated with the orders table and can proceed with appropriate actions.
