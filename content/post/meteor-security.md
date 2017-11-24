+++
title = "Meteor(18):Meteor安全"
description = "How to secure your Meteor app."
tags = [
    "JavaScript",
    "NodeJs",
    "Web application",
    "Mobile application",
    "Platform",
    "MongoDB",
]
date = "2017-11-24"
categories = [
    "Development",
    "nodejs",
]
thumbnail = "images/meteor-guide/Meteor-cordova.png"
+++

如何保护您的Meteor应用程序。

<!--more-->

阅读本指南后, 您将知道:

1.  Meteor应用程序应用的安全区。
2.  如何保护Meteor方法、发布和源代码。
3.  在开发环境和生产环境中存储密钥的位置。
4.  如何在审核应用程序时遵循安全检查表。

### 介绍

##  概念: 攻击面

因为Meteor应用通常是用一种将客户机和服务器代码放在一起的样式编写的, 所以要知道客户端上运行的是什么, 服务器上运行的是什么, 边界是什么, 这是格外重要的。这里有一个完整的地方安全检查列表需要在一个流星应用:

1.  **方法**: 通过方法参数来的任何数据都需要验证, 并且方法不应返回用户不应该访问的数据。
2.  **发布**: 通过发布参数来的任何数据都需要验证, 并且发布不应返回用户不应该访问的数据。
3.  **服务文件**: 您应该确保服务于客户端的源代码或配置文件都没有秘密数据。

每点都有自己的特殊部分。

### 避免允许/拒绝(allow/deny)

##  方法

### 验证所有参数

### mdg:validated-method

### 不要从客户端传递userId

### 每个操作一种方法

### 速率限制

就像 REST 端点一样, Meteor方法可以很容易地从任何地方调用--恶意程序、浏览器控制台中的脚本等。很容易在很短的时间内触发许多方法调用。这意味着攻击者可以很容易地测试大量不同的输入, 以找到一个有效的输入。Meteor有内置的速率限制密码登录, 以阻止密码暴力破解, 但它是由你来定义的。

在Todos示例应用程序中, 我们使用以下代码来设置所有方法的基本速率限制:

``` 
// Get list of all method names on Lists
const LISTS_METHODS = _.pluck([
  insert,
  makePublic,
  makePrivate,
  updateName,
  remove,
], 'name');
// Only allow 5 list operations per connection per second
if (Meteor.isServer) {
  DDPRateLimiter.addRule({
    name(name) {
      return _.contains(LISTS_METHODS, name);
    },
    // Rate limit per connection ID
    connectionId() { return true; }
  }, 5, 1000);
}
```

这将使每个方法限制每个连接每秒5次访问一次。这是用户根本不应该注意的速率限制, 但会阻止恶意脚本使用请求淹没服务器。您将需要调整限制参数, 以满足您的应用程序的需要。

##  发布

### 有关方法的规则仍然适用

### 始终限制字段

### 发布和userId

### 传递选项

## 服务文件

### 保护服务器代码

## 保护api key

### 客户端上的设置

### OAuth 的 API 密钥

##  SSL

**每个处理用户数据的Meteor应用程序都应该使用SSL来运行。**

### 设置 SSL

### 强制 SSL

### 安全检查表

