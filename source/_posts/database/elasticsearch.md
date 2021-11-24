---
title: elasticsearch
date: 2021-07-15 15:59:27
tags: 数据库
---

### 一、介绍

>  分布式文档存储，采用JSON文档

+ 集群中多个节点，文档分布在整个集群，任何节点可以访问文档。

+ 存储文档后，**近实时**地编入索引并完成搜索。
+ 倒排索引。支持全文搜索、精确搜索等
+ 无模式的能力（不明确字段类型的情况，映射为合适的类型）
+ 可拓展性和弹性（集群 -> 节点 -> 分片、索引）
  + 主分片和副本分片
  + 主分片创建时固定，副本分片可以随身更改



### 二、start searching

#### 2.1、创建索引和文档

customer：索引名称，不存在自动创建

_doc：创建文档，1为id

{"name": "zcx"}：添加的文档内容

```js
PUT /customer/_doc/1
{
	"name": "zcx",
    "settings" : {	// 设置
        "index" : {
            "number_of_shards" : 3, 	// 分片数量
            "number_of_replicas" : 2 	// 副分片数量
        }
    },
    "mappings" : {	// 配置文档的属性和类型
        "_doc" : {
            "properties" : {
                "field1" : { "type" : "text" }
            }
        }
    },
    "aliases" : {	// 添加别名
        "alias_1" : {},
        "alias_2" : {
            "filter" : {
                "term" : {"user" : "kimchy" }
            },
            "routing" : "kimchy"
        }
    }    
}
```



#### 2.2 search

```js
// 查询索引是否存在
head /customer
```



```js
GET /customer/_search
{
  "query": {
      "match": {"address":"mill lane"}	// 匹配到mill或者lane
      "match_phrase": {"address": "mill lane"}	// 匹配到mill lane这条语句
      "bool": {"must":[], "must_not":[], "should":[]}
  },	// 查询，类似where
  "sort": [],	// 排序
  "from": 0,	// 分页
  "size": 20
}

response：
{
  "took" : 3,
  "timed_out" : false,
  "_shards" : {
    "total" : 5,
    "successful" : 5,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : 1,
    "max_score" : 1.0,
    "hits" : [
      {
        "_index" : "customer",
        "_type" : "_doc",
        "_id" : "1",
        "_score" : 1.0,
        "_source" : {
          "name" : "zcx",
          "sno" : "31701028"
        }
      }
    ]
  }
}
```

+ `took`：执行时间（毫秒）
+ `time_out`：请求是否超时
+ `_shards`：分片总数、成功、失败、跳过
+ `max_score`：最相关文档的总数
+ `hits`：命中



```js
// 精准查询schools索引中，username=zcx的数据
GET schools/_search?q=username:zcx
```



在 index 后添加 `_stats`，获取索引相关的统计数据

```js
GET /school/_stats

response：
{
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "_all" : {
    "primaries" : {
      "docs" : {
        "count" : 0,
        "deleted" : 0
      },
      "shard_stats" : {
        "total_count" : 1
      },
      "store" : {
        "size_in_bytes" : 230,
        "total_data_set_size_in_bytes" : 230,
        "reserved_in_bytes" : 0
      },
      "indexing" : {
        "index_total" : 0,
        "index_time_in_millis" : 0,
        "index_current" : 0,
        "index_failed" : 0,
        "delete_total" : 0,
        "delete_time_in_millis" : 0,
        "delete_current" : 0,
        "noop_update_total" : 0,
        "is_throttled" : false,
        "throttle_time_in_millis" : 0
      },
      "get" : {
        "total" : 0,
        "time_in_millis" : 0,
        "exists_total" : 0,
        "exists_time_in_millis" : 0,
        "missing_total" : 0,
        "missing_time_in_millis" : 0,
        "current" : 0
      },
      "search" : {
        "open_contexts" : 0,
        "query_total" : 0,
        "query_time_in_millis" : 0,
        "query_current" : 0,
        "fetch_total" : 0,
        "fetch_time_in_millis" : 0,
        "fetch_current" : 0,
        "scroll_total" : 0,
        "scroll_time_in_millis" : 0,
        "scroll_current" : 0,
        "suggest_total" : 0,
        "suggest_time_in_millis" : 0,
        "suggest_current" : 0
      },
  },
  。。。省略
}
```



索引除了持久化到事务日志，也会持久化到 `Lucene`。数据恢复速度加快

```js
POST school/_flush
```





#### 2.3、聚合分析

待续







### 三、Mapping

#### 3.1、Dynamic Mapping

对于文档中未定义的字段，可以通过动态映射来定义是否动态添加。

类型通过字段的值动态推算，需要注意的几个：

| `JSON DataType` | `Elasticsearch datatype`    |
| --------------- | --------------------------- |
| integer         | long                        |
| array           | 第一个非空的value的类型数组 |
| string          | text                        |
| 浮点数          | float                       |

```js
PUT my_index
{
  "mappings": {
    "_doc": {
	  "dynamic": true		// 是否开启 动态Mapping（false、true、version7——runtime）
      "date_detection": true					// 是否开启date类型转换
      "dynamic_date_formats": ["MM/dd/yyyy"]	// 动态将字符串转为date类型，这样只有这种格式会转换为date类型
    }
  }
}
```



#### 3.2 Dynamic templates

```js
PUT my_index
{
  "mappings": {
    "_doc": {
      "dynamic_templates": [
        // long -> integer
        {
          "integers": {
            "match_mapping_type": "long",
            "mapping": {
              "type": "integer"
            }
          }
        },
        // 匹配到field name如果是long开头，不以text结束的类型：string -> long，否则为string类型
        {
          "longs_as_strings": {
            "match_mapping_type": "string",
            "match":   "long_*",
            "unmatch": "*_text",
            "mapping": {
              "type": "long"
            }
          }
        },
        // 正则匹配field name以profit开头+一串数字的类型：string -> long，否则为string类型
        {
          "longs_as_pattern_strings": {
            "match_mapping_type": "string"
            "match_pattern": "regex",
            "match": "^profit_\\d+$",
            "mapping": {
              "type": "long"
            }
          }
        }
      ]
    }
  }
}

```



### 四、index

打开和关闭索引

+ 关闭索引：只显示元数据，不能够读写数据
+ 打开索引：允许读写（正常操作）





### 五、`Cat APIs`

> `_cat` + index/nodes/templates，一般以 `JSON` 的形式返回数据类型，这里以表格的形式展示

```js
GET /_cat/indices

response:
healthy status index						   uuid					  pri rep docs.count docs.deleted store.size pri.store.size
green   open   .geoip_databases                F0S_AfxbS4qnRfmUZYGyBg 1   0   42         39           40.6mb     40.6mb
yellow open    my_index_01                     HU0-D83QQMasMtdlfPIUAw 1   1   0          0            230b       230b
yellow open  chapter                         nouKVwjsToaPqQR4QS4D-w 1 1  3     0  8.7kb  8.7kb
yellow close my_index_02                     to-6fbc0QsyMsBuwyOELKA 1 1                       
green  open  .apm-custom-link                zCu-FR7oR3CLVvdEp-JSRQ 1 0  0     0   208b   208b
yellow open  schools                         8YMa4HV0T1CxvZSVcEVoWw 1 1  1     0  4.6kb  4.6kb
green  open  .kibana-event-log-7.15.2-000001 WloRRcBOSfOmeS-Sc8N3vw 1 0  2     0 11.9kb 11.9kb
green  open  .apm-agent-configuration        Y9bRJgbiR06TwsQzCQqG4g 1 0  0     0   208b   208b
green  open  .kibana_pre6.5.0_001            L8NwjI1bSPGfA7XfPVWGgA 1 0  1     0  5.5kb  5.5kb
green  open  .kibana_7.15.2_001              7nAFYuyRSa6Tj5tGbM85bg 1 0 47    16  2.3mb  2.3mb
green  open  .tasks                          1zbUv88mTim-0--gLetpTQ 1 0  4     0 27.2kb 27.2kb
green  open  .kibana_task_manager_7.15.2_001 Adr8ANIjRiWtSbFVo2PGGg 1 0 15 11785  1.5mb  1.5mb
```



### 六、`cluster APIs`

> 集群相关的 `API`，/_nodes + address / _local

```
GET /_nodes/_local

response:
{
  "_nodes" : {
    "total" : 1,
    "successful" : 1,
    "failed" : 0
  },
  "cluster_name" : "elasticsearch",
  "nodes" : {
    "IebqG-UeTgykeuDuZsMz-w" : {
      "name" : "SK-20210419SSDQ",
      "transport_address" : "127.0.0.1:9300",
      "host" : "127.0.0.1",
      "ip" : "127.0.0.1",
      "version" : "7.15.2",
      "build_flavor" : "default",
      "build_type" : "zip",
      "build_hash" : "93d5a7f6192e8a1a12e154a2b81bf6fa7309da0c",
      "total_indexing_buffer" : 103795916,
      "roles" : [
        "data",
        "data_cold",
        "data_content",
        "data_frozen",
        "data_hot",
        "data_warm",
        "ingest",
        "master",
        "ml",
        "remote_cluster_client",
        "transform"
      ],
  },
  ...
}
```



> 检索当前 hot thread 的节点信息

```js
GET /_nodes/hot_threads
```



### 七、`Ingest Node`

#### 7.1、 `Pipeline`

> 管道（可以类比 Netty，对数据插入进行一系列操作）

```js
// 创建 pipeline
PUT _ingest/pipeline/test
{
  "processors": [
    {
      "lowercase": {
        "field": "username"
      }
    }
  ]
}

// pipeline 应用到索引
PUT my_index/_doc/1?pipeline=test

```

