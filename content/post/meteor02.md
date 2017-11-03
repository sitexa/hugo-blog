+++
title = "Meteor:开发Web应用和移动应用的利器（2）"
description = "title:应用程序组织结构"
tags = [
    "JavaScript",
    "NodeJs",
    "Web application",
    "Mobile application",
    "Platform",
]
date = "2017-11-02"
categories = [
    "Development",
    "nodejs",
]
thumbnail = "images/meteor-logo.jpg"
+++

本章讨论如何使用 ES2015 模块构建您的Meteor应用程序, 将代码发送到客户端和服务器, 并将代码拆分为多个应用程序。

<!--more-->

读完这篇文章后, 您将知道:

1.  Meteor应用程序在文件结构方面与其他类型的应用程序的比较。
2.  如何为小型应用和大型应用组织程序结构。
3.  如何设置代码的格式并以一致和可维护的方式命名应用程序的各个部分。


##  通用 JavaScript

Meteor是构建 JavaScript 应用程序的全栈框架。这意味着, Meteor应用程序不同于大多数应用程序, 
因为它们包括了在客户端上运行的代码（在web浏览器或Cordova移动应用程序内运行）， 在服务器上运行的代码（
在Node.js容器中运行）, 以及在两种环境中运行的通用代码。使用 "Meteor构建工具", 通过ES2015 import 和 export 
以及Meteor构建系统的默认文件加载规则，您可以轻松地指定哪些JavaScript代码 (包括任何支持的 UI 模板、CSS 规则和静态资源)
在每个环境中如何运行。 

### ES2015模块

从1.3 版起, Meteor完全支持 ES2015 模块。[ES2015模块标准](https://developer.mozilla.org/en/docs/web/javascript/reference/statements/import)是
[CommonJS](http://requirejs.org/docs/commonjs.html) 和 [AMD](https://github.com/amdjs/amdjs-api) 的替代品, 
它是常用的 JavaScript 模块格式和加载系统。

在 ES2015 中, 可以使用导出(export)关键字使变量在文件外部可用。若要在其他位置使用变量, 必须使用源路径导入(import)它们。
导出某些变量的文件称为 "模块", 因为它们表示可重用代码的单元。显式导入所使用的模块和包可帮助您以模块化方式编写代码,
避免引入全局符号和 "远距离操作"(action at a distance)。

因为这是在Meteor 1.3 中引入的一个新特性, 你会发现很多的在线代码使用的是更旧的、更集中的习惯 围绕包和应用程序来声明全局符号。
这些旧系统仍然有效, 因此, 要选择新的模块系统, 代码必须放在应用程序中的```import/```目录中。我们期望, 
未来的Meteor发布版将在默认情况下为所有代码开启模块功能, 因为这使更为广泛的JavaScript 社区编写代码保持一致。

您可以在[模块包自述文件](https://docs.meteor.com/#/full/modules)中详细阅读模块系统。
这个软件包作为 ecmascript [软件包](https://docs.meteor.com/#/full/ecmascript)的一部分自动包括在每一个新的流星应用中,
所以大多数应用程序不需要做任何事情就能马上开始使用模块。

### 使用导入和导出(import/export)简介

"Meteor" 不仅允许您导入应用程序中的 JavaScript, 而且还可以使用 CSS 和 HTML 来控制加载顺序:

``` 
import '../../api/lists/methods.js';  // import from relative path
import '/imports/startup/client';     // import module with index.js from absolute path
import './loading.html';              // import Blaze compiled HTML from relative path
import '/imports/ui/style.css';       // import CSS from absolute path
```

> 有关导入样式的更多方法, 请参见[构建系统](https://guide.meteor.com/build-tool.html#css-importing)文章。

Meteor还支持标准的 ES2015 模块导出(export)语法:

``` 
export const listRenderHold = LaunchScreen.hold();  // named export
export { Todos };                                   // named export
export default Lists;                               // default export
export default new Collection('lists');             // default export
```

### 从包中导入

在Meteor中, 使用导入(import)语法在客户端或服务器上加载 npm 包并访问包的导出符号与任何其他模块一样, 简单明了。
您也可以从Meteor Atmosphere 包导入, 但导入路径必须以 ```meteor/``` 为前缀, 以避免与 npm 包命名空间发生冲突。
例如, 从 npm 和 HTTP 从Atmosphere导入时:

``` 
import moment from 'moment';          // default import from npm
import { HTTP } from 'meteor/http';   // named import from Atmosphere
```

有关使用包导入的详细信息, 请参阅"Meteor指南"中的 ["使用包"](https://guide.meteor.com/using-packages.html)。

### 使用require

在Meteor中, 导入(import)语句编译生成 CommonJS ```require```语法。但是, 作为一个共识, 我们鼓励您使用导入(import)。

在某些情况下, 您可能需要使用require对外发出直接请求。一个显然的例子是, 当需要一个公共文件中的客户机或服务器代码时，就会发
生这种调用。由于import必须在 top-level 范围内, 因此您可能不会将它们放在 if 语句中, 因此需要编写如下代码:

``` 
if (Meteor.isClient) {
  require('./client-only-file.js');
}
```

请注意, ```require()``` 的动态调用 (名称可能在运行时更改) 无法正确分析, 有可能导致客户端捆绑中断。

如果需要从具有 ```default export```的 ES2015 模块中```require```, 可以使用 ```require("package").default``` 来获取导出。

### 使用CoffeeScript

参考文档：[ Modules » Syntax » CoffeeScript](https://docs.meteor.com/packages/modules.html#CoffeeScript)

``` 
// lists.coffee
export Lists = new Collection 'lists'
```
``` 
import { Lists } from './lists.coffee'
```

##  文件结构

要完全使用模块系统并确保我们的代码仅在要求时运行, 我们建议您将所有应用程序代码放在```import/```目录中。
这意味着, 如果使用import (也称为 "懒加载"(lazy loading)) 从另一个文件引用该文件, 则Meteor构建系统将只捆绑并包含该文档。

Meteor将使用默认的文件加载规则 (也称为 "急加载"), 加载任何名为 imports 的目录之外所有文件。
建议您精确地创建两个急加载的文件, 即 ```client/main.js``` 和 ```server/main.js```, 
以便为客户端和服务器定义明确的入口点。Meteor确保任何名为 ```server/``` 的目录中的任何文件都只在服务器上可用, 
对于任何名为 ```client/``` 的目录中的文件也是如此。这也排除了试图从名为 ```client/``` 的任何目录中导入要在服务器上使用的文件, 
即使它嵌套在 ```import/``` 目录中, 反之亦然。

这些 ```main.js``` 文件本身不会做任何事情, 但是它们应该导入一些启动模块, 在应用程序加载时它们将分别在客户端和服务器上立即运行。
这些模块应该对您在应用程序中使用的包进行任何必要的配置, 并导入应用程序代码的其余部分。

### 目录布局示例

首先, 让我们看看我们的[Todos示例应用程序](), 它是构建应用程序时要遵循的一个很好的示例. 下面是它的目录结构概述。
你可以用这个结构生成一个新的应用程序, 使用命令创建 ```meteor create appName --full```。

``` 
imports/
  startup/
    client/
      index.js                 # import client startup through a single index entry point
      routes.js                # set up all routes in the app
      useraccounts-configuration.js # configure login templates
    server/
      fixtures.js              # fill the DB with example data on startup
      index.js                 # import server startup through a single index entry point
  api/
    lists/                     # a unit of domain logic
      server/
        publications.js        # all list-related publications
        publications.tests.js  # tests for the list publications
      lists.js                 # definition of the Lists collection
      lists.tests.js           # tests for the behavior of that collection
      methods.js               # methods related to lists
      methods.tests.js         # tests for those methods
  ui/
    components/                # all reusable components in the application
                               # can be split by domain if there are many
    layouts/                   # wrapper components for behaviour and visuals
    pages/                     # entry points for rendering used by the router
client/
  main.js                      # client entry point, imports all client code
server/
  main.js                      # server entry point, imports all server code
```

### 结构化导入

现在, 我们已将所有文件放在 ```imports/``` 目录中, 让我们来考虑如何最好地使用模块来组织代码。
当应用程序在 ```imports/startup``` 目录中启动时, 将所有运行的代码放在一起是有意义的。
另一个好主意是从 UI 呈现代码中分离数据和业务逻辑。我们建议使用名为 "imports/api" 和 "imports/ui" 的目录进行逻辑拆分。

在 ```imports/api``` 目录中, 将代码拆分为基于代码提供 api 的域(domain)的目录是明智的, 
这通常对应于您在应用程序中定义的数据集合(collections)。例如, 在Todos示例应用程序中, 我们有 ```imports/api/lists``` 
和 ```imports/api/todos``` 域。在每个目录中, 我们定义了用于操作相关域数据(domain data)的集合(collections)、发布(publications)
和方法(methods)。

>   注意: 在较大的应用程序中, 考虑到Todos本身是列表的一部分, 将这两个域组合成一个更大的 "列表" 模块可能会有意义。
Todos的例子是足够小的, 我们分离这些只是为了演示模块化。

在 ```imports/ui``` 目录中, 根据它们定义的 ui 端代码的类型将文件分组到目录中通常是有意义的, 例如, 顶层页面(pages)、
环绕版式(layouts)或可重用组件(components)。

对于上面定义的每个模块, 用基本的 JavaScript 文件合用各种辅助文件是有意义的。例如, ```Blaze UI``` 
组件应该在同一个目录中有它的模板 HTML、JavaScript 逻辑和 CSS 规则。带有某些业务逻辑的 JavaScript 模块应该与该模块的单元测试共存。

### 启动文件

您的某些代码不会成为业务逻辑或 UI 的单元, 它只是一些安装程序或配置代码, 需要在应用程序的上下文中启动时运行。
在Todos示例应用程序中, ```imports/client/startup/useraccounts-configuration.js``` 文件配置 ```useraccounts```
登录模板 (有关 useraccounts 的详细信息, 请参阅[Accounts](https://guide.meteor.com/accounts.html)文章)。
```imports/startup/client/route.js``` 配置所有路由, 然后导入客户端上所需的所有其他代码:

``` 
import { FlowRouter } from 'meteor/kadira:flow-router';
import { BlazeLayout } from 'meteor/kadira:blaze-layout';
import { AccountsTemplates } from 'meteor/useraccounts:core';
// Import to load these templates
import '../../ui/layouts/app-body.js';
import '../../ui/pages/root-redirector.js';
import '../../ui/pages/lists-show-page.js';
import '../../ui/pages/app-not-found.js';
// Import to override accounts templates
import '../../ui/accounts/accounts-templates.js';
// Below here are the route definitions
```

然后, 我们将这两个文件导入 ```imports/startup/client/index.js```

``` 
import './useraccounts-configuration.js';
import './routes.js';
```

这样, 就可以很容易地导入所有客户端启动代码, 并将其作为我们的急加载的客户端入口点 ```client/main.js```:

``` 
import '/imports/startup/client';
```

在服务器上, 我们使用相同的技术将所有启动代码导入 ```imports/startup/server/index.js```:

``` 
// This defines a starting set of data to be loaded if the app is loaded with an empty db.
import '../imports/startup/server/fixtures.js';
// This file configures the Accounts package to define the UI of the reset password email.
import '../imports/startup/server/reset-password-email.js';
// Set up some rate limiting and other important security settings.
import '../imports/startup/server/security.js';
// This defines all the collections, publications and methods that the application provides
// as an API to the client.
import '../imports/api/api.js';
```

我们的主服务器入口点 ```server/main.js``` 于是导入这个启动模块。您可以看到, 在这里, 
我们实际上并没有从这些文件中导入任何变量-我们只是导入它们, 以便它们按此顺序执行。

### 导入 Meteor “pseudo-globals”

为向后兼容性，Meteor 1.3 仍然在您的应用程序中提供Meteor的全局命名空间的meteor核心包, 以及其他meteor包。
你还可以直接调函数，```methods.publish```, 如同以前版本的meteor, 没有首先导入它们。
但是, 建议您在使用 "meteor/package" 语法之前, 首先使用 ```import { Name } from 'meteor/package'```, 
这是最好的做法。例如:

``` 
import { Meteor } from 'meteor/meteor';
import { EJSON } from 'meteor/ejson';
```

##  默认文件加载顺序

尽管建议您使用 ES2015 模块和 imports/ 目录编写应用程序, 但meteor 1.3 仍然支持使用这些默认的加载顺序规则来满足对文件的急加载,
以提供应用程序的向后兼容性。在单个应用程序中, 您可以使用导入来合并急加载和懒加载。任何导入语句都按它们在文件中列出的顺序进行计算, 
并使用这些规则对该文件进行加载。


有几个加载顺序规则。它们按顺序应用于应用程序中的所有适用文件, 优先级如下所示:

1.  HTML模板文件总是最先加载
2.  以main开头的文件最后加载
3.  任何lib/目录中的文件下一个加载
4.  具有更深层次路径的文件下一个加载
5.  然后按整个路径的字母顺序加载文件

``` 
nav.html
main.html
client/lib/methods.js
client/lib/styles.js
lib/feature/styles.js
lib/collections.js
client/feature-y.js
feature-x.js
client/main.js
```

例如, 上面的文件按正确的加载顺序排列。因为 html 模板总是先被加载, 即使它以main开头, 因为规则1优先于规则2。
但是, 它将在nav.html后加载, 因为规则2的优先级高于规则5。

根据规则4 ```client/lib/styles.js``` 和 ```lib/feature/styles.js``` 具有相同的优先权，但是client在
字母顺序上优先于lib，所以应当先加载。

你也可以使用 ```meteor.startup``` 控制代码在服务器和客户端上何时运行。

### 特殊目录

默认情况下, 在您的 "Meteor" 应用程序文件夹中的任何 JavaScript 文件都捆绑并加载到客户端和服务器上。
但是, 项目内的文件和目录的名称可能会影响其加载顺序、加载位置以及其他一些特性。下面是由流星特别处理的文件和目录名的列表:

-   **imports**

    任何名为 "imports/" 目录中文件都不会主动加载，都必须使用 "import" 导入文件。
    
-   **node_modules**

    任何名为 node_modules 的目录都不会在任何地方加载。在 node_modules 目录中安装的js软件包必须
    根据package.json中Npm.depends使用import导入。
    
-   **client**

    服务器上不会加载任何名为 ```client/``` 的目录。就象包含在条件判断里的代码一样 ```if (Meteor.isClient) {...}```。
    在生产模式下, 在客户端上加载的所有文件都将自动串联并缩小。在开发模式中, JavaScript 和 CSS 文件不是缩小的, 
    目的是使调试更加容易。但css文件仍然被合并到一个文件中, 以确保生产和开发之间的一致性, 因为更改css文件的url会影响处理url的方式。
    
    >   在Meteor应用程序中, HTML文件的处理方式与服务器端框架有很大的不同。Meteor扫描您的目录中的所有HTML文件中三种顶级元素: ```<head>```, ```<body>``` 和 ```<template>```。
    head和body部分分别连接到一个单一的head和body中, 并在页面加载时传送到客户端。
    
-   **server**

    客户端上不会加载任何名为 ```client/``` 的目录。就象包含在条件判断里的代码一样 ```if (Meteor.isServer) {...}```, 除非客户端未收到任何代码。
    您不希望向客户端提供的任何敏感代码 (如包含密码或身份验证机制的代码) 都应保存在```server/```目录中。
    Meteor收集所有的 JavaScript 文件, 排除```client/```、```public/``` 和 ```private/``` 子目录下的任何内容, 并将它们加载到一个```node.js```服务器实例中。
    在Meteor中, 服务器代码在每个请求中以单线程的形式运行, 而不是在```node.js```的异步回调进程中。
    
-   **public**

    顶级目录```public/```下所有文件都是为客户端服务的。在引用这些资源时, 不要在 url 中包括```public/```, 就像它们都在顶层一样写url。
    例如, 引用```public/bg.png``` 写成 ```<img src='/bg.png'/>```。这是存放```favicon.ico```、```robots.txt``` 和类似文件的最佳位置。
    
-   **private**

    顶级目录```private/```中的所有文件只能通过服务器代码访问,可以通过```Assets``` API加载。这可以用于存放私有数据文件和项目目录中不希望从外部访问的任何文件。
    
-   **client/compatibility**

    此文件夹是为了与 JavaScript 库兼容, 它们依赖于将在顶层以 var 声明的变量作为全局变量导出。执行此目录中的文件时不会将其包装在新的变量范围内。这些文件在其他客户端 JavaScript 文件之前执行。
    
    >   建议使用npm管理第三方JavaScript库, 并使用import来控制加载文件的时机。
    
-   **tests**

    任何名为 ```tests/``` 的目录都不会加载到任何位置。这是在[Meteor内置测试工具](https://guide.meteor.com/testing.html)之外
    使用测试工具来进行测试的测试代码。
    
以下目录也不会作为应用程序代码的一部分进行加载:

-   以```.```开头的目录，比如```.git```和```.meteor```
-   ```packages/```,用来存放本地包
-   ```cordova-build-override/```,用于[自定义构建移动应用](https://guide.meteor.com/mobile.html#advanced-build)
-   ```programs```,由于遗留原因保留

### 特殊目录之外的文件

特殊目录之外的所有JavaScript文件都会在客户端和服务器上加载。Meteor提供变量```Meteor.isClient``` 
和 ```Meteor.isServer```进行判断, 以决定您的代码在客户端或者服务器上运行。

特殊目录之外的 CSS 和 HTML 文件只在客户端加载, 不能从服务器代码中使用。

##  分拆成多个应用程序

如果您正在编写一个足够复杂的系统, 可能有机会将您的代码拆分为多个应用程序。例如, 您可能希望为管理用户界面创建单独的应用程序 
(而不是通过网站的管理部件检查权限, 您只需检查一次), 或者将应用程序的移动版本和桌面版本的代码分开。

另一个非常常见的用例是将一个工作进程从主应用程序中剥离出来, 这样昂贵的作业不会因为锁定单个 web 服务器来影响访问者的用户体验。

以这种方式拆分应用程序有一些好处:

-   如果您将特定类型的用户永远不会使用的代码分开, 那么您的客户端 JavaScript 包可能会大大减少。
-   您可以使用不同的设置部署不同的应用程序, 并以不同的方式保护它们 (例如, 您可能会将对管理员应用程序的访问限制在防火墙后面的用户)。
-   您可以让不同团队独立地处理不同的应用程序。

但是, 在决定拆分之前应考虑到，这种拆分工作对您有一些挑战。

### 共享代码

主要的挑战是在不同应用程序之间正确地共享代码。处理此问题的最简单方法是简单地在不同的 web 服务器上部署相同的应用程序, 
通过不同的设置进行控制。这种方法使您可以轻松地部署不同要求的应用, 但却享受不到上面所述的其他优势。

如果你想用拆分代码方法创建Meteor应用, 你会有一些共享模块。如果这些模块可以更广泛的得到使用, 
你应该考虑将它们发布到[软件包系统](https://guide.meteor.com/writing-packages.html), 
无论是 npm 还是 Atmosphere, 这取决于代码是特定于Meteor应用框架的还是其他别的框架的。

如果代码是私有的, 或者对其他人没有兴趣, 则在两个应用程序中简单地包含相同的模块就行了 (您可以使用 [私有npm模块](https://www.npmjs.com/private-modules) 实现)。
有几种方法可以做到这一点:

-   直接了当的方法就是将通用代码作为两个应用程序的 git 子模块(git submodule)。
-   或者, 如果在单个代码库中同时包含两个应用程序, 则可以使用符号链接将公共模块包括在两个应用程序中。

### 共享数据

另一个重要的考虑是如何在不同的应用程序之间共享数据。

最简单的方法是将两个应用程序指向同一个 MONGO_URL, 并允许两个应用程序直接对数据库进行读写。
多亏了Meteor对数据库的反应式编程的支持，这种方案完美地解决了问题。当一个应用程序改变了 MongoDB 中的一些数据, 
其他连接到数据库的应用程序的用户将立即看到数据的变化。谢天谢地，感谢Meteor的livequery。

然而, 在某些情况下, 最好允许一个应用程序成为主控程序, 控制其他应用程序通过API访问数据的权限。
如果您希望在不同的时间上部署不同的应用程序, 并且需要对数据进行保守的更改, 这样做会有所帮助。

提供```server-server``` API 的最简单方法是直接使用Meteor的内置 DDP 协议。这与您的Meteor客户机从服务器获取数据的方式相同, 
但您也可以使用它在不同的应用程序之间进行通信。您可以使用 ```DDP.connect()``` 从 "客户端" 服务器连接到主服务器, 
然后使用返回的连接对象进行方法调用以从publication中读取数据。

### 共享帐户

如果有两台服务器访问同一数据库, 并且您希望经过身份验证的用户在两者之间进行 DDP 调用, 则可以使用在一个服务器连接上设置的 "恢复令牌"
登录到另一个服务器。

如果用户已连接到服务器A, 则可以使用 DDP.connect() 打开到服务器B的连接, 然后传入服务器A的恢复令牌以在服务器B上进行身份验证。
由于两个服务器都使用相同的数据库, 因此在这两种情况下, 相同的服务器令牌都会起作用。进行身份验证的代码如下:

```
// This is server A's token as the default `Accounts` points at our server
const token = Accounts._storedLoginToken();
// We create a *second* accounts client pointing at server B
const app2 = DDP.connect('url://of.server.b');
const accounts2 = new AccountsClient({ connection: app2 });
// Now we can login with the token. Further calls to `accounts2` will be authenticated
accounts2.loginWithToken(token);
```
您可以在[示例库](https://github.com/tmeasday/multi-app-accounts)中看到此体系结构概念的证明。
