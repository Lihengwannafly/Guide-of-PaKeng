### TensorFlow Serving



#### 1.部署TensorFlow Serving

墙裂推荐docker安装

#### 2.TensorFlow Estimator导出模型

需要先定义**serving_input_receiver_fn**

```
def serving_input_receiver_fn():
    features = {'user_id': tf.placeholder(dtype=tf.int32, shape=[None], name='user_id'),
                'item_id': tf.placeholder(dtype=tf.int32, shape=[None], name='item_id')}

    return tf.estimator.export.ServingInputReceiver(features, features)
```

然后在主函数中**tf.estimator.train_and_evaluate**之后，添加

```
export_dir = nn.export_savedmodel('export', serving_input_receiver_fn)
```

这是因为Estimator在读取数据的时候使用tf.data将数据转化为Tensor。但是部署到serving上不支持tf.data，仍然需要使用feed_dict这种方式把数据喂给模型进行计算，因此需要首先创建**serving_input_receiver_fn**定义输入的placeholer。

#### 3加载模型

这个是模型导出的文件夹**export**的树结构：

```
1/
├── saved_model.pb
└── variables
    ├── variables.data-00000-of-00001
    └── variables.index
```

下面是加载模型的指令：

```
docker run -p 8501:8501 --mount type=bind,source=/root/tensorlfow_estimator/export,target=/models/neural_cf -e MODEL_NAME=neural_cf -t tensorflow/serving &
```

上面这条指令值得注意的有两点：

- 模型的路径一定是绝对路径
- 路径只需要写到**export**这一级，因为**export**下的**1**文件夹是版本号，否则会报警告。

#### 4.查询预测结果

```
 curl -XPOST http://localhost:8501/v1/models/neural_cf:predict -d '{"instances":[{"user_id":1,"item_id":6}]}'
```

PS：***'{"instances":[{"user_id":1,"item_id":6}]}'***最外面的引号一定是单引号，其余是双引号。

#### 5. TensorFlow Serving Client

待续

#### 6. Model on HDFS

训练完成后，模型保存在HDFS上。但是如果使用docker安装的tensorflow serving，无法连接到HDFS，因此需要将HDFS挂载到本地，我们可以使用Hadoop提供的HDFS NFS Gateway达到这一目的。

1. 如果没有安装***NFSGateways***可现在ambari UI上快速安装，我们在dn4上安装。

2. 安装完之后，在该节点上创建挂载目录，并执行指令：

   ```
   mkdir /tmp/hdfs-mount
   mount -t nfs -o vers=3,proto=tcp,nolock,sync,rsize=1048576,wsize=1048576 dn4:/ /tmp/hdfs-mount/
   ```

3. docker启动tensorflow serving

   ```
   docker run -p 8500:8500 --mount type=bind,source=/tmp/hdfs-mount/liheng/base-model-output/,target=/models/neural_cf -e MODEL_NAME=neural_cf -t tensorflow/serving &
   
   docker run -p 8501:8501 --mount type=bind,source=/tmp/hdfs-mount/liheng/base-model-output/,target=/models/neural_cf -e MODEL_NAME=neural_cf -t tensorflow/serving &
   ```


#### 参考

[Using TensorFlow Serving with Docker]: https://github.com/tensorflow/serving/blob/master/tensorflow_serving/g3doc/docker.md

