+++
title = "Meteor(10):Meteor Blaze"
description = "How to use Blaze, Meteor's frontend rendering system, to build usable and maintainable user interfaces."
tags = [
    "JavaScript",
    "NodeJs",
    "Web application",
    "Mobile application",
    "Platform",
    "MongoDB",
]
date = "2017-11-20"
categories = [
    "Development",
    "nodejs",
]
thumbnail = "images/meteor-guide/blaze-logo.png"
+++

如何使用Blaze——Meteor的前端渲染系统, 建立可用的和可维护的用户界面。

<!--more-->

阅读本指南后, 您将知道:

1.  如何使用 Spacebars 语言来定义由Blaze引擎渲染的模板。
2.  在Blaze中编写可重用组件的最佳做法。
3.  Blaze渲染引擎如何在引擎盖下工作, 以及使用它的一些先进技术。
4.  如何测试Blaze模板。

Blaze是Meteor的内置响应式渲染库。通常, 模板是写在 [Spacebars](http://blazejs.org/guide/spacebars.html) 中的, 
它是一个[handlebars](http://handlebarsjs.com/)的变体, 以利用[Tracker](https://github.com/meteor/meteor/tree/devel/packages/tracker)的优势——Meteor的响应式系统。
这些模板被编译成由Blaze库渲染的 JavaScript UI 组件。

Blaze不是建立Meteor应用程序所必须的——你也可以很容易地使用[React](http://react-in-meteor.readthedocs.org/en/latest/)或[Angular](http://www.angular-meteor.com/)来开发你的UI。
但是, 这篇文章将带您完成在 "Blaze" 中构建应用程序的最佳实践, 它在所有其他文章中都用作 UI 引擎。

-----------------
##  Spacebars模板

Spacebars 是一个像Handlebar一样的模板语言, 建立在渲染一个反应变化的数据上下文的概念上。Spacebars 模板看起来像简单的 HTML 加上特殊的 "胡子" 标签, 由大括号```{{}}``` 分隔。

例如, 请考虑Todos示例应用程序中的 ```Todos_item``` 模板:

``` 
<template name="Todos_item">
  <div class="list-item {{checkedClass todo}} {{editingClass editing}}">
    <label class="checkbox">
      <input type="checkbox" checked={{todo.checked}} name="checked">
      <span class="checkbox-custom"></span>
    </label>
    <input type="text" value="{{todo.text}}" placeholder="Task name">
    <a class="js-delete-item delete-item" href="#">
      <span class="icon-trash"></span>
    </a>
  </div>
</template>
```

此代码段说明了几件事:

```{{#each .. in}}``` 块帮助器, 它为数组或游标中的每个元素重复 HTML 块, 或者如果不存在任何项, 则呈现 ```{{else}}``` 块的内容。
模板包含标记, ```{{>> Todos_item (todoArgs todo)}}```, 它使用从 ```todosArg``` 帮助器返回的数据上下文呈现 ```Todos_item``` 组件。
您可以阅读 [Spacebars](http://blazejs.org/api/spacebars.html) 中的完整语法。在本节中, 我们将尝试覆盖一些重要的细节, 不仅仅是语法。

### 数据上下文和查找

### 使用参数调用助手

### 模板包含

### 属性帮助

### 渲染原始 HTML

### 块帮助器

### 内置块帮助器

#### If / Unless

#### Each-in

#### Let

#### Each 和 With

### 阻止帮助的链接

###  严格

###  转义(Escaping)

-------------------
##  Blaze中可重用组件

### 验证数据上下文

### 将数据上下文命名为模板包含

### 首选 {{#each...}}

### 将数据传递给帮助器

### 使用模板实例

### 使用 reactive-dict 状态

### 将函数附加到实例

### 在模板实例的DOM范围内进行查找

### 使用事件映射的js选择器

### 将 HTML 内容作为模板参数传递

### 传递回调函数

### 对第三方库使用 onRendered ()

--------------------
## 用Meteor书写智能组件

### 从onCreated中订阅

### 从帮助器中获取数据

--------------------
##  在Meteor中重用代码

### 组合

### 类库

### 全器帮助器

--------------------
##  理解Blaze

### 重新渲染

### 控制重新渲染

### 属性帮助器

### 查找顺序

### Blaze和构建系统

### 什么是视图?

-------------
##  路由

### Iron Router

### Flow Router

### Flow Router Extra
    

--------
--------
## API

###  Templates

###  Blaze

###  spacebars
