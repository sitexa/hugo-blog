+++
title = "es:elasticsearch-analysis-ik(分词和拼音、同义词)"
description = "ElasticSearch是一个基于Lucene的搜索服务器。它提供了一个分布式多用户能力的全文搜索引擎，基于RESTful web接口。"
tags = [
    "technology"
]
date = "2019-04-05"
categories = [
    "es"
]
+++

elasticsearch-analysis-ik(分词和拼音、同义词)

<!--more-->

# elasticsearch-analysis-ik(分词和拼音、同义词)

- *分析*```analysis```
- *分析器*```analyzer```
- *分词器*```tokenizer```
- *过滤器*```filter```

##  安装中文分词插件

``` 
./bin/elasticsearch-plugin install https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v6.6.1/elasticsearch-analysis-ik-6.6.1.zip
```

##  安装拼音分词插件

``` 
./bin/elasticsearch-plugin install https://github.com/medcl/elasticsearch-analysis-pinyin/releases/download/v6.6.1/elasticsearch-analysis-pinyin-6.6.1.zip
```

##  创建索引，配置两种分词器

``` 
PUT /blog/
{
	"settings":{
		"number_of_shards": 3,
		"number_of_replicas": 1,
		"analysis": {
        "analyzer": {
				  "default":{
					  "tokenizer":"ik_max_word"
				  },				
          "pinyin_analyzer": {
            "type": "custom",
            "tokenizer": "my_pinyin",
            "filter": ["word_delimiter"]
          }
        },
        "tokenizer": {
          "my_pinyin" : {
            "type" : "pinyin",
            "keep_first_letter":true,
            "keep_separate_first_letter" : false,
            "keep_full_pinyin" : true,
            "keep_original" : false,
            "limit_first_letter_length" : 16,
            "lowercase" : true
            }
        }
    }
	}
}
```



##  针对索引配置字段映射

``` 
PUT /blog/_mapping/article
{    
	"properties" : {
		"title" : {
			"type" : "text",
			"analyzer" : "ik_max_word",
			"copy_to" : true,
			"fields" : {
				"pinyin" : {
					"type" : "text",
					"term_vector" : "with_positions_offsets",
					"analyzer" : "pinyin_analyzer",
					"boost" : 10
				  }
			 }
		},      
		"content":{
		  "type":"text",
		  "analyzer" : "ik_max_word"
		},
		"author":{
		  "type":"text",
		  "analyzer" : "ik_max_word"
		},
		"keyword":{
		  "type":"text",
		  "analyzer" : "ik_max_word"
		},
		"createtime":{			
			"type":"date",			
			"format":"yyyy-MM-dd HH:mm:ss || yyyy-MM-dd || epoch_millis"	
		}
	}
}
```


##  检查自定义的词语分析器是否生效

``` 
GET /blog/_analyze
{
	"text":"刘德华",
	"analyzer":"pinyin_analyzer"
}
```

##  填充一些内容

``` 
POST /blog/article/1
{
    "title":"刘德华2018年演唱会将在北京举行",
    "content":"刘德华2018年跨年演唱会将在北京市海淀剧院举行",
    "author":"王军",
    "createtime":"2018-12-26"
}
```

##  按照拼音搜索

``` 
POST /blog/article/_search
{
    "query":{
      "match":{
        "title.pinyin":"liudehua"
      }
    }
}
```

##  按照中文名进行搜索

``` 
POST /blog/article/_search
{
    "query":{
      "match":{
        "title":"刘德华"
      }
    }
}
```

##  按照中文名+拼音进行搜索

``` 
POST blog/article/_search
{
  "query": {
    "multi_match": {
      "type":"most_fields",
      "query":"刘德h",
      "fields":["title", "title.pinyin"]
    }
  }
}
```

##  分析执行结果

``` 
GET blog/article/_validate/query?explain
{
  "query": {
    "multi_match": {
      "type":"most_fields",
      "query":"刘德h",
      "fields":["title", "title.pinyin"]
    }
  }
}
```

##  关闭/打开/重新索引

``` 
POST web_document/_close
```

``` 
POST web_document/_open
```

``` 

```

##  配置同义词

``` 
PUT web_document/_settings?preserve_existing=true
{
  "index.analysis.analyzer.ik_syno.filter" : [
    "my_synonym_filter"
  ],
  "index.analysis.analyzer.ik_syno.tokenizer" : "ik_max_word",
  "index.analysis.analyzer.ik_syno.type" : "custom",
  "index.analysis.analyzer.ik_syno_smart.filter" : [
    "my_synonym_filter"
  ],
  "index.analysis.analyzer.ik_syno_smart.tokenizer" : "ik_smart",
  "index.analysis.analyzer.ik_syno_smart.type" : "custom",
  "index.analysis.filter.my_synonym_filter.synonyms_path" : "synonyms.txt",
  "index.analysis.filter.my_synonym_filter.type" : "synonym"
}'
```

> 网上很多博客修改 elasticsearch.yml 的做法，在 es6 中无法使用。

# web_document执行步骤

##  （1）创建索引、匹配分词器

``` 
PUT /web_document/
{
	"settings":{
		"analysis": {
        "analyzer": {
				  "default":{
					  "tokenizer":"ik_max_word"
				  },				
          "pinyin_analyzer": {
            "type": "custom",
            "tokenizer": "my_pinyin",
            "filter": ["word_delimiter"]
          }
        },
        "tokenizer": {
          "my_pinyin" : {
            "type" : "pinyin",
            "keep_first_letter":true,
            "keep_separate_first_letter" : false,
            "keep_full_pinyin" : true,
            "keep_original" : false,
            "limit_first_letter_length" : 16,
            "lowercase" : true
            }
        }
    }
	}
}
```

##  （2）针对索引配置字段映射


``` 
PUT /web_document/_mapping/doc
{    
	"properties" : {
	  "url": {
	    "type":"text",
      "analyzer" : "ik_max_word"
	  },
		"title" : {
			"type" : "text",
			"analyzer" : "ik_max_word",
			"copy_to" : true,
			"fields" : {
				"pinyin" : {
					"type" : "text",
					"term_vector" : "with_positions_offsets",
					"analyzer" : "pinyin_analyzer",
					"boost" : 10
				  }
			 }
		},      
		"contents":{
		  "type":"text",
		  "analyzer" : "ik_max_word",
		  "copy_to" : true,
      "fields" : {
        "pinyin" : {
          "type" : "text",
          "term_vector" : "with_positions_offsets",
          "analyzer" : "pinyin_analyzer",
          "boost" : 10
          }
       }
		},
		"cate":{
		  "type":"text",
		  "analyzer" : "ik_max_word"
		},
		"pubDate":{			
			"type":"date",			
			"format":"yyyy-MM-dd HH:mm || yyyy-MM-dd || epoch_millis"	
		}
	}
}
```

##  （3）检查自定义的词语分析器是否生效

``` 
GET /web_document/_analyze
{
	"title":"许达哲",
	"analyzer":"pinyin_analyzer"
}
```

##  (4)按照中文名进行搜索

``` 
POST /web_document/doc/_search
{
    "query":{
      "match":{
        "title":"许达哲"
      }
    }
}
```

##  (5)按照拼音进行搜索

``` 
POST /web_document/doc/_search
{
    "query":{
      "match":{
        "title.pinyin":"xudazhe"
      }
    }
}
```

##  (6)按中文+拼音搜索多个字段

``` 
POST /web_document/doc/_search
{
    "query":{
      "multi_match": {
        "type":"most_fields",
        "query":"许达zh",
        "fields":["title", "title.pinyin", "contents", "contents.pinyin"]
      }
    }
}
```

##  建立索引，配置分析器，分词器，过滤器

先关闭index, 修改之后再打开index

```
PUT /web_document/
{
	"settings":{
		"number_of_shards": 3,
		"number_of_replicas": 1,
		"analysis": {
        "analyzer": {
				  "default":{
					  "tokenizer":"ik_max_word"
				  },				
          "pinyin_analyzer": {
            "type": "custom",
            "tokenizer": "my_pinyin",
            "filter": ["word_delimiter"]
          },
          "ik_syno_smart": {
            "type":"custom",
            "tokenizer":"ik_smart",
            "filter": ["my_stop_filter", "my_syno_filter"],
              "char_filter" : ["my_char_filter"]
          }
        },
        "filter" : {
            "my_stop_filter" : {
                "type" : "stop",
                "stopwords" : [" "]
            },
            "my_syno_filter" : {
                "type" : "synonym",
                "synonyms_path" : "analysis-synonym/synonyms.txt"
            }
        },
        "char_filter" : {
            "my_char_filter" : {
                "type" : "mapping",
                "mappings" : ["| => |"]
            }
        },
        "tokenizer": {
          "my_pinyin" : {
            "type" : "pinyin",
            "keep_first_letter":true,
            "keep_separate_first_letter" : false,
            "keep_full_pinyin" : true,
            "keep_original" : false,
            "limit_first_letter_length" : 16,
            "lowercase" : true
            }
        }
    }
	}
}
```
