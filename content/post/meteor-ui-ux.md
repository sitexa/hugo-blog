+++
title = "Meteor(9):用户界面(User Interfaces)"
description = "General tips for structuring your UI code, independent of your view rendering technology."
tags = [
    "JavaScript",
    "NodeJs",
    "Web application",
    "Mobile application",
    "Platform",
    "MongoDB",
]
date = "2017-11-18"
categories = [
    "Development",
    "nodejs",
]
thumbnail = "images/meteor-guide/meteor-ux.jpg"
+++

阅读本指南后, 您将知道:

1.  如何在用户界面框架中构建可重用的客户端组件。
2.  如何构建样式指南, 使您能够直观地测试此类可重用组件。
3.  在Meteor中构建前端组件的高性能模式。
4.  如何以可维护和可扩展的方式构建用户界面。
5.  如何构建能够处理各种不同数据源的组件。
6.  如何使用动画以便用户了解变化。

<!--more-->

##  视图层

Meteor正式支持三种用户界面 (UI) 渲染库, Blaze, React和Angular。Blaze是作为Meteor的一部分在2011年推出的, 
React是Facebook 在 2013年创建的, Angular是由谷歌在2010年创建的。所有三种渲染库都已成功地用于大型生产应用程序。
Blaze是最容易学习和全栈式的Meteor包, 但React和Angular发展更快, 有更大的开发社区。

### 语法

-   Blaze使用一种易于学习的[handlebar](http://handlebarsjs.com/)样式的模板语法, 在HTML文件中穿插形如{{#if}} 和 {{#each}}逻辑。模板函数和 CSS 选择器事件映射是在 JavaScript 文件中编写的。
-   反应使用[JSX](https://facebook.github.io/react/docs/jsx-in-depth.html)在HTML中编写JavaScript代码。虽然它没有像大多数库那样的逻辑视图分离, 但它也具有最大的灵活性。模板函数和事件处理程序在与组件的HTML部分相同的文件中定义, 这通常便于理解它们是如何绑定在一起的。
-   Angular使用带有[特殊属性语法](https://angular.io/docs/ts/latest/guide/cheatsheet.html)的 HTML 来表达逻辑和事件。模板助手是在随附的 JavaScript 文件中编写的, 事件是由HTML属性中的名称调用的。
-   Angular和React采用更好的组件结构, 这样更容易开发大型的应用程序。(尽管您可以通过[遵循约定](http://blazejs.org/guide/reusable-components.html)
或使用[Blaze组件](http://components.meteorapp.com/)或[ViewModel](https://viewmodel.org/)包将组件结构添加到Blaze中。)

### 社区

-   在Atmosphere中有许多全栈式Blaze软件包, 例如 [useraccounts:core](https://atmospherejs.com/useraccounts/core) 和 [aldeed:autoform](https://atmospherejs.com/aldeed/autoform)。
-   React在github上有42k星，有13k npm库。
-   Angular在github上有12k星，有4k npm库。

### 性能

-   根据情况的不同, 渲染性能会有很大差异。三个库都能非常快速地呈现简单的应用程序, 但在复杂的应用程序上可能要花费可观的时间。
-   Angular和React有更多的性能优化工作投入,一般情况下他们比Blaze表现更好。但是, 在某些情况下Blaze做得更好(例如, {{#each}}在更改的游标上有更好的表现)。
-   根据[One test](http://info.meteor.com/blog/comparing-performance-of-blaze-react-angular-meteor-and-angular-2-with-meteor)的测试，Angular 2 表现最好，其次是React，再次是Angular 1, 最后是Blaze。

### 移动端

-   Cordova
    -   所有三个库都能在Cordova web视图中正常工作, 您可以使用移动css库 (如 Ionic css) 和任何视图库。
    -   最先进的移动web框架是[Ionic 2](http://ionicframework.com/docs/v2/), 它使用Angular 2。
    -   Ionic 1 使用 Angular 1, 它也有了[Blaze](http://meteoric.github.io//) 版本和 [React](http://reactionic.github.io/)版本。
    -   另一个不错的选择是[Onsen UI](https://onsen.io/v2/), 其中包括一个[React](https://onsen.io/v2/docs/guide/react/)版本。
-   Native
    -   你可以用任何本地的 iOS 或 Android 应用程序通过 DDP 连接Meteor服务器。对于 ios, 使用 [meteor-ios](https://github.com/martijnwalraven/meteor-ios) 框架。
    -   您可以在 JavaScript 中使用本机 UI 元素编写应用程序。有关如何使用Meteor Native的最新信息, 请参阅[此参考](https://github.com/spencercarli/react-native-meteor-index)。
    
##  UI组件

无论您使用的是什么视图层, 都有一些模式可以帮助您构建用户界面 (UI), 从而使您的应用程序代码更易于理解、测试和维护。这些模式非常类似于模块化的一般模式, 它围绕着使 UI 元素的接口非常清晰, 并避免使用绕过这些已知接口的技术。

在本文中, 我们将把用户界面中的元素称为 "组件"。虽然在某些系统中, 您可以将它们称为 "模板", 但最好将它们看作是一个组件, 它具有一个 API 和内部逻辑, 而不是模板那样仅仅是一些HTML片段。

首先, 让我们考虑两类对用户有用的 UI 组件, 即 "可重用" 和 "智能":

### 可重用组件

"可重用" 组件是一个组件, 它不依赖于它所呈现的环境中的任何内容。它纯粹基于它的直接输入 (在Blaze中的模板参数, 或在React中的属性) 和内部状态。

特别是在Meteor中, 这意味着一个组件不能访问来自任何全局资源的数据--集合、存储、路由器、用户数据或类似内容。
例如, 在Todos示例应用程序中, ```Lists_show``` 模板采用的是要呈现的列表和该列表的todos集, 它从不直接在```Todos```或```Lists```集合(Collections)中查找数据。

可重用组件有许多优点:

1.  它们很容易推理-您不需要了解全局存储中的数据是如何变化的, 只要了解组件的参数如何变化。
2.  他们很容易测试-你不需要担心你渲染他们的环境, 你需要做的就是提供正确的参数。
3.  它们易于添加到组件样式指南——正如我们将在 "组件样式指南" 一节中看到的那样, 创建样式指南时, 干净的环境会使操作更容易。
4.  您确切地知道需要为他们在不同的环境中工作提供哪些依赖关系。

还有一个更严格的可重用组件类型, 一个 "纯" 组件, 它没有任何内部状态。例如, 在Todos应用程序中, Todos_item 模板仅根据其参数呈现内容。
纯组件比可重复使用的部件更容易推理和测试, 因此应尽可能首先采用。

### 全局数据存储

那么, 有哪些全局数据存储, 您应该避免在重用组件中使用呢？确有几个。Meteor是为了优化开发速度而建立的, 这意味着你可以在全局范围内访问很多东西。
虽然这在构建 "智能" 组件 (见下文) 时很方便, 但您需要在可重用组件中避免这些数据源:

-   你的集合, 以及```Meteor.users```集合
-   帐户信息, 如```Meteor.user()``` 和```Meteor.loggingIn()```
-   当前路由信息
-   任何其他客户端数据存储 (在数据加载文章中读取更多信息)

### 智能组件

虽然应用程序中的大多数组件都应该是可重用的, 但他们需要从某处获取它们的数据。这就是 "智能" 组件出现的原因。这些组件通常执行以下操作:

-   订阅数据
-   从这些订阅中获取数据
-   从存储区 (如路由器、帐户和您自己的存储区) 获取全局客户端状态

理想情况下, 一旦智能组件组装了一组数据, 它就会传递给一个可重用的组件的部件以进行渲染。智能组件通常不会将任何内容与一个或多个可重用的子项分开。这使得在测试中区分呈现和数据加载变得很容易。

智能组件的典型用例是当您访问 URL 时, 路由器指向您的 "页面" 组件。这样的组件通常需要执行上面的三件事, 然后将结果参数传递到子组件中。在Todos示例应用程序中, listShowPage 完全做到了这一点, 从而生成一个具有非常简单的 HTML 的模板:

``` 
<template name="Lists_show_page">
  {{#each listId in listIdArray}}
    {{> Lists_show (listArgs listId)}}
  {{else}}
    {{> App_notFound}}
  {{/each}}
</template>
```

此组件的 JavaScript 负责订阅和获取 Lists_show 模板所使用的数据:

``` 
Template.Lists_show_page.onCreated(function() {
  this.getListId = () => FlowRouter.getParam('_id');
  this.autorun(() => {
    this.subscribe('todos.inList', this.getListId());
  });
});
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

##  直观地测试可重用组件

可重用组件的一个有用特性是, 您可以在任何地方呈现它们, 因为它们不依赖于复杂的环境。用组件资源管理器对组件管理对组件进行管理非常方便, 调试应用程序允许您浏览、可视化和测试UI组件。

![](/images/meteor-guide/chromatic-how-it-works.png)

组件资源管理器有两种情况:

-   对您的应用程序组件进行索引, 以便它们易于查找
-   使用开发人员定义的状态和存根数据呈现组件

例如, 在 Galaxy 中, 我们使用名为 "Chromatic" 的组件资源管理器一次或一次地呈现每个组件的规格。

使用Chromatic可以快速开发复杂的组件。通常在大型应用程序中, 完全通过 "使用" 应用程序来实现某些组件的状态是相当困难的。
例如, 如果同一应用程序的两个部署同时发生, 则 Galaxy 中的组件可以进入复杂状态。使用Chromatic, 我们可以在组件级别定义此状态, 
并独立于应用程序逻辑对其进行测试。

你可以在你的Meteor React应用程序中使用```meteor add mdg:chromatic```添加 [Chromatic component explorer](https://github.com/meteor/chromatic)。
在React中构建类似项目是使用PhilCockfield 的 [UI Harness]() 和 ArunodaSusiripala的[React Storybook](https://github.com/kadirahq/react-storybook)。

##  用户界面模式

以下是在构建Meteor应用程序的用户界面时要牢记的一些模式。

### 国际化

国际化 (i18n) 是一种将应用程序的 UI 泛化的过程, 它可以很容易地以不同的语言呈现所有文本。Meteor的包装生态系统包括适合您的前端框架选择的国际化选项。

**需要翻译的地方**

在系统中考虑用户可读字符串存在的不同位置并确保正确使用 i18n 系统在每个案例中生成这些字符串是很有用的。我们将在下面关于 tap:i18n 和 universe:i18n 的章节中讨论每个案例的实现。

1.  **HTML模板和组件**。这是用户看到的UI组件内容。
2.  **客户端JavaScript消息**。在客户端上生成的警告或其他消息将显示给用户。
3.  **服务器JavaScript消息和电子邮件**。服务器生成的消息或错误通常可以是用户可见的。最明显的是电子邮件和服务器生成的消息, 如移动推送通知, 返回值和错误消息。错误应以通用形式发送, 并在客户端上进行翻译。
4.  **数据库中的数据**。您可能需要翻译的最后一个地方是数据库中的实际用户生成的数据。例如, 如果您正在运行 wiki, 您可能希望有一个将 wiki 页面转换为不同语言的机制。如何解决可能对每个应用程序都不同。

**在JavaScript中使用```tap:i18n```**

在Meteor, [the excellent tap:i18n package](https://atmospherejs.com/tap/i18n) 提供了一个 API, 用于构建翻译, 并在组件和前端代码中使用它们。

要使用 ```tap:i18n```, 首先使用```meteor add tap:i18n```将其添加到您的应用程序中。然后, 我们需要给默认语言(英语-en)添加一个翻译JSON文件——我们可以把它在```i18n/en.i18n.JSON```文件夹中。
一旦完成, 我们可以导入和使用```TAPi18n.__()```函数来获取JavaScript代码中的字符串或键的翻译。

例如, 对于Todos示例应用程序中的错误, 我们创建了一个```error```模块, 它使我们能够轻松地为我们从方法中抛出所有错误的翻译错误:

``` 
import { TAPi18n } from 'meteor/tap:i18n';
export const displayError = (error) =>  {
  if (error) {
    // It would be better to not alert the error here but inform the user in some
    // more subtle way
    alert(TAPi18n.__(error.error));
  }
};
```
```error.error``` 字段是 ```Meteor.error``` 构造函数的第一个参数, 我们使用它来唯一地命名和定义我们在应用程序中使用的所有错误的名称空间。然后, 我们在 ```i18n/en.i18n.json``` 中定义这些错误的英文文本:

``` 
{
    "lists": {
    "makePrivate": {
      "notLoggedIn": "Must be logged in to make private lists.",
      "lastPublicList": "Cannot make the last public list private."
    },
    "makePublic": {
    ...
    }
    ...
}
```

**在Blaze中使用tap:i18n**

我们也可以很容易地在Blaze模板中使用翻译。为此, 我们可以使用 ```{{_}}```帮助器。在Todos应用程序中, 我们使用实际字符串(输出的英语)作为i18n键,
这意味着我们不需要提供一个英文翻译, 虽然也许在一个真正的应用程序, 你可能想从一开始提供关键字。

例如在```app-not-found.html```中:

``` 
<template name="App_notFound">
  <div class="page not-found">
    <nav>
      <div class="nav-group">
        <a href="#" class="js-menu nav-item"><span class="icon-list-unordered"></span></a>
      </div>
    </nav>
    <div class="content-scrollable">
      <div class="wrapper-message">
        <div class="title-message">{{_ 'Page not found'}}</div>
      </div>
    </div>
  </div>
</template>
```

**改变语言**

若要设置和更改用户所看到的语言, 应调用```TAPi18n setLanguage (fn)```, 其中 fn 是返回当前语言的 (可能是响应式的) 函数。例如, 你可以写

``` 
// A store to use for the current language
export const CurrentLanguage = new ReactiveVar('en');
import CurrentLanguage from '../stores/current-language.js';
TAPi18n.setLanguage(() => {
  CurrentLanguage.get();
});
```

然后在您的 UI 中的某个位置, 当用户选择一种新语言时, 您可以使用```CurrentLanguage.set ('es')```进行设置。

**在React中使用```universe:i18n```**

对于React-based应用程序, ```universe:18n``` 包为 ```tap:i18n``` 提供了另一种解决方案。```universe:i18n```
给```tap:i18n```提供类似的方便, 但也包括一个方便的drop-in响应式组件, 并省略```tap: i18n```依赖于Meteor的模块和jquery包。
```universe:i18n``` 的目的是为 "Meteor React" 应用程序使用ES2015 模块, 但它在没有React或模块时也可以使用。

**在JS中使用```universe:i18n```**

要开始, 运行```meteor add  universe:i18n```将其添加到您的应用程序中. 向您的应用程序中添加一个json格式的英语(en-us)翻译文件, 
名称为```en-us.i18n.JSON```。翻译文件可以通过文件名或 ```{"_locale": "en-us"}``` JSON属性来标识。还支持 YAML 文件格式。

如果你的应用程序从```client/main.js```和```server/main.js```入口点使用ES2015模块, 在其中导入JSON文件。```i18n.__()```函数现在将找到您传入的键。

借用上面的 ```tap:i18n``` 例子, 在 ```universe:i18n``` 中我们的 ```displayError``` 函数现在看起来像这样:

``` 
import i18n from 'meteor/universe:i18n';
export const displayError = (error) =>  {
  if (error) {
    alert(i18n.__(error.error));
  }
};
```

要更改用户的语言, 请使用 ```i18n setLocale ("en-us")```。```universe:i18n``` 允许通过方法检索其他翻译, 以及将 JSON 文件包含在客户端包中。

**在React组件中使用```universe:i18n```**

要在您的React组件中添加 reactive i18n, 只需使用 ```i18n.createComponent()```函数并将其密钥从您的翻译文件中传递进去。
下面是一个简单组件包装的 ```i18n's``` 翻译组件的示例:

``` 
import React from 'react';
import i18n from 'meteor/universe:i18n';
const T = i18n.createComponent();
// displays 'hello world!' where our en-us.i18n.json
// has the key/value { "hello": "hello world!" }
const Welcome = (props) => <div>
  <T>hello</T>
</div>;
export default Welcome;
```

有关其他选项和配置, 请参见 [universe:i18n](https://atmospherejs.com/universe/i18n) 的文档。

### 事件处理

UI 的很大一部分涉及响应用户的初始事件, 并且您应该采取一些步骤来确保应用程序在快速输入时能够很好地执行。应用程序滞后于响应用户操作是最引人注目的性能问题之一。

**对用户操作的节流方法调用**

当用户执行某项操作时, 通常会对数据库进行某种更改。然而, 重要的是要确保你不做得太快。例如, 如果希望用户在文本框中键入时保存用户的文本,
则应采取一些步骤以确保不要每隔几百毫秒就向服务器发送方法调用。

如果不这样做, 您将看到整个主板的性能问题: 您将通过大量的小改动来淹没用户的网络连接, UI将在每次按键时更新, 可能导致性能不佳,
并且您的数据库将遭受很多写操作.

为了限制写入, 典型的方法是使用 underscore的  [.throttle()](http://underscorejs.org/#throttle) 或 [.debounce()](http://underscorejs.org/#debounce) 函数。
例如, 在Todos示例应用程序中, 我们将用户输入的写操作限制为300ms:

``` 
import {
  updateText,
} from '../../api/todos/methods.js';
Template.Todos_item.events({
  // update the text of the item on keypress but throttle the event to ensure
  // we don't flood the server with updates (handles the event at most once
  // every 300ms)
  'keyup input[type=text]': _.throttle(function(event) {
    updateText.call({
      todoId: this.todo._id,
      newText: event.target.value
    }, (err) => {
      err && alert(err.error);
    });
  }, 300)
});
```

通常,  如果您对在用户系列操作期间发生的事件很确定(即您不介意随着时间的推移而发生的多个节流事件, 如本例所示)，您使用 ```.throttle()```； 
而如果您希望用户停止键入300ms或更长时间时，再进行该操作(在此示例中) ，您应使用```.debounce()```。

![](/images/meteor-guide/throttle-vs-debounce.png)

**限制渲染**

即使您不将每个用户输入的数据通过线上的保存到数据库, 有时您仍可能希望在每个用户更改时更新内存中的数据存储区。如果更新该数据存储区会触发大量的 UI 更改,
则当您经常更新它时, 您可以看到性能不佳和错过按键。在这种情况下, 您可以用类似的方式限制渲染。您还可以仅在用户停止键入后使用```.debounce()``` 来确保更改发生。

##  用户体验模式

用户体验 (或 UX) 描述用户与应用程序交互时的体验。有几个典型的Meteor应用的UX模式值得探索。这些模式中的许多都与用户与应用程序交互时加载数据的方式相关, 
因此在数据加载文章中也有类似的部分, 它们讨论如何使用Meteor的发布和订阅来实现这些模式。

### 订阅就绪

当您订阅Meteor中的数据时, 它不会立即在客户端上可用。通常情况下, 用户需要等待几百毫秒, 或者几秒钟 (取决于连接速度), 数据才能到达。
当应用程序首次启动或在显示全新数据的屏幕之间切换时, 这一点尤为明显。

有几个 UX 技术来处理这个等待期。最简单的方法就是在您等待所有数据 (通常是一个页面可能打开多个订阅) 时, 用一个通用的 "加载" 页面切换出要加载的页面。
例如, 在Todos示例应用程序中, 我们一直等到所有的公共列表和用户的私有列表加载完毕后, 才尝试呈现实际的页面:

``` 
{{#if Template.subscriptionsReady}}
  {{> Template.dynamic template=main}}
{{else}}
  {{> App_loading}}
{{/if}}
```

我们是通过Blaze的```Template.subscriptionsReady```实现的, 这是一个完美的方法, 因为它等待所有的订阅数据全部就绪后才显示页面。

**按组件加载状态**

通常, 尽可能快地显示尽可能多的屏幕, 并且只显示仍在等待数据的屏幕部分的加载状态，这样做会使用户体验更好。因此, 一个很好的可遵循的模式是 "每组件加载"。
我们在Todos应用程序中是这样做的，当访问列表页时, 立即呈现列表元数据, 如它的标题和隐私设置, 并在等待数据出现时显示Todos列表的加载状态。

![](/images/meteor-guide/todos-loading.png)

我们通过将Todos列表的就绪状态从订阅 (listShowPage) 的智能组件中传递到可重用的渲染组件中来实现这个目的:

``` 
{{> Lists_show todosReady=Template.subscriptionsReady list=list}}
```

然后我们使用该状态来确定可重用组件 (```listShow```) 中呈现的内容:

``` 
{{#if todosReady}}
  {{#with list._id}}
    {{#each todo in (todos this)}}
      {{> Todos_item (todoArgs todo)}}
    {{else}}
      <div class="wrapper-message">
        <div class="title-message">No tasks here</div>
        <div class="subtitle-message">Add new tasks using the field above</div>
      </div>
    {{/each}}
  {{/with}}
{{else}}
    <div class="wrapper-message">
      <div class="title-message">Loading tasks...</div>
    </div>
{{/if}}
```

**显示占位符**

您可以通过在等待数据加载时在上述UI中显示占位符来进一步改进用户体验。这是一个由 Facebook 首创的 UX 模式, 它给用户提供了一个更强烈的印象, 即数据从正从线上传来。
如果可以使占位符具有与最终元素相同的维度, 则它还会防止在数据加载时移动 UI 的某些部分。

例如, 在 Galaxy 中, 当您等待应用程序的日志加载时, 您会看到一个加载状态, 指示您可能看到的内容:

![](/images/meteor-guide/galaxy-placeholders.png)

**使用样式指南构造加载状态原型**

在开发环境中，由于订阅数据几乎可以立即加载，因此加载状态几乎不可察觉。因为在定义上加载状态是个暂态。

这就是[component style guide](https://guide.meteor.com/ui-ux.html#styleguides)如此有用的一个原因，因为它能够实现任何状态。
由于可重用组件```Lists_show```只是根据它的```todosReady```参数来选择渲染的内容，它并不关心订阅的数据，所以在样式指南中渲染它的状态是没有意义的。

### 分页

在“数据加载”文章中，我们讨论了通过“无限滚动”类型的订阅的分页模式，当用户向下滚动页面时，一次加载一页数据。使用这种UX模式消费数据并指出用户正在发生的事情是很有趣的。

**列表组件**

让我们考虑通用的项目列表组件。要专注于一个具体的例子，我们可以考虑Todos示例应用程序中的待办事项列表。 
虽然它不在我们目前的示例应用程序中，但是在将来的版本中，它可以通过为给定列表的待办事项分页。

这样的列表可以有多种状态：

1.   最初加载，还没有可用的数据。
2.   显示具有更多可用项目的子集。
3.   用更多的加载显示项目的子集。
4.   显示所有的项目 - 没有更多的可用项目。
5.   显示没有项目，因为不存在。

考虑需要什么参数来区别这个组件的五个状态是有益的。让我们考虑一个通用模式, 它在我们提供以下信息的所有情况下都可以使用:

-   项目总数的计数。
-   一个 countReady 布尔值, 它指示我们是否知道该计数 (记住, 我们甚至需要加载这些信息)。
-   我们求求的项目数。
-   我们当前知道的项目列表。

我们现在可以根据这些条件区分上述5种状态:

1.  ```countReady === false```, 或 ```count > 0 && items === empty``` (这些实际上是两种不同的状态, 但在视觉上将它们分开似乎并不重要).
2.  ```items.length === requested && requested < count```
3.  ```0 < items.length < requested```
4.  ```items.length === requested && requested === count && count > 0```
5.  ```count === 0```

您可以看到, 虽然情况有点复杂, 但它也完全由参数决定, 因此非常容易测试。组件样式指南可以帮助您轻松地看到所有这些状态! 
在 Galaxy 应用的风格指南中, 我们对应用程序的每个列表都有各自的状态, 可以确保所有的工作都按预期的方式显示出来, 并且正确无误:

![](/images/meteor-guide/galaxy-styleguide-list.png)

**分页“控制器”模式**

列表是了解拆分智能组件与可重用组件的好处的好机会。上面我们已经看到，正确地呈现和可视化展示列表的所有可能状态并不重要，
重要的是通过带参数的获取所需信息的可重用列表组件来实现却容易得多。

但是，我们仍然需要订阅项目和计数列表，并在某处收集数据。 要做到这一点，应当使用智能包装器组件（类似于MVC“控制器”），它的工作是订阅和获取相关数据。

在Todos示例应用程序中，我们已经有一个与路由器通话并设置订阅的列表的包装组件。 这个组件可以很容易地扩展来理解分页：

``` 
const PAGE_SIZE = 10;
Template.Lists_show_page.onCreated(function() {
  // We use internal state to store the number of items we've requested
  this.state = new ReactiveDict();
  this.getListId = () => FlowRouter.getParam('_id');
  this.autorun(() => {
    // As the `requested` state increases, we re-subscribe to a greater number of todos
    this.subscribe('List.todos', this.getListId(), this.state.get('requested'));
    this.countSub = this.subscribe('Lists.todoCount', this.getListId());
  });
  // The `onNextPage` function is used to increment the `requested` state variable. It's passed
  // into the listShow subcomponent to be triggered when the user reaches the end of the visible todos
  this.onNextPage = () => {
    this.state.set('requested', this.state.get('requested') + PAGE_SIZE);
  };
});
Template.Lists_show_page.helpers({
  listArgs(listId) {
    const instance = Template.instance();
    const list = Lists.findOne(listId);
    const requested = instance.state.get('requested');
    return {
      list,
      todos: list.todos({}, {limit: requested}),
      requested,
      countReady: instance.countSub.ready(),
      count: Counts.get(`list/todoCount${listId}`),
      onNextPage: instance.onNextPage
    };
  }
});
```

**用于显示新数据的UX模式**

在实时系统（如Meteor）中，一个有趣的UX挑战涉及如何将新的信息（如将列表中的数据更改）提示用户以引起用户的注意。
正如Dominic指出的那样，尽快更新列表的内容并不总是一个好主意，因为很容易错过更改或者对发生的事情感到困惑。

解决这个问题的一个办法是动画列表的变化（我们将在动画部分看看），但这并不总是最好的办法。例如，如果用户正在阅读评论列表，
他们可能不希望看到任何更改，直到他们完成当前的评论线程。

在这种情况下，一个选项是调用用户正在查看的数据的更改，而不实际进行UI更新。在默认情况下，如Meteor这样的系统，阻止这种变化不一定很容易！

但是，由于我们在智能和可重用组件之间的分离，可以做到这一点。可重用组件简单地呈现给它的内容，所以我们使用智能组件来控制这些信息。
我们可以使用本地集合来存储呈现的数据，然后在用户请求更新时将数据推入：

``` 
Template.Lists_show_page.onCreated(function() {
  // ...
  // The visible todos are the todos that the user can
  // actually see on the screen (whereas Todos are the
  // todos that actually exist)
  this.visibleTodos = new Mongo.Collection(null);
  this.getTodos = () => {
    const list = Lists.findOne(this.getListId());
    return list.todos({}, {limit: instance.state.get('requested')});
  };
  // When the user requests it, we should sync the visible todos to
  // reflect the true state of the world
  this.syncTodos = (todos) => {
    todos.forEach(todo => this.visibleTodos.insert(todo));
    this.state.set('hasChanges', false);
  };
  this.onShowChanges = () => {
    this.syncTodos(this.getTodos());
  };
  this.autorun((computation) => {
    const todos = this.getTodos();
    // If this autorun re-runs, the list id or set of todos must have
    // changed, so we should flag it to the user so they know there
    // are changes to be seen.
    if (!computation.firstRun) {
      this.state.set('hasChanges', true);
    } else {
      this.syncTodos(todos);
    }
  });
});
Template.Lists_show_page.helpers({
  listArgs(listId) {
    const instance = Template.instance();
    const list = Lists.findOne(listId);
    const requested = instance.state.get('requested');
    return {
      list,
      // we pass the *visible* todos through here
      todos: instance.visibleTodos.find({}, {limit: requested}),
      requested,
      countReady: instance.countSub.ready(),
      count: Counts.get(`list/todoCount${listId}`),
      onNextPage: instance.onNextPage,
      // These two properties allow the user to know that there
      // are changes to be viewed and allow them to view them
      hasChanges: instance.state.get('hasChanges'),
      onShowChanges:instance.onShowChanges
    };
  }
});
```

然后，可重用子组件可以使用hasChanges参数来确定是否应该向用户显示某种标注以指示有可用更改，然后使用onShowChanges回调触发它们显示。

### 乐观用户界面

一个比其他框架更容易的很好的UX模式就是Meteor的乐观界面(Optimistic UI)。乐观ui是在ui中显示用户生成的更改的过程, 它无需等待服务器确认更改是否成功, 
这会使得用户体验比实际可能更快, 因为您不需要等待任何服务器往返。因为在一个设计良好的应用程序中, 大多数用户操作都是成功的, 所以对于应用程序的所有部分来说, 这种方式都是乐观的。

然而, 乐观并不总是一个好主意。有时, 我们可能实际上想等待服务器的响应。例如, 当用户登录时, 您必须等待服务器检查密码是否正确, 然后才能开始允许他们进入站点。

那么, 你应该在什么时候等待服务器？它基本上归结为你有多乐观;有可能会出问题。在验证密码的情况下, 就不能保持乐观, 你需要保守。在其他情况下,
您可以相当确信方法调用将成功, 因此您可以继续。

例如, 在Todos示例应用程序中, 创建新列表时, 列表创建将基本上总是成功的, 因此我们编写:

``` 
import { insert } from '../../api/lists/methods.js';
Template.App_body.events({
  'click .js-new-list'() {
    const listId = insert.call((err) => {
      if (err) {
        // At this point, we have already redirected to the new list page, but
        // for some reason the list didn't get created. This should almost never
        // happen, but it's good to handle it anyway.
        FlowRouter.go('App.home');
        alert('Could not create list.');
      }
    });
    FlowRouter.go('Lists.show', { _id: listId });
  }
});
```

我们将```FlowRouter.go（'Lists.show'）```放在方法调用的回调之外，以便马上运行。 首先我们模拟该方法（在Minimongo本地创建一个列表），然后路由到它。 
最终当服务器返回时，通常创建完全相同的列表（用户甚至不会注意到）。 万一服务器调用失败，我们会显示一个错误并重定向回主页。

请注意，由list方法返回的listId（这是由客户端存根生成的）是保证与服务器上生成的相同的，这是由于Meteor生成ID能确保它们在客户端和服务器之间相同。

### 提示写入正在进行

有时用户可能有兴趣知道更新何时已经到达服务器。 例如，在聊天应用程序中，乐观地在聊天记录中显示消息是典型的模式，但是在服务器确认写入之前指示它是“挂起的”。 
我们可以通过简单地修改方法使其在客户机上采取不同的行为，这可以在Meteor中轻松实现：

``` 
Messages.methods.insert = new ValidatedMethod({
  name: 'Messages.methods.insert',
  validate: new SimpleSchema({
    text: {type: String}
  }).validator(),
  run(message) {
    // In the simulation (on the client), we add an extra pending field.
    // It will be removed when the server comes back with the "true" data.
    if (this.isSimulation) {
      message.pending = true;
    }
    Messages.insert(message);
  }
})
```

当然，在这种情况下，您还需要为服务器做好失败的准备，并再次以某种方式将其指示给用户。

### 意外的失败

我们已经看到了上面的失败的例子，你不会预料到会发生这种情况。防止每一个可能的错误是困难和低效的，也不太可能。但是，有一些全面的模式，您可以用于处理意外的失败。

由于Meteor自动处理乐观的用户界面，如果一个方法意外失败，乐观的变化将回滚，并且Minimongo数据库将以一致的状态结束。如果您直接从Minimongo渲染，则用户将看到一致的内容，
即使这不是他们预期的。在某些情况下，如果您有保持Minimongo以外的状态，则可能需要手动进行更改以反映此情况。您可以在上面的示例中看到，在操作失败后我们必须手动更新路由。

然而，简单地将用户跳到一个意想不到的状态，而不解释发生了什么，这是很糟糕的用户体验。我们使用了上面的```alert()```，这是一个非常糟糕的选择，但是完成了工作。
一个更好的方法是通过“flash通知”来指示更改，这是一个UI元素，通常在屏幕的右上方显示“out-of-band”，给用户一些指示。下面是Galaxy页面右上角的Flash通知示例：

![](/images/meteor-guide/galaxy-flash-notification.png)

##  动画

动画是随着时间而平滑地表示UI中的变化的过程，它不是即时的。 虽然动画经常被看作是“窗口装饰”或纯粹的审美，但实际上它是一个非常重要的工作，上面的变化列表说明了这个总是。 
在连接客户端的世界里，UI的变化并不总是由用户操作发起的（也就是说，有时候是因为服务器推送其他用户所做的更改而发生的），即时更改会导致用户很难理解发生了什么事情的用户体验。

### 动画可视化地表达改变

可能需要动画的最基本的UI变化类型是当项目出现或消失时。 在Blaze中，我们可以使用[percolate:momentum](https://atmospherejs.com/percolate/momentum)包
来将[velocity animation library](http://julian.com/research/velocity/)中的一组标准动画插入到状态变化中。

一个很好的例子就是Todos示例应用程序的列表编辑状态：

``` 
{{#momentum plugin="fade"}}
  {{#if instance.state.get 'editing'}}
    <form class="js-edit-form list-edit-form">...</form>
  {{else}}
    <div class="nav-group">...</div>
  {{/if}}
{{/momentum}}
```

Momentum通过覆盖子元素HTML元素出现和消失的方式工作。 在这种情况下，当列表组件进入```edit```状态时，```.nav-group```消失，```form```出现。 
Momentum负责确保两个项目都消褪，使变化更清晰。

### 动画表达属性的改变

另一种常见的动画类型是元素的属性发生变化。 例如，一个按钮可能会改变你点击它的颜色。 这些类型的动画最容易通过CSS转换([CSS transition](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Transitions/Using_CSS_transitions))来实现。 
例如，我们在Todos示例应用程序中使用链接悬停状态的CSS转换：

``` 
a {
  transition: all 200ms ease-in;
  color: @color-secondary;
  cursor: pointer;
  text-decoration: none;
  &:hover { color: darken(@color-primary, 10%); }
  &:active { color: @color-well; }
  &:focus { outline:none; } //removes FF dotted outline
}
```

### 动画表达页面的改变

最后，当用户在应用程序的路线之间切换时，动画是很常见的。 特别是在移动设备上，通过定位相对于彼此的页面，增加了对应用程序的导航感。 
这可以通过类似的方式来完成，使动画的东西出现和消失（一个页面出现，另一个页面消失），但也有一些值得注意的技巧。

我们来看看Todos示例应用程序的情况。 在这里，我们通过在主布局模板中使用Momentum来做类似的事情来实现页面之间的动画：

``` 
{{#momentum plugin="fade"}}
  {{#if Template.subscriptionsReady}}
    {{> Template.dynamic template=main}}
  {{else}}
    {{> App_loading}}
  {{/if}}
{{/momentum}}
```

这样做看起来应该是可行的，但有一个问题：有时渲染系统会喜欢简单地改变现有的组件，而不是将其切换出来并触发动画系统。 
例如，在Todos示例应用程序中，当您在列表之间导航时，默认情况下，Blaze将尝试简单地使用新的listId（更改后的参数）重新呈现Lists_show组件，
而不是将旧列表拉出并放入新列表。 原则上这是一个很好的优化，但是为了实现动画的目的，我们希望在这里避免这样做。 
更具体地说，我们要确保动画仅在listId发生变化时发生，而不发生在其他响应式变化上。

为了在这种情况下这样做，我们可以使用一个小技巧（这是特定于Blaze，尽管类似的技术适用于其他视图层）使用{{#each}}助手比较字符串数组，当他它改变的时候，重新渲染内容。

``` 
<template name="Lists_show_page">
  {{#each listId in listIdArray}}
    {{> Lists_show (listArgs listId)}}
  {{else}}
    {{> App_notFound}}
  {{/each}}
</template>
```

``` 
Template.Lists_show_page.helpers({
  // We use #each on an array of one item so that the "list" template is
  // removed and a new copy is added when changing lists, which is
  // important for animation purposes.
  listIdArray() {
    const instance = Template.instance();
    const listId = instance.getListId();
    return Lists.findOne(listId) ? [listId] : [];
  }
});
```

