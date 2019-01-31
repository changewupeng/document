# keepAlive 加mysql

## keepAlive的安装

1. 去[keepAlive的官网](http://www.keepalived.org/download.html)上下载keepAlive安装包。这里下载的是keepalived-1.1.15.tar.gz

2. 安装keepAlive

   - 解压：tar -zxvf keepalived-1.1.15.tar.gz
   - 编译

     ```
     shell>cd keepalived-1.1.15
     shell>./configure --prefix=/usr/local/keepalived
     shell>make
     shell>make install
     ```

     - 安装过程中出现的问题

       - Popt libraries is required

         ```
         sudo yum install popt-devel
         ```

       - configure: error: no acceptable C compiler found in $PATH

         ```
         这个主要是没有安装C语言的编译器,执行sudo yum install gcc即可
         ```

       - OpenSSL is not properly installed on your system

         ```
         sudo yum install -y openssl openssl-devel
         ```

   - 将keepAlive交给系统管理

     ```
     shell>cp /usr/local/keepalived/etc/rc.d/init.d/keepalived /etc/init.d/ 
     shell>cp /usr/local/keepalived/etc/sysconfig/keepalived /etc/sysconfig/ 
     shell>mkdir /etc/keepalived/ 
     shell>cp /usr/local/keepalived/etc/keepalived/keepalived.conf /etc/keepalived/ 
     shell>cp /usr/local/keepalived/sbin/keepalived /usr/sbin/
     ```

     

   - 
