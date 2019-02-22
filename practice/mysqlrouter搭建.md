# mysql router

MySQL Router是MySQL官方提供的一个轻量级中间件，可以在应用程序与MySQL服务器之间提供透明的路由方式。主要用以解决MySQL主从库集群的高可用、负载均衡、易扩展等问题。Router可以与MySQL Fabric无缝连接，允许Fabric存储和管理用于路由的高可用数据库服务器组，使管理MySQL服务器组更加简单。

        MySQL Router是一个可执行文件，可以与应用程序在同一平台上运行，也可以单独部署。虽然MySQL Router是InnoDB Cluster（MySQL 7.X）的一部分，MySQL 5.6 等版本数据库仍然可以使用Router作为其中间代理层。MySQL Router的配置文件中包含有关如何执行路由的信息。它与MySQL服务器的配置文件类似，也是由多个段组成，每个段中包含相关配置选项。
    
        MySQL Router是MySQL Proxy的替代方案，MySQL官方不建议将MySQL Proxy用于生产环境，并且已经不提供MySQL Proxy的下载。
## 安装mysql router

1.  去[官网](https://dev.mysql.com/downloads/router/)下载最新版本的mysql router。这里下载的二进制的文件：mysql-router-8.0.14-linux-glibc2.12-x86_64.tar.xz

2. 解压：tar -zxvf mysql-router-8.0.14-linux-glibc2.12-x86_64.tar.xz

3. 重命名：sudo mv mysql-router-8.0.14-linux-glibc2.12-x86_64.tar.xz   /usr/local/mysqlrouter

4. 编写配置文件（这里建议将配置文件交给mysql用户管理，此处配置文件是放在：/home/mysql/scripts/mysqlrouter/mysqlrouter-config.conf）

   ```
   [DEFAULT]
   logging_folder =/home/wupeng/media/dbdata/mysqlrouter
   plugin_folder = /usr/local/mysqlrouter/lib/mysqlrouter
   data_folder = /home/wupeng/media/dbdata/
   
   [logger]
   level = INFO
   
   [routing:basic_failover]
   bind_port = 7001
   ##此处必须填写，不然默认监听的是本机的ip，无法在其它机器上访问
   bind_address = 0.0.0.0
   max_connect_errors = 5
   max_connections = 2000
   routing_strategy = first-available
   destinations = 127.0.0.1:3308,127.0.0.1:3309
   socket = /home/mysql/scripts/mysqlrouter/mysqlrouter.sock
   
   [keepalive]
   interval = 60
   ```

5. 启动和关闭mysql router : 

   ```
   ##启动
   mysqlrouter -c /home/mysql/scripts/mysqlrouter/mysqlrouter-config.conf >/dev/null &
   ##关闭
   kill `pidof /usr/local/mysqlrouter/bin/mysqlrouter`
   ```



## 配置文件的说明



- routing_strategy说明：
  - **round-robin**:轮寻。将第一个connect转发到destinations里面的第一个地址，第二个connect转发到destinations 中的第二个地址，。。。以此类推。
  - **first-available**:新的连接被路由到列表中第一个可用的server。如果失败了，下一个可用的server被使用。这个循环一直持续到所有的server都不可用。
  - **next-available**:和first-available相似，新的连接被路由到列表中第一个可用的server。和first-available不同的是，当一个server被标记为不可用，那么它将会被丢弃，不会被再次使用。
  - **round-robin-with-fallback**: for load-balancing, each new connection is made to the next available secondary server in a round-robin fashion. If a secondary server is not available then servers from the primary list are used in round-robin fashion.(*还没明白表达的意思*)

