---
title: elasticsearch8
date: 2022-07-29 16:11:24
tags: 数据库
---

# Elasticsearch8+

> ECE 认证工程师备考

## 安装

使用 `Ubuntu16 `系统

```shell
apt-get install elasticsearch
/etc/elasticsearch | /var/log/elasticsearch | /usr/share/elasticsearch
wget https://xxxx/elasticsearch.tar.gz
```



```shell
===================================================================================
elasticsearch.keystore 密文文件，不小心误删
===================================================================================
./bin/elasticsearch-keystore create
```



```shell
./elasticsearch-setup-passwords interactive

=====================================================================================
Failed to authenticate user 'elastic' against http://10.0.16.8:9200/_security/_authenticate?pretty
Possible causes include:
 * The password for the 'elastic' user has already been changed on this cluster
 * Your elasticsearch node is running against a different keystore
   This tool used the keystore at /home/ubuntu/elasticsearch-8.3.2/config/elasticsearch.keystore

You can use the `elasticsearch-reset-password` CLI tool to reset the password of the 'elastic' user
=====================================================================================

修改 elasticsearch.yml 的 xpack.security.enabled: false
删除 .security-7 索引
重新设置 xpack.security.enabled: true
```

```shell
===================================================================================
kibana.yml配置不支持使用elastic用户名，使用其他用户名
===================================================================================
比较合理的 elasticsearch-setup-password 脚本会设置 kibana_system 的用户和密码

elasticsearch.username: "kibana_system"
elasticsearch.password: "xxx"
```





## **Data Management**

### Define an index that satisfies a given set of requirements —— 定义一个满足条件的索引

+ analysis
+ setting
+ mapper

### Define and use an index template for a given pattern that satisfies a given set of requirements

```json
PUT index-test
{
  "mappings": {
  	// 默认为true
    "date_detection": false
    // 设置时间格式
    "dynamic_date_formats": ["MM/dd/yyyy"]
    // 开启number类型检测，整数检测为long类型，浮点数检测为float类型
    "numeric_detection": true
  }
}
```

```json
PUT test_index
{
  "mappings": {
    "dynamic_templates": [
      {
      	// 自定义名称
        "my_template": {
          // 匹配条件(match_mapping_type、match、unmatch、path:e.g 类似对象、path_unmatch)
          "match": "name*",
          "mapping": {
            "type": "keyword"
          }
        }
      }
    ]
  }
}
```




### [Define and use a dynamic template that satisfies a given set of requirements](https://www.elastic.co/guide/en/elasticsearch/reference/current/dynamic-templates.html)

> 8+版本后 `_template` 逐渐被废弃，改为 `_component_template` 和 `_index_template`

```json
PUT _index_template/my_template_name
{
	"template": {
		"mappings": {},
		"settings": {}
	}
}
```



### Define an Index Lifecycle Management policy for a time-series index

> kibana： Stack Management >>> Index Lifecycle Management  



### Define an index template that creates a new data stream

类似 ILM

need

+ a matching index template

+ a `@timestamp` field
+ 

原理：read all indices ( now + back) / write now/newest index

索引格式：`.ds-<data-stream>-<yyyy.MM.dd>-000001`

add a new document： not use `PUT /<target>/_doc/<_id>` ，use `PUT /<target>/_create/<_id>`



### Use the Data Visualizer to upload a text file into Elasticsearch（不考）

kibana >>> machine learning  >>> data visualizer





## Searching Data

### Write and execute a search query for terms and/or phrases in one or more fields of an index

**match、term、match_phrase的区别**

+ `match`查询会进行分词，匹配到单个即可
+ `term`查询不会分词，必须完全匹配（适用keyword类型，类似like）
+ `match_phrase`查询会分词，必须完全匹配（适用text类型，类似like）

##### Boolean query

```json
POST _search
{
  "query": {
    "bool" : {
      "must" : {
        "term" : { "field" : "value" }
      },
      "filter": {
        "term" : { "field" : "value" }
      },
      "must_not" : {
        "range" : {
          "age" : { "gte" : 10, "lte" : 20 }
        }
      },
      "should" : [
        { "term" : { "field1" : "value1" } },
        { "term" : { "field2" : "value2" } }
      ],
      "minimum_should_match" : 1,
      "boost" : 1.0
    }
  }
}
```

`minimum_should_match` 存在 should 时默认为1，否则默认为0

`boost`：计算分数时使用

`bool.filter`：不计算分数

`_name`：每一个top query都可以添加，response的`matched_queries`查看结果满足哪个query。

##### Boosting query & Constant score query

```json
GET /_search
{
  "query": {
    "boosting": {
      "positive": {
        "term": {
          "text": "apple"
        }
      },
      // 匹配的document减negative_boost的分数
      "negative": {
        "term": {
          "text": "pie tart fruit crumble tree"
        }
      },
      "negative_boost": 0.5
    },
    "constant_score": {
      // 只有 filter 这一种
      "filter": {
        "term": { "user.id": "kimchy" }
      },
      // 匹配到数据分数指定为boost值
      "boost": 1.2
    }
  }
}
```

##### Disjunction max query

```json
GET /_search
{
  "query": {
    "dis_max": {
      // 满足多个 query 匹配的文档，根据 tie_breaker 增加分数
      // 0 <= tie_breaker <= 1
      "tie_breaker": 0.4,
      "boost": 1.2,
      "queries": [
        {"match":{}},
        {"match":{}},
        ...
      ]
    }
  }
}
```

##### [Intervals query](https://www.elastic.co/guide/en/elasticsearch/reference/8.3/query-dsl-intervals-query.html)

精确控制查询的terms顺序、terms之间的距离以及包含关系的灵活控制

```json
GET /_search
{
  "query": {
    "intervals": {
      "message": {
        "all_of": {
          // 按顺序匹配 xxxvalue1xxxvalue2xxx可以被查询到，xxxvalue2xxxvalue1xxx的文档不能配查询
          "ordered": true,
          // value1和value2的间距，0代表只能匹配xxxvalue1value2xxx的文档
          "max_gaps": 0,
          "intervals": [
            {
              "match": {
                "query": "value1"
              }
            },
            {
              "match": {
                "query": "value2"
              }
            }
          ]
        }
      }
    }
  }
}
```

##### [Match query](https://www.elastic.co/guide/en/elasticsearch/reference/8.3/query-dsl-match-query.html)

> 匹配 text、number、date or boolean value

```json
GET _search
{
  "query": {
    "match": {
      "message": {
        "query": "cuury",
        // 模糊匹配 cuury 可以匹配到 curry
        "fuzziness": 1,
        // fuzzy前缀n位不更改
        "prefix_length": 2,
        // "a b" >>> or >>> a or b 
        // "a b" >>> and >>> a and b
        "operator": "or/and"
      }
    }
  }
}
```



##### [Match boolean prefix](https://www.elastic.co/guide/en/elasticsearch/reference/8.3/query-dsl-match-bool-prefix-query.html)

```json
GET /_search
{
  "query": {
    "match_bool_prefix" : {
      "message" : "quick brown f"
    }
  }
}
等价于
GET /_search
{
  "query": {
    "bool" : {
      "should": [
        { "term": { "message": "quick" }},
        { "term": { "message": "brown" }},
        { "prefix": { "message": "f"}}
      ]
    }
  }
}
```

##### match phrase query & match phrase prefix query

```json
GET my-index-00002/_search
{
  "query": {
    "match_phrase": {
      "field": "value"     
    }
  }
}
```

##### [Combined fields](https://www.elastic.co/guide/en/elasticsearch/reference/8.3/query-dsl-combined-fields-query.html)

```json
GET _search
{
  "query": {
    "combined_fields": {
      "query": "quility.com Brogan Dante",
      // 查询 city 和 email 字段，^2代表匹配这个字段的boost=2
      "fields": ["city^2", "email"],
      // 默认为 or，or/and
      "operator": "or"
    }
  }
}
```







nested query

### Write and execute a search query that is a Boolean combination of multiple queries and filters

```json
GET my-index-000001/_msearch
{}
{"query" : {"match_all" : {}}, "from" : 0, "size" : 10}
{}
{"query" : {"match_all" : {}}}
{"index" : "my-index-000002"}
{"query" : {"match_all" : {}}}
```

返回的 response 数组结果集，每个元素为一个查询结果




### search template



### Write an asynchronous search

异步搜索

```json
POST  my_index/_async_search
{
}

GET _sql/async/<id>
```



### Write and execute metric and bucket aggregations

+ 前置条件：【doc_values: true】

+ 不能用于`text`类型
+ 缓存，aggs执行会将高频率的数据进行缓存，使用`"size":0`有效减少不必要的缓存，缓存适用于相同的`preference string`

demo search aggs 如下

```json
// typed_keys: 原本return name, 现在return type#name
GET my_index/_search?typed_keys
{
	"size": 0,	// 只返回 aggs 结果
	"aggs": {
		"my-first-agg-name": {
			"type (terms)": {
				"field": "my-field"
			},
			// 元数据：元数据也会带上这个field
			"meta": {
       	 		"my-metadata-field": "foo"
      		},
			// sub aggs
			"aggs": {
				"my-first-sub-agg-name": {
				}
			}
		},
		// 多个 aggs, 返回多个 result
		"my-second-agg-name": {
		}
	}
}
```





#### metric aggs

计算指标：Max/Min、Average、Sum

不支持 sub-aggregations

+ [Boxplot](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-aggregations-metrics-boxplot-aggregation.html)

  ```json
  "boxplot": {
    "field": "load_time" 
  }
  
  // response
  "aggregations": {
    "load_time_boxplot": {
      "min": 0.0,
      "max": 990.0,
      // 第一个四分位数（前一半数据的中位数）
      "q1": 165.0,
      // 第二个四分位数（中位数）
      "q2": 445.0,
      // 第三个四分位数（后一半数据的中位数）
      "q3": 725.0,
  	"lower": 0.0,
      "upper": 990.0
    }
  }
  ```

  

+ 

#### bucket aggs

支持 sub-aggregations

+ [Adjacency matrix](https://www.elastic.co/guide/en/elasticsearch/reference/8.3/search-aggregations-bucket-adjacency-matrix-aggregation.html)

  【向量】filter A B C -> return A B C A&B A&C B&C

+ [Auto-interval date histogram](https://www.elastic.co/guide/en/elasticsearch/reference/8.3/search-aggregations-bucket-autodatehistogram-aggregation.html)

  自定义`buckets`，es自动划分需要区间

  ```json
  "auto_date_histogram": {
  	"field": "date",
      "buckets": 10
  }
  ```

+ [Children](https://www.elastic.co/guide/en/elasticsearch/reference/8.3/search-aggregations-bucket-children-aggregation.html)

  适用于`join`类型

  ```json
  "aggs": {
    "join-aggs": {
      "children": {
        "type": "answer"
      }
    }
  }
  ```

+ [Composite](https://www.elastic.co/guide/en/elasticsearch/reference/8.3/search-aggregations-bucket-composite-aggregation.html)

  组合，合并多个aggs查询，source1：A1、A2，source2：B1、B2 -> A1B1、A1B2、A2B1、A2B2 的查询结果

  ```json
  "aggs": {
      "my-aggs-name": {
        "composite": {
          "sources": [
            {
              "name01": {
                "type": {
                  "field": "field-name"
                }
              }
            },
            {
              "name02": {
                "type": {
                  "field": "field-name"
                }
              }
            }
          ]
        }
      }
    }
  ```

+ [Histogram](https://www.elastic.co/guide/en/elasticsearch/reference/8.3/search-aggregations-bucket-histogram-aggregation.html)

  > 直方图/柱状图
  >
  > bucket_key = Math.floor((value - offset) / interval) * interval + offset

  ```json
  "histogram": {
    "field": "field-name",
    // 间隔
    "interval": 5,
    // 过滤 count < min_doc_count 的区间
    "min_doc_count": 1
  }
  ```

+ [Date histogram](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-aggregations-bucket-datehistogram-aggregation.html)

  > date 作为 field type 的特殊 histogram
  >
  > bucket_key = Math.floor(value / interval) * interval

+ [Range](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-aggregations-bucket-range-aggregation.html)

  >  范围聚合查询，from <= value < to

  ```json
  "aggs": {
    "price_ranges": {
      "range": {
        "keyed": true,
        // 可以查询script field
        "field": "price",
        "ranges": [
          // 可以没有to，也可以没有from
          { "to": 100.0 },
          // 范围可以重叠
          { "from": 80.0, "to": 200.0 },
          { "from": 200.0 }
        ]
      }
    }
  }
  ```

  

+ [Date range](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-aggregations-bucket-daterange-aggregation.html)

  

+ [Global](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-aggregations-bucket-global-aggregation.html)

  > 定义一个单独包含全部数据的bucket，使得当前聚合不受query查询的影响（全部数据作为聚合目标）

  ```json
  POST /sales/_search?size=0
  {
    "query": {
      "match": { "type": "t-shirt" }
    },
    "aggs": {
      // 全部商品平均价格（match_all）
      "all_products": {
        "global": {}, 
        "aggs": {     
        "avg_price": { "avg": { "field": "price" } }
        }
      },
      //  t-shirt商品平均价格（query有关）
      "t_shirts": { "avg": { "field": "price" } }
    }
  }
  ```

  

+ [Missing](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-aggregations-bucket-missing-aggregation.html)

  缺少字段或者字段值为null的`document`缺少`bucket`，通过`missing`聚合统计

  ```json
  "aggs": {
    "products_without_a_price": {
      "missing": { "field": "price" }
    }
  }
  ```

  

+ [IP prefix](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-aggregations-bucket-ipprefix-aggregation.html) 和 [IP range](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-aggregations-bucket-iprange-aggregation.html)

  > IP地址相关的聚合，xxx.xxx.xxx.xxx

+ [Random sampler](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-aggregations-random-sampler-aggregation.html)

  > 随机取样聚合，0 < probability < 0.5 || probability = 1
  >
  > 随机取集合 * probability 的集合，每次结果可能不相同

  ```json
  {
    "aggregations": {
      "sampling": {
        "random_sampler": {
          "probability": 0.1
        },
        "aggs": {
          "price_percentiles": {
            "percentiles": {
              "field": "taxful_total_price"
            }
          }
        }
      }
    }
  }
  ```

  

+ 

#### pipeline aggs



### Other

#### CCR（Cross-cluster replication） & CCS（Cross-cluster search）

配置 `elasticsearch.yml`

```yaml
transport.host: xxx.xxx.xxx.xxx
transport.port: 9300
```

证书问题

1）使用相同的CA证书：复制 `ca.crt` 和 `ca.key`

2）使用不同证书



##### 创建集群远程连接

Stack Management >>> Remote Clusters

或通过 Dev Tools 实现

```json

PUT _cluster/settings
{
  "persistent": {
    "cluster.remote": {
      "remote_cluster": {
        "seeds": [
          "192.168.0.8:9300"
        ]
      }
    }
  }
}
```



##### 配置跨集群复制

Stack Management >>> Cross Cluster Replication

| 参数               | 说明                                   |
| ------------------ | -------------------------------------- |
| **Remote cluster** | 添加的集群名称                         |
| **Leader index**   | 待迁移的索引。                         |
| **Follower index** | 迁移数据生成的索引。索引名称不可重复。 |

