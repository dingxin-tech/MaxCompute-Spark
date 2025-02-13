# 引用外部文件问题
需要引用到外部文件的场景
  + 用户作业需要读取一些配置文件
  + 用户作业需要额外的jar包/Python库

<h1 id="1">如何上传文件</h1>

上传文件有两种方式
* 通过Spark参数上传文件
* 通过MaxCompute Resource上传文件

## Spark参数
MaxCompute Spark支持Spark社区版原生的--jars，--py-files等参数，可以在作业提交时通过这些参数将文件上传，这些文件在任务运行时会被上传到用户的工作目录下。

在不同的运行模式下上传文件：
* 通过Spark客户端：直接使用spark-submit命令行参数
<pre>
**注意事项**
* --jars选项，会将配置的jar包上传至Driver和Executor的当前工作目录，多个文件逗号分隔，这些jar包都会加入Driver和Executor的Classpath，Spark作业中直接"./your_jar_name"即可引用，与社区版Spark行为相同。
* --files, --py-files选项，会将配置的 普通文件/python文件 上传至Driver和Executor的当前工作目录，多个文件逗号分隔，Spark作业中直接"./your_file_name"即可引用，与社区版Spark行为相同。
* --archives选项，与社区版Spark行为<b>略有不同</b>，多个逗号分隔，配置方式为xxx#yyy，会将配置的归档文件（例如.zip）解压到Driver和Executor的当前工作目录的<b>子目录</b>中。举例：当配置为xx.zip#yy时，应以"./yy/xx/"引用到归档文件中的内容；当仅配置xx.zip时，应以"./xx.zip/xx/"引用到内容。若一定需要将归档内容直接解压到当前目录，即直接引用"./xxx/"，请使用下面提到的<b>spark.hadoop.odps.cupid.resources</b>配置。
</pre>

* 通过DataWorks添加任务需要的资源，参见[文档](https://github.com/aliyun/MaxCompute-Spark/wiki/02.-Spark-on-Dataworks)

## MaxCompute Resource
MaxCompute Spark提供spark.hadoop.odps.cupid.resources参数，可以直接引用MaxCompute中的资源，这些资源在任务运行时会被上传到用户的工作目录下。

使用方式
```
1. 通过MaxCompute客户端将文件上传(单个文件最大支持500MB)
2. 在Spark作业配置中添加spark.hadoop.odps.cupid.resources参数
   格式为<projectname>.<resourcename>，如果需要引用多个文件，需要用逗号隔开
```
### spark.hadoop.odps.cupid.resources参数介绍
  + **配置说明** `该配置项指定了任务运行所需要的`[Maxcompute资源](https://help.aliyun.com/document_detail/27831.html?spm=5176.11065259.1996646101.searchclickresult.d55650ea0QU1qd&aly_as=45TiiTdO2)
  + **配置示例** spark.hadoop.odps.cupid.resources=public.python-python-2.7-ucs4.zip,public.myjar.jar
  + **使用说明** `指定的资源将被下载到driver和executor的当前工作目录，资源下载到工作目录后默认的名字是<projectname>.<resourcename>`
  + **文件重命名** `在配置时通过<projectname>.<resourcename>:<newresourcename>进行重命名`
  + **重命名示例** spark.hadoop.odps.cupid.resources=public.myjar.jar:myjar.jar
  + **注意** `该配置项必须要配置在spark-default.conf中或dataworks的配置项中才能生效，而不能写在代码中`
  
  
<h1 id="2">如何在代码中引用文件</h1>
通过上述两种方式可以将文件上传到任务的当前工作目录，文件读取示例：

```
val targetFile = "文件名"
val file = Source.fromFile(targetFile)
for (line <- file.getLines)
    println(line)
file.close
```