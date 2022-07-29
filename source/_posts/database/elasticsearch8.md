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

+ 

#### pipeline aggs
