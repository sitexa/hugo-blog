+++
title = "es:Elastic APM-server"
description = "ElasticSearch是一个基于Lucene的搜索服务器。它提供了一个分布式多用户能力的全文搜索引擎，基于RESTful web接口。"
tags = [
    "technology"
]
date = "2019-04-05"
categories = [
    "es"
]
+++

Elastic APM is an application performance monitoring system built on the Elastic Stack. It allows you to monitor software services and applications in real time, collecting detailed performance information on response time for incoming requests, database queries, calls to caches, external HTTP requests, etc. This makes it easier to pinpoint and fix performance problems quickly.

<!--more-->

# Elastic APM-server

Elastic APM is an application performance monitoring system built on the Elastic Stack. It allows you to monitor software services and applications in real time, collecting detailed performance information on response time for incoming requests, database queries, calls to caches, external HTTP requests, etc. This makes it easier to pinpoint and fix performance problems quickly.

Elastic APM also automatically collects unhandled errors and exceptions. Errors are grouped based primarily on the stacktrace, so you can identify new errors as they appear and keep an eye on how many times specific errors happen.

##  安装 APM-server

https://www.elastic.co/guide/en/apm/server/6.6/setup-repositories.html

### 1，Download and install the public signing key:

```sudo rpm --import https://packages.elastic.co/GPG-KEY-elasticsearch```

### 2，Create a file with a .repo extension (for example, elastic.repo) in your /etc/yum.repos.d/ directory and add the following lines:

``` 
[elastic-6.x]
name=Elastic repository for 6.x packages
baseurl=https://artifacts.elastic.co/packages/6.x/yum
gpgcheck=1
gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch
enabled=1
autorefresh=1
type=rpm-md
```

Your repository is ready to use. For example, you can install APM Server by running:

``` sudo yum install apm-server```

bin: /usr/bin/apm-server
location: /usr/share/apm-server
config: /etc/apm-server


### 3，To configure the Beat to start automatically during boot, run:

``` 
sudo chkconfig --add apm-server
```
