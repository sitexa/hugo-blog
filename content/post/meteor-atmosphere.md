+++
title = "Meteor(14):使用和编写Atmosphere包"
description = "Atmosphere包是专门为Meteor编写的软件包, 在与Meteor一起使用时, 与 npm 相比有几个优点。"
tags = [
    "JavaScript",
    "NodeJs",
    "Web application",
    "Mobile application",
    "Platform",
    "MongoDB",
]
date = "2017-11-22"
categories = [
    "Development",
    "nodejs",
]
thumbnail = "images/meteor-guide/meteor-angular.png"
+++

Atmosphere包是专门为Meteor编写的软件包, 在与Meteor一起使用时, 与 npm 相比有几个优点。

<!--more-->

##  搜索包

###  包命名

##  安装Atmosphere包

```meteor add kadira:flow-router```

##  使用Atmosphere包

```import { SimpleSchema } from 'meteor/aldeed:simple-schema';```

### 从Atmosphere包中导入样式

``` 
@import '{prefix:package-name}/buttons/styles.import.less';
```

### 对等的npm依赖项


##  Atmosphere包命名空间

#   编写atmosphere包

要开始编写atmosphere包, 请使用Meteor命令行工具:

```meteor create --package my-package```

##  添加文件和资源

### 添加javascript

### 添加css

### 添加 Sass, Less, or Stylus 变量

### 添加其他资源

##  导出

##  依赖

### Atmosphere依赖

**基于meteor版本**

**语义版本控制和版本约束**

### npm依赖

### 对等npm依赖

##  cordova插件

##  测试包

##  发布包

### 缓存格式

##  本地包

### 使用本地版本覆盖已发布的包