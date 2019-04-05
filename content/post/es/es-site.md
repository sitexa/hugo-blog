+++
title = "es:ElasticSearch站内搜索和应用内搜索"
description = "ElasticSearch是一个基于Lucene的搜索服务器。它提供了一个分布式多用户能力的全文搜索引擎，基于RESTful web接口。"
tags = [
    "technology"
]
date = "2019-04-05"
categories = [
    "es"
]
+++

ElasticSearch站内搜索和应用内搜索

<!--more-->

# ElasticSearch站内搜索和应用内搜索

##  难题：深入搜索

Elastic Site Search 和 Elastic App Search 均使用 Elasticsearch，这是一款开源、分布式的 RESTful 搜索引擎。没错，这两项服务都离不开 Elasticsearch！搜索固然是一个大难题，但 Elasticsearch 是一款久经考验和测试的工具，能够帮助弥补搜索过程中最难以克服的痛点。

我们可以将搜索过程中的难题细分为三个主要方面：

*摄取* --- 您拥有数据，并且希望可以对自己的数据进行搜索。摄取行为就是接收一个对象（一个网页、来自后端 API 的一个响应），并将其转换为可搜索文档。这一过程又称为索引：摄取数据并将其转化为最优形式以便搜索引擎可以处理。如何对数据进行索引？如何对基础架构进行托管？如果数据在公共网络上，应该怎么办？如果在私有网络上，又该怎么办？  

*您希望能够简化摄取过程，希望获得灵活性，而且还要将开发运行成本降至最低。*

*结果交付* --- 搜索引擎中充满了文档，而且您可以对其进行搜索。您希望返回什么内容？希望返回文档，没错，但是返回哪些文档呢？返回多少文档呢？返回的文档要采用什么格式呢？您希望营造何种体验？还要有自动完成功能呢，您会打造预测体验（想用户之所未想）吗？这些结果能够产生价值吗？

*您希望这些强相关度结果能够帮助您完成业务目标，改善投资回报率 (ROI)，并让用户享受卓越体验。*

*管理* --- 搜索体验的设计和开发工作完成之后，您要如何对其进行管理呢？如果需要开发人员投入时间进行构建，还需要多少时间来对其进行调整和改善呢？非技术型利益相关者可以参加持续优化吗？如何管理访问权限？您会采集分析数据并将搜索整合至您的分析管道吗？

*您的愿望是，目前运行良好的搜索能够帮助到所有利益相关者，为他们产生洞见，并且可以在产生最少冲突的前提下进行改善和改动。*

##  两种出色方案：各有千秋

Elastic App Search 和 Elastic Site Search 可以满足很多不同用例的深入搜索要求。无论您是电商平台还是媒体公司，也不论您要将搜索服务应用于知识库、游戏应用程序、Saas 服务还是多用途平台，这两种解决方案中肯定有一种能为您带来优质的搜索体验。

这两种解决方案都是完全托管型，可提供完备的文档，能够为用户提供世界级的支持和功能：

- 通过同义词、比重、结果排名等功能，可对相关度和细粒度相关性进行调整，堪称业内最佳。
- 提供针对语言进行优化的引擎，可以提高 14 种不同语言的搜索相关性。
- 深入分析内容能够在这些方面提供深入洞察：用户的搜索方式、用户找到的内容，以及用户没有找到的内容。
- 基于 API 的索引，以及查询阶段的可选参数（例如分面搜索、筛选、排序和提升）的高级搜索 API。

但是，我应该选择哪一项呢？这最终取决于您最希望通过什么方式来解决摄取、结果交付和管理问题。

您希望自动对网页进行扫描和索引，使用开箱即用型解决方案和插件，利用简洁且功能丰富的仪表板，并应用支持性 API 端点吗？如果这样的话，Elastic Site Search 符合您的要求。

或者，您希望将最深入、最多样的搜索和功能 API 端点融入到应用程序代码中，并通过易于理解的现代化仪表板对相关度进行微调吗？如果这样的话，您应该选择 Elastic App Search。

### Elastic Site Search：通过爬取对所有内容进行索引

Elastic Site Search 的核心是 Site Search 爬虫。Site Search 爬虫的运行方式与其他任何网络爬虫的运行方式相似。每个主流搜索引擎（Google、DuckDuckGo、Bing 等）都会向面向公众的页面分派一套名为爬虫的复杂系统。爬虫会对这些页面进行扫描和索引，收集指标，采集内容，并且在这一过程中创建文档。

Site Search 爬虫由 Elastic 进行托管和管理。其会代您完成数据引入过程。这是一款自动化工具，能够智能地对错误进行响应，而且不需要持续对其进行配置。添加域名，还可以选择对元数据标签、Robots.txt 文件、RSS/Atom 消息源或者网站地图进行整理，然后您即可高枕无忧，因为这些页面会自动转化为可搜索的索引式文档。

举例说明，如果您有一个包含诸多帮助文章的大型公共知识库。只需输入网站地址，Site Search 爬虫便会在每个页面爬取并创建索引。索引完成后，爬虫会将您的页面根据模式整理为字段。现在便可应用某项功能（例如比重）的相关度调整能力了。

您可以选择特定字段集，然后调整每个字段的“比重”。举例说明，如果我们想针对文档名称匹配用户搜索内容，并且不希望正文文本影响搜索结果，需要怎么做呢？如果我们更希望显示较为热门的结果（即点击量更高一些），又需要怎么做呢？

我们需要选择 title（名称）、body（正文）和 popularity（热门程度）字段，然后在 1-10 的滑尺上调整比重。

尽管自动引入和其他闪亮功能十分吸引人，但是创建搜索体验时又如何呢？Elastic Site Search 如何帮助我将这些强大的索引能力和细粒度相关度控制功能添加到我的网站上呢？Elastic Site Search 提供设计和自定义选项，从简单的开箱即用型解决方案，到可完全自定义的动态搜索体验，定有一项能满足您的需求。

通过安装 Site Search JavaScript 片段 ，并创建新搜索字段或者调整现有搜索字段，您便可使用可配置的开箱即用型 Site Search 覆盖层来显示搜索结果。

如果希望定制设计呢？Elastic Site Search 包括一个 API 集合，可帮助您进行定制设计。现提供支持 Java、Node.js、Python 和 Ruby 语言的自有客户端。我们针对搜索和自动完成功能提供热门的 JavaScript 库，这些可以作为起点来助您打造天马行空的定制体验：

爬虫能够轻松完成重要工作，Elastic Site Search 的强大功能在这一刻得到最充分展示。您只需投入极少时间即可开始使用，可以根据您的喜好进行调整，而且可以确保提供最新结果，性能值得信赖。Elastic Site Search API 和插件是特别实用的补充型工具。然而，某些功能仅在仪表板内提供，没有相对应的 API 接口。

为了对更深层的 API 接口进行完全编程式控制，我们推出了 Elastic App Search。

##  Elastic App Search：以 API 为中心，易于使用

Elastic App Search 以 API 为中心。现提供适用于 Ruby、JavaScript、Java、Node.js 和 Python 的 SDK 客户端。Elastic App Search 不使用爬虫。

使用 Elastic App Search API 时，开发人员将要决定如何生成需要索引的对象数据以及如何应用不同的 API 接口。摄取、结果交付和整体实施均通过编程方式来处理。无论您是在引人入胜的仪表板、复杂的网页或移动应用程序、游戏中创建搜索，还是在商店中创建搜索，只要您能对其进行编程，便可让您的对象具有可搜索性。

所有仪表板功能均转换为强大的细粒度 API，然后您便可将其写入您的应用程序代码中。举例说明，Elastic Site Search 和 Elastic App Search 均允许您创建同义词集合。同义词集合具有卓越的实用性；搜索者可能会使用完全不同的词语。对我来说是 car（轿车），对别人而言就是 vehicle（车辆），或者 jalopy（老爷车），或者 automobile（汽车），简直不胜枚举。

在 Elastic Site Search 中，您可以通过仪表板应用同义词：

![](/static/images/add_synonym.png)

在 Elastic App Search 仪表板内，您也可进行同样的操作：

![](/static/images/manage_synonyms.png)

或者您也可通过具有完备文档的 API 接口提交请求：

``` 
curl -X POST 'https://host-xxxxxx.api.swiftype.com/api/as/v1/engines/rent-a-car/synonyms' \
-H 'Content-Type: application/json' \
-H 'Authorization:Bearer private-xxxxxxxxxxxxxxxxxxxx' \
-d '{
  "synonyms": ["car", "vehicle", "jalopy"]
}'
```

Elastic 应用程序 API 则要深入得多。想一下，如果开发电商平台的话，人们如何使用 Analytics API 套件。搜索是以自由表达内容开始的，因此从中获取的分析洞见将会特别深刻。Analytics 套件可以返回有关用户查询的信息，并揭示在可调整时间框架内人们点击了哪些文档。

``` 
curl -X POST 'https://host-2376rb.api.swiftype.com/api/as/v1/engines/sample-engine/analytics/queries' \
-H 'Content-Type: application/json' \
-H 'Authorization:Bearer private-xxxxxxxxxxxxxxxxxxxxxxxx' \
-d '{
  "filters": {
    "date": {
      "from":"2018-06-15T12:00:00+00:00",
      "to":"2018-06-19T00:00:00+00:00"
    }
  }
}'
```

举例说明，如果您商店出售的某件商品在一夜之间忽然红遍大江南北，会怎样？忽然间，某个文档的搜索量和点击量出现爆炸式增长。这里潜藏着巨大的商机！您需要怎么做呢？

您可以写一个函数，此函数会使用 Analytics API 来采集通过搜索所找到的最热门文档。然后，您可以延展此函数，让其自动在主页中以醒目显眼的视图发布这些文档。用户搜索显示，需求和自动化功能会进行实时优化。尽管分析很深入且实用，但是 Elastic App Search 的最强项是搜索能力。

文档内的模式字段可以拥有后面这四种类型中的一种：文本、数字、日期和地理位置。Elastic App Search 对所有四种类型都具备深入的搜索功能。地理位置越来越热门，这是有充分原因的。某人此刻正在某处或者正准备前往一个新地方，他们想知道附近的情况。

通过利用用户的坐标，您能够将他们的地理位置写入搜索查询中，并根据距离提升搜索的相关度。

好比，您有一个移动应用，其中列出了世界各地的健康餐厅：

``` 
curl -X GET 'https://host-xxxxxx.api.swiftype.com/api/as/v1/engines/food-paradise/search' \
-H 'Content-Type: application/json' \
-H 'Authorization:Bearer search-xxxxxxxxxxxxxxxxxxxxxx' \
-d '{
  "boosts": {
    "current_location": {
      "type": "proximity",
      "function": "linear",
      "center":"37.6213, -122.3790",
      "factor":8
    }
  },
  "query": "sushi"
}'
```

搜索寿司的话，其会根据餐厅与中心的距离，将搜索结果的相关度与所提供的因数相乘。中心是根据从搜索者处收到的位置数据得出来的。这样的话，您便有机会基于地理位置提供内容，所以提供的结果也会高度相关。

第三个例子可能只有使用 Elastic App Search API 的深入功能才能实现，即高级分组。假设现在的用例是您需要提供文档。在更新产品版本时，您也需要更改文档的版本。

``` 
curl -X GET 'https://host-2376rb.api.swiftype.com/api/as/v1/engines/sample-engine/search' \
-H 'Content-Type: application/json' \
-H 'Authorization:Bearer search-o5bk7qpaedd2xmcsavb1d8os' \
-d '{
  "query": "meta tags",
  "result_fields": {
    "url": {
      "raw": {}
    },
    "title": {
      "raw": {}
    },
    "description": {
      "raw": {}
    },
    "version": {
      "raw": {}
    }
  },
  "group": {
    "field": "url"
  }
}'
```

如果用户需要查询某一特定功能，您如何协调出现此功能的不同文档的版本呢？分组查询可确保将同一主题的所有不同版本页面作为单一搜索结果返回。您可以允许用户从单一结果中选择他们希望查看的版本，而不是让他们分析数十条有着不同版本编号的结果。

Elastic 应用程序大大拓宽了搜索的范围，不仅仅局限于访客在搜索字段内寻求相关文档。搜索成为了一项功能性活动：您拥有文档，您的代码能够通过智能方式对他们进行搜索。无需使用某个搜索字段的总体上下文，您便可根据希望查找的内容将行为实现自动化。

乍看来，Elastic App Search 好像适用于“技术背景较强”的用户。API 肯定能够让希望实现灵活且高相关度搜索的开发人员感到满意，这一点毫无疑问。虽然 Elastic App Search 需要您自行创建界面并进行索引，但仪表板和仪表板的全部优势却适用于所有技术专长水平的用户。

##  结论

Elastic 解决方案能够满足您的需求。如果您希望在网站上或者应用程序内提供高品质的搜索体验，请在 Elastic Site Search 或 Elastic App Search 之间选择，挑选最能满足您需求的一项就可以啦。

访问 Elastic Site Search 或 Elastic App Search 解决方案页面了解详情。

