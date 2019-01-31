# mysql router

mysql router是一个在应用和mysql server之间轻量级的中间件，它可以通过有效地将数据库流量路由到适当的后端MySQL服务器来提供高可用性和可伸缩性，可拔插的架构也可以让开发人员扩展mysql router.

MySQL路由器是高可用性（HA）解决方案的构建块。它通过智能地将连接路由到MySQL服务器来简化应用程序开发，从而提高性能和可靠性。



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
   max_connect_errors = 5
   max_connections = 2000
   routing_strategy = first-available
   destinations = 127.0.0.1:3308,127.0.0.1:3309
   ##设置socket的文件的位置，在不使用tcp协议登录的时候
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

