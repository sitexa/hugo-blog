+++
title = "es:ElasticSearch应用介绍"
description = "ElasticSearch是一个基于Lucene的搜索服务器。它提供了一个分布式多用户能力的全文搜索引擎，基于RESTful web接口。"
tags = [
    "technology"
]
date = "2019-04-05"
categories = [
    "es"
]
+++

ElasticSearch应用介绍

<!--more-->
# ElasticSearch应用介绍

##  1，数据库搜索

数据库搜索是将数据库内容复制到Elasticsearch中进行索引，再利用elasticsearch进行搜索和展示。

##  2，logstash

Logstash 是开源的服务器端数据处理管道，能够同时从多个来源采集数据，转换数据，然后将数据发送到您最喜欢的 “存储库” 中。（我们的存储库当然是 Elasticsearch。）

### 输入

采集各种样式、大小和来源的数据

数据往往以各种各样的形式，或分散或集中地存在于很多系统中。 Logstash 支持各种输入选择 ，可以在同一时间从众多常用来源捕捉事件。能够以连续的流式传输方式，轻松地从您的日志、指标、Web 应用、数据存储以及各种 AWS 服务采集数据。

### 过滤器

实时解析和转换数据

数据从源传输到存储库的过程中，Logstash 过滤器能够解析各个事件，识别已命名的字段以构建结构，并将它们转换成通用格式，以便更轻松、更快速地分析和实现商业价值。

- 利用 Grok 从非结构化数据中派生出结构
- 从 IP 地址破译出地理坐标
- 将 PII 数据匿名化，完全排除敏感字段
- 简化整体处理，不受数据源、格式或架构的影响

我们的过滤器库丰富多样，拥有无限可能。

### 输出

选择您的存储库，导出您的数据
尽管 Elasticsearch 是我们的首选输出方向，能够为我们的搜索和分析带来无限可能，但它并非唯一选择。

Logstash 提供众多输出选择，您可以将数据发送到您要指定的地方，并且能够灵活地解锁众多下游用例。

### 即插即用

使用 Elastic Stack 更快获得洞察
Logstash 模块通过热门的数据源（如 ArcSight 和 Netflow ）呈现瞬间可视化的体验。通过立即部署摄入管道和复杂的仪表板，您在短短几分钟内便可开始数据探索。

##  3，logstash-input-jdbc

### Configuring multiple SQL statements

Configuring multiple SQL statements is useful when there is a need to query and ingest data from different database tables or views. It is possible to define separate Logstash configuration files for each statement or to define multiple statements in a single configuration file. When using multiple statements in a single Logstash configuration file, each statement has to be defined as a separate jdbc input (including jdbc driver, connection string and other required parameters).

Please note that if any of the statements use the sql_last_value parameter (e.g. for ingesting only data changed since last run), each input should define its own last_run_metadata_path parameter. Failure to do so will result in undesired behaviour, as all inputs will store their state to the same (default) metadata file, effectively overwriting each other’s sql_last_value.


##  4 es接口使用

1, curl -X GET weibo:9200

```
{
  "name" : "weibo-es-1",
  "cluster_name" : "weibo-es",
  "cluster_uuid" : "WTvsQFduQHG0mzquuZQmlg",
  "version" : {
    "number" : "6.6.1",
    "build_flavor" : "default",
    "build_type" : "tar",
    "build_hash" : "1fd8f69",
    "build_date" : "2019-02-13T17:10:04.160291Z",
    "build_snapshot" : false,
    "lucene_version" : "7.6.0",
    "minimum_wire_compatibility_version" : "5.6.0",
    "minimum_index_compatibility_version" : "5.0.0"
  },
  "tagline" : "You Know, for Search"
}
```

2, curl -X POST -H "Content-type:application/json" -d '{"name":"Oscar","lastname":"Peng","job_description":"Developer"}' weibo:9200/accounts/person/2

``` 
{
    "_index": "accounts",
    "_type": "person",
    "_id": "1",
    "_version": 1,
    "result": "created",
    "_shards": {
        "total": 2,
        "successful": 1,
        "failed": 0
    },
    "_seq_no": 0,
    "_primary_term": 1
}
```

3, curl -X GET weibo:9200/accounts/person/2

```
{"_index":"accounts","_type":"person","_id":"2","_version":1,"_seq_no":0,"_primary_term":1,"found":true,"_source":{"name":"Oscar","lastname":"Peng","job_description":"Developer"}}	
```

4, curl -X POST -H "Content-Type:application/json" -d '{"doc":{"job_description":"architecture"}}' weibo:9200/accounts/person/2/_update

``` 
{"_index":"accounts","_type":"person","_id":"2","_version":2,"result":"updated","_shards":{"total":2,"successful":1,"failed":0},"_seq_no":1,"_primary_term”:1}}
```

4, curl -X GET weibo:9200/_search?q=Oscar

``` 
{"took":61,"timed_out":false,"_shards":{"total":5,"successful":5,"skipped":0,"failed":0},"hits":{"total":1,"max_score":0.2876821,"hits":[{"_index":"accounts","_type":"person","_id":"2","_score":0.2876821,"_source":{"name":"Oscar","lastname":"Peng","job_description":"architecture"}}]}}
```

5,curl -X GET weibo:9200/_search?q=job_description:architecture

```
{"took":2,"timed_out":false,"_shards":{"total":5,"successful":5,"skipped":0,"failed":0},"hits":{"total":1,"max_score":0.2876821,"hits":[{"_index":"accounts","_type":"person","_id":"2","_score":0.2876821,"_source":{"name":"Oscar","lastname":"Peng","job_description":"architecture"}}]}}
```

6, curl -X GET weibo:9200/accounts/person/_search?q=job_description:architecture

```
{"took":5,"timed_out":false,"_shards":{"total":5,"successful":5,"skipped":0,"failed":0},"hits":{"total":1,"max_score":0.2876821,"hits":[{"_index":"accounts","_type":"person","_id":"2","_score":0.2876821,"_source":{"name":"Oscar","lastname":"Peng","job_description":"architecture"}}]}}
```

7, curl -X DELETE weibo:9200/accounts/person/1

```
{"_index":"accounts","_type":"person","_id":"1","_version":2,"result":"deleted","_shards":{"total":2,"successful":1,"failed":0},"_seq_no":1,"_primary_term":1}
```

8, curl -X GET weibo:9200/accounts/_search

```
{"took":2,"timed_out":false,"_shards":{"total":5,"successful":5,"skipped":0,"failed":0},"hits":{"total":1,"max_score":1.0,"hits":[{"_index":"accounts","_type":"person","_id":"2","_score":1.0,"_source":{"name":"Oscar","lastname":"Peng","job_description":"architecture"}}]}}
```

9,curl -H 'Content-Type:application/x-ndjson' -XPOST 'localhost:9200/bank/shakespeare/_bulk?pretty' --data-binary @shakespeare.json

curl localhost:9200/_cat/indices?v

##  5，常见查询及聚合的JAVA API

### 根据ID 进行单个查询

``` 
GetResponse response = client.prepareGet("accounts", "person", "1")
                .setOperationThreaded(false)
                .get();
```

### 分页查询所有记录

``` 
QueryBuilder qb=new MatchAllQueryBuilder();
        SearchResponse response= client.prepareSearch("accounts").setTypes("person").setQuery(qb).setFrom(0)
                .setSize(100).get();
        SearchHits searchHits =  response.getHits();
        for(SearchHit hit:searchHits.getHits()){
            logger.log(Level.INFO , hit.getSourceAsString());
        }
```

### 根据多条件组合与查询

``` 
 QueryBuilder qb=QueryBuilders.boolQuery().must(QueryBuilders.termQuery("title","JAVA开发工程师")).must(QueryBuilders.termQuery("age",30)) ;
 
        SearchRequestBuilder sv=client.prepareSearch("accounts").setTypes("person").setQuery(qb).setFrom(0)
                .setSize(100);
        logger.log(Level.INFO,sv.toString());
        SearchResponse response=  sv.get();
        SearchHits searchHits =  response.getHits();
        for(SearchHit hit:searchHits.getHits()){
            logger.log(Level.INFO , hit.getSourceAsString());
        }
```

### 多条件或查询

``` 
 QueryBuilder qb=QueryBuilders.termQuery("user","kimchy14");
        QueryBuilder qb1=QueryBuilders.termQuery("user","kimchy15");
 
        SortBuilder sortBuilder=SortBuilders.fieldSort("age");
        sortBuilder.order(SortOrder.DESC);
        QueryBuilder s=QueryBuilders.boolQuery().should(qb).should(qb1);//.must(qb5);
        SearchRequestBuilder sv=client.prepareSearch("accounts").setTypes("person").setQuery(s).addSort(sortBuilder).setFrom(0)
                .setSize(100);
        logger.log(Level.INFO,sv.toString());
        SearchResponse response=  sv.get();
        SearchHits searchHits =  response.getHits();
        for(SearchHit hit:searchHits.getHits()){
            logger.log(Level.INFO , hit.getSourceAsString());
        }
```

### 范围查询 

``` 
// RangeQueryBuilder rangeQueryBuilder = QueryBuilders.rangeQuery("age").from(30,true).to(30,true);
       // RangeQueryBuilder rangeQueryBuilder = QueryBuilders.rangeQuery("age").gt(30 );
        RangeQueryBuilder rangeQueryBuilder = QueryBuilders.rangeQuery("age").gte(30 );
        QueryBuilder s=QueryBuilders.boolQuery().must(rangeQueryBuilder);//.must(qb5);
        SearchRequestBuilder sv=client.prepareSearch("accounts").setTypes("person").setQuery(s).setFrom(0)
                .setSize(100);
        logger.log(Level.INFO,sv.toString());
        SearchResponse response=  sv.get();
        SearchHits searchHits =  response.getHits();
        for(SearchHit hit:searchHits.getHits()){
            logger.log(Level.INFO , hit.getSourceAsString());
        }
```

### 包含查询

``` 
 List<String> strs=new ArrayList<>();
        strs.add("kimchy14");
        strs.add("kimchy15");
        strs.add("kimchy16");
        QueryBuilder qb=QueryBuilders.termsQuery("user",strs);
 
        SearchRequestBuilder sv=client.prepareSearch("accounts").setTypes("person").setQuery(qb).setFetchSource("age",null).setFrom(0)
                .setSize(100);
        logger.log(Level.INFO,sv.toString());
        SearchResponse response=  sv.get();
        SearchHits searchHits =  response.getHits();
        for(SearchHit hit:searchHits.getHits()){
            logger.log(Level.INFO , hit.getSourceAsString());
        }
```

### 专门按id进行的包含查询

``` 
QueryBuilder qb=QueryBuilders.idsQuery(0+"");
 
        SearchRequestBuilder sv=client.prepareSearch("accounts").setTypes("person").setQuery(qb).setFetchSource("age",null).setFrom(0)
                .setSize(100);
        logger.log(Level.INFO,sv.toString());
        SearchResponse response=  sv.get();
        SearchHits searchHits =  response.getHits(); 
        for(SearchHit hit:searchHits.getHits()){
            logger.log(Level.INFO , hit.getSourceAsString());
        }
```

### 按通配符查询

``` 
  QueryBuilder qb = QueryBuilders.wildcardQuery("user", "k*hy17*");
         //Fuzziness fuzziness=Fuzziness.fromEdits(2);
 
      // QueryBuilder qb = QueryBuilders.fuzzyQuery("user","mchy2").fuzziness(fuzziness);
        //QueryBuilder qb = QueryBuilders.prefixQuery("user", "kimchy2");
        SearchRequestBuilder sv=client.prepareSearch("accounts").setTypes("person").setQuery(qb).setFetchSource("user",null).setFrom(0)
                .setSize(100);
        logger.log(Level.INFO,sv.toString());
        SearchResponse response=  sv.get();
        SearchHits searchHits =  response.getHits();
        for(SearchHit hit:searchHits.getHits()){
            logger.log(Level.INFO , hit.getSourceAsString());
        }
```

### 统计count

``` 
AggregationBuilder  termsBuilder = AggregationBuilders.count("ageCount").field("age");
 
        RangeQueryBuilder rangeQueryBuilder = QueryBuilders.rangeQuery("age").from(30,true).to(30,true);
        QueryBuilder s=QueryBuilders.boolQuery().must(rangeQueryBuilder);//.must(qb5);
        SearchRequestBuilder sv=client.prepareSearch("accounts").setTypes("person").setQuery(s).setFrom(0).setSize(100).addAggregation(termsBuilder);
        logger.log(Level.INFO,sv.toString());
        SearchResponse response=  sv.get();
        SearchHits searchHits =  response.getHits();
        for(SearchHit hit:searchHits.getHits()){
            logger.log(Level.INFO , hit.getSourceAsString());
        }
        ValueCount valueCount= response.getAggregations().get("ageCount");
        long value=valueCount.getValue();
```

### 查询最大值

``` 
 AggregationBuilder  termsBuilder = AggregationBuilders.max("max").field("age");
 
        RangeQueryBuilder rangeQueryBuilder = QueryBuilders.rangeQuery("age").from(30,true).to(30,true);
        QueryBuilder s=QueryBuilders.boolQuery().must(rangeQueryBuilder);//.must(qb5);
        SearchRequestBuilder sv=client.prepareSearch("accounts").setTypes("person").setQuery(s).setFrom(0).setSize(100).addAggregation(termsBuilder);
        logger.log(Level.INFO,sv.toString());
        SearchResponse response=  sv.get();
        SearchHits searchHits =  response.getHits();
        for(SearchHit hit:searchHits.getHits()){
            logger.log(Level.INFO , hit.getSourceAsString());
        }
        Max valueCount= response.getAggregations().get("max");
        double value=valueCount.getValue();
```

### 统计总和

``` 
 AggregationBuilder  termsBuilder = AggregationBuilders.sum("sum").field("age");
 
        RangeQueryBuilder rangeQueryBuilder = QueryBuilders.rangeQuery("age").from(30,true).to(30,true);
        QueryBuilder s=QueryBuilders.boolQuery().must(rangeQueryBuilder);//.must(qb5);
        SearchRequestBuilder sv=client.prepareSearch("accounts").setTypes("person").setQuery(s).setFrom(0).setSize(100).addAggregation(termsBuilder);
        logger.log(Level.INFO,sv.toString());
        SearchResponse response=  sv.get();
        SearchHits searchHits =  response.getHits();
        for(SearchHit hit:searchHits.getHits()){
            logger.log(Level.INFO , hit.getSourceAsString());
        }
        Sum valueCount= response.getAggregations().get("sum");
        double value=valueCount.getValue();
```

### 平均数

``` 
 AggregationBuilder  termsBuilder = AggregationBuilders.avg("avg").field("age");
 
        RangeQueryBuilder rangeQueryBuilder = QueryBuilders.rangeQuery("age").from(30,true).to(30,true);
        QueryBuilder s=QueryBuilders.boolQuery().must(rangeQueryBuilder);//.must(qb5);
        SearchRequestBuilder sv=client.prepareSearch("accounts").setTypes("person").setQuery(s).setFrom(0).setSize(100).addAggregation(termsBuilder);
        logger.log(Level.INFO,sv.toString());
        SearchResponse response=  sv.get();
        SearchHits searchHits =  response.getHits();
        for(SearchHit hit:searchHits.getHits()){
            logger.log(Level.INFO , hit.getSourceAsString());
        }
        Avg valueCount= response.getAggregations().get("avg");
        double value=valueCount.getValue();

```

### 统计样本基本指标

``` 
        AggregationBuilder  termsBuilder = AggregationBuilders.stats("stats").field("age");
 
        RangeQueryBuilder rangeQueryBuilder = QueryBuilders.rangeQuery("age").from(30,true).to(30,true);
        QueryBuilder s=QueryBuilders.boolQuery().must(rangeQueryBuilder);//.must(qb5);
        SearchRequestBuilder sv=client.prepareSearch("accounts").setTypes("person").setQuery(s).setFrom(0).setSize(100).addAggregation(termsBuilder);
        logger.log(Level.INFO,sv.toString());
        SearchResponse response=  sv.get();
        SearchHits searchHits =  response.getHits();
        for(SearchHit hit:searchHits.getHits()){
            logger.log(Level.INFO , hit.getSourceAsString());
        }
        Stats valueCount= response.getAggregations().get("stats");
        logger.log(Level.INFO,"max"+valueCount.getMaxAsString());
        logger.log(Level.INFO,"avg"+valueCount.getAvgAsString());
        logger.log(Level.INFO,"sum"+valueCount.getSumAsString());
        logger.log(Level.INFO,"min"+valueCount.getMinAsString());
        logger.log(Level.INFO,"count"+valueCount.getCount());
```

### 分组求各组数据

``` 
 AggregationBuilder  termsBuilder = AggregationBuilders.terms("by_age").field("age");
        AggregationBuilder  sumBuilder=AggregationBuilders.sum("ageSum").field("age");
        AggregationBuilder  avgBuilder=AggregationBuilders.avg("ageAvg").field("age");
        AggregationBuilder  countBuilder=AggregationBuilders.count("ageCount").field("age");
 
        termsBuilder.subAggregation(sumBuilder).subAggregation(avgBuilder).subAggregation(countBuilder);
        //TermsAggregationBuilder all = AggregationBuilders.terms("age").field("age");
        //all.subAggregation(termsBuilder);
        RangeQueryBuilder rangeQueryBuilder = QueryBuilders.rangeQuery("age").from(30,true).to(36,true);
        QueryBuilder s=QueryBuilders.boolQuery().must(rangeQueryBuilder);//.must(qb5);
        SearchRequestBuilder sv=client.prepareSearch("accounts").setTypes("person").setQuery(s).setFetchSource(null,"gender").setFrom(0).setSize(100).addAggregation(termsBuilder);
        logger.log(Level.INFO,sv.toString());
        SearchResponse response=  sv.get();
 
        Aggregations terms= response.getAggregations();
        for (Aggregation a:terms){
            LongTerms teamSum= (LongTerms)a;
            for(LongTerms.Bucket bucket:teamSum.getBuckets()){
                logger.info(bucket.getKeyAsString()+"   "+bucket.getDocCount()+"    "+((Sum)bucket.getAggregations().asMap().get("ageSum")).getValue()+"    "+((Avg)bucket.getAggregations().asMap().get("ageAvg")).getValue()+"    "+((ValueCount)bucket.getAggregations().asMap().get("ageCount")).getValue());
 
            }
        }
```

### 多分组求各组数据

``` 
 
        TermsAggregationBuilder all = AggregationBuilders.terms("by_gender").field("gender");
         AggregationBuilder age = AggregationBuilders.terms("by_age").field("age");
        AggregationBuilder  sumBuilder=AggregationBuilders.sum("ageSum").field("age");
        //AggregationBuilder  avgBuilder=AggregationBuilders.avg("ageAvg").field("age");
       // AggregationBuilder  countBuilder=AggregationBuilders.count("ageCount").field("age");
         all.subAggregation(age.subAggregation(sumBuilder));
        RangeQueryBuilder rangeQueryBuilder = QueryBuilders.rangeQuery("age").from(30,true).to(32,true);
        QueryBuilder s=QueryBuilders.boolQuery().must(rangeQueryBuilder);//.must(qb5);
        SearchRequestBuilder sv=client.prepareSearch("accounts").setTypes("person").setQuery(rangeQueryBuilder).addAggregation(all);
        logger.log(Level.INFO,sv.toString());
        SearchResponse response=  sv.get();
 
        Aggregations terms= response.getAggregations();
        for (Aggregation a:terms){
            StringTerms stringTerms= (StringTerms)a;
             for(StringTerms.Bucket bucket:stringTerms.getBuckets()){
              //  logger.info(bucket.getKeyAsString());
                Aggregation aggs=bucket.getAggregations().getAsMap().get("by_age");
                 LongTerms terms1= (LongTerms)aggs;
                for (LongTerms.Bucket bu:terms1.getBuckets()){
                     logger.info(bucket.getKeyAsString()+"  "+bu.getKeyAsString()+" "+bu.getDocCount()+"    "+((Sum)bu.getAggregations().asMap().get("ageSum")).getValue());
                }
 
            }
        }
```

### 总结    

精确查询用term 组合查询用bool 范围用range    and查询用must    or查询用should  not查询用must not  常见的接收聚合返回结果的类型 ValueCount   AVG  SUM  MAX  MIN  按照英文意义就可以理解  分组聚合查询时候还需要根据实际情况看是返回那种terms 
