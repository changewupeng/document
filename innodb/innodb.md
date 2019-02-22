# InnoDB

## Introduction to InnoDB

### Key Advantages of InnoDB

- Its [DML](https://dev.mysql.com/doc/refman/5.7/en/glossary.html#glos_dml) operations follow the [ACID](https://dev.mysql.com/doc/refman/5.7/en/glossary.html#glos_acid) model, with [transactions](https://dev.mysql.com/doc/refman/5.7/en/glossary.html#glos_transaction) featuring [commit](https://dev.mysql.com/doc/refman/5.7/en/glossary.html#glos_commit), [rollback](https://dev.mysql.com/doc/refman/5.7/en/glossary.html#glos_rollback), and [crash-recovery](https://dev.mysql.com/doc/refman/5.7/en/glossary.html#glos_crash_recovery)capabilities to protect user data. 
- Row-level [locking](https://dev.mysql.com/doc/refman/5.7/en/glossary.html#glos_locking) and Oracle-style [consistent reads](https://dev.mysql.com/doc/refman/5.7/en/glossary.html#glos_consistent_read) increase multi-user concurrency and performance. 
- `InnoDB` tables arrange your data on disk to optimize queries based on [primary keys](https://dev.mysql.com/doc/refman/5.7/en/glossary.html#glos_primary_key). Each `InnoDB` table has a primary key index called the [clustered index](https://dev.mysql.com/doc/refman/5.7/en/glossary.html#glos_clustered_index) that organizes the data to minimize I/O for primary key lookups. 
- To maintain data [integrity](https://dev.mysql.com/doc/refman/5.7/en/glossary.html#glos_referential_integrity), `InnoDB` supports [`FOREIGN KEY`](https://dev.mysql.com/doc/refman/5.7/en/glossary.html#glos_foreign_key) constraints. With foreign keys, inserts, updates, and deletes are checked to ensure they do not result in inconsistencies across different tables. 



![innodbfeture](/home/wupeng/media/github/document/innodb/img/innodbfeature.png)



### Benefits of Using InnoDB Tables

You may find `InnoDB` tables beneficial for the following reasons:

- If your server crashes because of a hardware or software issue, regardless of what was happening in the database at the time, you don't need to do anything special after restarting the database. `InnoDB` [crash recovery](https://dev.mysql.com/doc/refman/5.7/en/glossary.html#glos_crash_recovery) automatically finalizes any changes that were committed before the time of the crash, and undoes any changes that were in process but not committed. Just restart and continue where you left off.
- The `InnoDB` storage engine maintains its own [buffer pool](https://dev.mysql.com/doc/refman/5.7/en/glossary.html#glos_buffer_pool) that caches table and index data in main memory as data is accessed. Frequently used data is processed directly from memory. This cache applies to many types of information and speeds up processing. On dedicated database servers, up to 80% of physical memory is often assigned to the buffer pool.
- If you split up related data into different tables, you can set up [foreign keys](https://dev.mysql.com/doc/refman/5.7/en/glossary.html#glos_foreign_key) that enforce [referential integrity](https://dev.mysql.com/doc/refman/5.7/en/glossary.html#glos_referential_integrity). Update or delete data, and the related data in other tables is updated or deleted automatically. Try to insert data into a secondary table without corresponding data in the primary table, and the bad data gets kicked out automatically.
- If data becomes corrupted on disk or in memory, a [checksum](https://dev.mysql.com/doc/refman/5.7/en/glossary.html#glos_checksum) mechanism alerts you to the bogus data before you use it.
- When you design your database with appropriate [primary key](https://dev.mysql.com/doc/refman/5.7/en/glossary.html#glos_primary_key) columns for each table, operations involving those columns are automatically optimized. It is very fast to reference the primary key columns in [`WHERE`](https://dev.mysql.com/doc/refman/5.7/en/select.html) clauses, [`ORDER BY`](https://dev.mysql.com/doc/refman/5.7/en/select.html)clauses, [`GROUP BY`](https://dev.mysql.com/doc/refman/5.7/en/select.html) clauses, and [join](https://dev.mysql.com/doc/refman/5.7/en/glossary.html#glos_join) operations.
- Inserts, updates, and deletes are optimized by an automatic mechanism called [change buffering](https://dev.mysql.com/doc/refman/5.7/en/glossary.html#glos_change_buffering). `InnoDB` not only allows concurrent read and write access to the same table, it caches changed data to streamline disk I/O.
- Performance benefits are not limited to giant tables with long-running queries. When the same rows are accessed over and over from a table, a feature called the [Adaptive Hash Index](https://dev.mysql.com/doc/refman/5.7/en/glossary.html#glos_adaptive_hash_index) takes over to make these lookups even faster, as if they came out of a hash table.
- You can compress tables and associated indexes.
- You can create and drop indexes with much less impact on performance and availability.
- Truncating a [file-per-table](https://dev.mysql.com/doc/refman/5.7/en/glossary.html#glos_file_per_table) tablespace is very fast, and can free up disk space for the operating system to reuse, rather than freeing up space within the [system tablespace](https://dev.mysql.com/doc/refman/5.7/en/glossary.html#glos_system_tablespace) that only `InnoDB` can reuse.
- The storage layout for table data is more efficient for [`BLOB`](https://dev.mysql.com/doc/refman/5.7/en/blob.html) and long text fields, with the [DYNAMIC](https://dev.mysql.com/doc/refman/5.7/en/glossary.html#glos_dynamic_row_format) row format.
- You can monitor the internal workings of the storage engine by querying [INFORMATION_SCHEMA](https://dev.mysql.com/doc/refman/5.7/en/glossary.html#glos_information_schema) tables.
- You can monitor the performance details of the storage engine by querying [Performance Schema](https://dev.mysql.com/doc/refman/5.7/en/glossary.html#glos_performance_schema) tables.
- You can freely mix `InnoDB` tables with tables from other MySQL storage engines, even within the same statement. For example, you can use a [join](https://dev.mysql.com/doc/refman/5.7/en/glossary.html#glos_join) operation to combine data from `InnoDB` and [`MEMORY`](https://dev.mysql.com/doc/refman/5.7/en/memory-storage-engine.html) tables in a single query.
- `InnoDB` has been designed for CPU efficiency and maximum performance when processing large data volumes.
- `InnoDB` tables can handle large quantities of data, even on operating systems where file size is limited to 2GB.

## Best Practices for InnoDB Tables

This section describes best practices when using `InnoDB` tables.

- Specifying a [primary key](https://dev.mysql.com/doc/refman/5.7/en/glossary.html#glos_primary_key) for every table using the most frequently queried column or columns, or an [auto-increment](https://dev.mysql.com/doc/refman/5.7/en/glossary.html#glos_auto_increment)value if there is no obvious primary key.

- Using [joins](https://dev.mysql.com/doc/refman/5.7/en/glossary.html#glos_join) wherever data is pulled from multiple tables based on identical ID values from those tables. For fast join performance, define [foreign keys](https://dev.mysql.com/doc/refman/5.7/en/glossary.html#glos_foreign_key) on the join columns, and declare those columns with the same data type in each table. Adding foreign keys ensures that referenced columns are indexed, which can improve performance. Foreign keys also propagate deletes or updates to all affected tables, and prevent insertion of data in a child table if the corresponding IDs are not present in the parent table.

- Turning off [autocommit](https://dev.mysql.com/doc/refman/5.7/en/glossary.html#glos_autocommit). Committing hundreds of times a second puts a cap on performance (limited by the write speed of your storage device).

- Grouping sets of related [DML](https://dev.mysql.com/doc/refman/5.7/en/glossary.html#glos_dml) operations into [transactions](https://dev.mysql.com/doc/refman/5.7/en/glossary.html#glos_transaction), by bracketing them with `START TRANSACTION` and `COMMIT`statements. While you don't want to commit too often, you also don't want to issue huge batches of [`INSERT`](https://dev.mysql.com/doc/refman/5.7/en/insert.html), [`UPDATE`](https://dev.mysql.com/doc/refman/5.7/en/update.html), or[`DELETE`](https://dev.mysql.com/doc/refman/5.7/en/delete.html) statements that run for hours without committing.

- Not using [`LOCK TABLES`](https://dev.mysql.com/doc/refman/5.7/en/lock-tables.html) statements. `InnoDB` can handle multiple sessions all reading and writing to the same table at once, without sacrificing reliability or high performance. To get exclusive write access to a set of rows, use the [`SELECT ... FOR UPDATE`](https://dev.mysql.com/doc/refman/5.7/en/innodb-locking-reads.html) syntax to lock just the rows you intend to update.

- Enabling the [`innodb_file_per_table`](https://dev.mysql.com/doc/refman/5.7/en/innodb-parameters.html#sysvar_innodb_file_per_table) option or using general tablespaces to put the data and indexes for tables into separate files, instead of the [system tablespace](https://dev.mysql.com/doc/refman/5.7/en/glossary.html#glos_system_tablespace).

  The [`innodb_file_per_table`](https://dev.mysql.com/doc/refman/5.7/en/innodb-parameters.html#sysvar_innodb_file_per_table) option is enabled by default.

- Evaluating whether your data and access patterns benefit from the `InnoDB` table or page [compression](https://dev.mysql.com/doc/refman/5.7/en/glossary.html#glos_compression) features. You can compress `InnoDB` tables without sacrificing read/write capability.

- Running your server with the option [`--sql_mode=NO_ENGINE_SUBSTITUTION`](https://dev.mysql.com/doc/refman/5.7/en/server-system-variables.html#sysvar_sql_mode) to prevent tables being created with a different storage engine if there is an issue with the engine specified in the `ENGINE=` clause of [`CREATE TABLE`](https://dev.mysql.com/doc/refman/5.7/en/create-table.html).