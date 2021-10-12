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



#### 2.3、聚合分析

待续



### 三、配置（Configuration）

