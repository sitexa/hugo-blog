+++
title = "goit:容器平台"
description = "容器平台"
tags = [
    "architecture",
]
date = "2019-04-05"
categories = [
    "goit"
]
+++

容器是与业务无关代码。包括几部分：

1.  注册发现服务：go-eureka(8761)
2.  统一配置服务：go-config(8888)
3.  应用网关：go-gateway(9000)
<!--more-->

# 容器平台

## 1,项目文件结构

![](/images/goit/container01.png)

容器是与业务无关代码。包括几部分：

1.  注册发现服务：go-eureka(8761)
2.  统一配置服务：go-config(8888)
3.  应用网关：go-gateway(9000)

## 2,根项目: go-container

git 地址： http://10.10.4.25:9876/xnpeng/go-container.git

``` 
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.1.2.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>

    <groupId>com.weibo</groupId>
    <artifactId>go-container</artifactId>
    <version>1.0.0-SNAPSHOT</version>
    <packaging>pom</packaging>
    
    <modules>
        <module>go-eureka</module>
        <module>go-config</module>
        <module>go-gateway</module>
    </modules>

    <properties>
        <java.version>1.8</java.version>
        <spring-cloud.version>Greenwich.RC2</spring-cloud.version>
        <spring-cloud-netflix-eureka.version>2.0.1.RELEASE</spring-cloud-netflix-eureka.version>
        <spring-cloud-config.version>2.0.2.RELEASE</spring-cloud-config.version>
        <admin-server.version>1.5.5</admin-server.version>
    </properties>
```

## 3,注册发现服务: go-eureka

``` 
    <artifactId>go-eureka</artifactId>
    <version>1.0.0-SNAPSHOT</version>
    <name>go-eureka</name>
    <description>go eureka</description>

    <parent>
        <groupId>com.weibo</groupId>
        <artifactId>go-container</artifactId>
        <version>1.0.0-SNAPSHOT</version>
    </parent>
```

##  4,统一配置服务：go-config

``` 
    <parent>
        <groupId>com.weibo</groupId>
        <artifactId>go-container</artifactId>
        <version>1.0.0-SNAPSHOT</version>
    </parent>

    <artifactId>go-config</artifactId>
    <name>go-config</name>
```

##  5,配置仓库：config-repo

地址： https://gitee.com/sitexa/config

##  6,应用网关：go-gateway

``` 
    <parent>
        <groupId>com.weibo</groupId>
        <artifactId>go-container</artifactId>
        <version>1.0.0-SNAPSHOT</version>
    </parent>
    <artifactId>go-gateway</artifactId>
    <version>1.0-SNAPSHOT</version>
    <name>go-gateway</name>
    <description>go-gateway</description>

    <properties>
        <java.version>1.8</java.version>
        <spring-cloud.version>Greenwich.RC2</spring-cloud.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-zuul</artifactId>
        </dependency>

    </dependencies>
```
