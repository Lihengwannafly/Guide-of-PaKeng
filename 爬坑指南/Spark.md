## Spark爬坑指南

### Spark2.0 yarn方式启动报错

#### 报错信息

*16/08/25 17:29:28 WARN ObjectStore: Version information not found in metastore. hive.metastore.schema.verification is not enabled so recording the schema version 1.2.0
16/08/25 17:29:28 WARN ObjectStore: Failed to get database default, returning NoSuchObjectException
Exception in thread "main" java.lang.NoClassDefFoundError: com/sun/jersey/api/client/config/ClientConfig*

原因：yarn的lib包下面使用的是1.9的jar，而spark下使用的是2.22.2的jar包

解决方法：下载jersey-bundle-1.9.1.jar，并拷贝到$SPARK_HOME/jars（/usr/hdp/2.6.2.14-5/spark2/jars/）和/opt/rh/rh-python36/root/usr/lib/python3.6/site-packages/pyspark/jars/下：

```
scp /usr/hdp/2.6.2.14-5/spark2/jars/jersey-bundle-1.9.1.jar datanode5:/usr/hdp/2.6.2.14-5/spark2/jars/
scp /usr/hdp/2.6.2.14-5/spark2/jars/jersey-bundle-1.9.1.jar datanode5:/opt/rh/rh-python36/root/usr/lib/python3.6/site-packages/pyspark/jars/
```

