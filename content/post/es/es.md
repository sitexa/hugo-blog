# ElasticSearch

### 1，ElasticSearch是什么？

ElasticSearch是一个基于Lucene的搜索服务器。它提供了一个分布式多用户能力的全文搜索引擎，基于RESTful web接口。Elasticsearch是用Java开发的，并作为Apache许可条款下的开放源码发布，是当前流行的企业级搜索引擎。设计用于云计算中，能够达到实时搜索，稳定，可靠，快速，安装使用方便。

我们建立一个网站或应用程序的时候，需要添加搜索功能，但是想要完成搜索工作的创建是非常困难的。我们希望搜索解决方案要运行速度快，希望能有一个零配置和一个完全免费的搜索模式，希望能够简单地使用JSON通过HTTP来索引数据，希望搜索服务器始终可用，希望能够从一台开始并扩展到数百台，
希望实时搜索，希望简单的多租户，希望建立一个云的解决方案。因此我们利用Elasticsearch来解决所有这些问题及可能出现的更多其它问题。

### 2，安装配置及启动

下载：https://www.elastic.co/downloads/elasticsearch

安装：

``` 
sudo tar -xvf elasticsearch-6.6.1.tar.gz /opt/
```

配置环境变量

``` 
sudo vi /etc/security/limits.conf

*               soft    nofile          65536
*               hard    nofile          65536
*               soft    nproc           4096
*               hard    nproc           4096

sudo sysctl -w vm.max_map_count=262144

```

配置节点相关参数 config/elasticsearch.yml

``` 
cluster.name = my-application
node.name: node-1
path.data: /path/to/data
path.logs: /path/to/logs
network.host: 10.10.4.25
http.port: 9200
```

启动：
``` 
bin/elasticsearch -d -p pid
or
nohup bin/elasticsearch >/dev/null 2>&1 &
```

停止：
``` 
pkill -F pid
```

启动后有两个监听端口：

http://localhost:9200 //http协议端口，提供restful服务

localhost:9300 //tcp通信端口，为集群节点之间通信服务


### 3，kibana的安装配置和启动

下载：https://www.elastic.co/downloads/kibana-6.6.1-linux-x86_64.tar.gz

安装：
``` 
tar -xvf kibana-6.6.1-linux-x86_64.tar.gz /opt/
```

配置 config/kibana.yml
``` 
server.port: 5601
server.host: 10.10.4.25
elasticsearch.hosts: ["http://10.10.4.25:9200"]
```

启动与停止：

``` 
nohup bin/kibana >/dev/null 2>&1 &

kill -9 pid
```

### 4, logstash的安装配置和启动

下载：https://www.elastic.co/downloads/logstash-6.6.1.tar.gz

安装：
``` 
tar -xvf logstash-6.6.1.tar.gz /opt/
```

安装插件：logstash-input-jdnc *logstash 5.x以上版本已经集成了logstash-input-jdbc插件，无须另行安装*
``` 
gem install bundler

bundle config mirror.https://rubygems.org https://gems.ruby-china.org/

bin/logstash-plugin  install logstash-input-jdbc

```

配置：

``` 
#jdbc.conf

input {
    jdbc {
      jdbc_connection_string => "jdbc:mysql://10.10.4.25:3306/esite?characterEncoding=utf-8&useSSL=false&allowPublicKeyRetrieval=true&serverTimezone=GMT"
      jdbc_user => "root"
      jdbc_password => "WEIbo123!@#"
      jdbc_driver_library => "/opt/logstash-6.6.1/mysql-connector-java-5.1.47.jar"
      jdbc_driver_class => "com.mysql.jdbc.Driver"
      jdbc_paging_enabled => "true"
      jdbc_page_size => "50000"
      statement => "select * from sys_user"
      schedule => "* * * * *"
      type => "sysuser"
    }
}

input {
  jdbc {
    jdbc_connection_string => "jdbc:mysql://10.10.4.25:3306/esite?characterEncoding=utf-8&useSSL=false&allowPublicKeyRetrieval=true&serverTimezone=GMT"
    jdbc_user => "root"
    jdbc_password => "WEIbo123!@#"
    jdbc_driver_library => "/opt/logstash-6.6.1/mysql-connector-java-5.1.47.jar"
    jdbc_driver_class => "com.mysql.jdbc.Driver"
    jdbc_paging_enabled => "true"
    jdbc_page_size => "50000"
    statement => "select * from sys_org"
    schedule => "* * * * *"
    type => "sysorg"
  }
}

input {
  jdbc {
    jdbc_connection_string => "jdbc:oracle:thin:@10.10.4.18:1521:orcl"
    jdbc_user => "jkwwz"
    jdbc_password => "jkwwz"
    jdbc_driver_library => "/opt/logstash-6.6.1/ojdbc6-12.1.0.2.jar"
    jdbc_driver_class => "Java::oracle.jdbc.driver.OracleDriver"
    jdbc_paging_enabled => "true"
    jdbc_page_size => "50000"
    statement => "SELECT * FROM WWZ_LDXX_XJXX"
    schedule => "* * * * *"
    type => "wwzxjxx"
  }
}

filter {
    json {
        source => "message"
        remove_field => ["message"]
    }
}

output {
 if [type] == "sysuser" {
    elasticsearch {
      hosts => ["10.10.4.25:9200"]
      index => "sys_user"
      document_id => "%{uid}"
    }
  }
  if [type] == "sysorg" {
    elasticsearch {
      hosts => ["10.10.4.25:9200"]
      index => "sys_org"
      document_id => "%{oid}"
    }
  }
  if [type] == "wwzxjxx" {
    elasticsearch {
      hosts => ["10.10.4.25:9200"]
      index => "wwz_xjxx"
      document_id => "%{id}"
    }
  }
}
```

启动：

``` 
nohup bin/logstash -f jdbc.conf >/dev/null 2>&1 &

```

####  logstash-inpput-jdbc配置：

在config目录下，创建配置文件（logstash-mysql-es.conf）：

```
input {
  jdbc {
    # mysql相关jdbc配置
    jdbc_connection_string => "jdbc:mysql://10.112.76.30:3306/jack_test?useUnicode=true&characterEncoding=utf-8&useSSL=false"
    jdbc_user => "root"
    jdbc_password => "123456"

    # jdbc连接mysql驱动的文件目录，可去官网下载:https://dev.mysql.com/downloads/connector/j/
    jdbc_driver_library => "./config/mysql-connector-java-5.1.46.jar"
    # the name of the driver class for mysql
    jdbc_driver_class => "com.mysql.jdbc.Driver"
    jdbc_paging_enabled => true
    jdbc_page_size => "50000"

    jdbc_default_timezone =>"Asia/Shanghai"

    # mysql文件, 也可以直接写SQL语句在此处，如下：
    # statement => "select * from t_order where update_time >= :sql_last_value;"
    statement_filepath => "./config/jdbc.sql"

    # 这里类似crontab,可以定制定时操作，比如每分钟执行一次同步(分 时 天 月 年)
    schedule => "* * * * *"
    #type => "jdbc"

    # 是否记录上次执行结果, 如果为真,将会把上次执行到的 tracking_column 字段的值记录下来,保存到 last_run_metadata_path 指定的文件中
    #record_last_run => true

    # 是否需要记录某个column 的值,如果record_last_run为真,可以自定义我们需要 track 的 column 名称，此时该参数就要为 true. 否则默认 track 的是 timestamp 的值.
    use_column_value => true

    # 如果 use_column_value 为真,需配置此参数. track 的数据库 column 名,该 column 必须是递增的. 一般是mysql主键
    tracking_column => "update_time"
    
    tracking_column_type => "timestamp"

    last_run_metadata_path => "./logstash_capital_bill_last_id"

    # 是否清除 last_run_metadata_path 的记录,如果为真那么每次都相当于从头开始查询所有的数据库记录
    clean_run => false

    #是否将 字段(column) 名称转小写
    lowercase_column_names => false
  }
}

output {
  elasticsearch {
    hosts => "10.112.76.31:9200"
    index => "mysql_order"
    document_id => "%{id}"
    template_overwrite => true
  }

  # 这里输出调试，正式运行时可以注释掉
  stdout {
      codec => json_lines
  } 
}
```

这里有几个注意点：

- （1）jdbc_driver_library
        mysql-connector-java-5.1.46.jar的存放目录，这个一定要配置正确，支持全路径和相对路径。如果配置不对，将会报“can ”错误。
- （2）sql_last_value
        标志目前logstash同步的位置信息（类似offset）。比如id、updatetime。logstash通过这个标志，可以判断目前同步到哪一条数据。
- （3）statement、statement_filepath
        statement：执行同步的sql语句，可以同步部分数据。
        statement_filepath：存储执行同步的sql语句。不和statement同时使用。
- （4）schedule
        定时器，表示每隔多长时间同步一次数据。格式类似crontab。
- （5）tracking_column、tracking_column_type
        tracking_column：表示表中哪一列用于判断logstash同步的位置信息。与sql_last_value比较判断是否需要同步这条数据。
        tracking_column_type：racking_column指定列的类型。支持两种类型：numeric（默认）、timestamp。注意：如果列是时间字段（比如updateTime），一定要指定这个类型为timestamp。我就踩了这个大坑。。。一直同步不成功！！！
- （6）last_run_metadata_path
        存储sql_last_value值的文件名称及位置。
- （7）document_id
        生成elasticsearch的文档值，尽量使用同步的数据中已有的唯一标识。比如同步订单数据，可以使用订单号。

### 5, 给chrome浏览器安装ElasticSearch-Head插件
