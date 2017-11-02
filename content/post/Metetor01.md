+++
title = "Meteor:开发Web应用和移动应用的利器（1）"
description = "Meteor，是开发现代Web应用和移动应用的JavaScript全栈式平台。"
tags = [
    "JavaScript",
    "NodeJs",
    "Web application",
    "Mobile application",
    "Platform",
]
date = "2017-11-01"
categories = [
    "Development",
    "nodejs",
]
thumbnail = "images/meteor_logo.png"
+++


#  介绍

本文是一份Meteor的使用指南，Meteor是开发现代Web应用和移动应用的JavaScript全栈式开发平台。本文基于官网的介绍改编，目的是
为了学习和理解，对学习和实践的过程进行记录，供有兴趣的同学参考。

<!--more-->

## 什么是Meteor?

据官网宣称，Meteor是一款极其牛叉的工具。它是一款开发现代Web应用和移动应用的全栈式JavaScript开发平台。它包含了构建连接到
客户端的反应式应用程序(connected-client reactive applications)的关键技术集，一套构建工具，一些从Node.js和JavaScript
社区精选的工具包(package)。

-   用一种语言开发：JavaScript，在所有环境使用：应用服务器、Web浏览器、和移动设备。【一剑在手，天下行走！老外真能吹。】
-   使用在线数据(data on the wire)【对应的是data on the disk】，表示服务器发送数据，而不是HTML，客户端宣染数据。
-   Meteor拥抱JavaScript生态系统，把极其活跃的社区中的精心选择的最好的东西介绍给你。
-   Meteor提供全栈反应能力，让你的UI以最小的努力无逢的反应世界的真实状态。

## 快速开始

Meteor支持OSX,Windows,和Linux。

在Windows上使用，请下载[安装程序](https://install.meteor.com/windows)。

在OSX/Linux系统安状最新的发行版本(注：更新或升级版本使用同样的命令)，请在命令行输入以下命令：

``` curl https://install.meteor.com/ | sh ```

Windows发装程序支持Windows 7, Windows 8.1, Windows Server 2008, and Windows Server 2012.
命令行安装程序支持Mac OS X10.7 (Lion)及以上版本,Linux x86和x86_64架构。
（以下的命令除非特别说明，我们都在OSX系统上操作）

一旦安装完Meteor，就可以创建工程：

``` meteor create myapp ```

马上在本地运行：

``` 
cd myapp 
meteor npm install
meteor
# meteor application running on : http://localhost:3000/
```

> Meteor捆绑安装了npm，所以在运行命令meteor npm时不需要担心有没有安装npm。当然，如果你开心，你可以用一个全局安装的npm
来管理你的包。

## Meteor资源

1.  学习meteor的[官方教程](https://www.meteor.com/tutorials/blaze/creating-an-app).
2.  [StackOverflow](http://stackoverflow.com/questions/tagged/meteor)是提出和解答技术问题的最佳场所，别忘了给你的问题加上meteor标签。
3.  进入[meteor论坛](https://forums.meteor.com/)去通报工程，获得帮助，聊聊人生，或者谈谈内核的变化。
4.  [Meteor文档中心](https://docs.meteor.com/)是查看内核API文档的最好去处。
5.  [Atmosphere](https://atmospherejs.com/)是特别为Meteor设计的社区包的仓库。
6.  [Awesome Meteor](https://github.com/Urigo/awesome-meteor)是社区精选的包和资源的列表。

## 什么是Meteor指南？

本指南收集了一些文章，罗列了使用Meteor平台开发应用程序的最佳实践。涵盖了现代Web应用和移动应用开发的通用模式，所以文中
提到的一些概念不仅适用于Meteor，也可应用于任何现代应用程序、用户交互的开发工作。

这份指南并非构建meteor应用程序 **必须** 遵守的，你当然可以使用不同的甚至相反的方法使用meteor平台。然而，本指南尝试给出一个最佳
实践和社区公约，希望社区的多数人能够在使用该指南的时候获得帮助。

Meteor平台的API文档可以从[文档中心](https://docs.meteor.com/)获得，你可以在[Atmosphere](https://atmospherejs.com/)
上面浏览社区贡献的第三方包。

### 目标读者

本指南的目标读者是有一定基础的开发者，熟悉JavaScript、meteor平台、web应用。如果你是meteor的新手，建议你学习官方的
[入门教程](https://www.meteor.com/tutorials/blaze/creating-an-app)。

### 示例应用

许多文章都引用了Todos示例应用程序，该程序的代码是在本指南开发过程中开发出来的。你可以在[github](https://github.com/meteor/todos)中查看代码、提交issue或者通过
提交PR修改代码。

##  指南的编写

### 作出贡献

本指南的编写工作在[github](https://github.com/meteor/guide)上是开放的，我们鼓励大家提交文档、代码，或者提出问题，为指南的编写工作作出贡献。我们希望通过真诚和开放的
态度使将要进行的工作更加明确：指南中需要什么、将来的版本需要怎样改进。

### 目标

本指南里决策和最佳实践都是观点鲜明的。我们将突出某些最佳实践, 并忽略其他有效方法。我们的目标是围绕重大决策达成社区共识, 但在开发应用程序时总会有其他方法来解决问题。

我们认为重要的是要知道解决问题的 "标准" 方法，而不是为解决问题而寻求其他途径。如果有的解决办法确实很优秀，那么我们在将来的版本要把它写进指南中。

该指南的一个重要功能是指导meteor平台未来的发展方向。通过编写最佳实践，指南可以为将来的平台开发指明方向：如何变得更好、变得更容易、变得更高效。

同样，本指南里强调的一些平台还没有实现的重要方案，可以通过开发社区包的实现。我们鼓励你发现机会，为改进meteor工作流开发社区包。
如果你不确定如何更好地设计包的架构，请去社区论坛展开讨论。

#   代码风格（Code Style）

建议遵循的代码风格。

通过阅读本文，你将知道：

1.  为什么要坚持一贯的代码风格
2.  我们建议的JavaScript代码风格
3.  如何设置ESLint自动检查代码风格
4.  为meteor建议的代码模式，比如方法代码风格，发布方式，等等

##  坚持一贯风格的益处

多年来, 开发人员花了无数时间讨论单引号、双引号、括号、空格数以及其他各种外观代码样式问题。这些都
是与代码质量有关联的问题，但是，很容易因为他们外观可见而产生分歧。

尽管你的代码对字符串使单引号还是双引号并不一定重要，但在你的整个代码组织中进行一次决策并保持一致风格是有很大好处的。这些益处
同样适用于使meteor和整个JavaScript社区的开发工作保持一致。

###  容易阅读代码

就象阅读英文句子时并不是一个字一个字地阅读，在阅读代码的时候，并不是一个符号一个符号地阅读。通常，你只是看了表达式
的形状，或者重点，就断定代码是做什么的。如果所有代码都保持一贯风格，就能确保看起来相象的代码 **确实是** 相同的代码——没有隐藏
的标点符号或者陷阱，这样你就可以专注于理解逻辑而不是理解符号。一个例子就是缩进，在JavaScript代码中，缩进并没有意义，但
保持代码缩进的一贯性，在阅读代码时，并不需要阅读每一个括号细节以理解代码的用途。

``` 
// This code is misleading because it looks like both statements
// are inside the conditional.
if (condition)
  firstStatement();
  secondStatement();
```

``` 
// Much clearer!
if (condition) {
  firstStatement();
}
secondStatement();
```

###  自动检查错误

保持一贯的代码风格意味着容易使用标准的工具来检查错误。比如你采用一种约定，使用```let```和```const```代替```var```
申明变量，你就能用一种工具确保变量被正确申明。这意味着可以避免变量产生意外行为。同样，强制所有的变量在使用之前申明，就能
很容易地在代码运行前发现拼写错误。

### 深入理解

在学习一门程序语言的时候，很难一次性掌握所有的东西。例如，程序员刚接触JavaScript时，常常为```var```关键字和方法范围
(function scope)困惑。使用社区推荐的代码风格配合自动语法分析就能提前警示这些陷阱。这意味着你可以马上开始写代码，而不需要
把JavaScript的方方面面都学习之后再写代码。

##  JavaScrip风格指南

在meteor的世界里，我们坚信JavaScript是构建web应用程序的最好的语言，这是有很多原因的。JavaScript在不断地改进，JavaScript
ES2015标准把社区真正引领到了一起。下面的代码显示了ES2015标准关于JavaScript的编码建议。

![](/images/ben-es2015-demo.gif)

>   一段代码显示了从JavaScript到ES2015的重构过程

### 使用ecmascript包

ECMAScript, 是每个浏览器的JavaScript实现所基于的语言标准, 每年都有新的标准发布。最新的完整标准是ES2015, 
其中包括一些期待已久的和非常重要的对JavaScript语言的改进。Meteor的ECMAScript包使用流行的Babel编译器将ES2015标准语言
编译成常规的JavaScript语言，使得所有的浏览器都能理解执行。它是完全向后兼容到"常规"JavaScript的，所以如果你不乐意，
你就没必要使用新的特性。我们努力使一些高级的浏览器特性比如代码地图(source maps)跟Meteor包很好地协作，因此你可以使用你
最喜欢的开发工具，不需要查看编译结果。

默认情况下, ecmascript包包含在所有新应用程序和包中, 并自动编译带有.js文件扩展名的所有文件。
[查看ecmascript软件包支持的所有ES2015功能的列表](https://docs.meteor.com/packages/ecmascript.html#Supported-ES2015-Features)。

要获得完整的经验, 你也应该使用es5-shim包, 默认情况下这是包括在所有新的应用程序中。这意味着您可以依赖于运行时功能(如Array#forEach), 
而无需关心哪些浏览器支持它们。

本指南中的所有代码示例和未来的Meteor教程将使用所有新的ES2015功能。你也可以阅读更多关于ES2015和如何开始使用它的Meteor博客:

-   [Getting started with ES2015 and Meteor](http://info.meteor.com/blog/es2015-get-started)
-   [Set up Sublime Text for ES2015](http://info.meteor.com/blog/set-up-sublime-text-for-meteor-es6-es2015-and-jsx-syntax-and-linting)
-   [How much does ES2015 cost?](http://info.meteor.com/blog/how-much-does-es2015-cost)

### 遵循JavaScript样式指南

我们建议选择和坚持一种JavaScript风格的指南, 并使用工具来确保南彻它。我们推荐的一个主流选择是带有ES6扩展功能的Airbnb样式指南 
(带有React扩展选项)。

##  使用ESLint检查代码

"代码语法分析(linting)"是自动检查代码中常见错误或样式问题的过程。例如, ESLint可以发现您是否在变量名中有输入错误, 或者由于条件
写得不好而无法访问代码的某些部分。

我们建议使用Airbnb eslint配置来验证工具。[Airbnb样式指南](https://github.com/airbnb/javascript/tree/master/packages/eslint-config-airbnb)。

下面, 您可以在许多不同的开发阶段找到设置自动语法分析的方法。通常, 您希望尽可能频繁地运行语法分析器,因为它是识别错误和小错误的最快和最简单的方法。

### 安装和运行ESLint

若要在应用程序中安装ESLint, 可以安装以下npm程序包:

``` 
meteor npm install --save-dev babel-eslint eslint-config-airbnb eslint-plugin-import eslint-plugin-meteor eslint-plugin-react eslint-plugin-jsx-a11y eslint-import-resolver-meteor eslint @meteorjs/eslint-config-meteor
```

>   Meteor与npm捆绑在一起, 以便您可以键入流星npm, 而无需担心自己安装。如果愿意, 还可以使用全局安装的npm命令。

您还可以向```package.json```中添加```eslintConfig```节指定您要使用Airbnb配置, 并启用
[ESLint-meteor-Meteor](https://github.com/dferber90/eslint-plugin-meteor)。
您还可以设置要更改的任何额外规则, 以及添加一个```lint npm```命令:

``` 
{
  ...
  "scripts": {
    "lint": "eslint .",
    "pretest": "npm run lint --silent"
  },
  "eslintConfig": {
    "extends": "@meteorjs/eslint-config-meteor"
  }
}
```

要运行linter,只需输入命令：

``` 
meteor npm run lint
```

有关详细信息, 请阅读ESLint网站的[入门指南](http://eslint.org/docs/user-guide/getting-started)。

### 与编辑器集成

语法分析是在代码中查找潜在bug的最快方法。运行linter通常比运行应用程序或单元测试快, 所以一直运行它是一个好主意。
在您的编辑器中设置语法分析可能看起来很烦人, 因为它会在您保存格式较差的代码时经常抱怨, 但随着时间的推移, 
您要开发健壮的程序,就要在一开始就编写合格的代码。以下是在不同编辑器中设置 ESLint 的一些方法:

**Sublime Text**

您可以安装将它们集成到文本编辑器中的Sublime Text包。通常建议使用Package Control来添加这些包。如果您已经有了该设置,
您只需按名称添加这些软件包即可。如果没有, 请单击说明链接:

-   Babel (高亮语法 – [完整说明](https://github.com/babel/babel-sublime#installation))
-   SublimeLinter ([完整说明](http://sublimelinter.readthedocs.org/en/latest/installation.html))
-   SublimeLinter-contrib-eslint ([完整说明](https://github.com/roadhump/SublimeLinter-eslint#plugin-installation))

若要获得正确的语法高亮显示, 请转到.js文件, 然后通过 ```View``` 下拉菜单选择以下内容: 
 ```Syntax -> Open all with current extension as… -> Babel -> JavaScript (Babel)``` 。如果您使用的是jsx文件,
请从.jsx文件执行相同的操作。如果它工作,当你打开这些文件时，你会在窗口右下角看到 ```JavaScript (Babel)```。
有关兼容配色方案的信息, 请参阅[软件包自述文件](https://github.com/babel/babel-sublime)。
 

Emmet用户的便笺: 您可以使用 \n 扩展.jsx文件中的HTML标记, 它将正确扩展classes为React "className"属性。
您可以将这个动作绑定到tab键, 但您可能不想这样做。

**Atom**

在Atom中使用ESLint很简单。只需安装以下三软件包:

``` 
apm install language-babel
apm install linter
apm install linter-eslint
```
然后重新启动 (或按 Ctrl + Alt + R / Cmd + Opt + R) Atom以激活语法分析。

**WebStorm**

WebStorm 提供了[使用ESLint说明](https://www.jetbrains.com/webstorm/help/eslint.html)。在您安装ESLint Node包并设置您的 *package.json* 后, 只需启用ESLint并单击 "应用"。
您可以配置 WebStorm 应该如何找到您的 *.eslintrc* 文件, 但在我的机器上不做任何改变就能工作。它还自动建议切换到 "JSX Harmony" 语法高亮。

![](/images/webstorm-configuration.png)

语法分析可以逐个项目的在WebStorm上激活, 也可以将ESLint设置为默认：Editor > Inspections、选择默认配置文件、勾选 "ESLint" 并"应用"。

**Visual Studio Code**

在 VS Code 中使用 ESLint 需要安装第三方 ESLint 扩展。要安装扩展, 请按照下列步骤操作:

1.  启动 VS Code, 通过键入 Ctrl + P 打开快速打开的菜单;
2.  在命令窗口中粘贴```ext install vscode-eslint```, 然后按 Enter 键;
3.  重启VS Code.

##   Meteor代码样式

以上部分讨论了 javascript 代码的一般规范 -- 你可以很容易地应用在任何 javascript 应用程序中, 而不仅仅是Meteor应用。
然而, 有一些风格问题是特定于Meteor的, 特别是如何命名和组织应用程序不同组件的结构。

###  Collections

Collections应命名为复数名词, 使用PascalCase命名规范。数据库中集合的名称 (集合构造函数的第一个参数) 应与JavaScript符号的名称相同。

``` 
// Defining a collection
Lists = new Mongo.Collection('lists');
```

数据库中的字段应该使用camelCased法则, 就像您的 JavaScript 变量名一样。

``` 
// Inserting a document with camelCased field names
Widgets.insert({
  myFieldName: 'Hello, world!',
  otherFieldName: 'Goodbye.'
});
```

### Methods 和 publications

Method和publication名称使用camelCased法则，并冠以模块的名字空间：

``` 
// in imports/api/todos/methods.js
updateText = new ValidatedMethod({
  name: 'todos.updateText',
  // ...
});
```
请注意, 此代码示例使用[ValidatedMethod package recommended in the Methods article](https://guide.meteor.com/methods.html#validated-method)
中建议的ValidatedMethod包。如果您没有使用该包, 则可以使用该名称作为传递给 Meteor.methods 的属性。

以下是此命名约定在应用于publication时的外观:

``` 
// Naming a publication
Meteor.publish('lists.public', function listsPublic() {
  // ...
});
```
### Files, exports, and packages

您应该使用 ES2015 的import和export功能来管理您的代码。这将让您更好地理解代码的不同部分之间的依赖关系, 如果需要读取依赖项的源代码, 就很容易知道在哪里可以查找。

应用程序中的每个文件都应代表一个逻辑模块。避免使用可导出各种无关函数和符号的"垃圾箱"实用程序模块。通常, 这意味着每一个类、UI组件或每个集合各自放在一个文件, 
但是在某些情况下, 可以有例外, 例如, 如果您的ui组件中有一个小组件, 它没有在该文件之外使用。

当文件表示单个类或 UI 组件时, 该文件的名称应与它定义的对象相同, 其大小写相同。因此, 如果您有一个导出类的文件:

``` 
export default class ClickCounter { ... }
```

此类应在名为 ClickCounter 的文件中定义。当您导入它时, 它的外观会像这样:

``` 
import ClickCounter from './ClickCounter.js';
```

请注意, 导入使用相对路径, 并在文件名末尾包含文件扩展名。

对于[Atmosphere package](https://guide.meteor.com/using-packages.html), 作为较旧的 pre-1.3 api.export
语法允许每个包有多个导出, 您将倾向于看到用于符号的非默认导出。例如:

``` 
// You'll need to destructure here, as Meteor could export more symbols
import { Meteor } from 'meteor/meteor';
// This will not work
import Meteor from 'meteor/meteor';
```

### 模板和组件(Templates and components)

由于 Spacebars 模板始终是全局的, 因此不能作为模块导入和导出, 并且需要在整个应用程序中具有完全唯一的名称, 
因此建议使用命名空间的完整路径来命名Blaze模板, 并以下划线分隔。在这种情况下, 下划线是一个很好的选择,
因为这样您就可以在 JavaScript 中轻松地键入模板的名称作为一个符号。

``` 
<template name="Lists_show">
  ...
</template>
```

如果此模板是加载服务器数据并访问路由器的 "智能" 组件, 请将 _page 追加到该名称上:

``` 
<template name="Lists_show_page">
  ...
</template>
```

通常在处理模板或 UI 组件时, 您将有几个紧密耦合的文件要管理。它们可以是两个或更多的 HTML、CSS 和 JavaScript 文件。在这种情况下, 我们建议将这些内容放在同一目录中, 其名称相同:

``` 
# The Lists_show template from the Todos example app has 3 files:
show.html
show.js
show.less
```

整个目录或路径应表明这些模板与列表模块相关, 因此不必在文件名中重现该信息。请阅读[下面](https://guide.meteor.com/structure.html#javascript-structure)的目录结构。

如果您在React中编写, 则不需要使用下划线-拆分名称, 因为您可以使用 JavaScript 模块系统导入和导出组件。