

## Centos7下安装Ambari2.5.2

### 前期准备

1. 修改主机名

   ```
   hostnamectl set-hostname master
   ```

   修改网络配置

   ```
   vim /etc/sysconfig/network
   ```

   添加如下信息

   ```
   # Created by anaconda
   NETWORKING=yes
   HOSTNAME=master
   ```

   主机名视情况而定

2. 配置hosts

   ```
   vim /etc/hosts
   ```

   添加所有主机

   ```
   内网IP master
   内网IP slave1
   ...
   ```

3. 关闭防火墙

   ```
   systemctl disable firewalld.service
   systemctl stop firewalld.service
   ```

4. 关闭SElinux

   ```
   vim /etc/sysconfig/selinux
   ```

   将SELINUX=enforcing改为SELINUX=disabled

5. 时钟同步

   ```
   yum install -y ntp
   chkconfig --list ntpd
   systemctl is-enabled ntpd
   systemctl enable ntpd
   systemctl start ntpd
   ```

6. 安装JDK1.8

   检查系统JDK版本

   ```
   java -version
   ```

   如果为open jdk，则卸载

   ```
   yum remove *openjdk*
   ```

   安装oracle JDK1.8

   ```
   wget http://download.oracle.com/otn-pub/java/jdk/8u181-b13/96a7b8442fe848ef90c96a2fad6ed6d1/jdk-8u181-linux-x64.rpm?AuthParam=1539674177_f992c92ce0571724cd8bf6904bc4315f
   或者下载到本地，通过xftp传到云主机上
   rpm -i 安装包
   ```

   检查是否安装成功

   ```
   java -version
   ```

7. 相关服务安装

   ```
   umask 0022
   yum -y install lrzsz
   yum install -y openssh-clients
   ```

8. ssh免密登录

   每台节点上

   ```
   ssh-keygen
   ```

   在主节点上将公钥拷贝到authorized_keys

   ```
   cat id_rsa.pub >> authorized_keys
   ```

   登录其他子节点，将其的公钥文件内容都拷贝到主节点的authorized_keys文件中，命令如下：

   ```
   ssh-copy-id -i master #将公钥拷贝到master的authorized_keys中
   ```

   在主节点上

   ```
   chmod 700 ~/.ssh
   chmod 600 ~/.ssh/authorized_keys
   ```

   ```
   scp /root/.ssh/authorized_keys slave1:/root/.ssh/
   ```


### 制作ambari离线源

以下步骤仅在主节点master中操作

1. 下载ambari离线安装包

   ```
   wget http://public-repo-1.hortonworks.com/ambari/centos7/2.x/updates/2.5.2.0/ambari-2.5.2.0-centos7.tar.gz
   wget http://public-repo-1.hortonworks.com/HDP/centos7/2.x/updates/2.6.2.14/HDP-2.6.2.14-centos7-rpm.tar.gz
   wget http://public-repo-1.hortonworks.com/HDP-UTILS-1.1.0.21/repos/centos7/HDP-UTILS-1.1.0.21-centos7.tar.gz
   ```

2. 安装本地源工具

   ```
   yum install yum-utils createrepo yum-plugin-priorities -y
   vi /etc/yum/pluginconf.d/priorities.conf
   #添加 gpgcheck=0
   ```

3. 配置http服务

   ```
   yum install httpd
   chkconfig httpd on
   service httpd start
   ```

4. 创建本地源

   将下载的3个tar包解压到/var/www/html 相应目录下

   ```
   cd /var/www/html/
   mkdir ambari
   cd ..
   mkdir hdp
   
   tar -zxvf ambari-2.5.2.0-centos7.tar.gz -C /var/www/html/ambari
   tar -zxvf HDP-2.6.2.14-centos7-rpm.tar.gz -C /var/www/html/hdp
   tar -zxvf HDP-UTILS-1.1.0.21-centos7.tar.gz -C /var/www/html/hdp/HDP-UTILS
   ```

   创建centos repo

   ```
   cd /var/www/html/ambari/ambari/centos7
   createrepo ./
   cd /var/www/html/hdp/HDP/centos7
   createrepo ./
   cd /var/www/html/hdp/HDP-UTILS/centos7
   createrepo ./
   ```

5. 下载ambari.repo　HDP.repo，配置为本地源

   ```
   cd /etc/yum.repos.d/
   wget -nv http://public-repo-1.hortonworks.com/ambari/centos7/2.x/updates/2.5.2.0/ambari.repo
   vim ambari.repo 
   
   #VERSION_NUMBER=2.5.2.0-298
   [ambari-2.5.2.0]
   name=ambari Version - ambari-2.5.2.0
   baseurl=http://namenode/ambari/ambari/centos7/
   gpgcheck=0
   gpgkey=http://namenode/ambari/ambari/centos7RPM-GPG-KEY/RPM-GPG-KEY-Jenkins
   enabled=1
   priority=1
   ```

   ```
   wget -nv http://public-repo-1.hortonworks.com/HDP/centos7/2.x/updates/2.6.2.14/hdp.repo
   vim hdp.repo 
   
   #VERSION_NUMBER=2.6.2.14-5
   [HDP-2.6.2.14]
   name=HDP Version - HDP-2.6.2.14
   baseurl=http://namenode/hdp/HDP/centos7/
   gpgcheck=0
   gpgkey=http://namenode/hdp/HDP/centos7/RPM-GPG-KEY/RPM-GPG-KEY-Jenkins
   enabled=1
   priority=1
   
   
   [HDP-UTILS-1.1.0.21]
   name=HDP-UTILS Version - HDP-UTILS-1.1.0.21
   baseurl=http://namenode/hdp/
   gpgcheck=0
   gpgkey=http://namenode/hdp/RPM-GPG-KEY/RPM-GPG-KEY-Jenkins
   enabled=1
   priority=1
   ```

6. 执行yum命令

   ```
   yum clean all
   yum makecache
   ```

7. 查看本地源

8. 安装mysql

   ```
   cd /var/www/html/hdp/HDP-UTILS/mysql
   rpm -ivh mysql-community-release-el7-5.noarch.rpm  --nodeps --force
   ```

   安装mysql

   ```
   yum install -y mysql-server
   systemctl start mysqld
   service mysqld status
   ```

   ```
   yum install -y mysql-connector-java
   ```

9. 修改默认密码

   ```
   vi /etc/my.cnf
   添加skip-grant-tables 
   重启mysql：systemctl restart mysqld
   mysql -uroot
   mysql> USE mysql ; 
   mysql> UPDATE user SET Password = password ( '你的密码' ) WHERE User = 'root'; 
   mysql> flush privileges ; 
   退出mysql
   重启mysql：systemctl restart mysqld
   ```

10. 创建必要的数据库和数据库用户

    ambari用户和数据库

    ```
    create database ambari character set utf8;
    CREATE USER 'ambari'@'%' IDENTIFIED BY 'bigdata';
    GRANT ALL PRIVILEGES ON *.* TO 'ambari'@'%';
    FLUSH PRIVILEGES;
    ```

    hive用户和数据库

    ```
    create database hive character set utf8;
    CREATE USER 'hive'@'%' IDENTIFIED BY 'bigdata';
    GRANT ALL PRIVILEGES ON *.* TO 'hive'@'%';
    FLUSH PRIVILEGES;
    ```

    oozie用户和数据库

    ```
    create database oozie character set utf8;
    CREATE USER 'oozie'@'%' IDENTIFIED BY 'bigdata';
    GRANT ALL PRIVILEGES ON *.* TO 'oozie'@'%';
    FLUSH PRIVILEGES;
    ```

    上述三个用户，可能会出现输入密码无法登陆，但不输入密码却能登陆的情况。

    CREATE USER 'root'@'%' IDENTIFIED BY '417654321';

    GRANT ALL PRIVILEGES ON *.* TO 'root'@'%';

    ```
    select User,Password,Host from mysql.user;
    select user,host from mysql.user;
    delete from mysql.user where user='';
    +--------+-------------------------------------------+-----------+
    | User   | Password                                  | Host      |
    +--------+-------------------------------------------+-----------+
    | root   | *2340BB17528AEE5638CF94B2082CC542FD8F7E8C | localhost |
    | root   | *2340BB17528AEE5638CF94B2082CC542FD8F7E8C | namenode  |
    | root   | *2340BB17528AEE5638CF94B2082CC542FD8F7E8C | 127.0.0.1 |
    | root   | *2340BB17528AEE5638CF94B2082CC542FD8F7E8C | ::1       |
    | hive   | *C33A05FE652CA69965121A309F0DE7FA785D3916 | %         |
    | oozie  | *C33A05FE652CA69965121A309F0DE7FA785D3916 | %         |
    | ambari | *C33A05FE652CA69965121A309F0DE7FA785D3916 | %         |
    | ambari | *C33A05FE652CA69965121A309F0DE7FA785D3916 | localhost |
    | ambari | *C33A05FE652CA69965121A309F0DE7FA785D3916 | namenode  |
    | test   | *C33A05FE652CA69965121A309F0DE7FA785D3916 | %         |
    +--------+-------------------------------------------+-----------+
    
    ```

    如果用户名User为空，则将这一行删除。

### 安装ambari

1. 检查仓库是否可用

   ```
   yum repolist
   ```

2. 安装ambari

   ```
   yum install ambari-server
   ```

3. mysql导入Ambari-DDL-MySQL-CREATE.sql

   root用户登录mysql

   ```
   use ambari;
   source /var/lib/ambari-server/resources/Ambari-DDL-MySQL-CREATE.sql;
   ```

4. 配置Ambari-Server

   ```
   ambari-server setup
   Customize user account for ambari-server daemon [y/n] (n)? y
   Enter user account for ambari-server daemon (root): root
   Adjusting ambari-server permissions and ownership...
   ```

   配置jdk

   ```
   Checking JDK...Do you want to change Oracle JDK [y/n] (n)? y
   [1] Oracle JDK 1.8 + Java Cryptography Extension (JCE) Policy Files 8
   [2] Oracle JDK 1.7 + Java Cryptography Extension (JCE) Policy Files 7
   [3] Custom JDK
   
   ==========================================
   
   Enter choice (1): 3
   /usr/java/jdk1.8.0_181-amd64
   ```

   配置mysql数据库

   ```
   Configuring database...
   ========================================
   
   Choose one of the following options:
   
   [1] - PostgreSQL (Embedded)
   [2] - Oracle
   [3] - MySQL
   [4] - PostgreSQL
   [5] - Microsoft SQL Server (Tech Preview)
   [6] - SQL Anywhere
   
   ==========================================
   
   Enter choice (1): 3
   
   Hostname (localhost): mater
   其余配置不需要更改
   ```

   提示必须安装MySQL JDBC，已安装则直接通过。

   ```
   WARNING: Before starting Ambari Server, you must copy the MySQL JDBC driver JAR file to /usr/share/java.
   
   Press <enter> to continue.
   ```

5. 修改ssl验证，在***所有节点上***

   ```
   vi /etc/ambari-agent/conf/ambari-agent.ini
   
   在[security]下添加  
   force_https_protocol=PROTOCOL_TLSv1_2
   ```

   ```
   vi /etc/python/cert-verification.cfg
   
   [https]
   verify=disable
   ```

6. 启动ambari

   ```
   ambari-server start
   ```

### 其他常见问题

一般情况下，由于开发者使用的python解释器为python3，所以我们需要将重现安装一个新的python3。但是，ambari默认的是python2， 因此将默认的解释器改为python3会出现一系列问题。下面是解决方案：

1. 运行python3安装脚本，可在 服务器->安装脚本目录下找到

2. 默认解释器替换为python3之后，主要是以下两个文件报错：

   - /usr/bin/hdp-select
   - /etc/hadoop/conf/topology_script.py

   原因是因为它们的代码是python2版本，我们仅需要执行以下命令就可以将python2代码转换为python3。

   ```
   python3 2to3.py -w /usr/bin/hdp-select
   python3 2to3.py -w /etc/hadoop/conf/topology_script.py
   ```

### 磁盘扩容

待续



### 参考

[Ambari及其HDP集群安装及其配置教程](https://zhuanlan.zhihu.com/p/37322462)

[CentOS7 搭建Ambari-Server，安装Hadoop集群（一）](https://www.cnblogs.com/JasonMa1980/p/6912115.html)

[ambari安装问题 Confirm Hosts SSLError: Failed to connect. Please check openssl library versions](https://blog.csdn.net/jzero_2008/article/details/81135079)

[Access denied for user 'ambari'@localhost (using password: YES)](https://community.hortonworks.com/questions/146159/access-denied-for-user-ambari.html)

[How to Enable Python3 Support on HDP 2.6](https://stackoverflow.com/questions/52874985/how-to-enable-python3-support-on-hdp-2-6)

[Unable to start Pyspark jobs when running with Python 3](https://community.hortonworks.com/content/supportkb/186304/unable-to-start-pyspark-jobs-when-running-with-pyt.html)

[Use of Python version 3 scripts for pyspark with HDP 2.4](https://community.hortonworks.com/questions/46454/use-of-python-version-3-scripts-for-pyspark-with-h.html)

