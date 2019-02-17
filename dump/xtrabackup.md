# xtrabackup

## Xtrabackup介绍

Xtrabackup是由percona开源的免费数据库热备份软件，它能对InnoDB数据库和XtraDB存储引擎的数据库非阻塞地备份（对于MyISAM的备份同样需要加表锁）；mysqldump备份方式是采用的逻辑备份，其最大的缺陷是备份和恢复速度较慢，如果数据库大于50G，mysqldump备份就不太适合。

Xtrabackup安装完成后有4个可执行文件，其中2个比较重要的备份工具是innobackupex、xtrabackup

1. xtrabackup 是专门用来备份InnoDB表的，和mysql server没有交互；

2. innobackupex 是一个封装xtrabackup的Perl脚本，支持同时备份innodb和myisam，但在对myisam备份时需要加一个全局的读锁。

3. xbcrypt 加密解密备份工具

4. xbstream 流传打包传输工具，类似tar

 

## Xtrabackup优点

1. 备份速度快，物理备份可靠

2. 备份过程不会打断正在执行的事务（无需锁表）

3. 能够基于压缩等功能节约磁盘空间和流量

4. 自动备份校验

5. 还原速度快

6. 可以流传将备份传输到另外一台机器上

7. 在不增加服务器负载的情况备份数据

 

## Xtrabackup备份原理

备份开始时首先会开启一个后台检测进程，实时检测mysq redo的变化，一旦发现有新的日志写入，立刻将日志记入后台日志文件xtrabackup_log中，之后复制innodb的数据文件一系统表空间文件ibdatax，复制结束后，将执行flush tables with readlock,然后复制.frm MYI MYD等文件，最后执行unlock tables,最终停止xtrabackup_log



##  xtrabackup安装

到[percona官网](https://www.percona.com/software/mysql-database/percona-xtrabackup)上去下载对应平台的最新版本即可。



## 使用xtrabackup的前置条件


- 在执行备份的时候，执行程序的用户需要具备datadir的读写执行权限

- 数据库用户需要以下的权限

  - **RELOAD**和**LOCK TABLES** (除非在备份的时候--no-lock选项被申明) ，为了优先FLUSH TABLES WITH
    READ LOCK来复制文件
  - **REPLICATION CLIENT **,获取binlog的位置
  - **CREATE TABLESPACE** ,为了导入表，在恢复单个表的时候需要
  - **SUPER** 在replication环境中，开启和关闭slave线程。

  脚本如下：

  ```
  mysql> CREATE USER ’bkpuser’@’localhost’ IDENTIFIED BY ’s3cret’;
  mysql> GRANT RELOAD, LOCK TABLES, REPLICATION CLIENT ON *.* TO ’bkpuser’@’localhost’;
  mysql> FLUSH PRIVILEGES;
  ```

  

## 全备份：

```
 innobackupex --defaults-file=  --login-path=  /path/to/BACKUP-DIR/
```

当最后一条信息类似于：

```
innobackupex: Backup created in directory ’/path/to/BACKUP-DIR/2013-03-25_00-00-09’
innobackupex: MySQL binlog position: filename ’mysql-bin.000003’, position 1946
111225 00:00:53 innobackupex: completed OK!
```

的时候说明备份完毕。

**备注：**

  **1.--defaults-file是为了指定mysql的配置文件my.cnf的位置**

  **2.--login-path是为了使用login的登录方式，在备份的过程中，innobackupex需要登录mysql**

  **3.会在/path/to/BACKUP-DIR/目录下生成一个时间戳目录，里面放着全备份的文件。可以使用--no-timestamp不生成时间戳目录。**



备份的参数说明：

```
--compress：该选项表示压缩innodb数据文件的备份。
--compress-threads：该选项表示并行压缩worker线程的数量。
--compress-chunk-size：该选项表示每个压缩线程worker buffer的大小，单位是字节，默认是64K。
--encrypt：该选项表示通过ENCRYPTION_ALGORITHM的算法加密innodb数据文件的备份，目前支持的算法有ASE128,AES192,AES256。
--encrypt-threads：该选项表示并行加密的worker线程数量。
--encrypt-chunk-size：该选项表示每个加密线程worker buffer的大小，单位是字节，默认是64K。
--encrypt-key：该选项使用合适长度加密key，因为会记录到命令行，所以不推荐使用。
--encryption-key-file：该选项表示文件必须是一个简单二进制或者文本文件，加密key可通过以下命令行命令生成：openssl rand -base64 24。
--include：该选项表示使用正则表达式匹配表的名字[db.tb]，要求为其指定匹配要备份的表的完整名称，即databasename.tablename。
--user：该选项表示备份账号。
--password：该选项表示备份的密码。
--port：该选项表示备份数据库的端口。
--host：该选项表示备份数据库的地址。
--databases：该选项接受的参数为数据名，如果要指定多个数据库，彼此间需要以空格隔开；如："xtra_test dba_test"，同时，在指定某数据库时，也可以只指定其中的某张表。如："mydatabase.mytable"。该选项对innodb引擎表无效，还是会备份所有innodb表。此外，此选项也可以接受一个文件为参数，文件中每一行为一个要备份的对象。
--tables-file：该选项表示指定含有表列表的文件，格式为database.table，该选项直接传给--tables-file。
--socket：该选项表示mysql.sock所在位置，以便备份进程登录mysql。
--no-timestamp：该选项可以表示不要创建一个时间戳目录来存储备份，指定到自己想要的备份文件夹。
--ibbackup：该选项指定了使用哪个xtrabackup二进制程序。IBBACKUP-BINARY是运行percona xtrabackup的命令。这个选项适用于xtrbackup二进制不在你是搜索和工作目录，如果指定了该选项,innoabackupex自动决定用的二进制程序。
--slave-info：该选项表示对slave进行备份的时候使用，打印出master的名字和binlog pos，同样将这些信息以change master的命令写入xtrabackup_slave_info文件。可以通过基于这份备份启动一个从库。
--safe-slave-backup：该选项表示为保证一致性复制状态，这个选项停止SQL线程并且等到show status中的slave_open_temp_tables为0的时候开始备份，如果没有打开临时表，bakcup会立刻开始，否则SQL线程启动或者关闭知道没有打开的临时表。如果slave_open_temp_tables在--safe-slave-backup-timeount（默认300秒）秒之后不为0，从库sql线程会在备份完成的时候重启。
--rsync：该选项表示通过rsync工具优化本地传输，当指定这个选项，innobackupex使用rsync拷贝非Innodb文件而替换cp，当有很多DB和表的时候会快很多，不能--stream一起使用。
--kill-long-queries-timeout：该选项表示从开始执行FLUSH TABLES WITH READ LOCK到kill掉阻塞它的这些查询之间等待的秒数。默认值为0，不会kill任何查询，使用这个选项xtrabackup需要有Process和super权限。
--kill-long-query-type：该选项表示kill的类型，默认是all，可选select。
--ftwrl-wait-threshold：该选项表示检测到长查询，单位是秒，表示长查询的阈值。
--ftwrl-wait-query-type：该选项表示获得全局锁之前允许那种查询完成，默认是ALL，可选update。
--galera-info：该选项表示生成了包含创建备份时候本地节点状态的文件xtrabackup_galera_info文件，该选项只适用于备份PXC。
--stream：该选项表示流式备份的格式，backup完成之后以指定格式到STDOUT，目前只支持tar和xbstream。
--defaults-file：该选项指定了从哪个文件读取MySQL配置，必须放在命令行第一个选项的位置。
--defaults-extra-file：该选项指定了在标准defaults-file之前从哪个额外的文件读取MySQL配置，必须在命令行的第一个选项的位置。一般用于存备份用户的用户名和密码的配置文件。
----defaults-group：该选项表示从配置文件读取的组，innobakcupex多个实例部署时使用。
--no-lock：该选项表示关闭FTWRL的表锁，只有在所有表都是Innodb表并且不关心backup的binlog pos点，如果有任何DDL语句正在执行或者非InnoDB正在更新时（包括mysql库下的表），都不应该使用这个选项，后果是导致备份数据不一致，如果考虑备份因为获得锁失败，可以考虑--safe-slave-backup立刻停止复制线程。
--tmpdir：该选项表示指定--stream的时候，指定临时文件存在哪里，在streaming和拷贝到远程server之前，事务日志首先存在临时文件里。在 使用参数stream=tar备份的时候，你的xtrabackup_logfile可能会临时放在/tmp目录下，如果你备份的时候并发写入较大的话 xtrabackup_logfile可能会很大(5G+)，很可能会撑满你的/tmp目录，可以通过参数--tmpdir指定目录来解决这个问题。
--history：该选项表示percona server 的备份历史记录在percona_schema.xtrabackup_history表。
--incremental：该选项表示创建一个增量备份，需要指定--incremental-basedir。
--incremental-basedir：该选项表示接受了一个字符串参数指定含有full backup的目录为增量备份的base目录，与--incremental同时使用。
--incremental-dir：该选项表示增量备份的目录。
--incremental-force-scan：该选项表示创建一份增量备份时，强制扫描所有增量备份中的数据页。
--incremental-lsn：该选项表示指定增量备份的LSN，与--incremental选项一起使用。
--incremental-history-name：该选项表示存储在PERCONA_SCHEMA.xtrabackup_history基于增量备份的历史记录的名字。Percona Xtrabackup搜索历史表查找最近（innodb_to_lsn）成功备份并且将to_lsn值作为增量备份启动出事lsn.与innobackupex--incremental-history-uuid互斥。如果没有检测到有效的lsn，xtrabackup会返回error。
--incremental-history-uuid：该选项表示存储在percona_schema.xtrabackup_history基于增量备份的特定历史记录的UUID。
--close-files：该选项表示关闭不再访问的文件句柄，当xtrabackup打开表空间通常并不关闭文件句柄目的是正确的处理DDL操作。如果表空间数量巨大，这是一种可以关闭不再访问的文件句柄的方法。使用该选项有风险，会有产生不一致备份的可能。
--compact：该选项表示创建一份没有辅助索引的紧凑的备份。
--throttle：该选项表示每秒IO操作的次数，只作用于bakcup阶段有效。apply-log和--copy-back不生效不要一起用。
```





## 全备份prepare

一般情况下,在备份完成后，数据尚且不能用于恢复操作，因为备份的数据中可能会包含尚未提交的事务或已经提交但尚未同步至数据文件中的事务。因此，此时数据 文件仍处理不一致状态。--apply-log的作用是通过回滚未提交的事务及同步已经提交的事务至数据文件使数据文件处于一致性状态。

**语法**

```
innobackupex --apply-log [--use-memory=B]
             [--defaults-file=MY.CNF]
             [--export] [--redo-only] [--ibbackup=IBBACKUP-BINARY]
             BACKUP-DIR
```



**参数说明：**

```
--apply-log：该选项表示同xtrabackup的--prepare参数,一般情况下,在备份完成后，数据尚且不能用于恢复操			作，因为备份的数据中可能会包含尚未提交的事务或已经提交但尚未同步至数据文件中的事务。因此，此时			  数据 文件仍处理不一致状态。--apply-log的作用是通过回滚未提交的事务及同步已经提交的事务至数			据文件使数据文件处于一致性状态。

--use-memory：该选项表示和--apply-log选项一起使用，prepare 备份的时候，xtrabackup做crash 			   recovery分配的内存大小，单位字节。也可(1MB,1M,1G,1GB)，推荐1G。

--defaults-file：该选项指定了从哪个文件读取MySQL配置，必须放在命令行第一个选项的位置。

--export：这个选项表示开启可导出单独的表之后再导入其他Mysql中。

--redo-only：这个选项在prepare base full backup，往其中merge增量备份（但不包括最后一个）时候使用。
```

**一般来说，全库备份的prepare,只需要使用apply-log即可。**

```
innobackupex --apply-log  /home/mysql/backup/2019-02-15_23-04-13
```



## restore a full back

copy-back选项，将备份文件还原到datadir中

```
 innobackupex --copy-back /path/to/BACKUP-DIR
```

备注：

- datadir必须是空的innobackupex --copy-back选项不会拷贝datadir已经存在的文件
- 被恢复的instance必须是关闭的
- 在启动数据库实例之前，需要chown -R mysql:mysql datadir



## 增量备份

在每次备份中间不是所有的信息都改变，使用增量备份的策略，可以节省存储和带宽。

原理：每个innoDB页都有一个LSN(log sequence number,作为整个数据库的版本编号)，每次数据库发生改变，LSN增加。增量备份就是拷贝在指定LSN之后的所有的innoDB页。一旦所有的页按照各自的顺序被放置在一起，应用日志将会产生一个进程产生最近一次备份以来的数据。



步骤：

- 先做一个全量备份，产生的备份文件BASE-DIR=/home/mysql/backup/2019-02-15_23-04-13

  ```
  #做一个全量备份
  innobackupex --defaults-file=/home/mysql/scripts/mysql_data3306/my.cnf --login-path=local3306  /home/mysql/backup/
  ```

  如果此时检查BASE-DIR的xtrabackup_checkpoints，那么可以发现如下：

  ```
  backup_type = full-prepared
  from_lsn = 0
  to_lsn = 14557613182
  ```

- 在下一次做增量备份的时候，有如下两种方式

  - 基于上一次的BASEDIR

    ```
    innobackupex --defaults-file=/home/mysql/scripts/mysql_data3306/my.cnf --login-path=local3306 --incremental --incremental-basedir=/home/mysql/backup/2019-02-15_23-04-13   /home/mysql/backup/
    ```

  - 基于上一次备份的LSN

    ```
    ## 基于lsn的增量备份
    innobackupex --defaults-file=/home/mysql/scripts/mysql_data3306/my.cnf --login-path=local3306 --incremental --incremental-lsn=14557613182 /home/mysql/backup/
    ```

  如果此时检查BASE-DIR的xtrabackup_checkpoints，那么可以发现如下：

  ```
  backup_type = incremental
  from_lsn = 14557613182
  to_lsn = 14557613632
  ```

  

- 在下一次做增量备份的incremental-basedir就是上一次增量备份的BASE-DIR

  **注意： 增量备份只影响innoDB或者xtraDB的表，其他引擎的表在增量备份的时候，将会完全复制**



## 增量备份prepare

增量备份的prepare和全量有一些不太一样。

直接使用*--redo-only*选项。redo-only表示进行准备（应用日志）工作时，只进行redo操作，只会重做已提交但未应用的事务，不会回滚未提交的事务。原因是后面还有个增量备份，未提交的可能在后面增量备份时进行提交。



- 先对全量部分做准备工作

  ```
  innobackupex --apply-log --redo-only BASE-DIR
  ```

- 对增（差）量备份做准备工作

  ```
  innobackupex --apply-log [--redo-only] /PATH/TO/BACKUP/dir-quan --incremental-dir= /PATH/TO/BACKUP/dir-zeng
  ```

  

  --redo-only：**若只有一个增量备份或是最后那个增量备份文件，那么不需要这个选项**
  --incremental-dir=：此选项对应的目录为增量备份文件的目录

  **在对增量备份部分prepare的时候，会将增量备份部分合并到全量备份中去。**

- 一旦所有的增量备份都被合并到全量备份中，此时可以prepare全量备份，回滚未提交的事务。

  ```
  innobackupex --apply-log --use-memory=4G /path/to/BACKUP-DIR
  ```

  

  ```
  ##增量prepare的时候，prepare全量部分
  innobackupex --defaults-file=/home/mysql/scripts/mysql_data3306/my.cnf --login-path=local3306 --apply-log --redo-only  /home/mysql/backup/2019-02-15_23-04-13
  
  ##增量prepare的时候，prepare增量部分
  innobackupex --defaults-file=/home/mysql/scripts/mysql_data3306/my.cnf --login-path=local3306 --apply-log --redo-only  /home/mysql/backup/2019-02-15_23-04-13  --incremental-dir=/home/mysql/backup/2019-02-16_00-15-46
  ```



## 增量备份的恢复

在对增量备份进行prepare之后，basedir就相当于一次全量，此时按照全量备份的还原即可。





## 增量备份使用xbstream

- 先做一个全量备份

  ```
  innobackupex /data/backups
  ```

- 使用stream做一个全量备份

  ```
  innobackupex --incremental --incremental-lsn=LSN-number --stream=xbstream ./ > incremental.xbstream
  ```

- 解压一个xbstream

  ```
  xbstream -x < incremental.xbstream
  ```

- 做一个本地备份并且传递到远端并且解压

  ```
  innobackupex --incremental --incremental-lsn=LSN-number --stream=xbstream ./ | /
  ssh user@hostname " cat - | xbstream -x -C > /backup-dir/"
  ```



## 部分备份

*Percona XtraBackup*支持部分备份，可以只备份一些特定的表和数据库。当备份多个表时，需要用空格分开。*innodb_file_per_table*必须可用。

**唯一的警告是不要copy-back已经prepare后的部分备份，而应是导入表。**

### 创建部分备份

有三种方式来指定部分的库表。分别是正则表达式 (--include)，在一个文件中枚举表（--tables-file）或者提供一个数据库的列表(--databases)

- include

  ```
  innobackupex --defaults-file=/home/mysql/scripts/mysql_data3306/my.cnf --login-path=local3306 --include=’^tsintergy_pmss[.]sl_base_line’ /home/mysql/backup/
  ```

- --tables-file

  ```
  mysql@wupeng:~/backup$ echo "tsintergy_jzzh.sys_table_info" >> tables.txt
  mysql@wupeng:~/backup$ echo "tsintergy_pmss.sl_base_net" >> tables.txt
  mysql@wupeng:~/backup$ echo "tsintergy_pmss.sl_base_line" >> tables.txt
  
  
  mysql@wupeng:~/backup$ innobackupex --defaults-file=/home/mysql/scripts/mysql_data3306/my.cnf --login-path=local3306 --tables-file=/home/mysql/backup/tables.txt  /home/mysql/backup/
  ```

- --databases

  ```
  mysql@wupeng:~/backup$ innobackupex --defaults-file=/home/mysql/scripts/mysql_data3306/my.cnf --login-path=local3306 --databses="tsintergy_pmss tsintergy_jzzh.sys_table_info mysql "  /home/mysql/backup/
  ```

### prepare部分备份

操作：使用--apply-log和--export选项

```
innobackupex --defaults-file=/home/mysql/scripts/mysql_data3306/my.cnf --apply-log --export /home/mysql/backup/2019-02-16_22-28-25
```



## 时间点回复

原理：可以使用innobackupex和binlog将数据库恢复到一个具体的时刻,innobackupex备份的时候，包含了binlog的位置。只要有全量备份加上binlog的增量备份即可。