# 动态资源伸缩问题
## Spark 2.4.5/3.1.1 支持动态资源伸缩
* 首先需要切换到spark-2.4.5-odps0.34.0版本

```
* 从Dataworks提交任务，需要添加配置：spark.hadoop.odps.spark.version=spark-2.4.5-odps0.34.0，从而切换到新的spark版本

* 从本地提交任务，需要添加以下两个配置：
  spark.hadoop.odps.spark.libs.public.enable=true
  spark.hadoop.odps.spark.version=spark-2.4.5-odps0.34.0

* spark-3.1.1采用客户端提交可以直接使用动态资源伸缩功能
```

* 此外需要添加以下spark参数：
```
spark.dynamicAllocation.shuffleTracking.enabled = true （默认 false）
spark.dynamicAllocation.shuffleTracking.timeout = XXXs （默认 Long.MaxValue MILLISECONDS）
spark.dynamicAllocation.enabled = true

参考文档：https://spark.apache.org/docs/3.0.0/configuration.html#dynamic-allocation
```