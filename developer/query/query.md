# 查询 Tindex-Query-Json


- [Timeseries](#Timeseries)
- [GroupBy](#GroupBy)
- [NumericGroup](#NumericGroup)
- [CustomGroup](#CustomGroup)
- [NumericGroupBy](#NumericGroupBy)
- [Select](#Select)
- [Search](#Search)
- [TimeBoundary](#TimeBoundary)
- [SegmentMetadata](#SegmentMetadata)
- [UserGroup](#UserGroup)
- [Scan](#Scan)
- [FirstN](#FirstN)
- [MultiHaving](#MultiHaving)
- [Funnel](#Funnel)
- [Retention](#Retention)



`Query`，即查询。`Druid`包含多种查询类型。
> http-post请求地址格式为:http://{broker_ip:broker_port}/druid/v2/?pretty    
eg  http://192.168.0.223:8082/druid/v2/?pretty

## <a id="Timeseries" href="Timeseries"></a>  1. `Timeseries`


对于需要统计一段时间内的汇总数据，或者是指定时间粒度的汇总数据，`Druid`通过`Timeseries`来完成。


查询语句如下：

```
{
    "queryType": "lucene_timeseries",
    "dataSource": "userinfo",
    "intervals": "1000/3000",
    "granularity": " all",
    "context": {
        "timeout": 180000,
        "useOffheap": true,
        "groupByStrategy": "v2"
    },
    "aggregations": [
        {
            "name": "__VALUE__",
            "type": "lucene_count"
        }
    ]
}
```

相当于`SQL`语句的：`select count(*) from userinfo`

输出可能如下：
```
[
    {
        "timestamp": "2017-01-01T00:00:00.000Z",
        "result": {
            "__VALUE__": 100000
        }
    }
]
```

`Timeseries`查询包含如下部分。

字段名 | 描述 | 是否必须
---|---|---
queryType  | 对于Timeseries查询，该字段的值必须是lucene_timeseries | 是
dataSource | 要查询数据集`dataSource`名字。详见[`dataSource`](/developer/query/#dataSource) | 是
intervals  | 查询时间区间范围，ISO-8601 格式。详见[`interval`](/developer/query/#interval) | 是
granularity | 查询结果进行聚合的时间粒度 | 是
filter | 过滤条件。详见[`filter`](/developer/query/#filter) | 否
aggregations | 聚合。详见[`aggregation`](/developer/query/#aggregation) | 是
postAggregations | 后期聚合。详见[`post-aggregation`](/developer/query/#post-aggregation) | 否
descending | 是否降序 | 否
context | 指定一些查询参数，如结果是否进缓存等 | 否

`Timeseries`输出每个时间粒度内指定条件的统计信息，通过`filter`指定过滤条件，通过`aggregations`和`postAggregations`指定聚合方式。

`Timeseries`不能输出维度信息，`granularity`支持`all`,`none`,`second`,`minute`,`fifteen_minute`,`thirty_minute`,`hour`,`day`,`week`,`month`,`quarter`,`year`。

- `all`，汇总为1条输出。
- `none`，不推荐使用。
- 其他的，则输出相应粒度的统计信息。



## <a id="TopN" href="TopN"></a> 2. `TopN`
`TopN`返回指定维度和排序字段的有序`top-n`序列。`TopN`支持返回前N条记录，并支持指定`Metric`为排序依据。


查询示例如下：


```
{
    "queryType": "lucene_topN",
    "dataSource": "userinfo",
    "intervals": "1000/3000",
    "granularity": "all",
    "dimension":"userId",
    "threshold":3,
    "metric":{
        "type":"numeric",
        "metric":"sum(age)"
    },
    "aggregations":[
        {
            "name": "sum(age)",
            "type": "lucene_doubleSum",
            "fieldName": "age"
        }
    ]
}
```
`TopN`查询包含以下部分：

字段名 | 描述 | 是否必须
---|---|---
queryType  | 对于`TopN`查询，该字段的值必须是`lucene_topN` | 是
dataSource | 要查询数据集`dataSource`名字。详见[`dataSource`](/developer/query/#dataSource) | 是
intervals  | 查询时间区间范围，`ISO-8601`格式。详见详见[`interval`](/developer/query/#interval) | 是
granularity | 查询结果进行聚合的时间粒度 | 是
filter | 过滤条件。详见[`filter`](/developer/query/#filter) | 否
aggregations | 聚合。详见[`aggregation`](/developer/query/#aggregation) | 是
postAggregations | 后聚合器。详见[`post-aggregation`](/developer/query/#post-aggregation)| 否
dimension | 进行`TopN`查询的维度，一个`TopN`查询指定且只能指定一个维度。详见[`dimension`](/developer/query/#dimension) | 是
threshold | `TopN`的 N 取值 | 是
metric | 进行统计并排序的`Metric`| 是
context | 指定一些查询参数，如结果是否进缓存等 | 否

- `metric`: `TopN`专属，指定排序依据。它有如下使用方式：

```
"metric":"<metric_name>" //默认方式，升序排序
```

```
"metric":{
    "type":"numeric",   //指定按照numeric 降序排列
    "metric":"<metric_name>"
}
```

```
"metric":{
    "type":"inverted",     //指定按照numeric 升序排列
    "metric":"<delegate_top_n_metric_spec>"
}
```

```
"metric":{
    "type":"lexicographic", //指定按照字典序排序
    "previousStop":"<previousStop_value>", //如b,按照字典序，排到b开头的为止
}
```
```
"metric":{
    "type":"alphaNumeric",  //指定数字排序
    "previousStop":"<previousStop_value>"
}
```

需要注意的是，`topN`是一个近似算法，每一个`Segment`返回前1000条进行合并得到最后的结果，如果`dimension`的基数在1000以内，则是准确的，超过1000就是近似值。


## <a id="GroupBy" href="GroupBy"></a> 3. `GroupBy`
### 3.1 `GroupBy`

`GroupBy`类似于`SQL`中的`group by`操作，能对指定的多个维度进行分组，也支持对指定的维度进行排序，并输出`limit`行数。同时，支持`having`操作。

查询示例如下：

```
{
    "queryType": "lucene_groupBy",
    "dataSource": "userinfo",
    "intervals": "1000/3000",
    "granularity": "all",
    "context": {
        "timeout": 180000,
        "useOffheap": true,
        "groupByStrategy": "v2"
    },
    "dimensions": [
        {
            "type": "default",
            "dimension": "province",
            "outputName": "province"
        }
    ],
    "aggregations": [
        {
            "name": "sum(age)",
            "type": "lucene_doubleSum",
            "fieldName": "age"
        }
    ],
    "limitSpec": {
        "type": "default",
        "columns": [
            {
                "dimension": "province"
            }
        ],
        "limit": 3
    }
}

```

相当于`SQL`语句的：`select province,sum(age) from userinfo group by province limit 3;`

查询的结果如下：
```
[
    {
        "v": "v1",
        "timestamp": "1000-01-01T00:00:00.000Z",
        "event": {
            "province": "上海市",
            "sum(age)": 56642
        }
    },
    {
        "v": "v1",
        "timestamp": "1000-01-01T00:00:00.000Z",
        "event": {
            "province": "云南省",
            "sum(age)": 57850
        }
    },
    {
        "v": "v1",
        "timestamp": "1000-01-01T00:00:00.000Z",
        "event": {
            "province": "内蒙古",
            "sum(age)": 56473
        }
    }
]
```

`GroupBy`查询包含以下部分：

字段名 | 描述 | 是否必须
---|---|---
queryType  | 对于`GroupBy`查询，该字段的值必须是`lucene_groupBy` | 是
dataSource | 要查询数据集`dataSource`名字。详见[`dataSource`](/developer/query/#dataSource) | 是
dimensions | 进行`GroupBy`查询的维度集合。详见[`dimension`](/developer/query/#dimension) | 是
limitSpec  | 对统计结果进行排序，取`limit`的行数 | 否
having     | 对统计结果进行筛选。详见[`having`](/developer/query/#having) | 否
granularity | 查询结果进行聚合的时间粒度 | 是
filter | 过滤条件。详见[`filter`](/developer/query/#filter) | 否
aggregations | 聚合。详见[`aggregation`](/developer/query/#aggregation) | 是
postAggregations | 后聚合器。详见[`post-aggregation`](/developer/query/#post-aggregation) | 否
intervals  | 查询时间区间范围，`ISO-8601`格式。详见[`interval`](/developer/query/#interval) | 是
context    | 指定一些查询参数，如结果是否进缓存等 | 否

`GroupBy`特有的字段为`limitSpec`和`having`。

- **limitSpec**

指定排序规则和`limit`的行数。`JSON`示例如下：
```
{
    "type":"default",
    "limit":<integer_value>,
    "columns":[list of OrderByColumnSpec]
}
```
其中`columns`是一个数组，可以指定多个排序字段，排序字段可以使`demension`或`metric`，指定排序规则的拼写方式：
```
{
    "dimension":"<Any dimension or metric name>",
    "direction":<"ascending"|"descending">
}
```
示例如下：
```
"limitSpec":{
    "type":"default",
    "limit":1000,
    "columns":[
        {
            "dimension":"visitor_count",
            "direction":"descending"
        },
        {
            "dimension":"click_visitor_count",
            "direction":"ascending"
        }
    ]
}
```

- **having**

 类似于`SQL`中的`having`操作，对`GroupBy`的结果进行筛选，详见[`having`](/developer/query/#having)。

> 以上`GroupBy`仅支持对`String`类型的维度进行分组,而以下的`NumericGroup`,`CustomGroup`,`numericGroupBy`则支持对数字维度(包括`int`,`float`,`double`类型)的分组

### <a id="NumericGroup" href="NumericGroup"></a> 3.2 `NumericGroup`
`NumericGroup`支持对数字维度按指定的间隔(间隔必须大于1)进行分组

查询示例如下:
```
{
    "queryType": "lucene_groupBy",
    "dataSource": "userinfo",
    "intervals": "1000/3000",
    "granularity": "all",
    "context": {
        "timeout": 180000,
        "useOffheap": true,
        "groupByStrategy": "v2"
    },
    "dimensions": [
        {
            "type": "numericGroup",
            "dimension": "age",
            "outputName": "groupAge",
            "interval":5,
            "min":10,
            "max":29
            
        }
    ],
    "aggregations": [
        {
            "name": "sum(average)",
            "type": "lucene_doubleSum",
            "fieldName": "average"
        }
    ],
    "limitSpec": {
        "type": "default",
        "columns": [
            {
                "dimension": "sum(average)"
            }
        ],
        "limit": 3
    }
}

```
查询的结果如下：
```
[
    {
        "v": "v1",
        "timestamp": "1000-01-01T00:00:00.000Z",
        "event": {
            "groupAge": "25.000000~29.000000",
            "sum(average)": 1248982.8500000006
        }
    },
    {
        "v": "v1",
        "timestamp": "1000-01-01T00:00:00.000Z",
        "event": {
            "groupAge": "20.000000~25.000000",
            "sum(average)": 1252797.83
        }
    },
    {
        "v": "v1",
        "timestamp": "1000-01-01T00:00:00.000Z",
        "event": {
            "groupAge": "15.000000~20.000000",
            "sum(average)": 1258379.9299999997
        }
    }
]
```
`NumericGroup`与`GroupBy`相比,只有`dimensions`里的`dimension`属性不同,它使用类型为`numericGroup`的`dimension`,详见[`dimension`](/developer/query/#dimension)

### <a id="CustomGroup" href="CustomGroup"></a> 3.3 `CustomGroup`
`CustomGroup`支持对数字维度进行自定义的间隔进行分组

查询示例如下:
```
{
    "queryType": "lucene_groupBy",
    "dataSource": "userinfo",
    "intervals": "1000/3000",
    "granularity": "all",
    "context": {
        "timeout": 180000,
        "useOffheap": true,
        "groupByStrategy": "v2"
    },
    "dimensions": [
        {
            "type": "customGroup",
            "dimension": "age",
            "outputName": "groupAge",
            "groups":[
            	{
            		"name": "children",
            		"lower": 10,
            		"upper": 18
            	},
            	{
            		"name": "young people",
            		"lower": 18,
            		"upper": 29
            	}
        	],
        	"outOfBound":true
            
        }
    ],
    "aggregations": [
        {
            "name": "sum(average)",
            "type": "lucene_doubleSum",
            "fieldName": "average"
        }
    ],
    "limitSpec": {
        "type": "default",
        "columns": [
            {
                "dimension": "sum(average)"
            }
        ],
        "limit": 3
    }
}

```
查询结果如下:
```
[
    {
        "v": "v1",
        "timestamp": "1000-01-01T00:00:00.000Z",
        "event": {
            "groupAge": "OUT_OF_BOUND",
            "sum(average)": 249973.47
        }
    },
    {
        "v": "v1",
        "timestamp": "1000-01-01T00:00:00.000Z",
        "event": {
            "groupAge": "children",
            "sum(average)": 2026039.5100000002
        }
    },
    {
        "v": "v1",
        "timestamp": "1000-01-01T00:00:00.000Z",
        "event": {
            "groupAge": "young people",
            "sum(average)": 2750655.9500000016
        }
    }
]
```
`CustomGroup`与`GroupBy`相比,只有`dimensions`里的`dimension`属性不同,它使用类型为`customGroup`的`dimension`,详见[`dimension`](/developer/query/#dimension)

### <a id="NumericGroupBy" href="NumericGroupBy"></a> 3.4 `NumericGroupBy`
`NumericGroupBy`支持对数字维度进行间隔为1的分组

查询实例如下:
```
{
    "queryType": "lucene_groupBy",
    "dataSource": "userinfo",
    "intervals": "1000/3000",
    "granularity": "all",
    "context": {
        "timeout": 180000,
        "useOffheap": true,
        "groupByStrategy": "v2"
    },
    "dimensions": [
        {
            "type": "numericGroupBy",
            "dimension": "age",
            "outputName": "groupAge"
            
        }
    ],
    "aggregations": [
        {
            "name": "sum(average)",
            "type": "lucene_doubleSum",
            "fieldName": "average"
        }
    ],
    "limitSpec": {
        "type": "default",
        "columns": [
            {
                "dimension": "sum(average)"
            }
        ],
        "limit": 3
    }
}

```
查询结果如下:
```
[
    {
        "v": "v1",
        "timestamp": "1000-01-01T00:00:00.000Z",
        "event": {
            "groupAge": "14",
            "sum(average)": 240744.52999999997
        }
    },
    {
        "v": "v1",
        "timestamp": "1000-01-01T00:00:00.000Z",
        "event": {
            "groupAge": "28",
            "sum(average)": 242116.0299999999
        }
    },
    {
        "v": "v1",
        "timestamp": "1000-01-01T00:00:00.000Z",
        "event": {
            "groupAge": "19",
            "sum(average)": 243802.67
        }
    }
]
```
`NumericGroupBy`与`GroupBy`相比,只有`dimensions`里的`dimension`属性不同,它使用类型为`numericGroupBy`的`dimension`,详见[`dimension`](/developer/query/#dimension)




## <a id="Select" href="Select"></a> 4. `Select`

`Select`类似于`SQL`中的`select`操作，`Select`用来查看`Druid`中存储的数据，并支持按照指定过滤器和时间段查看指定维度和`Metric`。能通过`descending`字段指定排序顺序，并支持分页拉取，但不支持`aggregations`和`postAggregations`。

`JSON`示例如下：
```
{
    "queryType": "lucene_select",
    "dataSource": "userinfo",
    "intervals": "1000/3000",
    "granularity": "all",
    "context": {
        "timeout": 180000,
        "useOffheap": true,
        "groupByStrategy": "v2"
    },
    "dimensions": [
        "province"
    ],
    "pagingSpec": {
        "pagingIdentifiers": {},
        "threshold": 3
    }
}

```

相当于`SQL`语句：`select province from userinfo limit 3;`

查询结果如下：　　
```
[
  {
    "timestamp": "2017-01-01T00:00:00.000Z",
    "result": {
      "pagingIdentifiers": {
        "userinfo_2017-01-01T00:00:00.000Z_2018-01-01T00:00:00.000Z_2017-05-17T08:08:34.224Z": 2
      },
      "events": [
        {
          "segmentId": "userinfo_2017-01-01T00:00:00.000Z_2018-01-01T00:00:00.000Z_2017-05-17T08:08:34.224Z",
          "offset": 0,
          "event": {
            "timestamp": "2017-05-15T07:54:40.918Z",
            "province": "宁夏"
          }
        },
        {
          "segmentId": "userinfo_2017-01-01T00:00:00.000Z_2018-01-01T00:00:00.000Z_2017-05-17T08:08:34.224Z",
          "offset": 1,
          "event": {
            "timestamp": "2017-05-15T07:54:42.481Z",
            "province": "贵州省"
          }
        },
        {
          "segmentId": "userinfo_2017-01-01T00:00:00.000Z_2018-01-01T00:00:00.000Z_2017-05-17T08:08:34.224Z",
          "offset": 2,
          "event": {
            "timestamp": "2017-05-15T07:54:43.780Z",
            "province": "内蒙古"
          }
        }
      ]
    }
  }
]
```
在`pagingSpec`中指定分页拉取的`offset`和条目数，在结果中会返回下次拉取的`offset`。`JSON`示例如下：
```
{
    "pagingSpec":{
        "pagingIdentifiers":{},
        "thershold":5,
        "fromNext":true
    }
}
```

## <a id="Search" href="Search"></a> 5. `Search`
`Search`查询返回匹配中的维度，类似于`SQL`中的`topN`操作，但是支持更多的匹配操作。`JSON`示例如下：
```
{
    "queryType":"lucene_search",
    "dataSource":"userinfo",
    "granularity":"day",
    "intervals": "1000/3000",
    "limit":1,
    "searchDimensions":[
        "province",
        "time"
    ],
    "sort":{
        "type":"lexicographic"
    }
}
```
- `searchDimensions`:搜索的维度

需要注意的是，`Search`只是返回匹配中维度，不支持其他聚合操作。如果要将`Search`作为查询条件进行`TopN`、`GroupBy`或`Timeseries`等操作，则可以在`filter`字段中指定各种过滤方式。`filter`字段也支持正则匹配。
查询结果如下：
```
[
    {
        "timestamp": "2017-01-01T00:00:00.000Z",
        "result": [
            {
                "dimension": "province",
                "value": "新疆",
                "count": 1
            },
            {
                "dimension": "province",
                "value": "青海省",
                "count": 1
            },
            {
                "dimension": "province",
                "value": "黑龙江",
                "count": 1
            }
        ]
    }
]
```

## 6. 元数据查询
`Druid`支持对`DataSource`的基础元数据进行查询。

### <a id="TimeBoundary" href="TimeBoundary"></a> 6.1 `TimeBoundary`
通过`TimeBoundary`可查询`DataSource`的最早和最晚的时间点，查询`JSON`示例如下：
```
{
    "queryType": "lucene_timeBoundary",
    "dataSource": "userinfo",
    "bound":"maxtime"
}
```
- `bound`：最小最大时间，`maxTime or minTime`

返回结果如下：
```
[
    {
        "timestamp": "2017-05-17T07:54:36.337Z",
        "result": {
            "maxTime": "2017-05-17T07:54:36.337Z"
        }
    }
]
```

### <a id="SegmentMetadata" href="SegmentMetadata"></a> 6.2 `SegmentMetadata`
通过`SegmentMetadata`可查询`Segment`的元信息，如有哪些`column`、`metric`、`aggregator`，查询`JSON`示例如下：
```
{
    "queryType": "lucene_segmentMetadata",
    "dataSource": "userinfo",
    "intervals": "1000/3000",
    "merge": true,
    "analysisTypes": [
        "aggregators"
    ],
    "lenientAggregatorMerge": true,
    "usingDefaultInterval": false,
    "context": {
        "timeout": 180000,
        "useOffheap": true,
        "groupByStrategy": "v2"
    }
}
```
相当与`SQL`语句的 `desc userinfo;`

返回结果如下：
```
[
    {
        "id": "userinfo_2017-01-01T00:00:00.000Z_2018-01-01T00:00:00.000Z_2017-05-17T08:08:34.224Z",
        "intervals": null,
        "columns": {
            "UserID": {
                "type": "STRING",
                "hasMultipleValues": false,
                "size": 0,
                "cardinality": 0,
                "minValue": null,
                "maxValue": null,
                "errorMessage": null
            },
            "__time": {
                "type": "LONG",
                "hasMultipleValues": false,
                "size": 0,
                "cardinality": null,
                "minValue": null,
                "maxValue": null,
                "errorMessage": null
            },
          ...
        },
        "size": 0,
        "numRows": 100000,
        "aggregators": null,
        "queryGranularity": null
    }
]
```
`segmentMetadata`支持更多的查询字段，不过这些字段都不是必须的，具体如下：

字段名 | 描述 | 是否必须
---|--- |---
toInclude | 可以指定哪些`column`在返回结果中呈现，可以填`all`,`none`,`list`| 否
merge | 将多个`Segment`的元信息合并到一个返回结果中 | 否
analysisTypes | 指定返回`column`的哪些属性，如`size`,`intervals`等 | 是
lenientAggregatorMerge | `true`或`false`，设置为`true`时，将不同的`aggregator`合并显示 | 否
context | 查询`Context`，可以指定是否缓存查询结果等 | 否

- `toInclude`的使用方式如下：
```
"toInclude":{"type":"all"}
"toInclude":{"type":"none"}
"toInclude":{"type":"list","columns":[<string list of column names>]}
```
- `analysisTypes`支持指定的属性：`cardinality`,`minmax`,`size`,`intervals`,`queryGranularity`,`aggregators`。

## <a id="UserGroup" href="UserGroup"></a> 7. `UserGroup`

是用户分群查询，支持将多维度和多指标作为分析条件，有针对性地根据你的需要建立分群。`JSON`示例如下:
```
{
    "queryType":"user_group",
    "dataSource":"userinfo",
    "granularity":"all",
    "intervals": "1000/3000",
    "filter": {
        "type": "selector",
        "dimension": "province",
        "value": "广东省"
    },
    "dimension":"age",
    "dataConfig": {
        "hostAndPorts":"153.214.0.1:8046",  
        "clusterMode":true,  
        "groupId":"1"  
    },
    "aggregations":[
        {
            "name": "sum(age)",
            "type": "lucene_doubleSum",
            "fieldName": "age"
        }
    ],
    "context":{
        "timeout": 180000,
        "useOffheap": true,
        "groupByStrategy": "v2"
    }
}
```
- `dataConfig.hostAndPorts` redis集群ip和端口，逗号或分号隔开
- `dataConfig.clusterMode` redis是否是集群模式
- `dataConfig.groupId` 用户分群id



## <a id="Scan" href="Scan"></a> 8. `Scan`
用来查询原始数据，`JSON`示例如下:

```
{
    "queryType": "lucene_scan",
    "dataSource": "wuxianjiRT",
    "resultFormat": "compactedList",
    "batchSize": 1,
    "limit": 2,
    "columns": [
        "Province",
        "UserID"
    ],
    "filter": {
        "type": "and",
        "fields": [
            {
                "type": "in",
                "dimension": "ClientDeviceBrand",
                "values": [
                    "HUWEI"
                ]
            }
        ]
    },
    "intervals": [
        "2011-01-01/2017-06-30"
    ]
}

```
- `limit` 总共返回的数量
- `batchSize` 每一批次返回的数量

查询结果如下：
```
[
    {
        "segmentId": "wuxianjiRT_2017-02-23T00:00:00.000Z_2017-02-24T00:00:00.000Z_2017-02-23T00:00:00.905Z_14",
        "columns": [
            "timestamp",
            "Province",
            "UserID"
        ],
        "events": [
            [
                "2017-02-23T16:00:06.616Z",
                "辽宁省",
                "b5b41fac0361d157d9673ecb926af5ae"
            ]
        ]
    },
    {
        "segmentId": "wuxianjiRT_2017-02-23T00:00:00.000Z_2017-02-24T00:00:00.000Z_2017-02-23T00:00:00.905Z_14",
        "columns": [
            "timestamp",
            "Province",
            "UserID"
        ],
        "events": [
            [
                "2017-02-23T16:00:07.244Z",
                "安微省",
                "7f100b7b36092fb9b06dfb4fac360931"
            ]
        ]
    }
]
```

## <a id="FirstN" href="FirstN"></a> 9. `FirstN`
查询某个维度的前N个值（不用排序，不重复），`JSON`示例如下:


```
{
    "queryType":"lucene_firstN",
    "dataSource":"userinfo",
    "dimension":"province",
    "threshold":5,
    "intervals": "1000/3000",
    "granularity":"all",
    "context":{
        "timeout": 180000,
        "useOffheap": true,
        "groupByStrategy": "v2"
    }
}
```

查询结果如下：
```
[
    {
        "timestamp": "2017-01-01T00:00:00.000Z",
        "result": [
            "黑龙江",
            "重庆市",
            "青海省",
            "新疆",
            "四川省"
        ]
    }
]

```
## <a id="MultiHaving" href="MultiHaving"></a> 10. `MultiHaving`
用于对`GroupBy`分组数据进行不同的`having`条件过滤，分组时能对指定的多个维度进行分组，每个`having`过滤结果可以经过多个`aggregatorSpecs`进行聚合，`JSON`示例如下：

```
{
    "queryType": "multi_having",
    "dataSource": "userinfo",
    "intervals": "1000/3000",
    "granularity": "all",
    "dimensions":["province"],
    "aggregations":  [
        {
            "type":"lucene_count",
            "name":"_count_"
        }
    ],
    "havingAggregators": [
        {
            "name": "__havingAggregators",
            "havingSpec": {
                "type": "greaterThan",
                "aggregation": "_count_",
                "value": 3000
            },
            "aggregatorSpecs": [
                {
                    "type": "lucene_count",
                    "name": "_count1_"
                }
            ]
        }
    ],
    "context":{
        "timeout": 180000,
        "useOffheap": true,
    	"groupByStrategy": "v2"
    }
}
```
相当于`SQL`语句的`select count(_count_) _count1_ from (select count(*) _count_ from userinfo group by province having _count_ > 3000)`

查询结果如下：
```
[
    {
        "v": "data_row",
        "event": {
            "data": {
                "__havingAggregators": {
                    "_count1_": 4
                }
            },
            "multiHavingTime": 1,
            "groupByTime": 36
        }
    }
]
```

`MultiHaving`特有的字段为`havingAggregators`。

- **havingAggregators**

指定一组不同的having过滤条件，以及对过滤结果的聚合方式。JSON示例如下：

```
[<HavingAggregator>,<HavingAggregator>,<HavingAggregator>...]
```

其中`HavingAggregator`指定一个单独的having过滤条件，以及对该过滤结果的聚合方式，它包含的字段如
下：

字段名 | 描述 | 是否必须
--- | --- | ---|
name | 指定返回结果的属性名 | 是
havingSpec | 对分组数据进行having过滤，详见[`having`](/developer/query/#having) | 是
aggregatorSpecs | 对having过滤后的结果进行聚合的方式 | 是


## <a id="Funnel" href="Funnel"></a> 11. `Funnel`   
`Funnel查询`也叫漏斗查询,该查询可以设置多个步骤,每个步骤都会在上一个步骤的结果中查询出符合条件的数据,最后将每个步骤的结果返回,`JSON`示例如下：
```json
{
  "queryType":"funnel",
  "dataSource":"rollup-normal-test5",
  "context":
  {
    "groupByStrategy":"v2",
    "useOffheap":true,
    "timeout":180000
  },
  "intervals":"2017-08-20T00:00:00.000Z/2017-08-22T23:59:59.999Z",
  "granularity":  {
    "type":"period",
    "period":"P2D"
  },
  "dimension":null,
  "field":"uid",
  "steps":[
    {
      "name":"第 1 步",
      "filter":"event:访问"
    },
    
    {
      "name":"第 2 步",
      "filter":"event:登录"
    }
  ],
  "slidingWindow":
  {
    "type":"period",
    "period":"P1D"
  }
}
```

查询结果:
```json
[
    {
        "v": "FunnelResultRow",
        "timestamp": "2017-08-20T00:00:00.000Z",
        "event": {
            "第 1 步": 527595.2736384192,
            "第 2 步": 459295.1286562972
        },
        "type": "window"
    },
    {
        "v": "FunnelResultRow",
        "timestamp": "2017-08-21T00:00:00.000Z",
        "event": {
            "第 1 步": 388632.1614757022,
            "第 2 步": 245955.00990854224
        },
        "type": "window"
    },
    {
        "v": "FunnelResultRow",
        "timestamp": "2017-08-22T00:00:00.000Z",
        "event": {
            "第 1 步": 390788.5262813083,
            "第 2 步": 247582.08635253785
        },
        "type": "window"
    },
    {
        "v": "FunnelResultRow",
        "timestamp": "2017-08-20T00:00:00.000Z",
        "event": {
            "第 1 步": 574921.653267394,
            "第 2 步": 516034.8059116008
        },
        "type": "total"
    }
]
```
- `granularity.period` 指定`Funnel`的查询粒度,`P2D`表示查询粒度为2天
- `dimension` 指定查询的分组维度
- `field` 指定用于基数统计的维度
- `steps`  指定漏斗的查询条件
- `steps.name`  指定查询条件的名称
- `steps.filter`  指定查询具体的过滤条件,用`lucene`表达式设置
- `slidingWindow` 指定查询的滑动窗口,即展示结果的时间间隔,目前仅支持设为一天,即`P1D`

## <a id="Retention" href="Retention"></a> 12. `Retention`
`Retention查询`也叫留存查询,它要先设置一个起始条件,然后设置一个转化条件.然后在起始条件的结果里进行转化条件的过滤,最后返回各个条件的查询结果,`JSON`示例如下： 

```json
{
  "queryType":"retention",
  "dataSource":"rollup-normal-test5",
  "context":
  {
    "groupByStrategy":"v2",
    "useOffheap":true,
    "timeout":180000
  },
  "intervals":"2017-08-20T00:00:00.000Z/2017-08-27T23:59:59.999Z",
  "granularity":
  {
    "type":"period",
    "period":"P3D"

  },
  "field":"uid",
  "startStep":
  {
    "name":"total",
    "filter":"event:访问 "
  },
  "returnStep":
  {
    "name":"total_return",
    "filter":"event:购买"
  }
}
```
查询结果:
```json
[
    {
        "timestamp": "2017-08-25T00:00:00.000Z",
        "result": {
            "total": 358710.1861640895,
            "2017-08-25T00:00:00.000Z": 246955.86689676947
        }
    },
    {
        "timestamp": "2017-08-22T00:00:00.000Z",
        "result": {
            "total": 571790.5760884841,
            "2017-08-22T00:00:00.000Z": 561162.0445494938,
            "2017-08-25T00:00:00.000Z": 391989.24259191006
        }
    },
    {
        "timestamp": "2017-08-19T00:00:00.000Z",
        "result": {
            "2017-08-19T00:00:00.000Z": 490870.6724267715,
            "total": 527595.2736384192,
            "2017-08-22T00:00:00.000Z": 519098.24003230926,
            "2017-08-25T00:00:00.000Z": 362174.31900703744
        }
    }
]
```

- `granularity.period` 指定查询粒度
- `field` 指定用于基数统计的维度
- `startStep.filter` 指定起始条件,用`lucene`表达式设置
- `returnStep.filter` 指定结束条件,用`lucene`表达式设置
