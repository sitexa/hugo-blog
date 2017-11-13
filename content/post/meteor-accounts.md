+++
title = "Meteor(6):用户(Users)和帐户(Accounts)"
description = "How to build user login functionality into a Meteor app. Let your users log in with passwords, Facebook, Google, GitHub, and more."
tags = [
    "JavaScript",
    "NodeJs",
    "Web application",
    "Mobile application",
    "Platform",
    "MongoDB",
]
date = "2017-11-09"
categories = [
    "Development",
    "nodejs",
]
thumbnail = "images/meteor-guide/Meteor-ui.jpg"
+++

如何建立一个Meteor应用程序的用户登录功能，让您的用户使用密码、Facebook、Google、GitHub或别的认证系统登录应用程序。
<!--more-->

读完这篇文章后, 您将知道:

1.  Meteor的哪些核心功能启用了用户帐户
2.  在快速原型中如何使用```accounts-ui```
3.  如何使用 ```useraccounts``` 系列的软件包构建您的用户登录界面
4.  如何构建具有完整功能的密码登录体验
5.  如何通过像 ```Facebook``` 这样的 ```OAuth``` 提供商登录
6.  如何将自定义数据添加到Meteor的用户集合中
7.  如何管理用户角色(roles)和权限(permissions)


##  核心Meteor功能

在我们进入Meteor的所有不同的面向用户的帐户功能之前, 让我们去了解一些内置在流星DDP协议和```accounts-base```包。
如果在您的应用程序中需要用户帐户，你一定要知道Meteor的这部分功能。其他的大多数包都是可选的, 可通过包添加/删除来先取。

### DDP的userId

DDP是Meteor内置的 ```pub/sub``` 和 RPC 协议。您可以在[数据加载](https://guide.meteor.com/data-loading.html)和
[方法文章](https://guide.meteor.com/methods.html)中阅读有关如何使用它的信息。除了数据加载和方法调用的概念外, DDP还有一个功能, 
即在连接上建立```userId```字段的想法。这是跟踪登录状态的位置, 无论您使用的是哪个帐户 UI 包或登录服务。

这个内置的特性意味着你总是能在方法和发布中得到 ```this.userId```, 并且可以访问客户端上的```userID```。
这是构建您自己的自定义帐户系统的一个很好的起点, 而且大多数开发人员不需要担心该机制, 因为您将主要与 ```accounts-base``` 包进行交互。

### accounts-base

这个包是Meteor的"面向开发者"用户帐户功能的核心。这包括:

1.  一个使用标准模式的用户集合, 通过 ```Meteor.users``` 和客户端的单例模式的```Meteor.userId()``` 和 ```Meteor.user()```, 
代表了客户端上的登录状态。
2.  各种有用的通用方法用来跟踪登录状态、注销、验证用户等。请访问[文档的帐户部分](http://docs.meteor.com/#/full/accounts_api)以查找完整的列表。
3.  一个用于注册新登录处理程序的API, 它被所有其他帐户包用来与帐户系统集成。这个API没有任何正式的文档, 
但是你可以在 [MeteorHacks blog](https://web.archive.org/web/20160913210817/https://meteorhacks.com/extending-meteor-accounts) 上了解更多信息。

通常, 你不需要自己引入```accounts-base```包, 因为你在使用```accounts-password```或类似包的时个，它已经被自动引进来了。
但知道是怎么回事是有意义的。

##  使用```accounts-ui```快速构建原型

通常，当你开发一个新的应用系统的时候，你并不会一开始就建立一套复杂的帐户系统。因此，如果能有一些东西（帐户系统）让你快速引入进来是很有意义的。
这就是"帐户界面"(accounts-ui)的由来——只要一行代码就能使应用系统建立起帐户系统。引入帐户界面：

``` 
meteor add accounts-ui
```

然后，只要在Blaze模板中需要的地方包含进去：

``` 
{{> loginButtons}}
```

最后, 确保选择一个登录提供商;它们将自动与客户界面集成:

``` 
# pick one or more of the below
meteor add accounts-password
meteor add accounts-facebook
meteor add accounts-google
meteor add accounts-github
meteor add accounts-twitter
meteor add accounts-meetup
meteor add accounts-meteor-developer
```

现在, 只要打开你的应用程序, 按照配置步骤, 就可以了——如果你已经完成了[Meteor教程](https://www.meteor.com/tutorials/blaze/adding-user-accounts),
你就可以看到了实效了。当然, 在一个实际的应用程序, 您可能需要一个自定义的用户界面和一些逻辑来拥有更量身定制的用户交互界面(UX), 
这正是为什么我们有本指南的其余部分的原因。

这里有一个帐户界面的截图, 让你知道它到底是什么样子:

{{< figure src="/images/meteor-guide/accounts-ui.png" >}}

##  可定制的用户界面: useraccounts

一旦你已经得到了你的初始原型和可运行的帐户界面(accounts-ui), 你就会想得到更强大的和可配置的东西, 使您可以更好地将您的登录流程集成到您的应用程序的其余部分中去。
[useraccounts系列的软件包](https://github.com/meteor-useraccounts/core/blob/master/Guide.md)是如今可用于Meteor平台上的一套最强大的帐户管理界面控件。
如果你有更多的定制要求, 你也可以开发你自己的系统, 但```useraccounts```是第一个值得尝试的系统。

### 使用路由器或UI框架

关于 ```useraccounts```, 首先要了解的是, 核心帐户管理逻辑独立于HTML模板和路由包。这意味着您可以使用 ```useraccounts:core```
来构建您自己的登录模板集。通常, 您需要选择一个登录模板包和一个登录路由包。模板选项包括:

1.  [useraccounts: unstyled](https://atmospherejs.com/useraccounts/unstyled), 让你使用自己的CSS; 
这是在Todos示例应用程序中使用的, 使登录用户界面与应用程序的其余部分无缝地结合在一起。
2.  为[Bootstrap, Semantic UI, Materialize](https://github.com/meteor-useraccounts/core/blob/master/Guide.md#available-versions)等
预构建的模板。这些模板没有实际的CSS框架, 所以你可以选择你喜欢的界面包，例如Bootstrap包。

虽然路由器(Router)它是可选的, 基本功能可以在没有它的情况下工作, 但选择集成一个路由器是一个好主意:

-   [Flow Router](https://atmospherejs.com/useraccounts/flow-routing), 本指南[推荐的路由器](recommended in this guide)。
-   [Iron Router](https://atmospherejs.com/useraccounts/iron-routing), 流星社区的另一个流行的路由器。

在示例应用程序中, 我们使用的流程路由器集成非常成功。后面的部分将介绍如何自定义路由和模板, 以便更好地适应您的应用程序。

### 不带路由的用户界面

如果您不希望为登录流配置路由, 您可以直接引入 ```self-managing``` 帐户屏幕。无论您希望帐户UI模板呈现在哪里, 只需包括 ```atForm``` 模板, 如下如示:

``` 
{{> atForm}}
```

一但你根据[下面的部分](#customizing-routes)配置路由后, 您将希望删除此段代码。

### 自定义模板

对于某些应用程序, 由各种 ```useraccounts UI``` 软件包提供的现成登录模板就能完成正常工作, 
但大多数应用程序都希望自定义某些呈现形。使用```aldeed:template-extension```软件包的模板替换功能, 有一个简单的方法能够满足这个需要。

首先, 通过查看包的源代码来找出要替换的模板。例如, 在 ```useraccounts: unstyled``` 包中, 这些模板在GitHub的[这个目录](https://github.com/meteor-useraccounts/unstyled/tree/master/lib)中列出。
看一眼文件名并查找一些HTML字符串, 我们可以发现我们可能对替换 ```atPwdFormBtn``` 模板感兴趣。让我们来看看原始模板:

``` 
<template name="atPwdFormBtn">
  <button type="submit" class="at-btn submit {{submitDisabled}}" id="at-btn">
    {{buttonText}}
  </button>
</template>
```

一旦确定了需要替换的模板, 请定义一个新模板。在这种情况下, 我们要修改该按钮上的类, 以便与应用程序的其余部分一起使用 CSS。
在重写模板时要记住几件事:

1.  以与上一个模板相同的方式呈现帮助器。在这种情况下, 我们使用 ```buttonText```。
2.  保留任何```id```属性, 如```at-btn```, 因为它们用于事件处理。

下面是我们重写的新模板的外观:

``` 
<template name="override-atPwdFormBtn">
  <button type="submit" class="btn-primary" id="at-btn">
    {{buttonText}}
  </button>
</template>
```
然后, 使用模板上的```replaces```函数在 ```useraccounts``` 中重写现有模板:

``` 
Template['override-atPwdFormBtn'].replaces('atPwdFormBtn');
```

### <a name="customizing-routes"></a>自定义路由

除了对模板进行控制之外, 您还希望能够控制```useraccounts```提供的不同视图的路由和 url。由于```Flow Router```是官方推荐的流星的路由选择, 我们将详细介绍这一点。

首先, 我们需要配置在呈现帐户模板时要使用的布局:

``` 
AccountsTemplates.configure({
  defaultTemplate: 'Auth_page',
  defaultLayout: 'App_body',
  defaultContentRegion: 'main',
  defaultLayoutRegions: {}
});
```

在这种情况下, 我们希望对所有与帐户相关的页面使用 App_body 布局模板。此模板有一个名为 main 的内容区域。现在, 让我们配置一些路由:

``` 
// Define these routes in a file loaded on both client and server
AccountsTemplates.configureRoute('signIn', {
  name: 'signin',
  path: '/signin'
});
AccountsTemplates.configureRoute('signUp', {
  name: 'join',
  path: '/join'
});
AccountsTemplates.configureRoute('forgotPwd');
AccountsTemplates.configureRoute('resetPwd', {
  name: 'resetPwd',
  path: '/reset-password'
});
```

请注意, 我们已经指定了密码重置路由。通常情况下, 我们应当配置Meteor的帐户系统来发送此路由的密码重置电子邮件, 
但```useraccounts:flow-routing```软件包为我们做了这个工作。请参阅下面有关[配置电子邮件工作流的详细信息](#email-flows)。

既然在服务器上配置了路由, 就能从浏览器访问它们 (例如 ```example.com/reset-password```)。若要在模板中创建指向这些路由的链接,
最好使用路由器提供的帮助器方法。对于"流路由器", [arillo:flow-router-helpers](https://github.com/arillo/meteor-flow-router-helpers/)
软件包就是为这个目的提供的一个```pathFor``` 帮助器。安装后, 可以在模板中执行以下操作:

``` 
<div class="btns-group">
  <a href="{{pathFor 'signin'}}" class="btn-secondary">Sign In</a>
  <a href="{{pathFor 'join'}}" class="btn-secondary">Join</a>
</div>
```

您可以在 [useraccounts:flow-routing](https://github.com/meteor-useraccounts/flow-routing#routes) 的文档中找到不同可用路由的完整列表。

### 进一步的自定义

除了模板和路由,```useraccounts```还提供了许多其他自定义选项。阅读[useraccounts指南](https://github.com/meteor-useraccounts/core/blob/master/Guide.md)了解所有其他选项。

##  密码登录

Meteor带有一个安全和全功能的密码登录系统的模块。要使用它, 请添加包:

```
meteor add accounts-password
```
要查看对您可用的选项, 请阅读Meteor中的[accounts-password API](http://docs.meteor.com/#/full/accounts_passwords)的完整描述。

### 要求用户名或电子邮件

>   注意: 如果您使用的是 ```useraccounts```, 则不必这样做。它为您禁用常规的Meteor客户端帐户创建功能, 并进行自定义验证。

默认情况下, ```accounts-password``` 提供的```Accounts.createUser```功能允许您创建具有用户名、电子邮件或两者的帐户。
大多数应用程序都希望两者的特定组合, 因此您需要验证新用户的创建:

``` 
// Ensuring every user has an email address, should be in server-side code
Accounts.validateNewUser((user) => {
  new SimpleSchema({
    _id: { type: String },
    emails: { type: Array },
    'emails.$': { type: Object },
    'emails.$.address': { type: String },
    'emails.$.verified': { type: Boolean },
    createdAt: { type: Date },
    services: { type: Object, blackbox: true }
  }).validate(user);
  // Return true to allow user creation to proceed
  return true;
});
```

### 多个电子邮件

通常, 用户可能希望将多个电子邮件地址与同一帐户相关联。```accounts-password```通过将电子邮件地址存储为用户集合中的数组来处理此种情况。
有一些方便的 API 方法来处理[adding](http://docs.meteor.com/api/passwords.html#Accounts-addEmail)、
[removing](http://docs.meteor.com/api/passwords.html#Accounts-removeEmail)和
[verifying](http://docs.meteor.com/api/passwords.html#Accounts-verifyEmail)电子邮件。

为您的应用程序添加一个"主要的(primary)"电子邮件地址的概念可能有用。这样, 如果用户添加了多个电子邮件, 你知道给哪里发送确认电子邮件。

### 区分大小写

在Meteor1.2 之前, 数据库中的所有电子邮件地址和用户名都是区分大小写的(case-sensitive)。这意味着, 如果您注册了一个帐户作为 ```AdaLovelace@example.com```, 
使用 ```adalovelace@example.com```登录, 就会报错, 提示没有用户使用该电子邮件。这可能相当混乱, 所以我们决定在Meteor1.2以后的版本改善改变这种做法。
但情况并不像看起来那么简单;由于MongoDB没有的索引不区分大小写(case-insensitive), 因此无法保证在数据库级别上有唯一的电子邮件。
因此, 我们有一些用于查询和更新用户的特殊 api, 它们在应用程序级别管理区分大小写(case-sensitivity)的问题。

**这对我的应用程序意味着什么？**

只需遵循一个简单的规则: 不要直接通过用户名或电子邮件查询数据库。使用Meteor提供的方法 [Accounts.findUserByUsername](http://docs.meteor.com/api/passwords.html#Accounts-findUserByUsername)
和 [Accounts.findUserByEmail](http://docs.meteor.com/api/passwords.html#Accounts-findUserByEmail)，
就能运行区分大小写(case-insensitive)的查询, 就能找到您要查找的用户。

### Email工作流

当您有一个基于用户电子邮件的应用程序登录系统, 就有打开电子邮件帐户流程的可能性。所有这些工作流之间的共同之处在于, 
它们涉及向用户的电子邮件地址发送一个唯一的链接, 当单击该邮件时, 它会做一些特别的事情。让我们看看一些常见的例子, 
Meteor的 ```accounts-password``` 包支持的功能:

1.  **密码重置**。当用户单击电子邮件中的链接时, 他们将被带到一个页面, 在那里他们可以为他们的帐户输入新密码。
2.  **用户注册**。新用户由管理员创建, 但没有设置密码。当用户单击电子邮件中的链接时, 他们会被带到一个页面, 在那里他们可以为他们的帐户设置新的密码。与密码重置非常相似。
3.  **电子邮件验证**。当用户单击电子邮件中的链接时, 应用程序会记录此电子邮件确实属于正确的用户。

在这里, 我们讨论如何人工管理从开始到结束的整个过程。

**电子邮件在 ```accounts UI``` 软件包中的工作流程**

如果你只想要完成工作, 你可以直接使用 ```accounts-ui```  或 ```useraccounts```, 基本上为你实现了整个流程。
如果你确定要建立自己的电子邮件工作流程，就要按照下面的指引来做。

**发送邮件**

```accounts-password``` 具有很方便的功能, 让您可以从服务器调用它来发送电子邮件。他们的命名描述了他们的功能:

1.  [Accounts.sendResetPasswordEmail](http://docs.meteor.com/#/full/accounts_sendresetpasswordemail)
2.  [Accounts.sendEnrollmentEmail](http://docs.meteor.com/#/full/accounts_sendenrollmentemail)
3.  [Accounts.sendVerificationEmail](http://docs.meteor.com/#/full/accounts_sendverificationemail)

电子邮件是使用[Accounts.emailTemplates](http://docs.meteor.com/#/full/accounts_emailtemplates)中的电子邮件模板生成的, 
其中包含了与```Accounts.urls```一起生成的链接。稍后我们将详细介绍如何自定义电子邮件内容和 URL。

**在单击链接时标识它**

当用户收到电子邮件并单击里面的链接时, 他们的web浏览器将把它们带到您的应用程序中。现在, 您需要识别这些特殊的链接, 并采取适当的行动。
如果您还没有自定义链接URL, 那么您可以使用一些内置回调来确定应用程序在电子邮件流程中。

通常情况下, 当Meteor客户机连接到服务器时, 它的第一件事就是通过登录恢复令牌来重新建立上一个登录。但是, 当触发来自电子邮件流程的这些回调时, 
直到您的代码指示它已完成(finished)对请求的处理, 并调用传递到已注册回调的已完成(done)函数后, 才会发送该恢复令牌。
这意味着, 如果您以前以用户A身份登录, 然后单击了用户B的 "重置密码" 链接, 但随后通过调用```done()```取消了密码重置流程, 
则客户端将再次以用户A登录。

1.  [Accounts.onResetPasswordLink](http://docs.meteor.com/#/full/Accounts-onResetPasswordLink)
2.  [Accounts.onEnrollmentLink](http://docs.meteor.com/#/full/Accounts-onEnrollmentLink)
3.  [Accounts.onEmailVerificationLink](http://docs.meteor.com/#/full/Accounts-onEmailVerificationLink)

下面代码举例说明如何使用这些函数:

```
Accounts.onResetPasswordLink((token, done) => {
  // Display the password reset UI, get the new password...
  Accounts.resetPassword(token, newPassword, (err) => {
    if (err) {
      // Display error
    } else {
      // Resume normal operation
      done();
    }
  });
})
```
如果您想为重设密码页面提供不同的URL, 则需要使用```Accounts.urls```选项进行自定义:

``` 
Accounts.urls.resetPassword = (token) => {
  return Meteor.absoluteUrl(`reset-password/${token}`);
};
```

如果您已自定义了url, 则需要向路由器中添加一个处理您指定的url的新路由, 这样，默认的```Accounts.onResetPasswordLink```
和他的朋友们就不再为您工作了！

**显示适当的UI并完成流程**

既然您知道用户正在尝试重设密码、设置初始密码或验证其电子邮件, 则应显示适当的 UI 以允许他们这样做。
例如, 您可能希望显示一个带有表单的页面, 以便用户输入他们的新密码。

当用户提交表单时, 您需要调用相应的函数将其更改提交到数据库。这些函数都持有新值和从上一步事件中获得的令牌(token)。

1.  [Accounts.resetPassword](http://docs.meteor.com/#/full/accounts_resetpassword) - 用于重置密码和注册新用户;它接受两种令牌。
2.  [Accounts.verifyEmail](http://docs.meteor.com/#/full/accounts_verifyemail)

在您调用上述两个函数之一或用户取消该过程后, 请调用在链接回调中得到的完成函数(done function)。这将告诉Meteor,当你正在做一个电子邮件帐户流程时，离开它进入的那个特殊状态。


### 自定义帐户电子邮件

您可能会想自定义电子邮件让```accounts-password```代您发送。这可以很容易地通过[Accounts.emailTemplates](http://docs.meteor.com/#/full/accounts_emailtemplates) API来完成。
下面是Todos应用程序中的一些示例代码:

``` 
Accounts.emailTemplates.siteName = "Meteor Guide Todos Example";
Accounts.emailTemplates.from = "Meteor Todos Accounts <accounts@example.com>";
Accounts.emailTemplates.resetPassword = {
  subject(user) {
    return "Reset your password on Meteor Todos";
  },
  text(user, url) {
    return `Hello!
Click the link below to reset your password on Meteor Todos.
${url}
If you didn't request this email, please ignore it.
Thanks,
The Meteor Todos team
`
  },
  html(user, url) {
    // This is where HTML email content would go.
    // See the section about html emails below.
  }
};
```
正如您所看到的, 我们可以使用 ES2015 模板字符串功能生成包含密码重置 URL 的多行字符串。我们也可以设置一个自定义的地址和电子邮件主题。

**HTML电子邮件**

如果你需要从一个应用程序发送漂亮的HTML电子邮件, 它可能很快成为一个噩梦。流行的电子邮件客户端与基本的HTML功能的兼容性(如CSS),
众所周知是参差不齐的, 所以很难做出普遍适应的工具来。
从[responsive email template](https://github.com/leemunroe/responsive-html-email-template)
和 [framework](http://foundation.zurb.com/emails/email-templates.html)开始,有人做了这方面的工作，使用工具将电子邮件内容转换成与所有电子邮件客户端兼容的东西。
这篇由Mailgun写的[博客帖子](http://blog.mailgun.com/transactional-html-email-templates/)涵盖了与HTML电子邮件相关的一些主要问题。
从理论上讲, 一个社区包可以扩展Meteor的构建系统来为您进行电子邮件编译, 但在编写本文时, 我们还没有发现有任何此类软件包。

##  OAuth登录

在遥远的过去, 让您的应用程序使用```Facebook```或```Google```用户登录系统, 可能是一个头痛的问题。值得庆幸的是, 
大多数受欢迎的登录提供商已经标准化了一些版本的 [OAuth](https://en.wikipedia.org/wiki/OAuth), 而Meteor支持一些最受欢迎的登录服务。

### Facebook, Google, 和更多登录提供商

这里有一个Meteor维护进核心包中的完整的登录提供者列表:

1.  Facebook 使用 ```accounts-facebook```
2.  Google 使用 ```accounts-google```
3.  GitHub 使用 ```accounts-github```
4.  Twitter 使用 ```accounts-twitter```
5.  Meetup 使用 ```accounts-meetup```
6.  Meteor Developer Accounts 使用 ```accounts-meteor-developer```

有一个使用微博登录的包, 但它不再积极维护了。（微信呢？）

### 登录

如果您使用的是现成的登录用户界面 (如```accounts-ui```或 ```useraccounts```), 则添加在上面的列表中相关的包后无需编写任何代码。
如果您从头开始构建登录体验, 则可以使用[Meteor.loginWith<Service>](http://docs.meteor.com/#/full/meteor_loginwithexternalservice)函数以编程方式登录。
它看起来像这样:

``` 
Meteor.loginWithFacebook({
  requestPermissions: ['user_friends', 'public_profile', 'email']
}, (err) => {
  if (err) {
    // handle error
  } else {
    // successful login!
  }
});
```

### 配置OAuth

关于配置 OAuth 登录, 有几点需要了解:

1.  **客户端ID和密码**。最好将您的OAuth密钥保留在源代码之外, 并通过 ```Meteor.settings``` 传递它们。
阅读[安全文章](https://guide.meteor.com/security.html#api-keys-oauth)中的内容。
2.  **重定向URL**。在OAuth提供者的一侧, 您需要指定一个重定向URL。该URL将看起来像```https://www.example.com/_oauth/facebook```。
用您正在使用的服务的名称替换```facebook```。请注意, 您需要配置两个 url, 一个用于生产环境，一个用于开发环境, 其url可能类似于```http://localhost:3000/_oauth/facebook```。
3.  **权限**。每个登录服务提供程序都应该有关于哪些权限可用的文档。例如, 这里是 [Facebook](https://developers.facebook.com/docs/facebook-login/permissions)的页面。
如果您希望用户的数据在登录时有更多权限, 请将这些字符串中的一些 通过```requestPermissions```选项传递给```Meteor.loginWithFacebook```
或 ```Accounts.ui.config```。在下一节中, 我们将讨论如何检索这些数据。

### 调用服务API以获取更多数据

如果您的应用程序支持或甚至需要使用外部服务(如Facebook)进行登录, 那么也很自然地要使用该服务的API来请求有关该用户的其他数据。
例如, 您可能希望得到一张Facebook用户照片的列表。

首先, 在用户登录时, 您需要请求相关的权限。有关如何传递这些选项的内容,请参见上面的部分。

然后, 您需要获取用户的访问令牌。您可以在的```Meteor.users```集合中的```services```字段下找到此令牌。
例如, 如果您想获取特定用户的 ```Facebook``` 访问令牌:

``` 
// Given a userId, get the user's Facebook access token
const user = Meteor.users.findOne(userId);
const fbAccessToken = user.services.facebook.accessToken;
```

有关存储在用户数据库中的数据的详细信息, 请阅读下面有关访问用户数据的部分。

既然您有了访问令牌, 您就需要对相应的 API 进行实际的请求。这里有两个选项:

1.  使用[http](http://docs.meteor.com/#/full/http)包直接访问服务的API。您可能需要在头(header)中传递上面的访问令牌。有关详细信息, 您需要搜索该服务的API文档。
2.  使用来自Atmosphere或npm的软件包, 将API包装成一个漂亮的JavaScript接口。例如, 如果您正在尝试从 Facebook 加载数据,
则可以使用 [fbgraph](https://www.npmjs.com/package/fbgraph) npm 包。
有关如何与应用程序一起使用 npm 的详细信息, 请参阅[构建系统文章](https://guide.meteor.com/build-tool.html#npm)。

##  加载和显示用户数据

正如```accounts-base```中的实现，Meteor的帐户系统, 也包括一个数据库集合和一些常用功能, 以获取有关用户的数据。

### 当前登录用户

一旦用户使用上述方法之一登录到您的应用程序中, 就能够识别登录的用户并获取在注册过程中提供的数据。

**在客户端：Meteor.userId()**

对于在客户端上运行的代码, 全局```Meteor.userId()```响应函数将为您提供当前登录用户的 ID。

除了核心API之外, 还有一些有用的速记帮助类(helpers), ```Meteor.user()```, 这是完全等同于调用```Meteor.users.findOne(Meteor.userId())``` 
和 ```{{currentUser}}```Blaze帮助器(helpers), 返回```Meteor.user()```的值。

请注意, 限制您访问当前用户的位置对使您的UI更具可测试和模块化是有好处的。在[UI文章](https://guide.meteor.com/ui-ux.html#global-stores)中阅读有关详细信息。

**在服务端：this.userId**

在服务器上, 每个连接都有不同的登录用户, 因此没有定义进行全局登录的用户状态。由于Meteor跟踪每个方法调用的环境, 您仍然可以使用 "Meteor.userId()" 全局变量, 
它根据您调用的方法返回一个不同的值, 使您在处理异步代码时遇到危险情况。此外,```Meteor.userId()```不能在发布(publications)中工作。

我们建议在方法和发布的上下文中使用 ```this.userId``` 属性, 并将其通过函数参数传递到任何需要它的地方。

``` 
// Accessing this.userId inside a publication
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

``` 
// Accessing this.userId inside a Method
Meteor.methods({
  'todos.updateText'({ todoId, newText }) {
    new SimpleSchema({
      todoId: { type: String },
      newText: { type: String }
    }).validate({ todoId, newText }),
    const todo = Todos.findOne(todoId);
    if (!todo.editableBy(this.userId)) {
      throw new Meteor.Error('todos.updateText.unauthorized',
        'Cannot edit todos in a private list that is not yours');
    }
    Todos.update(todoId, {
      $set: { text: newText }
    });
  }
});
```

### Meteor.users集合

Meteor为用户数据附带了一个默认的MongoDB集合。它存储在名为```users```的数据库中, 并且可以通过 ```Meteor.users``` 在您的代码中访问。
此集合中的用户文档的架构将取决于创建帐户的登录服务。以下是使用 ```accounts-password``` 创建其帐户的用户的示例:

``` 
{
  "_id": "DQnDpEag2kPevSdJY",
  "createdAt": "2015-12-10T22:34:17.610Z",
  "services": {
    "password": {
      "bcrypt": "XXX"
    },
    "resume": {
      "loginTokens": [
        {
          "when": "2015-12-10T22:34:17.615Z",
          "hashedToken": "XXX"
        }
      ]
    }
  },
  "emails": [
    {
      "address": "ada@lovelace.com",
      "verified": false
    }
  ]
}
```

下面是同一个用户使用Facebook登录时的样子:

``` 
{
  "_id": "Ap85ac4r6Xe3paeAh",
  "createdAt": "2015-12-10T22:29:46.854Z",
  "services": {
    "facebook": {
      "accessToken": "XXX",
      "expiresAt": 1454970581716,
      "id": "XXX",
      "email": "ada@lovelace.com",
      "name": "Ada Lovelace",
      "first_name": "Ada",
      "last_name": "Lovelace",
      "link": "https://www.facebook.com/app_scoped_user_id/XXX/",
      "gender": "female",
      "locale": "en_US",
      "age_range": {
        "min": 21
      }
    },
    "resume": {
      "loginTokens": [
        {
          "when": "2015-12-10T22:29:46.858Z",
          "hashedToken": "XXX"
        }
      ]
    }
  },
  "profile": {
    "name": "Sashko Stubailo"
  }
}
```

请注意, 当用户使用不同的登录服务注册时, 架构是不同的。在处理此集合时有几件事需要注意:

1.  数据库中的用户文档具有访问密钥和哈希密码等机密数据。将用户数据发布到客户端时, 要格外小心, 不要包含客户端不应该看到的任何内容。
2.  DDP，Meteor的数据发布协议, 只知道如何解决字段的冲突。这意味着当你的一个发布发送```services.facebook.first_name```
，另一个发布发送```services.facebook.locale```的时候，只有一个发布能成功, 只有一个字段在客户端上可用。
解决此问题的最佳方法是将所需的数据反规范化(冗余)到自定义等级字段中, 就如[自定义用户数据](#custom-user-data)一节中所描述的那样。
3.  OAuth登录服务包填充```profile.name```。我们不建议使用此方法, 但如果您打算这样做, 请确保拒绝客户端写入配置文件。
请参阅有关[profile field on users](https://guide.meteor.com/accounts.html#dont-use-profile)。
4.  通过电子邮件或用户名查找用户时, 请确保使用 "accounts-password" 提供的"不区分大小写"的功能。
有关详细信息, 请参阅关于 [case-sensitivity](https://guide.meteor.com/accounts.html#case-sensitivity) 的部分。

##  用户自定义数据

当您的应用程序变得复杂时, 您需要存储单个用户的有关数据, 并且将这些数据放在 "Meteor.users" 集合中的其他字段上。
在数据更加规范化的情况下, 将Meteor的用户数据和你的用户数据用两个单独的表保存起来可能更好。
这是由于MongoDB不能很好地处理数据关联, 所以只使用一个集合是有好处的。

### 在用户文档的顶级字段上添加字段

将自定义数据存储到```Meteor.users```集合中的最佳方法是在用户文档上添加一个新的唯一命名的顶级字段。例如, 如果您想向用户添加一个邮寄地址, 您可以这样做:

``` 
// Using address schema from schema.org
// https://schema.org/PostalAddress
const newMailingAddress = {
  addressCountry: 'US',
  addressLocality: 'Seattle',
  addressRegion: 'WA',
  postalCode: '98052',
  streetAddress: "20341 Whitworth Institute 405 N. Whitworth"
};
Meteor.users.update(userId, {
  $set: {
    mailingAddress: newMailingAddress
  }
});
```

您可以使用除[帐户系统使用](http://docs.meteor.com/api/accounts.html#Meteor-users)的其他字段名称。

### 在用户注册时添加字段

上面的代码可以运行在服务器内的Meteor方法里来设置某人的邮寄地址。有时, 您希望在用户首次创建其帐户时设置字段, 
例如初始化默认值或从其社交数据中计算某项。您可以使用```Accounts.onCreateUser```来执行此操作:

``` 
// Generate user initials after Facebook login
Accounts.onCreateUser((options, user) => {
  if (! user.services.facebook) {
    throw new Error('Expected login with Facebook only.');
  }
  const { first_name, last_name } = user.services.facebook;
  user.initials = first_name[0].toUpperCase() + last_name[0].toUpperCase();
  // We still want the default hook's 'profile' behavior.
  if (options.profile) {
    user.profile = options.profile;
  }
  
  // Don't forget to return the new user object at the end!
  return user;
});
```

请注意, 所提供的用户对象还没有_id 字段。如果您需要在该函数中使用新用户的id, 则可以使用一个有用的技巧来生成id:

``` 
// Generate a todo list for each new user
Accounts.onCreateUser((options, user) => {
  // Generate a user ID ourselves
  user._id = Random.id(); // Need to add the `random` package
  // Use the user ID we generated
  Lists.createListForUser(user._id);
  // Don't forget to return the new user object at the end!
  return user;
});
```

### 不使用"profile"

有一个诱人的现有字段称为"profile", 默认情况下在新用户注册时被添加。这个名字在历史上是用来作为一个"用户特定数据"的划痕, 也许是他们的形象头像、名称、介绍文本, 等等。
因此, **每个用户的profile字段都将由该用户从客户端自动写入**。它还会自动发布给该特定用户的客户端。

事实证明, 如果字段在默认情况下是可写的, 而不做限制, 可能并不是个好主意。
有许多新的Meteor开发者存储诸如isAdmin字段在profile中..., 然后一个恶意的用户可以很容易地将其设置为"true", 使自己具有管理权。
即使您不关心这一点, 也不要让恶意用户在您的数据库中随意存储数据。

与其处理这些字段的细节, 不如完全忽略它的存在。只要您执行下列操作，就可以安全地拒绝客户端的所有写入:

``` 
// Deny all client-side updates to user documents
Meteor.users.deny({
  update() { return true; }
});
```

即使不管profile字段的安全性含义, 将所有应用程序的自定义数据放到一个字段中也不是一个好主意。正如在集合文章中所讨论的那样, 
Meteor的数据传输协议不会对字段进行深入的嵌套比较, 因此最好将对象拼合到文档中的多个顶级字段中。

### 发布自定义数据

如果您想访问您在UI中添加到```Meteor.users```中的自定义数据, 则需要将其发布到客户端。大多数情况下, 您只需遵循数据加载和安全文章中的建议即可。

要记住的最重要的一点是, 用户文档包含有关用户的私有数据。特别是, 用户文档包括哈希密码数据和外部api的访问密钥。这意味着, 筛选发送到任何客户端的用户文档的字段至关重要。

请注意, 在Meteor的发布和订阅系统中, 用不同的字段多次发布同一文档是完全可以的——它们将在内部合并, 客户端将看到一个所有字段合并在一起的一致文档。
因此, 如果您只是添加了一个自定义字段, 则只需编写一个具有该字段的发布。让我们看一个示例, 说明如何发布```initials```字段:

此发布将允许客户端传递它所感兴趣的用户 id 数组, 并获取所有这些用户的缩写。

##  角色和权限

您将登录系统添加到应用程序的主要原因之一是可能希望对您的数据具有权限。例如, 如果您正在运行一个论坛, 您希望管理员或版主能够删除任何帖子, 但普通用户只能删除自己的。这将揭示两种不同类型的权限:

1.  基于角色的权限
2.  基于数据的权限（基于文档的权限）

### alanning:roles

在Meteor中最流行的基于角色的权限包是```alanning:roles```。例如, 下面是如何使用户成为管理员或主持人:

``` 
// Give Alice the 'admin' role
Roles.addUsersToRoles(aliceUserId, 'admin', Roles.GLOBAL_GROUP);
// Give Bob the 'moderator' role for a particular category
Roles.addUsersToRoles(bobsUserId, 'moderator', categoryId);
```

现在, 假设您想检查某人是否被允许删除特定的论坛帖子:

``` 
const forumPost = Posts.findOne(postId);
const canDelete = Roles.userIsInRole(userId,
  ['admin', 'moderator'], forumPost.categoryId);
if (! canDelete) {
  throw new Meteor.Error('unauthorized',
    'Only admins and moderators can delete posts.');
}
Posts.remove(postId);
```

请注意, 我们可以一次检查多个角色, 如果某人在 GLOBAL_GROUP 中有一个角色, 那么他们被认为在每个组中都具有该角色。在这种情况下, 组是按类别ID, 但您可以使用任何唯一的标识符来组成一个组。

阅读[alanning:roles package documentation](https://atmospherejs.com/alanning/roles)了解更多信息。

### 基于文档的权限

有时, 将权限抽象为 "组" 是没有意义的——您只是希望文档拥有所有者。在这种情况下, 您可以使用集合帮助器进行更简单的策略。

``` 
Lists.helpers({
  // ...
  editableBy(userId) {
    if (!this.userId) {
      return true;
    }
    return this.userId === userId;
  },
  // ...
});
```

现在, 我们可以调用这个简单的函数来确定是否允许某个特定的用户编辑此列表:

``` 
const list = Lists.findOne(listId);
if (! list.editableBy(userId)) {
  throw new Meteor.Error('unauthorized',
    'Only list owners can edit private lists.');
}
```

了解有关如何在集合文章中使用集合帮助(helpers)的[更多信息](https://guide.meteor.com/collections.html#collection-helpers)。

