# hadoop接入
> 前提：  
> 1.部署好druid平台环境  
> 2.部署好hadoop环境

## 第一步：导入数据到hdfs：
在部署了hadoop环境的机器上进行操作。
1. 切换到hdfs用户，指令为:  
`su hdfs`
2. 在hdfs上创建存放数据的目录，指令为:  
`hadoop fs -mkdir -p dirName`  
例如，要把数据放到hdfs下的 /data1/tmp/druid 下，指令为:  
`hadoop fs -mkdir -p /data1/tmp/druid`
3. 将数据放到目录下，指令为：  
`hadoop fs -put fileName dirName`  
例如，将 /data/tmp/test.json 文件放到hdfs下的 /data1/tmp/druid 下，指令为:  
`hadoop fs -put /data/tmp/test.json /data1/tmp/druid`

## 第二步：创建json文件
创建、编辑、保存taskspec.json文件说明：
1. 按照数据的格式，创建taskspec.json文件，具体参数参照 [`task-spec.json`](#json)。  
2. 执行指令：  
`cd /data1/tmp/druid`  
进入该目录下，创建json文件。指令为  
`vim taskspec.json`  
将json内容拷贝，保存退出。

**说明**：taskspec.json 文件定义 csv 文件分2种，一种是有时间列，一种是没时间列，需要将json文件中的
```json
"timestampSpec":{
  "column":"ts",
  "format":"millis"
}
```  
改成：  
```json
"timestampSpec":{
  "missingValue":"2017-08-01T12:00:000Z"
}
```  
(固定时间，时间可改)。其中ts为表中时间列的列名。

## 第三步：执行命令,建立task
1. 进入存放 taskspec.json 文件的目录：  
`cd /data1/tmp/druid`  
2. 执行建立task的指令：  
```shell
curl -X 'POST' -H 'Content-Type:application/json' -d @task-spec.json http://{OverloadIP}:8090/druid/indexer/v1/task
```
- **`task-spec.json：`** json文件的名字  
- **`overlordIp`**： druid的overload节点ip地址


## 第四步：查看task的日志信息
1. 在浏览器上，查看 MiddleManagers 日志监控，如：  
`http://192.168.0.220:8090/console.html`  
这里的ip地址为 overload 节点ip地址。
2. 找到hadoop上传进程ID，点击 `log(all)` 查看 csv 上传过程的日志。

## 第五步：停止task（需要时再用）
在需要停止task时，可以发送如下http post请求停止task任务  
```shell
curl -X 'POST' -H 'Content-Type:application/json' http://{OverloadIP}:8090/druid/indexer/v1/task/{taskId}/shutdown
```  
- **`overlord_ip`**： druid的overload节点ip地址
- **`taskId`**： 在 `http://{OverloadIP}:8090/console.html` task详细页面对应 id 列的信息

## <a id="json" href="json"></a> task-spec.json详细配置如下：
```json
{
    "type": "lucene_index_hadoop",
    "spec": {
        "dataSchema": {
            "dataSource": "com_HyoaKhQMl_project_r1U1FDLjg",
            "parser": {
                "type": "string",
                "parseSpec": {
                    "format": "csv",
                    "timestampSpec": {
                        "column": "da",
                        "format": "millis"
                    },
                    "listDelimiter": ",",
                    "columns": [
                        "da",
                        "id",
                        "md5_str"
                    ],
                    "dimensionsSpec": {
                        "dynamicDimension": false,
                        "dimensions": [
                            {"name": "id","type": "int"},
                            {"name": "md5_str","type": "string"}
                        ]
                    }
                }
            },
            "metricsSpec": [],
            "granularitySpec": {
                "type": "uniform",
                "segmentGranularity": "MONTH",
                "queryGranularity": "NONE",
                "intervals": ["2015-05-21/2016-07-02"]
            }
        },
        "ioConfig": {
            "type": "hadoop",
            "inputSpec": {
                "type": "static",
                "paths": "/data1/tmp/test"
            }
        },
        "tuningConfig": {
            "type": "hadoop",
            "partitionsSpec": {
                "numShards": 3
            },
            "jobProperties": {
               "mapreduce.job.queuename": "root.default",
               "mapreduce.job.classloader": "true",
               "mapreduce.job.classloader.system.classes": "-javax.validation.,java.,javax.,org.apache.commons.logging.,org.apache.log4j.,org.apache.hadoop.,com.hadoop.compression"
           }
        }
    }
}
```
这个json是从hadoop上面接入csv文件的实例，这个csv文件的前五行数据如下：
```csv
1500629356487,0,cfcd208495d565ef66e7dff9f98764da
1500629107004,1,c4ca4238a0b923820dcc509a6f75849b
1500629183321,2,c81e728d9d4c2f636f067f89cc14862c
1500629285354,3,eccbc87e4b5ce2fe28308fd9f2a7baf3
1500628747465,4,a87ff679a2f3e71d9181a67b7542122c
```

- **`spec.dataSchema.parser.parseSpec.timestampSpec.column:`** 时间戳列

- **`spec.dataSchema.parser.parseSpec.timestampSpec.format:时间格式类型:`** 推荐`millis`  

  - **`yy-MM-dd HH:mm:ss.SSS:`** 自定义的时间格式
  - **`auto:`** 自动识别时间，支持`iso`和`millis`格式
  - **`iso:`** iso标准时间格式，如`2016-08-03T12:53:51.999Z`
  - **`posix:`** 从1970年1月1日开始所经过的秒数,10位的数字
  - **`millis:`** 从1970年1月1日开始所经过的毫秒数，13位数字

- **`spec.dataSchema.parser.parseSpec.listDelimiter:`** csv列分隔符  

- **`spec.dataSchema.parser.parseSpec.columns:`** 维度列表，包含时间戳列，如：`["ts","ProductID"]`  

- **`spec.dataSchema.parser.parseSpec.dimensionsSpec.dimensions:`** 维度定义列表，不包含时间戳列，每个维度的格式为：```{“name”: “name_string”, “type”:”type_string”}```。Type支持的类型：`string`、`int`、`float`、`long`、`date`

- **`spec.dataSchema.granularitySpec.intervals:`** 数据时间戳范围

- **`spec.dataSchema.granularitySpec.segmentGranularity:`** 段粒度，根据每天的数据量进行设置。  
小数据量建议DAY，大数据量（每天百亿）可以选择`HOUR`。可选项：`SECOND`、`MINUTE`、`FIVE_MINUTE`、`TEN_MINUTE`、`FIFTEEN_MINUTE`、`HOUR`、`SIX_HOUR`、`DAY`、`MONTH`、`YEAR`。

- **`spec.dataSchema.dataSource:`** 数据源的名称，类似关系数据库中的表名

- **`spec.ioConfig.type:`** 这里是从`hadoop`接入数据，固定为`hadoop`  
- **`spec.ioConfig.inputSpec.paths:`** 数据文件在hdfs上的目录


- **`spec.tuningConfig.partitionsSpec.numShards:`** 指定分片数
- **`spec.tuningConfig.jobProperties.mapreduce.job.queuename:`** 指定yarn上的队列名
- **`spec.tuningConfig.jobProperties.mapreduce.job.classloader:`** 如果为true，`task`会用 `druid` 中的 `hadoop-dependencies` ，而不是 `hadoop` 集群的配置

### json文件中需要特别注意的地方是一下几个参数：
1. **`spec.dataSchema.parser.parseSpec.columns:`** 包含时间戳列，并且先后顺序要和源数据对应
2. **`spec.dataSchema.parser.parseSpec.dimensionsSpec.dimensions:`** 不包含时间戳列，并且`name`要和`parser.parseSpec.columns`对应，`type`要和源数据对应起来。
3. **`spec.ioConfig.inputSpec.paths:`** 数据文件在hdfs上的目录，注意是否配置正确
4. 数据源有 date 类型时，要对 taskspec.json 文件 date 类型按要求定义：
    > Date类型格式 2017-01-01 00:00，则定义为："date","format":"yy-MM-dd HH:mm"  
    > Date类型格式 2017-01-01 00:00:00，则定义为："date","format":"yy-MM-dd HH:mm:ss"  
    > Date类型格式 2017-01-01，则定义为："date","format":"yy-MM-dd"  
    > Date类型格式 2017-01，则定义为："date","format":"yyyy-MM"  
    > Date类型格式 2017，则定义为："date","format":"yyyy"  
    > Date类型格式是从1970年1月1日开始所经过的秒数，10位的数字，则定义为："date","format":"posix"  
    > Date类型格式是从1970年1月1日开始所经过的毫秒数，13位数字，则定义为："date","format":"millis"  
5. 如果 `segmentGranularity` 即段粒度较小， `interval` 的范围也要较小，否则分片过多，非常耗时。  