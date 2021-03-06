+++
title = "goit:微服务架构"
description = "微服务架构"
tags = [
    "architecture",
]
date = "2019-04-05"
categories = [
    "goit"
]
+++

微服务(Microservices Architecture)是一种架构风格，一个软件应用由一个或多个微服务组成。系统中的各个微服务可被独立部署，各个微服务之间是松耦合的。每个微服务仅关注于完成一件任务并很好地完成该任务。在所有情况下，每个任务代表着一个小的业务能力。

<!--more-->

# 微服务架构

## 1，什么是微服务架构

微服务(Microservices Architecture)是一种架构风格，一个软件应用由一个或多个微服务组成。系统中的各个微服务可被独立部署，各个微服务之间是松耦合的。每个微服务仅关注于完成一件任务并很好地完成该任务。在所有情况下，每个任务代表着一个小的业务能力。

![](/images/goit/microservice01.png)

## 2，什么是前后端分离

前端通常指浏览器或者APP，后端通常指服务器。前端处理界面逻辑，通常不存储数据(cookies除外)，所有数据都从服务器端获取；后端通常只处理存储逻辑和非界面逻辑，通过接口的方式向前端提供数据。

![](/images/goit/microservice02.png)

## 3，什么是单体架构

单体架构(Monolithic Architecture ) 是把所有功能放到同一个单体架构中去。比如：常见的ERP、CRM等系统都以单体架构的方式运行，同时由于提供了大量的业务功能，随着功能的升级，整个研发、发布、定位问题，扩展、升级这样一个“怪物”系统会变得越来越困难。

![](/images/goit/microservice03.png)

##  4，单体架构的微服务化

将单体架构下的应用进行服务分离和前后端分离，达到微服务架构下的设计目标。

![](/images/goit/microservice04.png)

##  5，政府网站一体化项目微服务架构

采用Spring-Cloud微服务架构体系，建设政府网站一体化平台。其基本骨架为应用网关、注册与发现中心、统一配置中心、微服务群。应用网关具有负载均衡作用，还起到链路溶断的作用，同时发挥前后端分离作用。前端应用部署在Web服务器上，可以是apache,nginx,tomcat，也可以是别的服务器，移动APP本身就是前端应用。

![](/images/goit/microservice05.png)

