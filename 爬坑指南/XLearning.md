## XLearning 爬坑指南

### 1.安装相关软件

只需要在namenode上安装git和maven（自行谷歌）

[maven]: https://www.tecmint.com/install-apache-maven-on-centos-7/
[git]: https://my.oschina.net/antsky/blog/514586

```
git clone https://github.com/Qihoo360/XLearning.git
在源码根目录下，执行:
maven package
```

### 2.设置环境变量

#### 2.1 修改系统环境变量

```
vim /etc/profile
export XLEARNING_HOME=/root/xlearning-1.3
```

#### 2.2 修改XLearning环境变量

1. log4j.properties文件

   去掉注释

   ```
   log4j.rootCategory=INFO, console
   log4j.appender.console=org.apache.log4j.ConsoleAppender
   log4j.appender.console.target=System.err
   log4j.appender.console.layout=org.apache.log4j.PatternLayout
   log4j.appender.console.layout.ConversionPattern=%d{yy/MM/dd HH:mm:ss} %p %c{1}: %m%n
   log4j.logger.org.apache.zookeeper=ERROR
   log4j.logger.org.apache.hadoop.hdfs.distributedavatarfilesystem=ERROR
   log4j.logger.org.apache.hadoop.util.NativeCodeLoader=ERROR
   # Settings to quiet third party logs that are too verbose
   
   # Settings the HistoryServer logs
   log4j.logger.net.qihoo.xlearning.jobhistory=DEBUG,RFA
   log4j.additivity.net.qihoo.xlearning.jobhistory=false
   log4j.appender.RFA=org.apache.log4j.RollingFileAppender
   log4j.appender.RFA.File=/tmp/XLearning/logs/XLearningHistoryServer.log
   log4j.appender.RFA.Encoding=UTF-8
   log4j.appender.RFA.Append=true
   log4j.appender.RFA.MaxFileSize=100MB
   log4j.appender.RFA.MaxBackupIndex=5
   log4j.appender.RFA.layout=org.apache.log4j.PatternLayout
   log4j.appender.RFA.layout.ConversionPattern=%d{yy/MM/dd HH:mm:ss} %p %c{1}: %m%n
   ```

2. xlearning-env.sh文件

   添加JAVA_HOME和HADOOP_CONF_DIR

   ```
   export XLEARNING_HOME="$(cd "`dirname "$0"`"/..; pwd)"
   
   # When running a distributed configuration it is best to
   # set JAVA_HOME in this file, so that it is correctly defined on
   # remote nodes.
   # The java implementation to use.
   export JAVA_HOME=/usr/java/jdk1.8.0_181-amd64
   
   # Set Hadoop-specific environment variables here.
   export HADOOP_CONF_DIR=/usr/hdp/2.6.2.14-5/hadoop/etc/hadoop
   
   export XLEARNING_CONF_DIR=$XLEARNING_HOME/conf/
   export XLEARNING_CLASSPATH="$XLEARNING_CONF_DIR:$HADOOP_CONF_DIR"
   
   for f in $XLEARNING_HOME/lib/*.jar; do
       export XLEARNING_CLASSPATH=$XLEARNING_CLASSPATH:$f
   done
   
   export XLEARNING_CLIENT_OPTS="-Xmx1024m"
   ```

3. xlearning-site.xml

   配置JobHistory信息，修改0.0.0.0为主机名，如namenode

   ```
   <configuration>
       <property>
           <name>xlearning.staging.dir</name>
           <value>/tmp/XLearning/staging</value>
       </property>
       <property>
           <name>xlearning.cleanup.enable</name>
           <value>true</value>
       </property>
       <property>
           <name>xlearning.container.maxFailures.rate</name>
           <value>0.5</value>
       </property>
       <property>
           <name>xlearning.history.webapp.address</name>
           <value>namenode:19886</value>
       </property>
       <property>
           <name>xlearning.history.webapp.port</name>
           <value>19886</value>
       </property>
       <property>
           <name>xlearning.tf.board.history.dir</name>
           <value>/tmp/XLearning/eventLog</value>
       </property>
       <property>
           <name>xlearning.history.log.dir</name>
           <value>/tmp/XLearning/history</value>
       </property>
       <property>
           <name>xlearning.history.webapp.https.address</name>
           <value>namenode:19885</value>
       </property>
       <property>
           <name>xlearning.history.webapp.https.port</name>
           <value>19885</value>
       </property>
   </configuration>
   ```

### 3.启动XLearning JobHistoryServer服务

```
xlearning-1.3/sbin/start-history-server.sh
```

### 4.修改yarn.application.classpath

添加在YARN配置中的yarn.application.classpath添加以下两项：

\$HADOOP_MAPRED_HOME/*
\$HADOOP_MAPRED_HOME/lib/*

```
/usr/hdp/current/hadoop-mapreduce-client/*,/usr/hdp/current/hadoop-mapreduce-client/lib/*
```

### 5.libhdfs.so

将*/usr/hdp/2.6.2.14-5/usr/lib*的*libhdfs.so、libhdfs.so.0.0.0*复制到*/usr/hdp/2.6.2.14-5/hadoop/lib/native*

```
cp /usr/hdp/2.6.2.14-5/usr/lib/* /usr/hdp/2.6.2.14-5/hadoop/lib/native
```



### 6.XLearning客户端

创建目录

```
hadoop fs -mkdir -p /tmp/XLearning/history
hadoop fs -mkdir -p /tmp/XLearning/eventLog
hadoop fs -mkdir -p /tmp/XLearning/staging
```

上传数据到HDFS

```
hadoop fs -put data /tmp/
```

执行示例

```
cd examples/tensorflow
sh run.sh
```

## Iuuse

#### I tensorflow/core/distributed_runtime/master.cc:193] CreateSession still waiting for response from worker: /job:worker/replica:0/task:0 

我认为对于`tf.train.MonitoredTrainingSession`协调协议，这是 “预期的行为”。在[recent answer](https://stackoverflow.com/a/43007657/3574081)中，我解释了这个协议是如何适应长期运行training工作的，因此worker会在检查变量是否被初始化之间休眠30秒。

worker1运行初始化操作和worker2检查变量之间存在竞争条件，并且如果worker2“胜利”竞争，它会观察到一些变量未初始化，并且它将进入30秒在再次检查之前睡觉。

但是，程序中的总计算量非常小，因此在此30秒期间，worker1将能够完成其工作并终止。当worker2检查变量是否被初始化时，它会创建一个新的`tf.Session`，尝试连接到其他任务，但worker1不再运行，因此您将看到类似这样的日志消息（每10秒重复一次或所以）：

```
I tensorflow/core/distributed_runtime/master.cc:193] CreateSession still waiting for response from worker: /job:worker/replica:0/task:0 
```

当training大大超过30秒时，这不成问题。

一种解决方法是通过设置“设备过滤器”来消除工人之间的相互依赖关系。因为在一个典型的图形之间配置的worker不沟通，你可以告诉TensorFlow忽略不存在另一worker在会话创建时，使用`tf. ConfigProto`：

```
# Each worker only needs to contact the PS task(s) and the local worker task. 
config = tf.ConfigProto(device_filters=[ 
    '/job:ps', '/job:worker/task:%d' % arguments.task_index]) 

with tf.train.MonitoredTrainingSession(
    master=server.target, 
    config=config, 
    is_chief=(arguments.task_index == 0 and (
       arguments.job_name == 'worker'))) as sess: 
```

