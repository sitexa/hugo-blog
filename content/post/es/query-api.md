+++
title = "es:Query DSL"
description = "ElasticSearch是一个基于Lucene的搜索服务器。它提供了一个分布式多用户能力的全文搜索引擎，基于RESTful web接口。"
tags = [
    "technology"
]
date = "2019-04-05"
categories = [
    "es"
]
+++

Query DSL

<!--more-->

# Query DSL

https://www.elastic.co/guide/en/elasticsearch/reference/6.6/query-dsl.html

https://www.elastic.co/guide/en/elasticsearch/client/java-api/current/java-compound-queries.html#java-query-dsl-bool-query

Elasticsearch provides a full Query DSL (Domain Specific Language) based on JSON to define queries. Think of the Query DSL as an AST (Abstract Syntax Tree) of queries, consisting of two types of clauses:

- Leaf query clauses.
  Leaf query clauses look for a particular value in a particular field, such as the match, term or range queries. These queries can be used by themselves.
- Compound query clauses.
  Compound query clauses wrap other leaf or compound queries and are used to combine multiple queries in a logical fashion (such as the bool or dis_max query), or to alter their behaviour (such as the constant_score query).

Query clauses behave differently depending on whether they are used in *query context* or *filter context*.

##  Query and filter context

The behaviour of a query clause depends on whether it is used in query context or in filter context:

- Query context
  A query clause used in query context answers the question “How well does this document match this query clause?” Besides deciding whether or not the document matches, the query clause also calculates a _score representing how well the document matches, relative to other documents.

Query context is in effect whenever a query clause is passed to a query parameter, such as the query parameter in the search API.

- Filter context
  In filter context, a query clause answers the question “Does this document match this query clause?” The answer is a simple Yes or No — no scores are calculated. Filter context is mostly used for filtering structured data, e.g.

  - Does this timestamp fall into the range 2015 to 2016?
  - Is the status field set to "published"?

  Frequently used filters will be cached automatically by Elasticsearch, to speed up performance.

  Filter context is in effect whenever a query clause is passed to a filter parameter, such as the filter or must_not parameters in the bool query, the filter parameter in the constant_score query, or the filter aggregation.

``` 
GET /_search
{
  "query": { 
    "bool": { 
      "must": [
        { "match": { "title":   "Search"        }}, 
        { "match": { "content": "Elasticsearch" }}  
      ],
      "filter": [ 
        { "term":  { "status": "published" }}, 
        { "range": { "publish_date": { "gte": "2015-01-01" }}} 
      ]
    }
  }
}
```

##  Match All Query

``` 
GET /_search
{
    "query": {
        "match_all": {}
    }
}

GET /_search
{
    "query": {
        "match_all": { "boost" : 1.2 }
    }
}
```

##  Match None Query

``` 
GET /_search
{
    "query": {
        "match_none": {}
    }
}
```

##  Full text queries

The high-level full text queries are usually used for running full text queries on full text fields like the body of an email. They understand how the field being queried is analyzed and will apply each field’s analyzer (or search_analyzer) to the query string before executing.

- match query.
  The standard query for performing full text queries, including fuzzy matching and phrase or proximity queries.
- match_phrase query.
  Like the match query but used for matching exact phrases or word proximity matches.
- match_phrase_prefix query.
  The poor man’s search-as-you-type. Like the match_phrase query, but does a wildcard search on the final word.
- multi_match query.
  The multi-field version of the match query.
- common terms query.
  A more specialized query which gives more preference to uncommon words.
- query_string query.
  Supports the compact Lucene query string syntax, allowing you to specify AND|OR|NOT conditions and multi-field search within a single query string. For expert users only.
- simple_query_string query.
  A simpler, more robust version of the query_string syntax suitable for exposing directly to users.

##  Match Query

``` 
GET /_search
{
    "query": {
        "match" : {
            "message" : "this is a test"
        }
    }
}
```

##  Match Phrase Query

The *match_phrase* query analyzes the text and creates a phrase query out of the analyzed text. For example:

``` 
GET /_search
{
    "query": {
        "match_phrase" : {
            "message" : "this is a test"
        }
    }
}
```

##  Multi Match Query

The multi_match query builds on the match query to allow multi-field queries:

``` 
GET /_search
{
  "query": {
    "multi_match" : {
      "query":    "this is a test", 
      "fields": [ "subject", "message" ] 
    }
  }
}
```

##  Constant Score Query

A query that wraps another query and simply returns a constant score equal to the query boost for every document in the filter. Maps to Lucene ConstantScoreQuery.

``` 
GET /_search
{
    "query": {
        "constant_score" : {
            "filter" : {
                "term" : { "user" : "kimchy"}
            },
            "boost" : 1.2
        }
    }
}
```

``` 
constantScoreQuery(termQuery("name","kimchy")).boost(2.0f);           
```

##  Regexp Query

The regexp query allows you to use regular expression term queries. See Regular expression syntax for details of the supported regular expression language. The "term queries" in that first sentence means that Elasticsearch will apply the regexp to the terms produced by the tokenizer for that field, and not to the original text of the field.

``` 
GET /_search
{
    "query": {
        "regexp":{
            "name.first": "s.*y"
        }
    }
}
```

### 1-Match any character
The period "." can be used to represent any character. For string "abcde":

```
ab...   # match
a.c.e   # match
```

### 2-One-or-more

The plus sign "+" can be used to repeat the preceding shortest pattern once or more times. For string "aaabbb":

```
a+b+        # match
aa+bb+      # match
a+.+        # match
aa+bbb+     # match
```

### 3-Zero-or-more
      
The asterisk "*" can be used to match the preceding shortest pattern zero-or-more times. For string "aaabbb":
      
```
a*b*        # match
a*b*c*      # match
.*bbb.*     # match
aaa*bbb*    # match
```

### 4-Zero-or-one
The question mark "?" makes the preceding shortest pattern optional. It matches zero or one times. For string "aaabbb":

```
aaa?bbb?    # match
aaaa?bbbb?  # match
.....?.?    # match
aa?bb?      # no match
```

### 5-Min-to-max
Curly brackets "{}" can be used to specify a minimum and (optionally) a maximum number of times the preceding shortest pattern can repeat. The allowed forms are:

```
{5}     # repeat exactly 5 times
{2,5}   # repeat at least twice and at most 5 times
{2,}    # repeat at least twice

a{3}b{3}        # match
a{2,4}b{2,4}    # match
a{2,}b{2,}      # match
.{3}.{3}        # match
a{4}b{4}        # no match
a{4,6}b{4,6}    # no match
a{4,}b{4,}      # no match
```
   
### 6-Grouping
Parentheses "()" can be used to form sub-patterns. The quantity operators listed above operate on the shortest previous pattern, which can be a group. For string "ababab":

```
(ab)+       # match
ab(ab)+     # match
(..)+       # match
(...)+      # no match
(ab)*       # match
abab(ab)?   # match
ab(ab)?     # no match
(ab){3}     # match
(ab){1,2}   # no match
```

### 7-Alternation
The pipe symbol "|" acts as an OR operator. The match will succeed if the pattern on either the left-hand side OR the right-hand side matches. The alternation applies to the longest pattern, not the shortest. For string "aabb":

```
aabb|bbaa   # match
aacc|bb     # no match
aa(cc|bb)   # match
a+|b+       # no match
a+b+|b+a+   # match
a+(b|c)+    # match
```

### 8-Character classes
Ranges of potential characters may be represented as character classes by enclosing them in square brackets "[]". A leading ^ negates the character class. The allowed forms are:

```
[abc]   # 'a' or 'b' or 'c'
[a-c]   # 'a' or 'b' or 'c'
[-abc]  # '-' or 'a' or 'b' or 'c'
[abc\-] # '-' or 'a' or 'b' or 'c'
[^abc]  # any character except 'a' or 'b' or 'c'
[^a-c]  # any character except 'a' or 'b' or 'c'
[^-abc]  # any character except '-' or 'a' or 'b' or 'c'
[^abc\-] # any character except '-' or 'a' or 'b' or 'c'

ab[cd]+     # match
[a-d]+      # match
[^a-d]+     # no match
```

##  Query String Query

A query that uses a query parser in order to parse its content. Here is an example:

``` 
GET /_search
{
    "query": {
        "query_string" : {
            "default_field" : "content",
            "query" : "this AND that OR thus"
        }
    }
}


GET /_search
{
    "query": {
        "query_string" : {
            "default_field" : "content",
            "query" : "(new york city) OR (big apple)" 
        }
    }
}
```

####  Synonyms

The query_string query supports multi-terms synonym expansion with the *synonym_graph* token filter. When this filter is used, the parser creates a phrase query for each multi-terms synonyms. For example, the following synonym: ny, new york would produce:

```(ny OR ("new york"))```  

``` 
GET /_search
{
   "query": {
       "query_string" : {
           "default_field": "title",
           "query" : "ny city",
           "auto_generate_synonyms_phrase_query" : false
       }
   }
}
```

##  Bool Query

A query that matches documents matching boolean combinations of other queries. The bool query maps to Lucene BooleanQuery. It is built using one or more boolean clauses, each clause with a typed occurrence. The occurrence types are:

|Occur|Description|
|---|---|
must|The clause (query) must appear in matching documents and will contribute to the score.
filter|The clause (query) must appear in matching documents. However unlike must the score of the query will be ignored. Filter clauses are executed in filter context, meaning that scoring is ignored and clauses are considered for caching.
should|The clause (query) should appear in the matching document. If the bool query is in a query context and has a must or filter clause then a document will match the bool query even if none of the should queries match. In this case these clauses are only used to influence the score. If the bool query is in a filter context or has neither must or filter then at least one of the should queries must match a document for it to match the bool query. This behavior may be explicitly controlled by setting the minimum_should_match parameter.
must_not|The clause (query) must not appear in the matching documents. Clauses are executed in filter context meaning that scoring is ignored and clauses are considered for caching. Because scoring is ignored, a score of 0 for all documents is returned.

``` 
POST _search
{
  "query": {
    "bool" : {
      "must" : {
        "term" : { "user" : "kimchy" }
      },
      "filter": {
        "term" : { "tag" : "tech" }
      },
      "must_not" : {
        "range" : {
          "age" : { "gte" : 10, "lte" : 20 }
        }
      },
      "should" : [
        { "term" : { "tag" : "wow" } },
        { "term" : { "tag" : "elasticsearch" } }
      ],
      "minimum_should_match" : 1,
      "boost" : 1.0
    }
  }
}
```

``` 
boolQuery()
        .must(termQuery("content", "test1"))                 
        .must(termQuery("content", "test4"))                 
        .mustNot(termQuery("content", "test2"))              
        .should(termQuery("content", "test3"))               
        .filter(termQuery("content", "test5"));       
```

##  Dis Max Query

A query that generates the union of documents produced by its subqueries, and that scores each document with the maximum score for that document as produced by any subquery, plus a tie breaking increment for any additional matching subqueries.

``` 
GET /_search
{
    "query": {
        "dis_max" : {
            "tie_breaker" : 0.7,
            "boost" : 1.2,
            "queries" : [
                {
                    "term" : { "age" : 34 }
                },
                {
                    "term" : { "age" : 35 }
                }
            ]
        }
    }
}
```

``` 
disMaxQuery()
        .add(termQuery("name", "kimchy"))                    
        .add(termQuery("name", "elasticsearch"))             
        .boost(1.2f)                                         
        .tieBreaker(0.7f);      
```

## Function Score Query

https://www.elastic.co/guide/en/elasticsearch/reference/6.6/query-dsl-function-score-query.html

The function_score allows you to modify the score of documents that are retrieved by a query. This can be useful if, for example, a score function is computationally expensive and it is sufficient to compute the score on a filtered set of documents.

``` 
GET /_search
{
    "query": {
        "function_score": {
            "query": { "match_all": {} },
            "boost": "5",
            "random_score": {}, 
            "boost_mode":"multiply"
        }
    }
}
```


``` 
import static org.elasticsearch.index.query.functionscore.ScoreFunctionBuilders.*;
FilterFunctionBuilder[] functions = {
        new FunctionScoreQueryBuilder.FilterFunctionBuilder(
                matchQuery("name", "kimchy"),                
                randomFunction()),                           
        new FunctionScoreQueryBuilder.FilterFunctionBuilder(
                exponentialDecayFunction("age", 0L, 1L))     
};
functionScoreQuery(functions);
```

##  Boosting Query

The boosting query can be used to effectively demote results that match a given query. Unlike the "NOT" clause in bool query, this still selects documents that contain undesirable terms, but reduces their overall score.

``` 
GET /_search
{
    "query": {
        "boosting" : {
            "positive" : {
                "term" : {
                    "field1" : "value1"
                }
            },
            "negative" : {
                 "term" : {
                     "field2" : "value2"
                }
            },
            "negative_boost" : 0.2
        }
    }
}
```

``` 
boostingQuery(
            termQuery("name","kimchy"),                      
            termQuery("name","dadoonet"))                    
        .negativeBoost(0.2f);     
```
