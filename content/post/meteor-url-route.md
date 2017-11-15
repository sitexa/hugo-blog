+++
title = "Meteor(8):地址和路由(url & routing)"
description = "How to drive your Meteor app's UI using URLs with FlowRouter."
tags = [
    "JavaScript",
    "NodeJs",
    "Web application",
    "Mobile application",
    "Platform",
    "MongoDB",
]
date = "2017-11-15"
categories = [
    "Development",
    "nodejs",
]
thumbnail = "images/meteor-guide/meteor-router.jpg"
+++

如何使用 ```FlowRouter``` 的url驱动您的Meteor应用程序的UI。

<!--more-->

阅读本指南后, 您将知道:

1.  url 在客户端渲染的应用程序中扮演什么角色, 以及它与传统的服务器渲染应用程序的区别。
2.  如何使用```Flow Router```为您的应用程序定义客户端和服务器路由。
3.  如何让您的应用程序根据 URL 显示不同的内容。
4.  如何以编程方式构造到路由的链接, 以及如何转到路由。

##  客户端路由(Client-side Routing)

在 web 应用程序中, 路由是使用 url 来驱动用户界面 (UI) 的过程。url 在每一个 web 浏览器中都是一个重要的功能, 
并且从用户的角度来看, 有几个主要功能:

1.  Bookmarking —— 用户可以在 web 浏览器中为 url 添加书签以保存内容, 以便他们以后再来。
2.  Sharing ——  用户可以通过发送指向特定页面的链接来与他人共享内容。
3.  Navigation —— url 用于驱动 web 浏览器的后退/前进功能。

在传统的 web 应用程序堆栈中, 服务器一次呈现一个HTML页面, URL 是用户访问应用程序的基本入口点。
用户通过单击 url (通过 HTTP 将 url 发送到服务器) 来导航应用程序, 并且服务器通过服务器端路由器进行适当的响应。

跟传统应用程序相反，Meteor的操作是基于"在线数据(data on the aire)"原理, 服务器不考虑 url 或 HTML 页面。客户端应用程序通过 DDP 与服务器进行通信。
通常, 当应用程序加载时, 它将初始化一系列订阅, 以获取呈现应用程序所需的数据。当用户与应用程序交互时, 可能会加载不同的订阅, 
但在这个过程中不需要使用 url, 您可以轻松地拥有一个url 不更改的Meteor应用程序。

然而, 以上列出的大多数面向用户的url功能仍然适用于典型的Meteor应用。由于服务器不是url驱动的, 因此url仅仅成为用户当前正在查看的客户端状态的表示。
但是, 与服务器呈现url的应用程序中不同, url不需要描述用户当前状态的全部;它只需要包含要链接的部分。
例如, URL应包含在页面上要用的所有搜索条件, 但不一定是下拉菜单或弹出窗口的状态。

##  使用```Flow Router```

要将路由添加到您的应用程序, 请安装[kadira:flow-router](https://atmospherejs.com/kadira/flow-router):

``` 
meteor add kadira:flow-router
```

```Flow router```是一种用于Meteor的社区路由包。

##  定义简单路由

路由器的基本用途是匹配特定的url并执行相应的操作。这一切发生在客户端, 在应用程序的用户浏览器或移动应用程序的容器内。
让我们以Todos示例应用程序为例:

``` 
FlowRouter.route('/lists/:_id', {
  name: 'Lists.show',
  action(params, queryParams) {
    console.log("Looking at a list?");
  }
});
```

此路由处理程序将在两种情况下运行: 页面在最初与url模式匹配的url上加载, 或者在页打开时，将url更改为与模式匹配的地址。
请注意, 与服务器端呈现的应用程序不同, URL可以在没有任何其他请求的情况下更改。

当路由匹配时, 将执行所匹配的操作方法, 然后您可以执行任何所需的操作。路由的 name 属性是可选的, 但以后将让我们更方便地引用此路由。

### URL模式匹配

请考虑上面的代码段中使用的以下 URL 模式:

``` 
'/lists/:_id'
```

上面的模式将匹配某些 url。您可能会注意到, url有一个部分的前缀是":" ——这意味着这部分是url参数, 并且将匹配该路径段中存在的任何字符串。
FlowRouter 将使该部分的 URL 在当前路由的```params```属性中可用。

此外, URL 还可以包含HTTP查询字符串 (在"?"部分之后)。如果是这样, FlowRouter 也会将它拆分为命名参数, 称为 queryParams。

下面是一些示例 url 和结果 params 和 queryParams:

URL	| matches pattern? | params	| queryParams
----|------------------|--------|------------
/   | no               |        |   
/about|no|  |
/lists/|no| |
/lists/eMtGij5AFESbTKfkT|yes|{ _id: “eMtGij5AFESbTKfkT”|{}
/lists/1|yes|{ _id: “1”}|{}
/lists/1?todoSort=to|yes|{ _id:1}|{ todoSort: “top” }

请注意, ```params``` 和 ```queryParams``` 中的所有值始终是字符串, 因为 url 没有任何方式对数据类型进行编码。
例如, 如果希望一个参数表示一个数字, 则可能需要在访问它时使用 ```parseInt (value, 10)``` 对其进行转换。

##  访问路由信息

除了将参数(parameters)作为参数(arguments)传递给路由上的```action```函数之外, FlowRouter 还通过全局单例类```FlowRouter```
中的函数 (响应式函数和其他函数) 来提供各种信息。当用户在您的应用程序里面导航时, 这些函数的值会相应地改变 (在某些情况下是响应式的)。

与应用程序中的任何其他全局单单例一样 (请参阅有关存储的数据加载的信息), 最好限制对 ```FlowRouter``` 的访问。
这样, 你的应用程序的部件将保持模块化和独立性。在 ```FlowRouter``` 的情形中,无论是在 "页面" 组件中, 还是在包装组件的布局中，
最好仅从组件层次结构的顶部访问它。阅读有关访问UI中的数据的文章以了解详细信息。

### 当前路由

在代码中访问有关当前路由的信息是很有用的。以下是一些可以调用的响应式函数:

-   ```FlowRouter.getRouteName()``` 得到路由名称
-   ```FlowRouter.getParam(paramName)``` 返回一个URL参数(param)的值
-   ```FlowRouter.getQueryParam(paramName)``` 返回一个URL查询参数(queryParam)的值

在Todos应用程序的列表页示例中, 我们使用 ```FlowRouter.getParam ("_id")``` 访问当前列表的id(我们将在下面看到更多内容)。

### 突出显示活动路由

一种情况是, 在通过导航组件渲染链接时, 访问全局```FlowRouter```单例来访问当前路由的(在组件层次结构中更深层次的)信息是明智之举。
通常需要以某种方式突出显示"活动"路由 (这是用户当前正在查看的站点的路由或环节)。

一个方便的包是[zimme:active-route](https://atmospherejs.com/zimme/active-route):

``` 
https://atmospherejs.com/zimme/active-route
```

在Todos示例应用程序中, 我们在 ```App_body``` 模板中链接到每个用户知道的列表:

``` 
{{#each list in lists}}
  <a class="list-todo {{activeListClass list}}">
    ...
    {{list.name}}
  </a>
{{/each}}
```

我们可以确定用户当前是否正在使用 ```activeListClass``` 帮助器查看列表:

``` 
Template.App_body.helpers({
  activeListClass(list) {
    const active = ActiveRoute.name('Lists.show')
      && FlowRouter.getParam('_id') === list._id;
    return active && 'active';
  }
});
```

##  基于路由的呈现

现在我们了解了如何定义路由和访问有关当前路由的信息, 我们可以执行用户访问路由时通常要执行的操作-将用户界面呈现给代表它的屏幕。

在本节中, 我们将讨论如何使用 "Blaze" 作为UI引擎来呈现路由。如果使用"Reactive"或"Angular"来构建应用程序, 
概念是类似的, 但是代码会有一点不同。

