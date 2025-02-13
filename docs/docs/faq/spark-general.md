---
sidebar_position: 1
---

# Spark常见问题
本文档列举了一些常见问题以及错误码，用户可以根据关键字快速搜索本页面以找到对应的解决方法，如搜索不到对应问题，可联系开发团队支持。

参考 [如何查看Driver ERROR日志](https://github.com/aliyun/MaxCompute-Spark/wiki/05.-%E4%BD%9C%E4%B8%9A%E8%AF%8A%E6%96%AD)

## 项目工程自检 !!!重点推荐!!!

* pom.xml 检查

```
<dependency>
    <groupId>org.apache.spark</groupId>
    <artifactId>spark-core_${scala.binary.version}</artifactId>
    <version>${spark.version}</version>
    <scope>provided</scope> // spark-xxxx_${scala.binary.version} 依赖scope一定得是provided
</dependency>
```

* 主类代码 hard-code spark.master

```
val spark = SparkSession
      .builder()
      .appName("SparkPi")
      .config("spark.master", "local[4]") // 如果是yarn-cluster方式提交，代码里如果还有 local[N]的配置，将会出错
      .getOrCreate()
```

* 主类scala代码 必须是object不是class

```
object SparkPi { // 必须得是object，有的用户在IDEA创建文件的时候会不小心写成class，这样子main函数是无法加载的
  def main(args: Array[String]) {
    val spark = SparkSession
      .builder()
      .appName("SparkPi")
      .getOrCreate()
```

* 主类 hard-code配置

```
val spark = SparkSession
      .builder()
      .appName("SparkPi")
      .config("key1", "value1")
      .config("key2", "value2")
      .config("key3", "value3")
      ...  // 开发者在做local测试的时候习惯把MaxCompute的配置hard-code在代码里，但是有一些配置写在代码里是无法生效的
      .getOrCreate()

强烈建议yarn-cluster方式提交的时候，把配置项都写在spark-defaults.conf里面
```

## User signature dose not match

```
异常堆栈:
Invalid signature value - User signature dose not match

解决方法:
一般是spark-defaults.conf里的
spark.hadoop.odps.access.id
spark.hadoop.odps.access.key
填写有误导致的，请在阿里云 AccessKey 管理页面再次确认，确保复制粘贴的过程中没有出错
```

## You have NO privilege 'odps:CreateResource'

```
异常堆栈:
com.aliyun.odps.OdpsException: ODPS-0420095: 
Access Denied - Authorization Failed [4019], You have NO privilege 'odps:CreateResource' on {acs:odps:*:projects/*}

解决方法:
一般是用来提交Spark作业的accessId accessKey对应的云账号或者子账号没有对应权限导致的
找该项目的project owner授权目标用户的相关权限，授权语句如下

grant CreateInstance, CreateResource, ReadResource on PROJECT <projectName> TO USER <userId>;
详细授权语法参考
https://help.aliyun.com/document_detail/27935.html?spm=a2c4g.11186623.6.847.29642ab5zftWuB
```

## The task is not in release range: CUPID

```
异常堆栈:
The task is not in release range: CUPID

解决方案:
直接钉钉扫上文二维码，一般是因为region中没有开启MaxCompute Spark支持，联系开发人员该region的服务
```

## No space left on device

```
异常堆栈:
No space left on device

解决办法:
Spark使用网盘作为本地存储，driver和每个executor都有一个，Shuffle数据以及BlockManager溢出的数据均存储在网盘上。
网盘的大小通过参数spark.hadoop.odps.cupid.disk.driver.device_size控制，默认20g，最大100g。
如果调整到100g仍然会出现此错误，需要分析具体原因，常见案例：
1. 数据倾斜，在shuffle或者cache过程中数据集中分布在某些block；
2. 可以缩小单个executor的并发（spark.executor.cores），增加executor的数量(spark.executor.instances)。
```

## ClassNotFound类错误

```
异常堆栈:
java.lang.ClassNotFoundException: xxxx.xxx.xxxxx

解决办法:
1. jar -tf <作业jar包> | grep <类名称> 可初步断定你提交的jar包里是否有该类定义
2. 一般就是pom.xml的dependency没有写对，或者没有用shade包提交

PS. 提交spark作业的时候，请记得提交带 **shaded** 关键字的那个包，带shaded关键字表示把工程中的依赖都全部打包进来
```

## Jar包版本冲突类错误

```
异常堆栈:
User class threw exception: java.lang.NoSuchMethodError

解决办法:
1. 在$SPARK_HOME/jars路径下面尝试找出异常类在哪一个jar下面
2. 大致用命令 "grep <异常类类名> $SPARK_HOME/jars/*.jar" 可定位到三方库的坐标以及版本
3. 在spark作业工程根目录下，用命令 "mvn dependency:tree" 可以查看整个工程所有依赖
4. 找到对应的依赖引入后，可用maven dependency exclusions把引入的冲突包给排掉
5. 重新编译代码重新提交

具体mvn命令样例网上有很多说明，只需google或者baidu一下即可
```

## Shutdown hook called before final status was reported

```
现象:
提交作业后很快报错：App Status: SUCCEEDED, diagnostics: Shutdown hook called before final status was reported.

解决办法:
这个是由于提交到集群的user main并没有通过am申请集群资源，直接退出了。常见的情况有：用户没有new SparkContext；用户在代码里面设置spark.master为local。
```

## 中文乱码
```
现象:
运行spark作业打印的中文乱码解决方法

解决办法:
添加如下配置
"--conf" "spark.executor.extraJavaOptions=-Dfile.encoding=UTF-8 -Dsun.jnu.encoding=UTF-8"
"--conf" "spark.driver.extraJavaOptions=-Dfile.encoding=UTF-8 -Dsun.jnu.encoding=UTF-8"
如果是pyspark作业需要设置下如下两个参数：
spark.yarn.appMasterEnv.PYTHONIOENCODING=utf8
spark.executorEnv.PYTHONIOENCODING=utf8
另外在python脚本的最前面加上如下的代码：
# -*- coding: utf-8 -*-
import sys
reload(sys)
sys.setdefaultencoding('utf-8')
```

## 找不到Table或视图
```
现象:
Table or view not found:xxx

解决办法:
首先检查下报错的表在project是否真的存在。如果确实存在，则检查下是否打开了hive的catalog配置，需要去掉hive的配置。比如一个常见的导致报错的pyspark的写法如下：
spark = SparkSession.builder.appName(app_name).enableHiveSupport().getOrCreate()
需要把中间的enableHiveSupport去掉。
```

## com.aliyun.odps.cupid.CupidException: Row Not Found
```
现象:
com.aliyun.odps.cupid.CupidException: Row Not Found

原因：
该错误一般是由于用户代码里配置的project name，和实际运行作业的project name不一致导致的
需要注意：spark.hadoop.odps.project.name配置应当是运行spark作业的project名字，而不是访问表所在的project
```

## 如何把jar包当成资源来引用
答：spark.hadoop.odps.cupid.resources 可以把jar包当成resources传到项目里面，可以通过这个参数来指定需要引用的资源  例如spark.hadoop.odps.cupid.resources = projectname.xx0.jar,projectname.xx1.jar 。 这种是可以多个项目共享的。只要把权限设置好就行

## 如何调节读MaxCompute表的并发度？
答：配置split大小：spark.hadoop.odps.input.split.size，单位MB，默认是256. 如果表的数据量太小，也可以把数据读出来后通过repartition来调大后续的并发。

## 如何在代码中使用Maxcompute Java SDK？
答：确认以provided的方式添加了cupid-sdk的依赖，然后直接通过CupidSession.get().odps()获取Odps对象

## 如何Kill一个Spark任务？
答：通常通过两种方式kill正在运行的Spark任务
* 通过MaxCompute客户端执行 kill + instanceId; 
* 通过dataworks界面执行stop

注意：直接在spark客户端或者dataworks的任务提交界面执行Ctrl + C是无法kill一个Spark任务的

## com.aliyun.odps.cupid.ExceedMaxOpenFileException
动态分区插入报错问题
示例：com.aliyun.odps.cupid.ExceedMaxOpenFileException: NumPartitions: 1024, maxPartSpecNum: 1000, openFiles: 98, maxOpenFiles: 100000, exceeded Max limit，出现这个报错有两种情况：

(1)当NumPartitions * openFiles>maxOpenFiles会报这个错误，maxOpenFiles的默认参数是通过spark.hadoop.odps.cupid.open.file.max控制的，因此此时调节spark.hadoop.odps.cupid.open.file.max 只要大于NumPartitions*openFiles即可

(2)当openFiles > maxPartSpecNum时也会包这个错误，maxPartSpecNum的默认参数是通过spark.hadoop.odps.cupid.partition.spec.max控制的，此时调节spark.hadoop.odps.cupid.partition.spec.max只要大于openFiles即可。