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

使用 FlowRouter 时, 在页面上为不同的 url 显示不同视图的最简单方法是使用补充 BlazeLayout 包。
首先, 确保安装了 BlazeLayout 软件包:

``` 
meteor add kadira:blaze-layout
```

要使用这个包, 我们需要定义一个 "布局" 组件。在Todos示例应用程序中, 该组件称为 App_body:

``` 
<template name="App_body">
  ...
  {{> Template.dynamic template=main}}
  ...
</template>
```

(这不是整个 ```App_body``` 组件, 但我们强调了这里最重要的部分)。
在这里, 我们使用称为```Template.dynamic```的 Blaze 功能来呈现一个附加到数据上下文的```main```属性的模板。
使用 ```Blaze Layout```, 我们可以在访问路由时更改该```main```属性。

我们在 ```Lists.show``` 路由定义的 ```action``` 功能中这样做:

``` 
FlowRouter.route('/lists/:_id', {
  name: 'Lists.show',
  action() {
    BlazeLayout.render('App_body', {main: 'Lists_show_page'});
  }
});
```

这意味着当用户访问表单 ```/lists/X``` 的 URL 时, ```Lists.show``` 路由将启动, 触发 ```BlazeLayout``` 调用以设置 ```App_body``` 组件的```main```属性。

##  组件作为页面

请注意, 我们将要呈现的组件称为 ```Lists_show_page``` (而不是 ```Lists_show```)。这表示此模板由 ```FlowRouter``` 操作直接呈现, 并形成此 URL 的呈现层次结构的 "顶部"。

```Lists_show_page``` 模板在**不带参数**的情况下呈现: 此模板负责从当前路由收集信息, 然后将这些信息向下传递到其子模板中。
相应地, ```Lists_show_page``` 模板与呈现它的路由密切相关, 因此它需要是一个智能组件。有关智能和可重用组件的更多信息, 请参阅 [UI/UX](https://guide.meteor.com/ui-ux.html) 的文章。

对于像 Lists_show_page 这样的 "页面" 智能组件来说, 它要做以下一些事情:

1.  收集路由信息,
2.  订阅相关订阅,
3.  从这些订阅中获取数据, 并
4.  将数据传递到组件。

在这种情况下, ```Lists_show_page``` 的 HTML 模板看起来非常简单, 而 JavaScript 代码中的大部分逻辑都是这样的:

``` 
<template name="Lists_show_page">
  {{#each listId in listIdArray}}
    {{> Lists_show (listArgs listId)}}
  {{else}}
    {{> App_notFound}}
  {{/each}}
</template>
```
(```{{#each listId listIdArray}} 是 **页面到页面转换** 的动画(animation)技术。)

``` 
Template.Lists_show_page.helpers({
  // We use #each on an array of one item so that the "list" template is
  // removed and a new copy is added when changing lists, which is
  // important for animation purposes.
  listIdArray() {
    const instance = Template.instance();
    const listId = instance.getListId();
    return Lists.findOne(listId) ? [listId] : [];
  },
  listArgs(listId) {
    const instance = Template.instance();
    return {
      todosReady: instance.subscriptionsReady(),
      // We pass `list` (which contains the full list, with all fields, as a function
      // because we want to control reactivity. When you check a todo item, the
      // `list.incompleteCount` changes. If we didn't do this the entire list would
      // re-render whenever you checked an item. By isolating the reactiviy on the list
      // to the area that cares about it, we stop it from happening.
      list() {
        return Lists.findOne(listId);
      },
      // By finding the list with only the `_id` field set, we don't create a dependency on the
      // `list.incompleteCount`, and avoid re-rendering the todos when it changes
      todos: Lists.findOne(listId, {fields: {_id: true}}).todos()
    };
  }
});
```
它是 ```listShow``` 组件 (一个可复用组件), 它实际处理呈现页面内容的作业。当页面组件将参数传递到可复用组件时, 
它能够相当准确地分离"与路由器对话"和"呈现页面"。

### 注销时更改页面

呈现逻辑的类型是与路由相关的, 但也似乎与用户界面呈现相关。一个典型的例子是授权; 例如, 如果用户尚未登录, 
您可能需要为页面的某些子集呈现登录表单。
 
最好将所有的逻辑放在组件层次结构 (即渲染组件的树) 中。因此, 此授权应发生在组件内。
假设我们想把这个添加到我们在上面看到的 ```Lists_show_page```，我们可以这样做:

``` 
<template name="Lists_show_page">
  {{#if currentUser}}
    {{#each listId in listIdArray}}
      {{> Lists_show (listArgs listId)}}
    {{else}}
      {{> App_notFound}}
    {{/each}}
  {{else}}
    Please log in to edit posts.
  {{/if}}
</template>
```

当然, 我们可能会发现我们需要在需要访问控制的应用程序的多个页面之间共享此功能。我们可以轻松地在模板之间共享功能, 
就是将它们封装在一个包装 "布局" 组件中, 且其中包括我们所需的功能。

您可以使用 Blaze 的 "template as block helper" 功能创建包装组件 (请参见 [Blaze Article](http://blazejs.org/guide/spacebars.html#Block-Helpers))。
下面是我们如何编写授权模板的方法:

``` 
<template name="App_forceLoggedIn">
  {{#if currentUser}}
    {{> Template.contentBlock}}
  {{else}}
    Please log in see this page.
  {{/if}}
</template>
```

一旦该模板存在, 我们可以简单地包装我们的 ```Lists_show_page```:

``` 
<template name="Lists_show_page">
  {{#App_forceLoggedIn}}
    {{#each listId in listIdArray}}
      {{> Lists_show (listArgs listId)}}
    {{else}}
      {{> App_notFound}}
    {{/each}}
  {{/App_forceLoggedIn}}
</template>
```

这种方法的主要优点是, 访问页面时，能立即预见到用户在查看 ```Lists_show_page``` 时会发生什么行为。

此类型的多个行为可以通过在多个包装中包装模板或创建组合多个包装模板的 meta-wrapper 来组成。

##  更改路由

当用户到达新的路由时呈现更新的UI却不给用户某种方式来到达新的路由，并不那么有用。最简单的方法是使用可靠的 <a> 标记和 URL。
您可以使用```FlowRouter. pathFor```来生成url, 但是使用[arillo:flow-router-helpers](https://github.com/arillo/meteor-flow-router-helpers/)包来为您定义一些帮助器更方便:

``` 
meteor add arillo:flow-router-helpers
```

现在有了这个包, 您可以在模板中使用帮助程序来显示指向特定路由的链接。例如, 在Todos示例应用程序中, 我们的导航链接如下所示:

``` 
<a href="{{pathFor 'Lists.show' _id=list._id}}" title="{{list.name}}"
    class="list-todo {{activeListClass list}}">
```

### 以编程方式进行路由

在某些情况下, 您希望根据用户操作 (在它们之外单击链接) 来更改路由。例如, 在示例应用程序中, 当用户创建新列表时, 
我们希望将它们路由到刚创建的列表。当知道新列表的 id 时，我们通过调用```FlowRouter. go()```来实现:

``` 
import { insert } from '../../api/lists/methods.js';
Template.App_body.events({
  'click .js-new-list'() {
    const listId = insert.call();
    FlowRouter.go('Lists.show', { _id: listId });
  }
});
```

使用```FlowRouter.setParams()``` 和 ```FlowRouter.setQueryParams()```, 我们也可以只更改部分 URL。例如, 
如果我们查看一个列表时, 想去另一个列表, 我们可以写:

``` 
FlowRouter.setParams({_id: newList._id});
```

当然, 调用```FlowRouter.go()``` 将始终有效, 因此除非您正在尝试针对特定情况进行优化, 否则最好使用该方法。

### 在 URL 中存储数据

通常, 如果要将任意可序列化数据存储在 URL 参数中, 可以使用``EJSON.stringify()``` 将其转换为字符串。
您需要使用 ```encodeURIComponent``` 对字符串进行 url 编码, 以删除在 url 中具有意义的任何字符:

``` 
FlowRouter.setQueryParams({data: encodeURIComponent(EJSON.stringify(data))});
```

然后, 您可以使用```EJSON.parse()``` 从流路由器中获取数据。请注意, ```Flow Router``` 会自动为您进行 URL 解码:

``` 
const data = EJSON.parse(FlowRouter.getQueryParam('data'));
```

##  定向(Redirecting)

有时候, 你的用户会在一个不适合他们的页面上结束。也许他们正在寻找的数据已经移动, 也许他们是在一个管理面板页退出, 
或者他们刚创建了一个新的对象, 你希望他们在刚刚创建东西的页面上结束。

通常, 我们可以通过调用```FlowRouter.go()```和相关函数对用户的操作进行重定向, 就像在上面的列表创建示例中一样,
但是如果用户直接浏览到不存在的 URL, 就应当知道如何立即重定向。

如果 url 只是过期(有时可能更改应用程序的 url 方案), 则可以在路由的操作函数内重定向:

``` 
FlowRouter.route('/old-list-route/:_id', {
  action(params) {
    FlowRouter.go('Lists.show', params);
  }
});
```

###  动态重定向

上述方法只对静态重定向起作用。但是, 有时您需要加载一些数据来找出重定向到的位置。在这种情况下, 您需要呈现组件层次结构的一部分来订阅所需的数据。
例如, 在Todos示例应用程序中, 我们希望使根 (```/```) 路由重定向到第一个已知的列表。为了实现这一点, 
我们需要呈现一个特殊的 ```App_rootRedirector``` 路由:

``` 
FlowRouter.route('/', {
  name: 'App.home',
  action() {
    BlazeLayout.render('App_body', {main: 'App_rootRedirector'});
  }
});
```

```App_rootRedirector``` 组件是在 ```App_body``` 布局中呈现的, 它负责在呈现其组件之前订阅用户知道的一组列表, 
我们被保证至少有一个这样的列表。这意味着, 如果 ```App_rootRedirector``` 最终被创建, 将会有一个列表加载, 所以我们可以简单地做:

``` 
Template.App_rootRedirector.onCreated(function rootRedirectorOnCreated() {
  // We need to set a timeout here so that we don't redirect from inside a redirection
  //   which is a limitation of the current version of FR.
  Meteor.setTimeout(() => {
    FlowRouter.go('Lists.show', Lists.findOne());
  });
});
```

如果您需要等待在创建时尚未订阅的特定数据, 则可以使用```autorun``` 和 ```subscriptionsReady()``` 来等待该订阅:

``` 
Template.App_rootRedirector.onCreated(function rootRedirectorOnCreated() {
  // If we needed to open this subscription here
  this.subscribe('lists.public');
  // Now we need to wait for the above subscription. We'll need the template to
  // render some kind of loading state while we wait, too.
  this.autorun(() => {
    if (this.subscriptionsReady()) {
      FlowRouter.go('Lists.show', Lists.findOne());
    }
  });
});
```

### 在用户操作后重定向

通常, 您只想在用户完成某一操作时以编程方式转到新的路由。在上面我们看到一个案例 (创建一个新的列表), 
当我们从服务器上听到该方法成功之前，我们想乐观地做这件事。我们可以这样做, 因为我们合理地期望该方法在几乎所有情况下都能成功
 (请参阅 [UI/UX](https://guide.meteor.com/ui-ux.html#optimistic-ui) 文章以进一步讨论此问题)。
 
但是, 如果要等待从服务器返回的方法, 我们可以将重定向放在方法的回调中:

``` 
Template.App_body.events({
  'click .js-new-list'() {
    lists.insert.call((err, listId) => {
      if (!err) {
        FlowRouter.go('Lists.show', { _id: listId });  
      }
    });
  }
});
```

您还需要显示某种指示, 表明该方法在单击按钮和完成重定向之间工作。如果方法返回错误, 请不要忘记提供反馈。

##  高级路由

### 缺少页面
    
如果用户键入了不正确的 URL, 您可能希望向他们显示某种有趣的未找到的页面。实际上有两类未找到的页面。
第一个是在键入的 URL 与您的任何路由定义不匹配时。您可以使用 FlowRouter 一套来处理此操作:

``` 
// the App_notFound template is used for unknown routes and missing lists
FlowRouter.notFound = {
  action() {
    BlazeLayout.render('App_body', {main: 'App_notFound'});
  }
};
```

第二个是 URL 是有效的, 但实际上并不匹配任何数据。在这种情况下, URL 与路由匹配, 但一旦路由成功订阅, 就会发现没有数据。
在这种情况下, 对于页面组件 (订阅和读取数据) 来说, 它通常是有意义的, 以呈现一个没有找到的模板, 而不是页面的通常模板:

``` 
<template name="Lists_show_page">
  {{#each listId in listIdArray}}
    {{> Lists_show (listArgs listId)}}
  {{else}}
    {{> App_notFound}}
  {{/each}}
<template>
```

### 分析

这是很常见的需求, 统计分析应用程序的哪些页面访问访问量大, 用户来自哪里。您可以阅读有关如何在部署指南中设置基于 ```FlowRouter``` 的分析。

### 服务端路由

正如我们已经讨论过的, Meteor是一个为客户端渲染的应用程序框架, 但这并不意味着没有对服务器呈现的路由的要求。服务器端路由主要有两个用例。

**用于 API 访问的服务器路由**

尽管流星允许您编写"低级"(low-level)连接处理程序来创建您喜欢的服务器端的任何类型的API, 但如果您想做的只是创建一个RESTFul的方法和发布,
那么您通常可以使用 [simple:rest](http://atmospherejs.com/simple/rest) 包来轻松完成此操作。
有关详细信息, 请参阅数据加载和方法文章。

如果您需要更多的控制, 您可以使用全面的 [nimble:restivus](https://atmospherejs.com/nimble/restivus) 包, 
在任何您需要的本体中或多或少地创建您所需的任何东西。

**服务器呈现**

Blaze UI 库不支持服务器端呈现, 因此如果您使用Blaze, 则不可能在服务器上呈现您的页面。但是, React库是这样做的。
这意味着, 如果您使用 React 作为渲染框架, 则可以在服务器上呈现 HTML。

虽然 FlowRouter 可以或多或少的渲染React组件, 正如我们在上面对Blaze描述的那样。在撰写本书的时候, FlowRouter 对 SSR 的支持仍然是实验性的。
然而, 如果现在你想使用Meteor做SSR，它可能是最好的方法。





