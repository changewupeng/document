# 基于GTIDs

GTID:global transaction identifier

GTID是一个和master上所提交的每一个提交的事务关联的唯一标识符。这个唯一不仅仅是在产生这个事务的机器上唯一，在一个给定的复制拓扑里面也是唯一。



当一个事物在master上被提交，那么它就和一个GTID关联，以写进binlog为准



客户端事物被确保拥有不间断的单调递增的数字。如果一个事物没有被写进binlog，就不会和GTID关联



复制的事务获取来自master的事物相同的GTID，在复制的事务开始执行之前，GTID就被展现，就算slave的复制事务没有被写进binlog或者被slave过滤掉，GTID也会被持久化



mysql.gitid_executed表被用来存储mysql server已经执行过的所有事物，除了那些被存储在currently active binary log file

auto-skip功能确保在master上提交的事务在每个slave上至多执行一次，以此来保证一致性。一旦一个绑定GTID的事物在给定的服务器上被提交，任何随之而来的绑定相同的GTID的事务都会被服务器忽略，没有错误抛出也没有事物执行



如果一个给定的GTID是事务开始执行但是还没有提交或者回滚。任何尝试开始一个并发的有相同gtid的是事务都会被阻塞。mysql server既不会执行并发的事务，也不会将控制权交给客户端。一旦事物提交或者回滚，被堵塞的事务将继续。

- 事务被提交，那么被堵塞的功能就会被auto-skip忽视掉
- 事务被回滚，那么一个被堵塞的事务继续执行，其它的继续堵塞

GTID是以一组被冒号分隔的坐标展现：

```
GTID = source_id:transaction_id
```

- source_id是mysql的server-id,存储在data目录下的my.auto里面
- transaction_id是大于0的正整数



When binary logging is enabled, the `mysql.gtid_executed` table does not hold a complete record of the GTIDs for all executed transactions. That information is provided by the global value of the [`gtid_executed`](https://dev.mysql.com/doc/refman/5.7/en/replication-options-gtids.html#sysvar_gtid_executed) system variable. Always use `@@GLOBAL.gtid_executed`, which is updated after every commit, to represent the GTID state for the MySQL server, and do not query the `mysql.gtid_executed` table.



## GTID的生命周期

1. 当一个事务在master上执行并提交，这个事务和一个新生成的GTID关联，GTID被写进binlog.
2. 如果一个GTID和事务关联，那么GTID会被自动持久化到binlog(在事务的开头)，不管何时binlog旋转护着实例关闭，server将所有以前已经写进binlog的日志的GTID写进mysql.gtid_executed。
3. 当一个GTID和事务关联，GTID被立即写进gtid_executed系统变量（@@GLOBAL.gtid_executed），这个GTID set包含了所有已经提交的GTID事务，将在复制的时候被使用。在master上，系统变量gtid_executed记录这所有的已提交的事务，但是mysql.gtid_executed表里面不是所有的，大部分最近提交的历史GTID事务还在当前的binlog日志中。
4. 在binlog数据被转移到slave，并被存储在relaylog中，slave读取GTID，并将它的[`gtid_next`](https://dev.mysql.com/doc/refman/5.7/en/replication-options-gtids.html#sysvar_gtid_next) 设置为这个GTID, git_next变量是告诉slave下一个必须使用这个GTID被记录。

5. slave必须确保没有其它线程掌握gtid_next的数值，在执行事务之前，slave不仅需要确保这个GTID没有被使用，也需要确保没有其它线程已经读取了这个GTID，但是还没有提交关联的事务。如果有多个客户端尝试并发执行相同的GTID的事务，slave仅仅让其中一个执行。gt_owned变量（@@GLOBAL.gtid_owned）展示了每个当前在使用的GTID以及使用的线程id.
6. 如果GTID没有被使用，slave执行这个事务。因为gtid_next是已经被master产生的关联的GTID，slave不会尝试产生一个新的GTID,只是会将这个GTID存储在gtid_next.

7. 如果slave的binlog可用，GTID在提交的时候自动写入binlog的事务的开头。在binlog旋转和实例关闭的时候，server会将所有写进binlog的GTID写进mysql.gtid_executed

8. 如果binlog在slave上不可用，GTID被直接mysql.gtid_executed。在这种情况下， mysql.gtid_executed记录着所有在slave上执行的事务。

   注意：mysql5.7中，读与DML操作，insert GTID进入表是自动的，但是对于DML操作不是，如果在一个事务执行了DDL操作之后服务器意外退出，可能会导致不一致性。在mysql8.0中不存在这个问题

9. 在slave这边的 是复制事务提交后的很短时间，GTID被写进gtid_executed这个变量里面。

   

master中gtid_executed变量记录着所有的GTID事务，在slave中，如果binlog不可用，mysql.gtid_executed 记录这所有的GTID事务，如果binlog可用，gtid_executed变量记录着所有的GTID事务。



## GTID自动定位

GTID需要确定在主从中开始、停止和恢复数据流的点，当使用GTIDs,从master获取数据只需要直接获取数据流

在change master to的配置中，不需要MASTER_LOG_FILE和MASTER_LOG_POS，需要确保MASTER_AUTO_POSITION默认不可用，如果是多数据源的复制，需要设置每个数据源的replication channel。

如果将MASTER_AUTO_POSITION设置为不可用将会切换到file-based replication

当slave设置[`GTID_MODE=ON`](https://dev.mysql.com/doc/refman/5.7/en/replication-options-gtids.html#sysvar_gtid_mode), `ON_PERMISSIVE,` or `OFF_PERMISSIVE` ,并且MASTER_AUTO_POSITION可用



In the initial handshake, the slave sends a GTID set containing the transactions that it has already received, committed, or both. This GTID set is equal to the union of the set of GTIDs in the[`gtid_executed`](https://dev.mysql.com/doc/refman/5.7/en/replication-options-gtids.html#sysvar_gtid_executed) system variable (`@@GLOBAL.gtid_executed`), and the set of GTIDs recorded in the Performance Schema[`replication_connection_status`](https://dev.mysql.com/doc/refman/5.7/en/replication-connection-status-table.html) table as received transactions (the result of the statement `SELECT RECEIVED_TRANSACTION_SET FROM PERFORMANCE_SCHEMA.replication_connection_status`).



The master responds by sending all transactions recorded in its binary log whose GTID is not included in the GTID set sent by the slave. This exchange ensures that the master only sends the transactions with a GTID that the slave has not already received or committed.



### 异常情况处理

1. 如果任何需要被master传递的事务已经从master的binlog清除，或者已经被添加进gtid_purged。master将会传递一个错误**ER_MASTER_HAS_PURGED_REQUIRED_GTIDS**给slave,并且复制不会开始。slave不会自动恢复，因为它需要的历史事务已经被清除。

   解决办法：从其他源来复制这个丢失的数据，或者用一个新的备份重新创建一个slave.

   预防办法：将master的binlog过期时间稍微设置长一些来确保此类情况不再发生。

2. 如果在事务交流的过程中发现slave收到或提交了来自master的GTID。但是master本身没有记录，master将会向slave发送一个**ER_SLAVE_HAS_MORE_GTIDS_THAN_MASTER**的错误，并且复制不会开始。

   发生的原因：master没有设置sync_binlog=1，并且经历了停电，操作系统的崩溃，丢失了已经提交了但还没有同不到binlog日志的事务，但是slave已经收到了。

If during the exchange of transactions it is found that the slave has received or committed transactions with the master's UUID in the GTID, but the master itself does not have a record of them, the master sends the error**ER_SLAVE_HAS_MORE_GTIDS_THAN_MASTER** to the slave and replication does not start. This situation can occur if a master that does not have [`sync_binlog=1`](https://dev.mysql.com/doc/refman/5.7/en/replication-options-binary-log.html#sysvar_sync_binlog) set experiences a power failure or operating system crash, and loses committed transactions that have not yet been synchronized to the binary log file, but have been received by the slave. The master and slave can diverge if any clients commit transactions on the master after it is restarted, which can lead to the situation where the master and slave are using the same GTID for different transactions. The correct approach to recover from this situation is to check manually whether the master and slave have diverged. If the same GTID is now in use for different transactions, you either need to perform manual conflict resolution for individual transactions as required, or remove either the master or the slave from the replication topology. If the issue is only missing transactions on the master, you can make the master into a slave instead, allow it to catch up with the other servers in the replication topology, and then make it a master again if needed.





## 配置基于GTID的复制

## 1. 同步服务器

这一步骤只有在没有使用GTID复制的时候才需要，对于新服务器，转步骤3.通过在每个服务器上设置read_only=on来确保当前服务器是只读。

```
mysql> SET @@GLOBAL.read_only = ON;
```

等到所有在运行的事务提交或者回滚，然后让slave追赶上master.在继续下一步之前确保slave已经执行了有的更新非常重要。

注意：没有GTID的事务不能被用于开启了GTID的服务器。需要确保没有GTID的事物不能存在与GTID的拓扑结构中。

## 2. 关闭mysql实例

使用shutdown命令

## 3. 启动master和slave确保GTID可用

```
##开启GTID
gitid_mode=on
##确保只有对于GTID复制安全的事务才会被日志记录
enforce-gtid-consistency=true
```

另外，在配置好slave之前启动slave实例，需要添加--skip-slave-start选项，避免实例启动的时候，slave自动开启

对于slave而言，binlog不是必须的，如果不需要开启binlog，slave设置 [`--skip-log-bin`](https://dev.mysql.com/doc/refman/5.7/en/replication-options-binary-log.html#option_mysqld_log-bin) 和  [`--skip-log-slave-updates`](https://dev.mysql.com/doc/refman/5.7/en/replication-options-slave.html#option_mysqld_log-slave-updates)选项即可。

### 4.**Configure the slave to use GTID-based auto-positioning**

```mysql
mysql> CHANGE MASTER TO
     >     MASTER_HOST = host,
     >     MASTER_PORT = port,
     >     MASTER_USER = user,
     >     MASTER_PASSWORD = password,
     >     MASTER_AUTO_POSITION = 1;
```

### 5.做一个新的备份

（这一步看情况而定）

### 6.**Start the slave and disable read-only mode.**

```
start slave
SET @@GLOBAL.read_only = OFF;
```





# 使用GTID进行故障切换和水平扩展

The easiest way to reproduce all identifiers and transactions on a new server is to make the new server into the slave of a master that has the entire execution history, and enable global transaction identifiers on both servers. See[Section 16.1.3.4, “Setting Up Replication Using GTIDs”](https://dev.mysql.com/doc/refman/5.7/en/replication-gtids-howto.html), for more information.

Once replication is started, the new server copies the entire binary log from the master and thus obtains all information about all GTIDs.

This method is simple and effective, but requires the slave to read the binary log from the master; it can sometimes take a comparatively long time for the new slave to catch up with the master, so this method is not suitable for fast failover or restoring from backup. This section explains how to avoid fetching all of the execution history from the master by copying binary log files to the new server.



## 简单复制

对于一个有完整执行记录历史的master而言，将一个新的server设置成slave最见简单的办法就是在复制开启之后，slave从master获取binlog日志并执行，因此slave会获取所有的GTID信息

优点：简单，有效

缺点：要求slave从master读取binlog日志，可能会花费比较长的时间。不适用于快速故障切换和从备份数据中恢复。



## 向slave中拷贝数据和事务

将一个源服务器上的数据快照，binlog和全局事务信息导入一个新slave。源服务器既可以是master也可以是slave.

### 备份数据

1. 使用mysqldump从源服务器上备份数据。有以下几个参数必须设置
   - --master-data:包含一个binlog日志信息的change master to申明。
   - --set-gtid-purged=[auto|on]:包含已经执行过的事务信息。

2. 也可以将目标服务器和源服务器都停止。开启slave的gtid_mode,将源服务器的datadir里面的数据拷贝一份到目标服务器的datadir，然后启动slave即可。

### 完整的事务历史

前提： 拥有完整的事务历史在binlog中。（@@GLOBAL.gtid_purged为空）

使用binlog从master上读取日志来本机执行。或者将master上的binlog日志拷贝下来，在本机执行。

