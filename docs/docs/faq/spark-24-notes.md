# Spark 2.4.5 使用注意事项
## 如何使用Spark 2.4.5提交作业
* 直接使用Yarn-cluster模式在本地提交任务, 添加 spark.hadoop.odps.spark.libs.public.enable=true和spark.hadoop.odps.spark.version=spark-2.4.5-odps0.34.0 这两个参数可以加速包上传速度

* 或在Dataworks中配置参数 spark.hadoop.odps.spark.version=spark-2.4.5-odps0.34.0，注意，若Dataworks独享资源组尚未升级到Spark 2.4.5，用户可以采用公共资源组进行调度，或联系Dataworks平台官方人员进行升级

## Spark 2.4.5 使用变化
* 如果使用Yarn-cluster模式在本地提交任务，需要新增环境变量 export HADOOP_CONF_DIR=$SPARK_HOME/conf

* 如果使用local模式进行调试，需要在$SPARK_HOME/conf目录下新建odps.conf文件，并添加以下配置：
```
odps.project.name = 
odps.access.id = 
odps.access.key =
odps.end.point =
```

## Spark 2.4.5 参数配置变化

* `spark.sql.catalogImplementation`
  + **配置值** `hive`
* `spark.sql.sources.default`
  + **配置值** `hive`
* `spark.sql.odps.columnarReaderBatchSize`
  + **默认值** `4096`
  + **配置说明**  `向量化读每个batch包含的行数`
* `spark.sql.odps.enableVectorizedReader`
  + **默认值** `true`
  + **配置说明**  `开启向量化读`
* `spark.sql.odps.enableVectorizedWriter`
  + **默认值** `true`
  + **配置说明**  `开启向量化写`
* `spark.sql.odps.split.size`
  + **默认值** `256m`
  + **配置说明**  `该配置可以用来调节读Maxcompute表的并发度，默认每个分区为256MB`
* `spark.hadoop.odps.cupid.vnet.capacity`
  + **默认值** `802`
  + **配置说明**  `该配置用于设置最大的instance数量，建议配置值为spark.executor.instances + 2，否则可能会遇到create virtual net failed错误。该参数需要设置到spark-defaults.conf或Dataworks配置项中`



