# 多数据源复制

- 定义：多数据源可以让一个slave同时从多个master上接收事务。
- 用途：将多个server的数据备份到一个服务器上.
- 限制： 多数据源复制不会进行冲突检查

## 配置多数据源

- 复制的配置信息需要配置为存放在表中（默认是存放在文件中)

  ```
  master-info-repository=TABLE
  relay-log-info-repository=TABLE 
  ```

  

- 使用change master to ...FOR CHANNEL *channel* 

  ```
  CHANGE MASTER TO MASTER_HOST='master1', MASTER_USER='rpl', MASTER_PORT=3451, MASTER_PASSWORD='', MASTER_AUTO_POSITION = 1 FOR CHANNEL 'master-1';
  ```

  ```
  CHANGE MASTER TO MASTER_HOST='master1', MASTER_USER='rpl', MASTER_PORT=3451, MASTER_PASSWORD='' ,MASTER_LOG_FILE='master1-bin.000006', MASTER_LOG_POS=628 FOR CHANNEL 'master-1';
  ```

  

- 开启多数据源的slave

  ```
  ##开启所有的
  START SLAVE thread_types;
  ##开启channel1的复制线程
  START SLAVE thread_types FOR CHANNEL channel;
  ```

- 关闭多数据源的slave

  ```
  ## 关闭所有的
  STOP SLAVE thread_types;
  ##关闭channel1的复制线程
  STOP SLAVE thread_types FOR CHANNEL channel;
  ```

## 多数据源的监控

- 通过replication_connection_status
- SHOW SLAVE STATUS FOR CHANNEL *channel* ，如果FOR CHANNEL *channel*不添加，那么是查询所有的channel