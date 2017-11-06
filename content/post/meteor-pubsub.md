+++
title = "Meteor(4):发布(Publications)与订阅(Subscriptions)"
description = "How and where to load data in your Meteor app using publications and subscriptions."
tags = [
    "JavaScript",
    "NodeJs",
    "Web application",
    "Mobile application",
    "Platform",
    "MongoDB",
]
date = "2017-11-04"
categories = [
    "Development",
    "nodejs",
]
thumbnail = "images/meteor-guide/SynchronizedPUB-SUBPattern.jpg"
+++


在Meteor应用程序中使用发布(publications)和订阅(subscriptions)加载数据(loading data)。

<!--more-->

阅读本指南后, 您将知道:

1.  在Meteor上什么是发布(publications)和订阅(subscriptions)
2.  如何在服务器上定义发布(publication)。
3.  在客户端什么地方使用哪个模板(templates)订阅(subscribe)发布(publication)。
4.  一些用于管理订阅(subscriptions)的好用的模式(patterns)。
5.  如何使用反应式(reactively)方法发布(publish)相关数据。
6.  如何确保您的发布(publications)对反应式(reactively)变化的安全性。
7.  如何使用低级发布 API 发布内容。
8.  订阅(subscribe)发布(publication)时会发生什么。
9.  如何将第三方 REST 端点转换为发布。
10. 如何将应用程序中的发布(publications)转换为 REST 端点。

##  发布(publications)和订阅(subscriptions)

在传统的http模式下的web应用程序中, 客户端和服务器以 "请求-响应" 的方式进行通信。通常情况下, 客户端对服务器发出 rest 风格的 HTTP 请求, 
并接收 HTML形式 或 JSON形式的数据响应, 但在后端发生更改时, 服务器无法将数据 "推送" 到客户端。

Meteor是建立在分布式数据协议 (DDP)基础上的平台, 允许数据双向传输。构建一个Meteor应用程序并不要求您设置 REST 端点来序列化和发送数据。
而是创建可以将数据从服务器推送(push)到客户端的发布(publish)端点(endpoint)。

在Meteor中, 发布(publication)是服务器上的命名API, 它构造一组要发送到客户端的数据。客户端启动一个连接到发布的订阅, 并接收该数据。
该数据由在初始化订阅时发送的第一批数据组成, 然后在发布的数据发生更改时进行增量更新。

因此, 订阅可以被看作是一组随时间变化的数据。通常, 订阅结果是订阅 "桥接" 服务器端的 [MongoDB集合](https://guide.meteor.com/collections.html#server-collections), 
以及该集合的客户端 [Minimongo缓存](https://guide.meteor.com/collections.html#client-collections)两者组成。您可以将订阅看作是一个管道, 它将 "真实"集合的子集与客户端的版本连接起来, 
并不断更新来自服务器上的最新信息。

##  定义发布(publication)

应在服务器文件中定义发布。例如, 在Todos示例应用程序中, 我们希望将一组公共列表发布给所有用户:

``` 
Meteor.publish('lists.public', function() {
  return Lists.find({
    userId: {$exists: false}
  }, {
    fields: Lists.publicFields
  });
});
```

关于这个代码块有几件事情需要了解。首先, 我们用一个字符串```lists.public```命名了该发布。这将是我们从客户端访问该发布的地方。
第二, 我们只是从发布函数返回一个MongoDB游标。请注意, 游标被筛选为只返回集合中的某些字段, 详见[安全文章](https://guide.meteor.com/security.html#fields)。

这意味着, 发布只是简单地确保对订阅该查询的任何客户端都可用的数据集进行匹配。在这种情况下, 所有列表都没有设置userId。
因此,在该订阅打开时,客户机上的名为 "Lists" 的集合将具有服务器集合中名为 "Lists" 的所有可用的公共列表。
在Todos应用程序这个特定示例中, 当应用程序启动且未停止时, 订阅将被初始化, 但稍后部分将讨论[订阅的生命周期](https://guide.meteor.com/data-loading.html#patterns)。

每个发布都采用两种类型的参数:

1.  ```this```上下文中有关于当前 DDP 连接的信息。例如, 您可以使用```this.userId```来访问当前使用者 _id。
2.  可以在调用```Meteor.subscribe```时给发布传递参数。

>   注意: 由于我们需要访问 "this" 的上下文, 因此需要使用 "function() {}" 形式来发布而不是 ES2015 的"() = >> {}"形式。
您可以禁用发布文件的 ```eslint-disable prefer-arrow-callback``` 的箭头函数语法检查规则。未来版本的发布 API 将更适合于 ES2015。

在这份加载有私有lists的发布中, 我们需要使用```this.userId```来只获取只属于特定用户的 todo 列表。

``` 
Meteor.publish('lists.private', function() {
  if (!this.userId) {
    return this.ready();
  }
  return Lists.find({
    userId: this.userId
  }, {
    fields: Lists.publicFields
  });
});
```

得益于 DDP 和Meteor的帐户系统提供的保证, 上述发布可以确保它只会把私人列表发布给属于他们用户。请注意, 如果用户注销 (或再次返回),
则发布将重新运行, 这意味着已发布的私有列表集将随着活动用户的更改而更改。

在用户登出的情况下, 我们显式调用 ```this.ready()```, 表明我们已经给订阅发送了我们最初要发送的所有数据 (在本例中为 none) 。
重要的是要知道, 如果您不从发布中返回游标或调用 ```this.ready()```, 用户的订阅将永远不会准备就绪, 并且他们可能永远只看到加载状态。

下面是一个带有命名参数的发布示例。请注意, 必须检查通过网络传来的参数类型。

``` 
Meteor.publish('todos.inList', function(listId) {
  // We need to check the `listId` is the type we expect
  new SimpleSchema({
    listId: {type: String}
  }).validate({ listId });
  // ...
});
```

当我们在客户端订阅此发布时, 我们可以通过调用Meteor.subscribe()提供此参数:

``` 
Meteor.subscribe('todos.inList', list._id);
```

### 组织发布

将发布与它所针对的功能放在一起是有意义的。例如, 有时候, 发布提供非常具体的数据, 这些数据的视图只对开发者有用。
在这种情况下, 将发布放在与视图代码相同的模块或目录中才合理。

然而, 通常情况下, 大都是通用发布。例如, 在Todos示例应用程序中, 我们创建了一个 ```todos.inList``` 发布, 它发布了列表中的所有todos。
尽管在应用程序中我们只在一个位置 (在 Lists_show 模板中) 使用它, 但在一个较大的应用程序中, 我们很有可能在其他位置访问所有todos的列表。
因此, 将发布放在todos包中是一种明智的做法。

##  订阅数据

要使用发布, 您需要在客户端上创建对它的订阅。要这样做, 需要通过发布的名称调用 ```Meteor.subscribe()``` 。
当这样做时, 它将打开对该发布的订阅, 并且服务器开始将数据发送到线上, 以确保您的客户端集合包含由发布指定的数据的最新副本。


```Meteor.subscribe()``` 还会返回一个具有```.ready()``` 的属性的"订阅句柄"。这是一个反应函数, 当发布标记就绪时返回 true 
(除非显示调用 "ready()", 否则发送过来的是返回游标的初始内容)。

``` 
const handle = Meteor.subscribe('lists.public');
```

### 停止订阅

订阅句柄还具有另一个重要属性: ```.stop()``` 方法。当您订阅时, 确保在您完成订阅时始终调用 ```.stop()``` 是非常重要的。
这样可以确保从本地 Minimongo 缓存中清除订阅发送的文档, 并且服务器停止执行为订阅服务所需的工作。如果忘记了停止调用, 
则会在客户端和服务器上消耗不必要的资源。

然而, 如果你在反应式上下文中有条件地调用 ```Meteor.subscribe()```(例如autoRun, 或在Reactive中用 getMeteorData) 
，或在Blaze组件上通过```this.subscribe()```调用, Meteor的反应式系统会在适当的时间自动调用 ```this.stop()```。

### 在UI组件中订阅

最好将订阅尽可能地放置在需要订阅数据的地方。这样可以减少 "远距离操作", 并使您更容易理解通过应用程序的数据流。
如果订阅和提取是分开的, 我们很难搞清楚对订阅的更改 (如更改参数) 为什么和怎么样影响游标的内容的。

实际上, 这意味着您应该将订阅调用放在组件中。在Blaze中, 最好在 onCreated () 回调中执行此操作:

``` 
Template.Lists_show_page.onCreated(function() {
  this.getListId = () => FlowRouter.getParam('_id');
  this.autorun(() => {
    this.subscribe('todos.inList', this.getListId());
  });
});
```

在这段代码段中, 我们可以看到两个重要的技术, 用于订阅Blaze模板:

1.  调用```this.subscribe()```(而不是 ```Meteor.subscribe()```), 它将一个特殊的 ```subscriptionsReady()``` 函数附加到模板实例上,
该函数在该模板内的所有订阅都准备就绪时返回```true```。
2.  调用```this.autorun``` 会设置一个反应式上下文, 每当反应式功能```this.getListId()```发生变化时，它(```this.autorun```)将重新初始化订阅, 。

有关Blaeze订阅的详细信息，请参阅[Blaze article](http://blazejs.org/api/templates.html#Blaze-TemplateInstance-subscribe)，
有关在UI组件内跟踪加载状态的内容，请参阅[UI article](https://guide.meteor.com/ui-ux.html#subscription-readiness)。


### 获取数据

订阅数据并将它放在客户端集合中。要在用户界面中使用数据, 您需要查询您的客户端集合。这样做的时候, 有几条重要的规则要遵循。

**总是使用特定的查询来获取数据**

如果要发布数据的子集, 只需查询集合中可用的所有数据 (即```Lists.find()```), 以便在客户机上获取该集合的子集, 
而不用重新指定您以前用来发布该数据的Mongo选择器。

但是, 如果您这样做, 那么如果另一个订阅将数据推入到同一集合中时, 您就会遇到问题, 因为通过 ```Lists.find()``` 返回的数据可能不再是您所期望的了。
在一个活跃开发的应用程序中, 通常很难预测将来会发生什么变化, 这可能是很难理解 bug 的根源。

另外, 在订阅之间进行更改时, 会有一个简短的时间段, 期间加载了两个订阅 ([更改参数时的发布行为](https://guide.meteor.com/data-loading.html#publication-behavior-with-arguments)),
因此, 在执行类似于分页的操作时, 非常可能会出现这种情况。

**获取您订阅的附近的数据**

我们这样做的原因与我们在靠近组件的地方订阅的理由相同--避免"远距离行动", 并且更容易理解数据的来源。常见的模式是在父模板中获取数据, 
然后将其传递到 "纯" 子组件中, 就像我们在 [UI文章](https://guide.meteor.com/ui-ux.html#components)中看到的那样。

请注意, 第二条规则有一些例外。一个常见的例外是```Meteor.user()``` —— 虽然对订阅(自动通常)来说这是严格的，但作为一个参数通过组件层次传递给每个组件时，是很复杂的。
所以请记住, 最好不要在太多的地方使用它, 因为它会使组件更难测试。

### 全局订阅

有一种情况是你不需要在组件中订阅数据，那是因为你知道你总是需要那个数据。例如，在你的应用程序的每个屏幕上你都需要用户数据(请参阅[帐户文章](https://guide.meteor.com/accounts.html))。

如果有一些屏幕不需要这些数据，那么使用布局组件 (将所有组件都封装在其中) 来订阅这种订阅，并在类似的情况下保持一致，就会使系统变得更加录活。

##  数据加载的模式

在整个Meteor应用中, 有必要了解一些常见的客户端数据加载和管理模式。我们将在 [UI/UX文章](https://guide.meteor.com/ui-ux.html)中详细介绍其中的一些内容。

### 订阅就绪

订阅不会立即提供其数据，了解这一点很关键。从服务器上的发布的数据到达客户端上订阅的数据, 将会有一个延迟。
您还应该知道, 对于您的用户来说, 此延迟可能比您在本地开发中的时间长得多!

尽管在构建应用程序时通常不需要过多地考虑这一点, 但是如果您想让用户体验更好, 您还是需要知道数据何时准备好。

去发现 "流星" 订阅 (), 并返回一个 "订阅句柄", 其中包含称为的反应数据源. 准备就绪 ():

为了说明这一点，假使Meteor用```Meteor.subscribe()```订阅发布，并返回一个订阅句柄，其中包含一个反应式数据源叫做```.ready()```:

``` 
const handle = Meteor.subscribe('lists.public');
Tracker.autorun(() => {
  const isReady = handle.ready();
  console.log(`Handle is ${isReady ? 'ready' : 'not ready'}`);  
});
```

当我们尝试向用户显示数据时, 或当我们显示加载屏幕时, 我们可以使用这些信息。

### 反应式改变订阅参数

我们已经看到一个例子, 当(反应式)订阅参数发生改变时，会使用```autorun```重新订阅发布。在这种情况下会发生什么，值得挖掘更多的细节。

``` 
Template.Lists_show_page.onCreated(function() {
  this.getListId = () => FlowRouter.getParam('_id');
  this.autorun(() => {
    this.subscribe('todos.inList', this.getListId());
  });
});
```
在我们的示例中, 当 ```this.getListId()``` 变化时,（最终是因为 FlowRouter getParam ("_id") 变化） ```autorun```将重新运行,
但其他常见的反应式数据源是:

1.  模板数据上下文 (您可以Templates.currentData()访问反应式结果)。
2.  当前用户状态 (Meteor.user() 和Meteor.loggingIn())。
3.  其他应用程序特定的客户端数据存储的内容。

从技术上讲, 当其中一个反应源发生变化时会发生以下情况:

1.  反应式数据源使自动运行(autorun)的计算无效 (标记它以便在下一个跟踪器刷新周期中运行)。
2.  订阅检测到这一点, 并考虑到在下一次计算运行时，任何情况都可能的发生，便打上"销毁自己"的标记。
3.  通过```.subscribe()```使用相同或不同的参数重新计算。
4.  如果使用相同的参数运行订阅, 则 "新" 订阅发现旧的标记着 "销毁自己" 的订阅, 而且数据已经准备就绪, 就直接重用它。
5.  如果使用不同的参数运行订阅, 则会创建一个新的订阅, 它将连接到服务器上的发布。
6.  在刷新周期的末尾 (即在计算完成运行后), 旧的订阅会检查它是否被重用, 如果不是, 则向服务器发送一条消息, 通知服务器关闭它。

上面步骤4说明了一个重要的细节——系统巧妙地不去订阅那个 参数相同而且"autorun"重新运行过的订阅(subscription)。
即使在模板层次结构中的其他位置设置了新订阅, 也是如此。例如, 如果用户在两个订阅了完全相同订阅的页面之间导航, 则将启动相同的机制, 不会发生不必要的订阅。


### 参数更改时的发布行为

在新订阅启动且旧的预订已停止时，有必要了解服务器上发生了什么。

在从旧订阅中删除数据之前, 服务器会显式等待, 直到所有数据发送完成(新订阅已就绪)。这是为了避免闪烁。如果需要, 你可以继续显示旧的订阅的数据, 
直到新的数据准备就绪, 然后立即切换到新的订阅的完整数据集。

这意味着一般情况下, 在更改订阅时, 会有一段时间的超额数据：客户端上的数据比您请求的数据多。
这就是为什么您应该始终获取您订阅的相同数据 (而不是过度获取数据)的原因。

### 订阅分页

一个非常常见的数据访问模式是分页。这是指一次获取一个 "页面" 的有序数据列表的做法, 通常是一个特定数量, 比如每页二十条。

通常使用两种类型的分页, 一种是"一页接一页" 样式, 其中一次只显示一页结果, 从某一偏移量开始 (用户可以控制)；另一种是 "无限滚动" 样式, 
当用户在列表中移动时，显示越来越多的项目页,  (这是典型的 "feed" 样式的用户界面)。

在本节中, 我们将讨论第二个无限滚动分页样式的发布/订阅技术。"一页接一页"技术在Meteor中有一些困难, 因为在客户端难以计算偏移量。
如果需要这样做, 您可以遵循我们在这里使用的许多相同的技术, 并使用[过滤:从发布查找](https://atmospherejs.com/percolate/find-from-publication)包, 
以跟踪哪些记录来自您的发布。

在无限的滚动发布中, 我们只需要向发布添加一个新参数, 以控制要加载的项目数。假设我们想在我们的Todos示例应用程序中分页todo项目:

``` 
const MAX_TODOS = 1000;
Meteor.publish('todos.inList', function(listId, limit) {
  new SimpleSchema({
    listId: { type: String },
    limit: { type: Number }
  }).validate({ listId, limit });
  const options = {
    sort: {createdAt: -1},
    limit: Math.min(limit, MAX_TODOS)
  };
  // ...
});
```

重要的是, 我们在查询中设置一个```sort```参数 (以确保在请求更多页时可重复列出列表项的顺序), 并在用户可以请求的项目数上设置最大值 
(至少在列表可以不受约束地增长的情况下)。

然后在客户端, 我们设置了某种类型的反应式状态变量(reactive state variable)来控制要请求的项数:

``` 
Template.Lists_show_page.onCreated(function() {
  this.getListId = () => FlowRouter.getParam('_id');
  this.autorun(() => {
    this.subscribe('todos.inList',
      this.getListId(), this.state.get('requestedTodos'));
  });
});
```

当用户单击 "加载更多" (或者可能只是滚动到页面底部时), 我们会增加请求的Todos变量。

数据分页时有一条信息很有用：项目总数。 [tmeasday:publish-counts](https://atmospherejs.com/tmeasday/publish-counts) 包可以用于发布此信息。
我们可以添加一个```Lists.todoCount```给发布，请看下面的代码：

``` 
Meteor.publish('Lists.todoCount', function({ listId }) {
  new SimpleSchema({
    listId: {type: String}
  }).validate({ listId });
  Counts.publish(this, `Lists.todoCount.${listId}`, Todos.find({listId}));
});
```

然后在客户端, 订阅该发布后, 我们可以访问计数。

```
Counts.get(`Lists.todoCount.${listId}`)
```

##  具有反应式存储的客户端数据

在Meteor中, 持久的或共享的数据会通过发布从在线数据上传输。但是, 有一些类型的数据不需要在用户之间存储或共享。
例如, 当前用户的 "logged-in-ness" 或他们正在查看的路由。

虽然客户端状态通常最好包含为单个模板的状态 (并在必要时将状态作为参数沿模板层次结构传递下去), 但有时您需要在模板层次的不相关部分之间共享 "全局" 状态.

通常这种状态是存储在一个全局的单例对象中——我们称之为存储。单例是一种数据结构, 逻辑上只存在一个副本。当前用户和路由是这样的单例的典型例子。

### 存储的类型

在Meteor中, 最好使存储成为反应式数据源(reactive data sources), 这样它们就会自然地与生态系统的其余部分紧密联系。你可以使用几个不同的包做存储。

如果存储是单维度的(single dimensional), 您可以使用 ReactiveVar 存储它 (由 [reactive-var](https://atmospherejs.com/meteor/reactive-var) 包提供)。
ReactiveVar 有两个属性,get()和set():

``` 
DocumentHidden = new ReactiveVar(document.hidden);
$(window).on('visibilitychange', (event) => {
  DocumentHidden.set(document.hidden);
});
```

如果存储是多维的(multi-dimensional), 您可以需要使用 ReactiveDict (由 [reactive-dict](https://atmospherejs.com/meteor/reactive-dict) 包提供):

``` 
const $window = $(window);
function getDimensions() {
  return {
    width: $window.width(),
    height: $window.height()
  };
};
WindowSize = new ReactiveDict();
WindowSize.set(getDimensions());
$window.on('resize', () => {
  WindowSize.set(getDimensions());
});
```

ReactiveDict 的优点是您可以单独访问每个属性 (```WindowSize.get("width")```), 而字典将对字段进行比较, 并逐个跟踪更改 (您的模板的更新次数就会减少)。

如果您需要查询存储，或存储许多相关的项目, 则最好使用本地集合 (请参阅[Collection Article](https://guide.meteor.com/collections.html#local-collections))。

### 访问存储

您应该像访问模板中的其他反应式数据一样访问数据存储, 这意味着集中访问存储, 非常类似于集中订阅和集中获取数据。
对于Blaze模板, 要么使用帮助类(helper), 或者在 回调函数 onCreated()的```this.autorun```中。

这样你就可以获得存储的整个反应式强大功能。

### 更新存储

如果由于用户操作而需要更新存储, 则可以从事件处理程序中更新存储, 就像调用方法一样。

如果您需要在更新中执行复杂的逻辑 (例如, 不只是调用. set () 等), 那么最好在存储中定义一个触发器(mutator)。由于存储是单例, 因此您可以直接将函数附加到对象:

``` 
WindowSize.simulateMobile = (device) => {
  if (device === 'iphone6s') {
    this.set({width: 750, height: 1334});
  }
}
```

##  高级发布

有时, 从发布函数返回查询的简单机制不能满足您的需要。这时, 您可以使用一些更强大的发布模式。

### 发布关系数据

在给定的页面上, 需要来自多个集合的相关数据是很常见的情况。例如, 在Todos应用程序中, 当我们呈现一个 todo 列表时, 我们需要列表本身, 以及属于该列表的todos集合。

要这样做的一种方法是从出版物函数中返回多个游标:

``` 
Meteor.publish('todos.inList', function(listId) {
  new SimpleSchema({
    listId: {type: String}
  }).validate({ listId });
  const list = Lists.findOne(listId);
  if (list && (!list.userId || list.userId === this.userId)) {
    return [
      Lists.find(listId),
      Todos.find({listId})
    ];
  } else {
    // The list doesn't exist, or the user isn't allowed to see it.
    // In either case, make it appear like there is no list.
    return this.ready();
  }
});
```
但是, 此示例将无法按预期工作。原因是反应式编程在服务器上的工作方式与在客户端上的不一样。在客户端, 一个反应式功能的任何变化, 都会引起整个功能的重新运行, 结果是相当直观的。

在服务器上, 反应式功能仅限于从发布函数返回的游标的行为。您会看到查询的数据的任何更改, 但**它们的查询永远不会更改**。

因此, 在上述情况下, 如果用户订阅了一个列表，后来此列表被其他用户改为私有数据, 尽管 "list.userId" 将更改，不再满足发布的条件, 
但发布的主体(body)却不会重新运行, 因此对Todos集合的查询 ({lisId}) 不会更改。因此, 第一个用户将继续看到他们不应该看到的项目。

但是, 我们可以编写对集合中的更改进行适当反应的发布。为此, 我们使用[reywood:publish-composite](https://atmospherejs.com/reywood/publish-composite)包。

此程序包的工作方式是首先在一个集合上建立游标, 然后在第二个集合中使用第一个游标的结果显式地(explicitly)设置第二级游标。
包使用后台的查询观察器来触发订阅更改, 并在源数据更改时重新运行查询。

``` 
Meteor.publishComposite('todos.inList', function(listId) {
  new SimpleSchema({
    listId: {type: String}
  }).validate({ listId });
  const userId = this.userId;
  return {
    find() {
      const query = {
        _id: listId,
        $or: [{userId: {$exists: false}}, {userId}]
      };
      // We only need the _id field in this query, since it's only
      // used to drive the child queries to get the todos
      const options = {
        fields: { _id: 1 }
      };
      return Lists.find(query, options);
    },
    children: [{
      find(list) {
        return Todos.find({ listId: list._id }, { fields: Todos.publicFields });
      }
    }]
  };
});
```

在这个例子中, 我们编写了一个复杂的查询, 以确保我们只找到一个我们被允许看到列表, 
然后, 一旦我们发现每个列表 (可以是一个或零个，取决于存取), 我们就为该列表发布托多斯列表。
如果列表没有匹配原始查询或者别的查询, 则发布复合体(Publish Composite)将停止并启动从属游标。

### 复杂授权

我们还可以使用 ```publish-composite``` 在发布中执行复杂的授权。例如, 假设我们有一个 ```Todos.admin.inList``` 发布, 
该发布默认对具有管理标志集的用户进行安全检查，却允许管理员绕过安全检查(security)。

我们可能想写：

``` 
Meteor.publish('Todos.admin.inList', function({ listId }) {
  new SimpleSchema({
    listId: {type: String}
  }).validate({ listId });
  const user = Meteor.users.findOne(this.userId);
  if (user && user.admin) {
    // We don't need to worry about the list.userId changing this time
    return [
      Lists.find(listId),
      Todos.find({listId})
    ];
  } else {
    return this.ready();
  }
});
```

但是, 由于上面讨论的相同的原因, 如果用户的```admin```状态更改, 发布却不会重新运行。如果这是很可能发生的事情, 并且需要作出反应式的改变, 
那么我们就需要使发布成为反应式的。我们可以通过同样的技术来做到这一点:

``` 
Meteor.publishComposite('Todos.admin.inList', function(listId) {
  new SimpleSchema({
    listId: {type: String}
  }).validate({ listId });
  const userId = this.userId;
  return {
    find() {
      return Meteor.users.find({_id: userId, admin: true}, {fields: {admin: 1}});
    },
    children: [{
      find(user) {
        // We don't need to worry about the list.userId changing this time
        return Lists.find(listId);
      }
    },
    {
      find(user) {
        return Todos.find({listId});
      }
    }]
  };
});
```
请注意, 我们显式地设置了 'Meteor.user' 查询字段, 即 "publish-composite" 将所有返回的游标发布到客户端, 并在游标更改时运行 "子计算"。

限制结果有双重目的: 它既防止敏感字段被泄露给客户端, 又限制计算到相关字段 (即管理字段)。

### 使用低级别API自定义发布

在我们所有的例子中 (除了使用```Meteor.publishComposite()```), 我们已经从我们的```Meteor.publish()```处理程序中返回了一个游标。
这样做可以确保Meteor在服务器和客户端之间保持同步的内容的工作。但是, 还有另一个 API 可以用于发布函数, 这更接近于底层分布式数据协议 (DDP) 的工作方式。

DDP 使用三个主要消息来传达发布数据的更改: 添加、更改和删除的消息。因此, 我们同样可以对发布进行同样的操作:

```
Meteor.publish('custom-publication', function() {
  // We can add documents one at a time
  this.added('collection-name', 'id', {field: 'values'});
  // We can call ready to indicate to the client that the initial document sent has been sent
  this.ready();
  // We may respond to some 3rd party event and want to send notifications
  Meteor.setTimeout(() => {
    // If we want to modify a document that we've already added
    this.changed('collection-name', 'id', {field: 'new-value'});
    // Or if we don't want the client to see it any more
    this.removed('collection-name', 'id');
  });
  // It's very important to clean up things in the subscription's onStop handler
  this.onStop(() => {
    // Perhaps kill the connection with the 3rd party server
  });
});
```

从客户的角度来看, 像这样发布的数据看起来没有什么不同--实际上, 由于 DDP 消息是相同的, 客户端无法知道差异。因此, 即使您正在连接和镜像一些深奥的数据源, 在客户端上, 它也会像任何其他的芒果集合一样出现。

需要注意的一点是, 如果您允许用户修改您以这种方式发布的 "pseudo-collection" 中的数据, 您将需要确保通过发布重新发布(re-publish)对它们的修改, 以获得乐观的用户体验。

### 订阅的生命周期

尽管您可以通过直观的理解在Meteor中使用发布和订阅, 但有时在订阅数据时准确了解引擎盖下发生的情况是很有用的。

假设您有以下形式的简单发布:

``` 
Meteor.publish('Posts.all', function() {
  return Posts.find({}, {limit: 10});
});
```

然后当一个客户调用```Meteor.subscribe('Posts.all')```时， Meteor里面发生以下事情:

1.  客户端通过 DDP 发送一个带有订阅名称的订阅消息。
2.  服务器通过运行发布处理函数来启动订阅。
3.  发布处理程序标识返回值为游标。这为发布游标提供了方便的模式。
4.  服务器在该游标上设置一个查询观察器, 如果此类观察器已经存在于服务器上 (对于任何用户), 就重用该观察者。
5.  观察器读取与游标匹配的当前文档集, 并将它们传递回订阅 (通过this.added()回调函数)。
6.  订阅将添加的文档传递给订阅客户端的连接mergebox, 这是已发布到此特定客户端的文档的服务器缓存。
每个文档都与客户端知道的文档的任何现有版本合并, 这样该添加了 (如果该文档是新的客户端) 或已更改 (如果已知但此订阅是添加或更改字段),
的文档就通过DDP消息发送给了订阅客户端。请注意, mergebox 在顶级字段进行操作, 因此, 如果两个订阅发布嵌套的字段 
(例如, sub1发布```doc.a.b=7```, sub2发布```doc.a.c=8```), 那么 "合并" 文档可能不像您所期望的那样 (在本例中，如果sub2在sub1之后发生,则结果为```doc.a={c:8}```)。
7.  该发布调用```.ready()``` 回调, 它向客户端发送 DDP 就绪消息。客户端上的订阅句柄标记为就绪。
8.  观察者观察查询。通常, 它使用 MongoDB 的 Oplog 来通知影响查询的更改。如果它看到了相关的变化, 比如有了新匹配文档，或匹配文档中字段发生更改, 
它就会调用订阅(```.added()，.changed()，.removed()```), 然后将更改发送到 mergebox, 再通过 DDP 向客户端传递。

此操作将一直运行, 直到客户端停止订阅, 触发以下行为:

1.  客户端发送取消订阅的DDP消息。
2.  服务器停止其内部订阅对象, 触发以下行为:
3.  任何由发布处理程序设置的回调```this.onStop()```将运行。在这种情况下, 它是一个单一的自动回调设置。当从处理程序返回游标时，它将停止查询观察器并在必要时对其进行清理。
4.  由该订阅跟踪的所有文档都将从mergebox中删除, 这意味着可能会也可能不会把它们也从客户端中删除。
5.  "无订阅"(nosub)消息将发送到客户端, 以指示订阅已停止。

##  使用REST API

在Meteor的DDP协议中, 发布和订阅是处理数据的主要方式, 但大量的数据源使用流行的 REST 协议做为它们的API。能够在两者之间进行转换是很有用的。

### 从发布的REST端点加载数据

作为使用低级API([low-level-api](https://guide.meteor.com/data-loading.html#custom-publication))的一个具体示例, 
请考虑您有一些第三方 REST 端点的情况, 它提供了一组对用户有价值的不断变化的数据。如何使这些数据可用呢？

一个选项是提供一个简单的到端点的代理方法, 轮询和处理不断变化的数据则是客户端的责任。因此, 处理数据的本地数据缓存、更改发生时更新UI等都是客户端的问题。
尽管可以这样做(例如, 您可以使用本地集合来存储轮询数据), 但更简单、更自然的方法是创建一个为客户端进行该查询的发布。

用于转变成轮询的REST端点的模式类似于:

``` 
const POLL_INTERVAL = 5000;
Meteor.publish('polled-publication', function() {
  const publishedKeys = {};
  const poll = () => {
    // Let's assume the data comes back as an array of JSON documents, with an _id field, for simplicity
    const data = HTTP.get(REST_URL, REST_OPTIONS);
    data.forEach((doc) => {
      if (publishedKeys[doc._id]) {
        this.changed(COLLECTION_NAME, doc._id, doc);
      } else {
        publishedKeys[doc._id] = true;
        this.added(COLLECTION_NAME, doc._id, doc);
      }
    });
  };
  poll();
  this.ready();
  const interval = Meteor.setInterval(poll, POLL_INTERVAL);
  this.onStop(() => {
    Meteor.clearInterval(interval);
  });
});
```

事情会变得更复杂。例如, 您可能需要处理被删除的文档, 或者共享多个用户之间的轮询工作 (在轮询数据不是私有的情况下), 
而不是对每个感兴趣的用户进行完全相同的轮询。

### 以REST端点的形式访问发布

如果要发布的数据有第三方使用, 通常是通过REST方式, 则会发生相反的情况。如果我们要发布的数据与我们通过发布提供的信息相同, 
那么我们可以使用 [simple:rest](https://atmospherejs.com/simple/rest)包来很容易地做到这一点。

在Todos示例应用程序中, 我们已经这样做了, 您现在可以通过 HTTP 访问我们的发布:

``` 
$ curl localhost:3000/publications/lists.public
{
  "Lists": [
    {
      "_id": "rBt5iZQnDpRxypu68",
      "name": "Meteor Principles",
      "incompleteCount": 7
    },
    {
      "_id": "Qzc2FjjcfzDy3GdsG",
      "name": "Languages",
      "incompleteCount": 9
    },
    {
      "_id": "TXfWkSkoMy6NByGNL",
      "name": "Favorite Scientists",
      "incompleteCount": 6
    }
  ]
}
```

您还可以访问需要认证的发布 (如```list.private```)。假设我们已通过web面界注册了用户```user@example.com```, 并使用密码```password```，
并创建了一个私有列表。于是, 我们可以这样访问它:

``` 
# First, we need to "login" on the commandline to get an access token
$ curl localhost:3000/users/login  -H "Content-Type: application/json" --data '{"email": "user@example.com", "password": "password"}'
{
  "id": "wq5oLMLi2KMHy5rR6",
  "token": "6PN4EIlwxuVua9PFoaImEP9qzysY64zM6AfpBJCE6bs",
  "tokenExpires": "2016-02-21T02:27:19.425Z"
}
# Then, we can make an authenticated API call
$ curl localhost:3000/publications/lists.private -H "Authorization: Bearer 6PN4EIlwxuVua9PFoaImEP9qzysY64zM6AfpBJCE6bs"
{
  "Lists": [
    {
      "_id": "92XAn3rWhjmPEga4P",
      "name": "My Private List",
      "incompleteCount": 5,
      "userId": "wq5oLMLi2KMHy5rR6"
    }
  ]
}
```

