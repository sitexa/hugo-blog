+++
title = "goit:开发环境"
description = "开发环境"
tags = [
    "architecture",
]
date = "2019-04-05"
categories = [
    "goit"
]
+++

开发环境

<!--more-->

# 开发环境

##  1，开发服务器：Linux/CentOS 7

IP地址：10.10.4.25

##  2，Java版本：jdk1.8

``` 
[xnpeng@localhost ~]$ java -version
openjdk version "1.8.0_191"
OpenJDK Runtime Environment (build 1.8.0_191-b12)
OpenJDK 64-Bit Server VM (build 25.191-b12, mixed mode)
```

##  3，数据库：MySQL, Oracle

```
Oracle: 10.10.4.18
MySQL: 10.10.4.25:3306 root/WEIbo123!@#
```

客户端工具：MySQLWorkbench。 请自官网下载：[https://dev.mysql.com/downloads/workbench/](https://dev.mysql.com/downloads/workbench/)

##  4，构建工具：Maven

请上官网下载Maven工具：[http://maven.apache.org/download.cgi](http://maven.apache.org/download.cgi)

```
Apache Maven 3.3.9 (bb52d8502b132ec0a5a3f4c09453c07478323dc5; 2015-11-11T00:41:47+08:00)
Maven home: /Users/open/Library/apache-maven-3.3.9
Java version: 1.8.0_192, vendor: Oracle Corporation
Java home: /Library/Java/JavaVirtualMachines/jdk1.8.0_192.jdk/Contents/Home/jre
Default locale: en_CN, platform encoding: UTF-8
OS name: "mac os x", version: "10.14.2", arch: "x86_64", family: "mac"
```

基本命令：

1.  mvn clean
2.  mvn package
3.  mvn install
4.  mvn jetty:run
5.  mvn tomcat:run

##  5，Maven仓库

```
http://10.10.4.25:8081
admin/admin123
```

##  6，版本控制：GitLab

```
http://10.10.4.25:9876
```

请上官网下载git客户端工具：[https://git-scm.com/](https://git-scm.com/)

请自己注册用户，并学习使用。上面建了一个练习项目go-test。

基本命令：

1.  git clone http://10.10.4.25:9876/root/go-test.git //克隆中央git仓库中的go-test项目
2.  git status //查看版本状态
3.  git add . //将修改后的文件加入git本地仓库
4.  git commit -m '修改README' //提交修改后的版本到本地仓库
5.  git push  //推送本地仓库中的版本到中央仓库

> .gitignore的重要性：为了保持仓库干净，一切无用的代码都不允许提交到版本库，包括本地环境文件，编译生成文件，过程文件。因此，必须在项目根目录上建立.gitignore文件，将需要排除的文件、目录写进去。

```.gitignore 
.idea/
.target/
**/target/
.out/
**/out/
.build/
**/build/
logs/
**/logs/
.svn/
**/.svn/
.project/
**/.project/
.settings/
**/.settings/
go-config/basedir/

*.iml
**/*.iml
**/*.gz
**/*.log
```

##  7，开发工具：Intellij IDEA

请上官网下载：[https://www.jetbrains.com/idea/](https://www.jetbrains.com/idea/)

##  8，微服务体系：Spring Cloud v2

请上官网查看版本依赖：[http://spring.io/projects/spring-cloud](http://spring.io/projects/spring-cloud)

我们的项目使用最新发行版：```SpringCloud 2.1.1.RELEASE，SpringBoot Finchley.RELEASE```. 许多组件的版本需要精确匹配，不能随意更改。

##  9，前端技术栈： NodeJS v8.12.0 + Vue v2.5

请上官网下载安装：

NodeJS: [https://nodejs.org/en/](https://nodejs.org/en/)

VueJS: [https://cn.vuejs.org/](https://cn.vuejs.org/)

##  10, 前端UI框架：ElementUI

官网：[http://element-cn.eleme.io/#/zh-CN](http://element-cn.eleme.io/#/zh-CN)

##  11, 跨平台移动APP框架：Cordova, Flutter

官网： [https://cordova.apache.org/](https://cordova.apache.org/)

官网： [https://flutter.io/](https://flutter.io/)


