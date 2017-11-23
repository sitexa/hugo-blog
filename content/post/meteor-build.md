+++
title = "Meteor(13):Meteor Build"
description = "Atmosphere vs. npm"
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

从头开始构建应用程序是一个很大的挑战。这是为什么要使用Meteor的主要的原因。你可以专注于编写应用程序代码, 而不是重新发明的轮子, 如用户登录和数据同步。
为了进一步简化工作流程, 使用 npm 和 atmosphere 中的社区包是很有帮助的。这些软件包中的许多都是在指南中推荐的, 您可以在联机目录中找到更多信息。

<!--more-->

随着版本1.3 的发布, Meteor已完全支持 npm。将来, 将来会将所有包迁移到 npm, 但目前这两个系统都有好处。

##  何时使用Atmosphere包

Atmosphere包是专门为Meteor编写的软件包, 在与Meteor一起使用时, 与 npm 相比有几个优点。特别是, Atmosphere包可以:

-   依靠核心的Meteor包, 如 ddp 和 Blaze
-   显式包括 non-javascript 文件, 包括 CSS, Less, Sass, Stylus和静态资源
-   利用Meteor的构建系统自动 transpiled 像 CoffeeScript 这样的语言
-   有一个定义良好的方式为客户端和服务器运送不同的代码, 在每个上下文中启用不同的行为
-   直接访问eteor的包空间和封装全局出口, 而无需显式使用 ES2015 导入
-   使用eteor约束冲突解决程序在包之间强制执行精确版本依赖项
-   包括构建插件的eteor的构建系统
-   为不同的服务器体系结构 (如 Linux 或 Windows) 包括预构建的二进制代码

如果你的包依赖于Atmosphere包 (在流星 1.3, 包括Meteor核心包), 或需要利用Meteor的构建系统, 编写一个Atmosphere包可能是目前最好的选择。

##  何时使用npm包

npm 是一个通用 JavaScript 包的存储库。这些软件包最初仅用于Node.js 服务器端环境, 但是随着 JavaScript 生态系统的成熟, 解决方案出现了, 可以在其他环境 (如浏览器) 中使用 npm 包。现在, npm 用于所有类型的 JavaScript 包。

如果要分发和重用为Meteor应用程序编写的代码, 则应该考虑在 npm 上发布该代码 (如果它的一般性足以被更广泛的 JavaScript 访问群体所使用)。在Meteor应用中使用 npm 包是很简单的, 
并且可以在Atmosphere包中使用 npm 包, 因此即使您的主要受众是Meteor开发者, npm 也可能是最好的选择。
 
>   流星与 npm 捆绑在一起, 以便您可以键入 ```meteor npm```, 而无需担心自己安装。如果愿意, 还可以使用全局安装的 npm 来管理包。

