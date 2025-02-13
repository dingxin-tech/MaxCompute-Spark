# 访问VPC和OSS的问题
<h1 id="1">访问OSS常见问题</h1>

## 线上作业time out
   + 用户在local模式通常需要采用公网的oss域名，在线上需要改成inernal域名

## 依赖的问题
   + 示例中的hadoop-fs-oss必须以compile的方式打入用户主jar包
   + 如果用户使用PySpark，一定要把包含hadoop-fs-oss的fat jar包也上传到集群，否则出现找不到OSS相关类的问题
   
<h1 id="2">访问VPC常见问题</h1> 

## 线上作业time out
   + vpc.domain.list 需要压缩成一行：建议通过 [网站](http://www.bejson.com/) 进行压缩，不要有空格
   + 如果不是使用ENI专线，则需在要访问的服务中添加ip白名单，允许100.104.0.0/16网段的访问
   + 如果是使用ENI专线，需要在要访问的服务中添加安全组白名单（开通ENI专线时使用的安全组）
   + smartnat只有北京和上海region可用，且必须设置为true
   + 用户要保证所有可能访问到的IP都已经加到vpc.domain.list，例如如果用户要访问位于hdfs，hbase这种多个节点的服务，一定要把所有的节点都添加进来，不然肯定会遇到time out的情况
   
## 访问公网
目前MaxCompute Spark运行在网络隔离环境中，如果需要访问公网，只能通过以下两种方式：

* 中国公共云：
   + 提工单设置 project 级别白名单，如把 google.com:443 加到odps.security.outbound.internetlist 里面
   + 在Spark作业中配置公网访问白名单:spark.hadoop.odps.cupid.internet.access.list=google.com:443和spark.hadoop.odps.cupid.smartnat.enable=true

* 开通专线
   + 提工单开通专线，配置专线参数spark.hadoop.odps.cupid.eni.info和spark.hadoop.odps.cupid.eni.enable=true
   + 在Spark作业中配置公网访问白名单:spark.hadoop.odps.cupid.internet.access.list=google.com:443