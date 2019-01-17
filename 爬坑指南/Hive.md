### Hive爬坑指南

#### 通过Hive建立Hbase外表

由于直接使用spark sql读取hbase难度太大，因此我们会创建hive外表与hbase连接，然后去用spark sql读取hive外表。但是，有可能会报一下错误：

```
2019-01-17 15:33:10 INFO  ClientCnxn:975 - Opening socket connection to server VM_0_23_centos/127.0.0.1:2181. Will not attempt to authenticate using SASL (unknown error)
2019-01-17 15:33:10 WARN  ClientCnxn:1102 - Session 0x0 for server null, unexpected error, closing socket connection and attempting reconnect
java.net.ConnectException: Connection refused
	at sun.nio.ch.SocketChannelImpl.checkConnect(Native Method)
	at sun.nio.ch.SocketChannelImpl.finishConnect(SocketChannelImpl.java:717)
	at org.apache.zookeeper.ClientCnxnSocketNIO.doTransport(ClientCnxnSocketNIO.java:361)
	at org.apache.zookeeper.ClientCnxn$SendThread.run(ClientCnxn.java:1081)
```

解决方法：

上述原因是缺少hbase配置文件造成的，执行以下命令即可。

```
cp /usr/hdp/2.6.2.14-5/hbase/conf/hbase-site.xml /usr/hdp/2.6.2.14-5/hadoop/conf/
```

