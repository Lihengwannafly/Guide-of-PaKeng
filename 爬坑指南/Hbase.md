### hbase遇到的疑难杂症

#### 问题Table Namespace Manager not fully initialized

在HBase Shell命令行中执行create命令时出现ERROR: java.io.IOException: Table Namespace Manager not fully initialized, try again later

![image](https://github.com/jasonzhn/pictureResource/blob/master/b4-2.png?raw=true)

解决方法：

从Zookeeper和HDFS清除HBase相关数据：

首先进入**hdfs**用户，然后执行执行：

```
hbase clean --cleanAll
```

如果出现类似`ZNode(s) [datanode1,16020,1540190478770, datanode2,16020,1540190482957, datanode3,16020,1540190482438] of regionservers are not expired. Exiting without cleaning hbase data.`的错误，先关掉hbase的相关服务，HBase Master和RegionServer，然后执行上述指令。