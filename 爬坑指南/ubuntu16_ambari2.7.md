### ubuntu18.04安装ambari2.7

### 1. 前期准备

#### 1.1 ubuntu系统允许root用户登陆

1. 创建root账户：

   ```
   sudo passwd root 
   ```

2. 修改配置文件，文件路径/etc/lightdm/lightdm.conf

   ```
   sudo gedit /etc/lightdm/lightdm.conf
   ```

   打开后，在文档末尾输入

   ```
   [Seat:*]
   autologin-guest=false
   autologin-user=root
   autologin-user-timeout=0
   greeter-session=lightdm-gtk-greeter
   ```

3. 修改/root/.profile文件

   ```
   gedit /root/.profile
   ```

​	文档最后一行 mesg n || true 前添加  tty -s && 即 tty -s &&mesg n || true

4. 重启系统，终端界面输入 #reboot

#### 1.2 ubuntu安装相关软件

1. 安装ssh服务

   ```
   apt-get install openssh-server
   ```

2. 为ssh服务打开使用root用户登录的权限

   ```
   gedit /etc/ssh/sshd_config
   ```

   将*PermitRootLogin without-password*修改为*PermitRootLogin yes*

   重启ssh即可实现root用户使用ssh登录

   ```
   service ssh restart
   ```

3. 安装jdk

   下载jdk压缩包

   创建目录

   ```
   sudo mkdir /usr/lib/jvm
   ```

   解压缩包到该目录

   ```
   sudo tar -zxvf jdk-8u181-linux-x64.tar.gz -C /usr/lib/jvm
   ```

   修改环境变量

   ```
   sudo gedit ~/.bashrc
   ```

   末尾追加

   ```
   #set oracle jdk environmentexport 
   JAVA_HOME=/usr/lib/jvm/jdk1.8.0_181 
   export JRE_HOME=${JAVA_HOME}/jre  
   export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib  
   export PATH=${JAVA_HOME}/bin:$PATH  
   ```

   环境变量生效

   ```
   source ~/.bashrc
   ```


4. 安装NTP时钟同步

   ```
   apt-get install ntp
   update-rc.d ntp defaults
   ```

5. 防火墙配置

   ```
   sudo ufw disable
   sudo iptables -X
   sudo iptables -t nat -F
   sudo iptables -t nat -X
   sudo iptables -t mangle -F
   sudo iptables -t mangle -X
   sudo iptables -P INPUT ACCEPT
   sudo iptables -P FORWARD ACCEPT
   sudo iptables -P OUTPUT ACCEPT
   ```

   其他软件安装详情请见ambari官方文档

[1]: https://docs.hortonworks.com/HDPDocuments/Ambari-2.7.0.0/bk_ambari-installation/bk_ambari-installation.pdf	"ambari官方文档"

#### 1.3 ssh免密登陆

1. 每台节点配置hosts文件，如下所示：

   ```
   192.168.1.116	master
   192.168.1.125 	slave1
   192.168.1.115	slave2
   ```

2. 每台节点生成ssh公私钥对

   ```
   ssh-keygen
   ```

   一路回车

3. 在主节点上将公钥拷贝到authorized_keys

   ```
   cat id_rsa.pub >> authorized_keys
   ```

4. 登录其他子节点，将其的公钥文件内容都拷贝到主节点的authorized_keys文件中，命令如下：

   ```
   ssh-copy-id -i master #将公钥拷贝到master的authorized_keys中
   ```

5. 登录master,输入命令：

   ```
   chmod 700 ~/.ssh
   chmod 600 ~/.ssh/authorized_keys
   ```

6. 将authorized_keys从主节点拷贝到其他子节点上

   ```
   scp /root/.ssh/authorized_keys slave1:/root/.ssh/
   scp /root/.ssh/authorized_keys slave2:/root/.ssh/
   ```

### 2. 安装ambari

​	根据提示一步一步来即可。

[2]: https://docs.hortonworks.com/HDPDocuments/Ambari-2.7.0.0/bk_ambari-installation/content/download_the_ambari_repo_ubuntu16.html	"ambari toturial"

set database configure for hive and oozie, please look at this link 

https://developer.ibm.com/hadoop/2015/10/29/using-postgresql-for-oozie-and-hive/

```
su - postgres
psql
CREATE DATABASE HIVE;
CREATE USER HIVE with PASSWORD ‘bigdata’;
GRANT ALL PRIVILEGES ON DATABASE hive to hive;
```

