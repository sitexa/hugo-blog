+++
title = "es:Spring Client API"
description = "ElasticSearch是一个基于Lucene的搜索服务器。它提供了一个分布式多用户能力的全文搜索引擎，基于RESTful web接口。"
tags = [
    "technology"
]
date = "2019-04-05"
categories = [
    "es"
]
+++

Spring Client API

<!--more-->

# Spring Client API

from : https://github.com/dadoonet/spring-elasticsearch


Actually, since version 1.4.1, this project has been split in two parts:

- Elasticsearch Beyonder which find resources in project classpath to automatically create indices, types and templates.
- Spring-elasticsearch: which is building Client beans using Spring framework.

From 5.0, *Spring-elasticsearch* project provides 2 implementations of an elasticsearch Client:

- The REST client
- The Transport client (deprecated)

From 6.0, the REST client implementation has been replaced by a *High Level REST client*. It now also supports X-Pack for official security.

