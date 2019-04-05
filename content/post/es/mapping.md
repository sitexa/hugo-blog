+++
title = "es:es-mapping"
description = "ElasticSearch是一个基于Lucene的搜索服务器。它提供了一个分布式多用户能力的全文搜索引擎，基于RESTful web接口。"
tags = [
    "technology"
]
date = "2019-04-05"
categories = [
    "es"
]
+++

mapping是类似于数据库中的表结构定义，主要作用如下：

- 定义index下的字段名
- 定义字段类型，比如数值型、浮点型、布尔型等
- 定义倒排索引相关的设置，比如是否索引、记录position等

<!--more-->

# es-mapping

##  什么是Mapping?

mapping是类似于数据库中的表结构定义，主要作用如下：

- 定义index下的字段名
- 定义字段类型，比如数值型、浮点型、布尔型等
- 定义倒排索引相关的设置，比如是否索引、记录position等

##  查看mapping

```
GET /shakespeare/_mapping
```

``` 
{
  "shakespeare" : {
    "mappings" : {
      "line" : {
        "properties" : {
          "line_id" : {
            "type" : "long"
          },
          "line_number" : {
            "type" : "text",
            "fields" : {
              "keyword" : {
                "type" : "keyword",
                "ignore_above" : 256
              }
            }
          },
          "play_name" : {
            "type" : "text",
            "fields" : {
              "keyword" : {
                "type" : "keyword",
                "ignore_above" : 256
              }
            }
          },
          "speaker" : {
            "type" : "text",
            "fields" : {
              "keyword" : {
                "type" : "keyword",
                "ignore_above" : 256
              }
            }
          },
          "speech_number" : {
            "type" : "long"
          },
          "text_entry" : {
            "type" : "text",
            "fields" : {
              "keyword" : {
                "type" : "keyword",
                "ignore_above" : 256
              }
            }
          }
        }
      }
    }
  }
}
```

##  自定义mapping

``` 
PUT my_index
{
  "mappings":{
    "doc":{
      "properties":{
        "title":{
          "type":"text"
        },
        "name":{
          "type":"keyword"
        },
        "age":{
          "type":"integer"
        }
      }
    }
  }
}
```

mapping中的字段类型一旦设置，禁止直接修改，因为 lucene实现的倒排索引生成后不允许修改，应该重新建立新的索引，然后做reindex操作。

##  ```dynamic```参数

- true：默认值，表示允许选自动新增字段
- false：不允许自动新增字段，但是文档可以正常写入，但无法对字段进行查询等操作
- strict：严格模式，文档不能写入，报错 

``` 
PUT my_index2
{
  "mappings": {
    "doc": {
      "dynamic": false,
      "properties": {
        "title": {
          "type": "text"
        },
        "name": {
          "type": "keyword"
        },
        "age": {
          "type": "integer"
        }
      }
    }
  }
}
```

##  插入记录

``` 
PUT my_index/doc/1
{
  "title": "hello world",
  "desc": "this is book"
}
```

##  查询记录

``` 
GET my_index/doc/_search
{
  "query":{
    "match":{
      "title":"hello"
    }
  }
}
```
或者：
``` 
GET my_index/doc/_search
{
  "query": {
    "match": {
      "desc": "book"
    }
  }
}
```
结果都是：
``` 
{
  "took" : 0,
  "timed_out" : false,
  "_shards" : {
    "total" : 5,
    "successful" : 5,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : 1,
    "max_score" : 0.2876821,
    "hits" : [
      {
        "_index" : "my_index",
        "_type" : "doc",
        "_id" : "1",
        "_score" : 0.2876821,
        "_source" : {
          "title" : "hello world",
          "desc" : "this is book"
        }
      }
    ]
  }
}

```

##  ”dynamic”: strict模式：

``` 
PUT my_index
{
  "mappings": {
    "doc": {
      "dynamic": "strict",
      "properties": {
        "title": {
          "type": "text"
        },
        "name": {
          "type": "keyword"
        },
        "age": {
          "type": "integer"
        }
      }
    }
  }
}
```
插入记录：

``` 
PUT my_index/doc/3
{
  "name":"peng",
  "desc":"hello"
}
```

结果：
``` 
{
  "error": {
    "root_cause": [
      {
        "type": "strict_dynamic_mapping_exception",
        "reason": "mapping set to strict, dynamic introduction of [desc] within [doc] is not allowed"
      }
    ],
    "type": "strict_dynamic_mapping_exception",
    "reason": "mapping set to strict, dynamic introduction of [desc] within [doc] is not allowed"
  },
  "status": 400
}
```

##  查询计数

``` 
GET /web_document/doc/_count
{
    "query" : {
        "match" : { "url" : "http://www.gov.cn/" }
    }
}
```
