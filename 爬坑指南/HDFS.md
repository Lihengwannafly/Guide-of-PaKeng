### HDFS 疑难杂症

#### Standy Namenode

1. Put Active NN in safemode

   ```
   sudo -u hdfs hdfs dfsadmin -fs hdfs://datanode1 -safemode enter
   或者
   sudo -u hdfs hdfs dfsadmin -fs hdfs://datanode1:8020 -safemode enter（不带端口号应该也是可以的）
   ```

2. Do a savenamespace operation on Active NN

   ```
   sudo -u hdfs hdfs dfsadmin -fs hdfs://datanode1 -saveNamespace
   ```

3. Leave Safemode

   ```
   sudo -u hdfs hdfs dfsadmin -fs hdfs://datanode1 -safemode leave
   ```

4. Login to Standby NN

5. Run below command on **Standby namenode** to get latest fsimage that we saved in above steps.

   ```
   sudo -u hdfs hdfs namenode -bootstrapStandby -force
   ```
