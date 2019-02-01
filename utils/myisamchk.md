# myisamchk

myisamchk获取数据库表信息，检查，修复，优化表。myisamchk只对myisam表,也可以使用CHECK TABLE和REPAIR TABLE来检查和修复myisam表。

 

myisamchk不支持分区表

 

在使用myisamchk修复表的时候，最好先做一个备份。在有些情况下，修复操作会导致数据丢失。可能的原因包含但是不限于文件系统错误。

使用[myisamchk](https://dev.mysql.com/doc/refman/5.7/en/myisamchk.html)的语法

```
shell> myisamchk [*options*] *tbl_name* ...
```

shell> myisamchk [*options*] *tbl_name* ...

myisamchk --silent --fast */path/to/datadir/\*/**.MYI

 

**重要***在运行**myisamchk*时，您必须确保没有其他程序正在使用这些表 。这样做的最有效方法是在运行[myisamchk时](https://dev.mysql.com/doc/refman/5.7/en/myisamchk.html)关闭MySQL服务器，或者锁定[myisamchk](https://dev.mysql.com/doc/refman/5.7/en/myisamchk.html)正在使用的所有表。否则，当您运行[myisamchk时](https://dev.mysql.com/doc/refman/5.7/en/myisamchk.html)，它可能会显示以下错误消息：warning: clients are using or haven't closed the table properly这意味着您正在尝试检查已被另一程序（如[mysqld](https://dev.mysql.com/doc/refman/5.7/en/mysqld.html)服务器）更新的表，该表 尚未关闭该文件，或者在没有正确关闭该文件的情况下死亡，这有时会导致损坏一个或多个 MyISAM表格。如果[mysqld](https://dev.mysql.com/doc/refman/5.7/en/mysqld.html)正在运行，则必须强制其使用刷新仍在内存中缓存的所有表修改[FLUSH TABLES](#flush-tables)。然后，您应确保在运行[myisamchk](https://dev.mysql.com/doc/refman/5.7/en/myisamchk.html)时没有人使用这些表格但是，避免此问题的最简单方法是使用 [CHECK TABLE](https://dev.mysql.com/doc/refman/5.7/en/check-table.html)而不是 [myisamchk](https://dev.mysql.com/doc/refman/5.7/en/myisamchk.html)来检查表  

 

myisam表维护也可以使用sql语句达到类似于myisamchk的效果：

- 要检查MyISAM表格，请使用 [CHECK TABLE](https://dev.mysql.com/doc/refman/5.7/en/check-table.html)。

- 要修复MyISAM表格，请使用 [REPAIR TABLE](https://dev.mysql.com/doc/refman/5.7/en/repair-table.html)。

- 要优化MyISAM表格，请使用 [OPTIMIZE TABLE](https://dev.mysql.com/doc/refman/5.7/en/optimize-table.html)。

- 要分析MyISAM表格，请使用 [ANALYZE TABLE](https://dev.mysql.com/doc/refman/5.7/en/analyze-table.html)。

这些语句可以直接使用或者使用[mysqlcheck](https://dev.mysql.com/doc/refman/5.7/en/mysqlcheck.html) 工具。

这些语句相对于 [myisamchk](https://dev.mysql.com/doc/refman/5.7/en/myisamchk.html) 的优点是：服务器做了所有的工作，如果使用myisamchk，必须确保服务器没有使用这些表

使用myisamchk来进行崩溃恢复。myisamchk可以解决数据破坏，但是如果频繁的数据破坏，需要找到原因。

 

 

 