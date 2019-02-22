# my.cnf



```



[mysqld]
## 默认引擎为innodb
default-storage-engine=innodb

innodb_file_per_table=on

##隔离级别
tx_isolation=READ-COMMITTED

innodb_flush_log_at_trx_commit=
sync_binlog=
```

