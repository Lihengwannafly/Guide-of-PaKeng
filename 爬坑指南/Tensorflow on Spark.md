### Tensorflow on Spark

#### 准备工作

Centos7，Spark2

#### 开始

1. 安装python3.6

   可使用已经写好的**python3.6-install.sh**进行安装

2. 下载MNIST数据集

   ```
   mkdir mnist
   pushd mnist >/dev/null
   curl -O "http://yann.lecun.com/exdb/mnist/train-images-idx3-ubyte.gz"
   curl -O "http://yann.lecun.com/exdb/mnist/train-labels-idx1-ubyte.gz"
   curl -O "http://yann.lecun.com/exdb/mnist/t10k-images-idx3-ubyte.gz"
   curl -O "http://yann.lecun.com/exdb/mnist/t10k-labels-idx1-ubyte.gz"
   zip -r mnist.zip *
   popd >/dev/null
   ```

2. github下载tensorflow on spark工程，将lib文件夹中的tensorflow-hadoop-1.0-SNAPSHOT.jar上传到master中，执行

   ```
   hadoop fs -put tensorflow-hadoop-1.0-SNAPSHOT.jar
   ```

3. 修改配置文件

   ```
   vim /etc/profile
   ```

   末尾添加：

   ```
   # tensorflow on spark test
   export PYTHON_ROOT=/opt/rh/rh-python36/root
   export LD_LIBRARY_PATH=/usr/hdp/2.6.2.14-5/hadoop/lib/native
   export PYSPARK_PYTHON=${PYTHON_ROOT}/bin/python3
   export PYSPARK_DRIVER_PYTHON=${PYTHON_ROOT}/bin/python3
   export SPARK_YARN_USER_ENV="PYSPARK_PYTHON=/opt/rh/rh-python36/root/bin/python3"  
   export PATH=${PYTHON_ROOT}/bin/:/usr/lib64/qt-3.3/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/root/bin 
   export HADOOP_CONF_DIR=/usr/hdp/2.6.2.14-5/hadoop/etc/hadoop 
   export YARN_CONF_DIR=/usr/hdp/2.6.2.14-5/hadoop-yarn/etc/hadoop 
   # set paths to libjvm.so, libhdfs.so, and libcuda*.so 
   export LIB_HDFS=/usr/hdp/2.6.2.14-5/usr/lib              # path to libhdfs.so, for TF acccess to HDFS 
   export LIB_JVM=/usr/java/jdk1.8.0_191-amd64/jre/lib/amd64/server/                        # path to libjvm.so
   ```

4. 下载jersey-bundle-1.9.1.jar，并复制到集群每个节点的/opt/rh/rh-python36/root/lib/python3.6/site-packages/pyspark/jars和/usr/hdp/2.6.2.14-5/spark2/jars

5. 在Ambari的Yarn配置Custom yarn-site，添加hdp.version=2.6.2.14-5

6. 将MNIST压缩文件转换为HDFS文件

   ```
   spark-submit \
   --master yarn \
   --deploy-mode cluster \
   --queue ${QUEUE} \
   --archives TensorFlowOnSpark/mnist/mnist.zip#mnist \
   TensorFlowOnSpark/examples/mnist/mnist_data_setup.py \
   --output TensorFlowOnSpark/mnist/csv \
   --format csv
   ```

7. Training

   ```
   spark-submit \
   --master yarn \
   --deploy-mode cluster \
   --queue ${QUEUE} \
   --num-executors 2 \
   --executor-memory 4G \
   --py-files TensorFlowOnSpark/examples/mnist/spark/mnist_dist.py \
   --conf spark.dynamicAllocation.enabled=false \
   --conf spark.yarn.maxAppAttempts=1 \
   --conf spark.executorEnv.LD_LIBRARY_PATH=$LIB_JVM:$LIB_HDFS:$LD_LIBRARY_PATH \
   TensorFlowOnSpark/examples/mnist/spark/mnist_spark.py \
   --images TensorFlowOnSpark/mnist/csv/train/images \
   --labels TensorFlowOnSpark/mnist/csv/train/labels \
   --mode train \
   --model mnist_model
   ```

   ```
   spark-submit \
   --master yarn \
   --deploy-mode cluster \
   --queue ${QUEUE} \
   --num-executors 2 \
   --executor-memory 16G \
   --py-files TensorFlowOnSpark/examples/mnist/spark/mnist_dist.py \
   --conf spark.dynamicAllocation.enabled=false \
   --conf spark.yarn.maxAppAttempts=1 \
   --conf spark.executorEnv.LD_LIBRARY_PATH=$LIB_JVM:$LIB_HDFS:$LD_LIBRARY_PATH \
   TensorFlowOnSpark/examples/mnist/spark/mnist_spark.py \
   --images /user/root/TensorFlowOnSpark/mnist/csv/train/images \
   --labels /user/root/TensorFlowOnSpark/mnist/csv/train/labels \
   --mode train \
   --model mnist_model
   ```



   ```
   spark-submit \
   --master yarn \
   --deploy-mode cluster \
   --queue ${QUEUE} \
   --num-executors 2 \
   --executor-memory 8G \
   --py-files TensorFlowOnSpark/tfspark.zip,TensorFlowOnSpark/examples/mnist/spark/mnist_dist.py \
   --conf spark.cores.max=2 \
   --conf spark.task.cpus=1 \
   --conf spark.executorEnv.LD_LIBRARY_PATH=$LIB_JVM:$LIB_HDFS \
   TensorFlowOnSpark/examples/mnist/spark/mnist_spark.py \
   --images /user/root/TensorFlowOnSpark/mnist/csv/train/images \
   --labels /user/root/TensorFlowOnSpark/mnist/csv/train/labels \
   --format csv \
   --mode train \
   --model /user/root/TensorFlowOnSpark/model/mnist_model
   ```

   ```
   spark-submit \--master yarn \--deploy-mode cluster \--queue ${QUEUE} \--num-executors 2 \--executor-memory 4G \--archives TensorFlowOnSpark/mnist/mnist.zip#mnist \--jars hdfs:///user/root/tensorflow-hadoop-1.0-SNAPSHOT.jar \--conf spark.executorEnv.LD_LIBRARY_PATH=$LIB_JVM:$LIB_HDFS \TensorFlowOnSpark/examples/mnist/mnist_data_setup.py \	--output TensorFlowOnSpark/mnist/tfr \--format tfr
   ```

   ```
   spark-submit \
   --master yarn \
   --deploy-mode cluster \
   --queue ${QUEUE} \
   --num-executors 2 \
   --executor-memory 5G \
   --py-files TensorFlowOnSpark/tfspark.zip,TensorFlowOnSpark/examples/mnist/tf/mnist_dist.py \
   --conf spark.dynamicAllocation.enabled=false \
   --conf spark.yarn.maxAppAttempts=1 \
   --conf spark.executorEnv.LD_LIBRARY_PATH=$LIB_JVM:$LD_LIBRARY_PATH \
   TensorFlowOnSpark/examples/mnist/tf/mnist_spark.py \
   --images /user/root/TensorFlowOnSpark/mnist/tfr/train \
   --format tfr \
   --mode train \
   --model /user/root/TensorFlowOnSpark/model/mnist_model
   ```

   ```
   hadoop fs -mkdir /TensorFlowOnSpark/model
   hadoop fs -chmod 777 /TensorFlowOnSpark/model
   hadoop fs -mkdir /TensorFlowOnSpark/export
   hadoop fs -chmod 777 /TensorFlowOnSpark/export
   spark-submit \
   --master yarn \
   --num-executors 2 \
   --executor-memory 5G \
   --conf spark.executorEnv.JAVA_HOME="$JAVA_HOME" \
   --py-files TensorFlowOnSpark/tfspark.zip,TensorFlowOnSpark/examples/mnist/spark/mnist_dist.py \
   --conf spark.dynamicAllocation.enabled=false \
   --conf spark.yarn.maxAppAttempts=1 \
   --conf spark.executorEnv.LD_LIBRARY_PATH=$LIB_JVM:$LIB_HDFS \
   TensorFlowOnSpark/examples/mnist/keras/mnist_mlp.py \
   --input_mode spark \
   --images hdfs:///TensorFlowOnSpark/mnist/csv/train/images \
   --labels hdfs:///TensorFlowOnSpark/mnist/csv/train/labels \
   --epochs 1 \
   --mode train \
   --model_dir  hdfs:///TensorFlowOnSpark/model \
   --export_dir  hdfs:///TensorFlowOnSpark/export
   ```

```
   spark-submit \
   --master yarn \
   --num-executors 3 \
   --executor-memory 4G \
   --conf spark.executorEnv.JAVA_HOME="$JAVA_HOME" \
   --py-files TensorFlowOnSpark/tfspark.zip \
   --conf spark.dynamicAllocation.enabled=false \
   --conf spark.yarn.maxAppAttempts=1 \
   --conf spark.executorEnv.LD_LIBRARY_PATH=$LIB_JVM:$LIB_HDFS:$LD_LIBRARY_PATH \
   TensorFlowOnSpark/examples/mnist/keras/mnist_mlp_estimator.py \
   --input_mode spark \
   --images hdfs:///TensorFlowOnSpark/mnist/csv/train/images \
   --labels hdfs:///TensorFlowOnSpark/mnist/csv/train/labels \
   --epochs 1 \
   --mode train \
   --model_dir  hdfs:///TensorFlowOnSpark/model \
   --export_dir  hdfs:///TensorFlowOnSpark/export \
   --steps 100
```

