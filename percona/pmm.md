# PMM(PERCONA MONITORING AND MANAGEMENT)

Percona Monitoring and Management (PMM)是一款开源的用于管理和监控MySQL和MongoDB性能的开源平台，通过PMM客户端收集到的DB监控数据用第三方软件Grafana画图展示出来。

- PMM Client 安装在每个想监控的数据库服务器上。收集mysql服务器的度量信息，一般的系统信息，查询统计数据。
- PMM Server是PMM的中心，聚合pmm client收集的信息，以表格，仪表盘，图表的形式展示。



![PMM架构图](/home/wupeng/media/github/document/percona/img/struct..webp)






## PMM SERVER

PMM SERVER包含的工具：

- Query Analytics ：
- Metrics Monitor
- Orchestrator:





Orchestrator
Orchestrator is a MySQL replication topology management and visualization tool. If it is enabled, you can access it
using the /orchestrator URL after PMM Server address. Alternatively, you can click the MySQL Replication
Topology Manager button on the PMM Server landing page.
To use it, create a MySQL user for Orchestrator on all managed instances:
GRANT SUPER, PROCESS, REPLICATION SLAVE, RELOAD ON *.*
TO 'orc_client_user'@'%'
IDENTIFIED BY 'orc_client_password’;
Note: The credentials in the previous example are default. If you use a different user name or password, you have
to pass them when running PMM Server using the ORCHESTRATOR_PASSWORD and ORCHESTRATOR_USER
options.
$ docker run ... -e ORCHESTRATOR_ENABLED=true ORCHESTRATOR_USER=name -e ORCHESTRATOR_
˓ → PASSWORD=pass ... percona/pmm-server:latest
Then you can use the Discover page in the Orchestrator web interface to add the instances to the topology.
Note: Orchestrator is not enabled by default starting with PMM 1.3.0
Orchestrator was included into PMM for experimental purposes. It is a standalone tool, not integrated with PMM
other than that you can access it from the landing page.
**In version 1.3.0 and later, Orchestrator is not enabled by default. To enable it, see Additional options in the Running PMM Server via Docker section.**

### 简单自动安装

下载脚本

```
curl -fsSL https://raw.githubusercontent.com/percona/pmm/master/get-pmm.sh -o get-pmm.sh

sh get-pmm.sh
```

脚本需要使用root帐号执行，或者可以运行docker container的非root，或者sudo.

脚本主要做以下工作：

- 检查docker是否安装，如果没有安装尝试安装；
- 运行docker
- 下载PMM Server镜像
- 生成pmm-data容器
- 配置并启动pmm server容器。



### 使用docker安装

- 搭建docker环境，运行docker的守护进程。细节不描述

- 下载pmm server镜像

  ```
  docker pull percona/pmm-server:latest
  ```

  这里是下载最新版本的，也可以制定下载版本。

- 创建pmm-data容器

  ```
  $ docker create -v /opt/prometheus/data -v /opt/consul-data -v /var/lib/mysql 
  -v /var/lib/grafana --name pmm-data percona/pmm-server:latest /bin/true
  --------------------------------------------------------------------------
  
  
  The previous command does the following:
  • The docker create command instructs the Docker daemon to create a container from an image.
  • The -v options initialize data volumes for the container.
  • The --name option assigns a custom name for the container that you can use to reference the container within
  a Docker network. In this case: pmm-data.
  • percona/pmm-server:latest is the name and version tag of the image to derive the container from.
  • /bin/true is the command that the container runs.
  ```

​      

- 创建并运行PMM Server容器

  ```
  $ docker run -d \
  -p 80:80 \
  --volumes-from pmm-data \
  --name pmm-server \
  --restart always \
  percona/pmm-server:latest
  
  ---------------------------------------------------------------------------------
  • The docker run command runs a new container based on the percona/pmm-server:latest image.
  • The -d option starts the container in the background (detached mode).
  • The -p option maps the port for accessing the PMM Server web UI. For example, if port 80 is not available,
  you can map the landing page to port 8080 using -p 8080:80.
  • The -v option mounts volumes from the pmm-data container (see Creating the pmm-data Container).
  • The --name option assigns a custom name to the container that you can use to reference the container within
  the Docker network. In this case: pmm-server.
  • The --restart option defines the container’s restart policy. Setting it to always ensures that the Docker
  daemon will start the container on startup and restart it if the container exits.
  • percona/pmm-server:latest is the name and version tag of the image to derive the container from.
  ```

### 对当前的版本的pmm server container进行备份

如果升级不成功，或者尝试了新版本之后，依然使用以前的版本，那么需要对当前的版本进行一个备份。

- 停止当前的pmm-server container

  ```
  $ docker stop pmm-server
  ```

- 对当前版本的pmm-server进行重命名

  ```
  $ docker rename pmm-server pmm-server-backup
  ```

### 基于新的pmm server 镜像创建新的docker container

- 下载新版本的pmm server

- 在下载了新版本的pmm server后，使用docker run创建一个新的容器

  ```
  docker run -d \
  -p 80:80 \
  --volumes-from pmm-data \
  --name pmm-server \
  --restart always \
  percona/pmm-server:latest
  ```

**Note that this command also refers to pmm-data as the value of --volumes-from option. This way, your new version will continue to use the existing data.**



### 删除备份的容器

在已经尝试了新版本的所有特性之后，当决定使用新版本，那么备份容器将不再使用，可以删除掉。

```
$ docker rm pmm-server-backup
```

### 恢复旧版本

如果想恢复就版本，只需要停止并删除新版本，重命名旧版本即可

```
$ docker stop pmm-server && docker rm pmm-server
$ docker pmm-server-backup rename pmm-server 
$ docker start pmm-server
```

### 从容器中备份pmm data

pmm server是在容器中运行，它的数据是在pmm-data这个容器中。为避免数据丢失可将数据备份到外面。



为了备份来自pmm-data的数据，需要在本地创建一个包含一系列子目录的目录，然后使用docker命令来拷贝文件。

- 创建备份目录，并进入

  ```
  $ mkdir pmm-data-backup; 
  $ cd pmm-data-backup
  ```

- 创建必要的子目录

  ```
  $ mkdir -p opt/prometheus
  $ mkdir -p var/lib
  ```

- 关闭的docker container

  ```
  $ docker stop pmm-server
  ```

- 拷贝数据

  ```
  $ docker pmm-data:/opt/prometheus/data opt/prometheus/
  $ docker pmm-data:/opt/consul-data opt/
  $ docker pmm-data:/var/lib/mysql var/lib/
  $ docker pmm-data:/var/lib/grafana var/lib/
  ```

- 开启的docker container

  

  

  

  

  

## PMM client

 条件：

- 确保pmm server是可以访问的
- 安装client的机器需要具有root权限。



从官网上下载安装包，执行install脚本即可。

### 存储要求

至少100MB的空间，由于client和server之间的常连接，不需要额外的内存，但是如果数据不能被立即送到server, 那么client就需要空间存储。



- 配置client连接server

  ```
  $ pmm-admin config --server 192.168.100.1:8080
  ```

  

client需要保持下面的端口通畅：

```
PMM Server should keep ports 80 or 443 ports open for computers where PMM Client is installed to access the
PMM web interface.
42000 For PMM to collect genenal system metrics.
42001 This port is used by a service which collects query performance data and makes it available to QAN.
42002 For PMM to collect MySQL server metrics.
42003 For PMM to collect MongoDB server metrics.
42004 For PMM to collect ProxySQL server metrics.
```

- 收集mysql

  ```
  $ pmm-admin add mysql
  ```






关闭pmm client

```
pmm-admin stop mysql:metrics localhost.localdomain
pmm-admin stop linux:metrics localhost.localdomain
pmm-admin stop mysql:queries localhost.localdomain
```

