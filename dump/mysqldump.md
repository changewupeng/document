# mysqldump



## 简介

mysqldump是mysql提供的一个逻辑备份工具，它可以将本地或远端的mysql实例数据生成对应的DDL和DML语句，存储在一个文件中，它可以完成全库备份和部分表备份。



## 基本操作

- 备份指定数据库的制定表的数据

  ```
  mysqldump [OPTIONS] database [tables]
  ```

- 备份多个数据库

  ```
  mysqldump [OPTIONS] --databases [OPTIONS] DB1 [DB2 DB3...]
  ```

- 备份全库

  ```
  mysqldump [OPTIONS] --all-databases [OPTIONS]
  ```





## 参数说明

mysqldump默认从以下文件中读取[mysqldump]信息

/etc/my.cnf /etc/mysql/my.cnf /usr/local/mysql/etc/my.cnf ~/.my.cnf 



| 参数                    | 简称 | 解释                                                         |
| ----------------------- | ---- | ------------------------------------------------------------ |
| --print-defaults        |      |                                                              |
| --no-defaults           |      | 除了login file,不读取任何默认的配置文件                      |
| --defaults-file=#       |      | 指定读取的配置文件                                           |
| --defaults-extra-file=# |      | 在全局文件被读取之后读取此文件                               |
|                         |      |                                                              |
| --login-path=#          |      | 读取login file的login-path信息                               |
| --all-databases         | -A   | 全库备份                                                     |
| --add-drop-database     |      | 在每个create database前面加上drop操作                        |
| --add-drop-table        |      | 在每个drop table操作前加上drop操作，默认开启，使用--skip-add-drop-table关闭 |
| --add-drop-trigger      |      | 在每个创建触发器的操作之前drop trigger                       |
| --add-locks             |      | 在insert操作之前加锁，默认是开启的，使用--skip-add-locks关闭 |
| --allow-keywords        |      | 允许字段名是关键字                                           |
| --comments              | -i   | 写入注释信息，默认开始，使用--skip-commnets关闭              |
| --compatible            |      | 设置备份数据的兼容性。默认导出的格式是mysql.合法的值是：ansi, mysql323, mysql40,postgresql, oracle, mssql, db2, maxdb, no_key_options,no_table_options, no_field_options，可以是多个值的组合，中间用逗号分割。需要mysql的版本是4.1以上。 |
| --compact               |      | 把一些冗长的注释信息删除掉                                   |
| --complete-insert       | -c   | 一个表导出一条insert语句。                                   |
| --compress              | -C   | Use compression in server/client protocol.                   |
| --create-options        | -a   | 包含所有的create信息，默认开始，使用--skip-create-options关闭 |
| --databases             | -B   | 导出一个或多个指定的数据库                                   |
| --delete-master-logs    |      | 在dump之后，删除二进制日志，此选项默认关闭，开启此选项会激活--master-data. |
|                         |      |                                                              |
|                         |      |                                                              |
|                         |      |                                                              |
|                         |      |                                                              |
|                         |      |                                                              |
|                         |      |                                                              |
|                         |      |                                                              |
|                         |      |                                                              |
|                         |      |                                                              |
|                         |      |                                                              |
|                         |      |                                                              |
|                         |      |                                                              |
|                         |      |                                                              |
|                         |      |                                                              |
|                         |      |                                                              |
|                         |      |                                                              |
|                         |      |                                                              |
|                         |      |                                                              |
|                         |      |                                                              |



```


  -K, --disable-keys  '/*!40000 ALTER TABLE tb_name DISABLE KEYS */; and
                      '/*!40000 ALTER TABLE tb_name ENABLE KEYS */; will be put
                      in the output.
                      (Defaults to on; use --skip-disable-keys to disable.)
  --dump-slave[=#]    This causes the binary log position and filename of the
                      master to be appended to the dumped data output. Setting
                      the value to 1, will printit as a CHANGE MASTER command
                      in the dumped data output; if equal to 2, that command
                      will be prefixed with a comment symbol. This option will
                      turn --lock-all-tables on, unless --single-transaction is
                      specified too (in which case a global read lock is only
                      taken a short time at the beginning of the dump - don't
                      forget to read about --single-transaction below). In all
                      cases any action on logs will happen at the exact moment
                      of the dump.Option automatically turns --lock-tables off.
  -E, --events        Dump events.
  -e, --extended-insert 
                      Use multiple-row INSERT syntax that include several
                      VALUES lists.
                      (Defaults to on; use --skip-extended-insert to disable.)
  --fields-terminated-by=name 
                      Fields in the output file are terminated by the given
                      string.
  --fields-enclosed-by=name 
                      Fields in the output file are enclosed by the given
                      character.
  --fields-optionally-enclosed-by=name 
                      Fields in the output file are optionally enclosed by the
                      given character.
  --fields-escaped-by=name 
                      Fields in the output file are escaped by the given
                      character.
  -F, --flush-logs    Flush logs file in server before starting dump. Note that
                      if you dump many databases at once (using the option
                      --databases= or --all-databases), the logs will be
                      flushed for each database dumped. The exception is when
                      using --lock-all-tables or --master-data: in this case
                      the logs will be flushed only once, corresponding to the
                      moment all tables are locked. So if you want your dump
                      and the log flush to happen at the same exact moment you
                      should use --lock-all-tables or --master-data with
                      --flush-logs.
  --flush-privileges  Emit a FLUSH PRIVILEGES statement after dumping the mysql
                      database.  This option should be used any time the dump
                      contains the mysql database and any other database that
                      depends on the data in the mysql database for proper
                      restore. 
  -f, --force         Continue even if we get an SQL error.
  -?, --help          Display this help message and exit.
  --hex-blob          Dump binary strings (BINARY, VARBINARY, BLOB) in
                      hexadecimal format.
  -h, --host=name     Connect to host.
  --ignore-error=name A comma-separated list of error numbers to be ignored if
                      encountered during dump.
  --ignore-table=name Do not dump the specified table. To specify more than one
                      table to ignore, use the directive multiple times, once
                      for each table.  Each table must be specified with both
                      database and table names, e.g.,
                      --ignore-table=database.table.
  --include-master-host-port 
                      Adds 'MASTER_HOST=<host>, MASTER_PORT=<port>' to 'CHANGE
                      MASTER TO..' in dump produced with --dump-slave.
  --insert-ignore     Insert rows with INSERT IGNORE.
  --lines-terminated-by=name 
                      Lines in the output file are terminated by the given
                      string.
  -x, --lock-all-tables 
                      Locks all tables across all databases. This is achieved
                      by taking a global read lock for the duration of the
                      whole dump. Automatically turns --single-transaction and
                      --lock-tables off.
  -l, --lock-tables   Lock all tables for read.
                      (Defaults to on; use --skip-lock-tables to disable.)
  --log-error=name    Append warnings and errors to given file.
  --master-data[=#]   This causes the binary log position and filename to be
                      appended to the output. If equal to 1, will print it as a
                      CHANGE MASTER command; if equal to 2, that command will
                      be prefixed with a comment symbol. This option will turn
                      --lock-all-tables on, unless --single-transaction is
                      specified too (in which case a global read lock is only
                      taken a short time at the beginning of the dump; don't
                      forget to read about --single-transaction below). In all
                      cases, any action on logs will happen at the exact moment
                      of the dump. Option automatically turns --lock-tables
                      off.
  --max-allowed-packet=# 
                      The maximum packet length to send to or receive from
                      server.
  --net-buffer-length=# 
                      The buffer size for TCP/IP and socket communication.
  --no-autocommit     Wrap tables with autocommit/commit statements.
  -n, --no-create-db  Suppress the CREATE DATABASE ... IF EXISTS statement that
                      normally is output for each dumped database if
                      --all-databases or --databases is given.
  -t, --no-create-info 
                      Don't write table creation info.
  -d, --no-data       No row information.
  -N, --no-set-names  Same as --skip-set-charset.
  --opt               Same as --add-drop-table, --add-locks, --create-options,
                      --quick, --extended-insert, --lock-tables, --set-charset,
                      and --disable-keys. Enabled by default, disable with
                      --skip-opt.
  --order-by-primary  Sorts each table's rows by primary key, or first unique
                      key, if such a key exists.  Useful when dumping a MyISAM
                      table to be loaded into an InnoDB table, but will make
                      the dump itself take considerably longer.
  -p, --password[=name] 
                      Password to use when connecting to server. If password is
                      not given it's solicited on the tty.
  -P, --port=#        Port number to use for connection.
  --protocol=name     The protocol to use for connection (tcp, socket, pipe,
                      memory).
  -q, --quick         Don't buffer query, dump directly to stdout.
                      (Defaults to on; use --skip-quick to disable.)
  -Q, --quote-names   Quote table and column names with backticks (`).
                      (Defaults to on; use --skip-quote-names to disable.)
  --replace           Use REPLACE INTO instead of INSERT INTO.
  -r, --result-file=name 
                      Direct output to a given file. This option should be used
                      in systems (e.g., DOS, Windows) that use carriage-return
                      linefeed pairs (\r\n) to separate text lines. This option
                      ensures that only a single newline is used.
  -R, --routines      Dump stored routines (functions and procedures).
  --set-charset       Add 'SET NAMES default_character_set' to the output.
                      (Defaults to on; use --skip-set-charset to disable.)
  --set-gtid-purged[=name] 
                      Add 'SET @@GLOBAL.GTID_PURGED' to the output. Possible
                      values for this option are ON, OFF and AUTO. If ON is
                      used and GTIDs are not enabled on the server, an error is
                      generated. If OFF is used, this option does nothing. If
                      AUTO is used and GTIDs are enabled on the server, 'SET
                      @@GLOBAL.GTID_PURGED' is added to the output. If GTIDs
                      are disabled, AUTO does nothing. If no value is supplied
                      then the default (AUTO) value will be considered.
  --single-transaction 
                      Creates a consistent snapshot by dumping all tables in a
                      single transaction. Works ONLY for tables stored in
                      storage engines which support multiversioning (currently
                      only InnoDB does); the dump is NOT guaranteed to be
                      consistent for other storage engines. While a
                      --single-transaction dump is in process, to ensure a
                      valid dump file (correct table contents and binary log
                      position), no other connection should use the following
                      statements: ALTER TABLE, DROP TABLE, RENAME TABLE,
                      TRUNCATE TABLE, as consistent snapshot is not isolated
                      from them. Option automatically turns off --lock-tables.
  --dump-date         Put a dump date to the end of the output.
                      (Defaults to on; use --skip-dump-date to disable.)
  --skip-opt          Disable --opt. Disables --add-drop-table, --add-locks,
                      --create-options, --quick, --extended-insert,
                      --lock-tables, --set-charset, and --disable-keys.
  -S, --socket=name   The socket file to use for connection.
  --secure-auth       Refuse client connecting to server if it uses old
                      (pre-4.1.1) protocol. Deprecated. Always TRUE
  --ssl-mode=name     SSL connection mode.
  --ssl               Deprecated. Use --ssl-mode instead.
                      (Defaults to on; use --skip-ssl to disable.)
  --ssl-verify-server-cert 
                      Deprecated. Use --ssl-mode=VERIFY_IDENTITY instead.
  --ssl-ca=name       CA file in PEM format.
  --ssl-capath=name   CA directory.
  --ssl-cert=name     X509 cert in PEM format.
  --ssl-cipher=name   SSL cipher to use.
  --ssl-key=name      X509 key in PEM format.
  --ssl-crl=name      Certificate revocation list.
  --ssl-crlpath=name  Certificate revocation list path.
  --tls-version=name  TLS version to use, permitted values are: TLSv1, TLSv1.1
  -T, --tab=name      Create tab-separated textfile for each table to given
                      path. (Create .sql and .txt files.) NOTE: This only works
                      if mysqldump is run on the same machine as the mysqld
                      server.
  --tables            Overrides option --databases (-B).
  --triggers          Dump triggers for each dumped table.
                      (Defaults to on; use --skip-triggers to disable.)
  --tz-utc            SET TIME_ZONE='+00:00' at top of dump to allow dumping of
                      TIMESTAMP data when a server has data in different time
                      zones or data is being moved between servers with
                      different time zones.
                      (Defaults to on; use --skip-tz-utc to disable.)
  -u, --user=name     User for login if not current user.
  -v, --verbose       Print info about the various stages.
  -V, --version       Output version information and exit.
  -w, --where=name    Dump only selected records. Quotes are mandatory.
  -X, --xml           Dump a database as well formed XML.
  --plugin-dir=name   Directory for client-side plugins.
  --default-auth=name Default authentication client-side plugin to use.
  --enable-cleartext-plugin 
                      Enable/disable the clear text authentication plugin.


```

