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




