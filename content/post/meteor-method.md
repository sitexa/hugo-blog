+++
title = "Meteor(5):方法(Methods)"
description = "How to use Methods, Meteor's remote procedure call system, to write to the database."
tags = [
    "JavaScript",
    "NodeJs",
    "Web application",
    "Mobile application",
    "Platform",
    "MongoDB",
]
date = "2017-11-07"
categories = [
    "Development",
    "nodejs",
]
thumbnail = "images/meteor-guide/meteor-method.png"
+++

**Meteor的远程程序调用系统, 如何使用方法写入数据库。**

1.  在Meteor中什么是方法，他们是如何工作的。
2.  定义和调用方法的最佳做法。
3.  如何使用方法抛出和处理错误。
4.  如何从有单中调用方法。

<!--more-->

##  什么是方法

方法是Meteor的远程过程调用 (RPC) 系统, 用于保存用户输入事件和来自客户端的数据。如果您熟悉 REST api 或 HTTP, 
您可以将它们看作是对服务器的 POST 请求, 但有许多为构建现代 web 应用程序而优化的好功能。稍后在本文中, 
我们将详细介绍从 HTTP 端点中无法获得的一些方法带来的好处。

在Meteor核心, 方法是您的服务器的 API 端点;您可以在服务器及其对应的客户端上定义方法, 然后用一些数据调用它, 向数据库中写入, 并在回调中获取返回值。
Meteor的方法紧密结合了"发布/订阅"和Meteor的数据加载系统, 以允许[Optimistic UI](http://info.meteor.com/blog/optimistic-ui-with-meteor-latency-compensation)--
在客户端上模拟服务器端操作的能力, 使您的应用程序感觉比实际更快。

我们将提到的陨石方法加上前缀M, 以区分JavaScrip的类方法。

##  定义和调用方法

### 基本方法

在一个基本的应用中, 定义Meteor方法就像定义一个函数一样简单。在一个复杂的应用程序中, 您需要一些额外的功能, 使方法更强大和易于测试。
首先, 我们将探讨如何使用Meteor核心 API 来定义方法, 在后面的小节中, 我们将详细说明如何使用我们创建的有用的包装包来启用更强大的方法工作流。

**定义**

这里是介绍如何使用内置的[Meteor.methods](http://docs.meteor.com/#/full/meteor_methods) API 定义方法。请注意, 
应始终在客户端和服务器上加载的公共代码中定义方法, 以启用```Optimistic UI```。如果您的方法中有一些秘密代码, 请参阅[安全文章](https://guide.meteor.com/security.html#secret-code),
了解如何将其隐藏在客户端中。

本示例使用```aldeed:simple-schema```包 (在其他几篇文章中推荐) 来验证方法参数。

``` 
Meteor.methods({
  'todos.updateText'({ todoId, newText }) {
    new SimpleSchema({
      todoId: { type: String },
      newText: { type: String }
    }).validate({ todoId, newText });
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

**调用**

此方法可通过```Meteor.call```从客户端和服务器调用。请注意, 在需要从客户端调用某些代码的情况下, 只应使用方法。如果只想将只从服务器调用的代码模块化, 请使用常规 JavaScript 函数, 而不是方法。

以下是如何从客户端调用此方法:

``` 
Meteor.call('todos.updateText', {
  todoId: '12345',
  newText: 'This is a todo item.'
}, (err, res) => {
  if (err) {
    alert(err);
  } else {
    // success!
  }
});
```
如果该方法引发错误, 则会在回调的第一个参数中得到它。如果该方法成功, 则在第二个参数中得到结果, 并且第一个参数 err 将是 undefined。
有关错误的详细信息, 请参阅下面有关错误处理的部分。

### 高级方法样板

Meteor的方法有几个不太明显的功能, 但每个复杂的应用程序有时都需要它们。这些功能是以向后兼容的方式在数年内逐步增加的, 
因此解密这些方法的全部功能将需要大量的样板。在本文中, 我们将首先向您展示您需要为每个功能编写的所有代码,
然后下一节将讨论我们开发的方法包装程序包, 以使其更容易使用。

下面是一个理想方法的一些功能:

1.  运行验证代码本身而不运行方法体。
2.  轻松地重写测试方法。
3.  使用 ```custom user ID``` 轻松调用该方法, 尤其是在测试中 (如[Discover Meteor two-tiered methods pattern](https://www.discovermeteor.com/blog/meteor-pattern-two-tiered-methods/)所建议的)。
4.  通过JS模块, 而不是```神奇的字符串```来引用该方法。
5.  获取方法模拟返回值以获取插入文档的 id。
6.  如果客户端验证失败, 避免调用服务器端方法, 因此我们不会浪费服务器资源。

**定义**

``` 
export const updateText = {
  name: 'todos.updateText',
  // Factor out validation so that it can be run independently (1)
  validate(args) {
    new SimpleSchema({
      todoId: { type: String },
      newText: { type: String }
    }).validate(args)
  },
  // Factor out Method body so that it can be called independently (3)
  run({ todoId, newText }) {
    const todo = Todos.findOne(todoId);
    if (!todo.editableBy(this.userId)) {
      throw new Meteor.Error('todos.updateText.unauthorized',
        'Cannot edit todos in a private list that is not yours');
    }
    Todos.update(todoId, {
      $set: { text: newText }
    });
  },
  // Call Method by referencing the JS object (4)
  // Also, this lets us specify Meteor.apply options once in
  // the Method implementation, rather than requiring the caller
  // to specify it at the call site.
  call(args, callback) {
    const options = {
      returnStubValue: true,     // (5)
      throwStubExceptions: true  // (6)
    }
    Meteor.apply(this.name, [args], options, callback);
  }
};
// Actually register the method with Meteor's DDP system
Meteor.methods({
  [updateText.name]: function (args) {
    updateText.validate.call(this, args);
    updateText.run.call(this, args);
  }
})
```

**调用**

现在调用该方法就像调用 JavaScript 函数一样简单:

``` 
import { updateText } from './path/to/methods.js';
// Call the Method
updateText.call({
  todoId: '12345',
  newText: 'This is a todo item.'
}, (err, res) => {
  if (err) {
    alert(err);
  } else {
    // success!
  }
});
// Call the validation only
updateText.validate({ wrong: 'args'});
// Call the Method with custom userId in a test
updateText.run.call({ userId: 'abcd' }, {
  todoId: '12345',
  newText: 'This is a todo item.'
});
```

正如您所看到的, 这种调用方法的方法会产生更好的开发工作流-您可以更轻松地单独处理方法的不同部分, 更容易地测试您的代码, 
而不必处理Meteor内部逻辑。但是, 这种方法要求您在方法定义方面编写大量的样板。

### 通过mdg:validated-method使用高级方法

为了缓解一些在正确的方法定义中涉及的样板, 我们已经发布了一个称为 ```mdg:validated-method``` 的包装包, 它为您做了大部分工作。
这里的方法与上面的相同, 但用包定义:

``` 
import { ValidatedMethod } from 'meteor/mdg:validated-method';
export const updateText = new ValidatedMethod({
  name: 'todos.updateText',
  validate: new SimpleSchema({
    todoId: { type: String },
    newText: { type: String }
  }).validator(),
  run({ todoId, newText }) {
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

您调用它的方法与调用上面的高级方法相同, 但方法定义非常简单。我们相信这种方法可以让您清楚地看到重要的部分-通过在线发送的方法的名称、
预期参数的格式以及可以引用该方法的 JavaScript 命名空间。验证的方法只接受单个参数和一个回调函数。

##  错误处理

在常规 JavaScript 函数中, 通过抛出错误对象来指示错误。从Meteor中掷出错误方法差不多与此相同, 但有点复杂的是, 在某些情况下, 
错误对象将通过websocket发送到客户端。

### 从方法中抛出错误

Meteor引入了两种新的JavaScript错误: ```Meteor.Error``` 和 ```ValidationError```。这些错误类型和常规JavaScript错误类型应在不同的情况下使用:

**内部服务器错误作为常规Error处理**

当您有一个不需要向客户端报告但却是服务器内部的错误时, 抛出一个常规的 JavaScript 错误对象。这将作为完全不透明的内部服务器错误报告给客户端, 没有详细信息。

**常规运行时错误作为Meteor.Error处理**

当条件已知,而服务器无法完成用户所需的操作时, 你应该抛出一个描述性的Meteor.Error对象给客户端。在Todos示例应用程序中,
我们使用这些方法来报告当前用户无权完成某项操作的情况, 或者在应用程序中不允许执行此操作的情况(例如, 删除最后一个公共列表)。

Meteor.Error接受三个参数: error、reason 和 details。

1.  Error应该是一个简短的、唯一的、机器可读的错误代码字符串, 客户端可以解释它以了解发生了什么。最好用方法的名称做前缀, 
以方便国际化, 例如: ```todos.updateText.unauthorized```。
2.  Reason应该是对开发人员错误的简短描述。它应该给你的同事足够的信息, 以便能够调试错误。原因参数不应直接打印到最终用户, 
因为这意味着您现在必须在发送错误消息之前在服务器上进行国际化, 而UI开发人员也必须考虑将其显示在UI中。
3.  Details是可选的, 可以在需要额外数据帮助客户了解错误的地方使用。特别是, 它可以与 "Error" 字段组合, 以便向最终用户打印更有用的错误消息。

**参数校验错误将作为ValidationError处理**

因为参数的类型不正确而导致方法调用失败时, 最好抛出一个 ```ValidationError```。这就像```Meteor.Error```一样, 
但它是一个自定义构造函数, 它强制执行可由不同表单和验证库读取的标准错误格式。特别是, 如果在表单中调用此方法, 
则抛出```ValidationError``` 将很容易在表单中的特定字段旁边显示错误信息。

在上面的演示中，当您使用```mdg:validated-method```与```aldeed:simple-schema```时, 将为您抛出这种类型的错误(ValidationError)。

有关 [mdg:validation-error](https://atmospherejs.com/mdg/validation-error) 的错误格式的详细信息, 请参阅他的文档。

### 处理错误

调用方法时, 它引发的任何错误都将在回调中返回。此时, 应确定它是哪个错误类型, 并向用户显示相应的消息。在这种情况下, 该方法不太可能引发 ValidationError 或内部服务器错误, 因此我们将只处理未经授权的错误:

``` 
// Call the Method
updateText.call({
  todoId: '12345',
  newText: 'This is a todo item.'
}, (err, res) => {
  if (err) {
    if (err.error === 'todos.updateText.unauthorized') {
      // Displaying an alert is probably not what you would do in
      // a real app; you should have some nice UI to display this
      // error, and probably use an i18n library to generate the
      // message from the error code.
      alert('You aren\'t allowed to edit this todo item');
    } else {
      // Unexpected error, handle it in the UI somehow
    }
  } else {
    // success!
  }
});
```

我们将在下面的窗体部分中讨论如何处理```ValidationError```。

### 方法模拟中的错误

当调用某个方法时, 它通常在客户端上运行两次, 以模拟 "Optimistic UI" 的结果, 并再次在服务器上对数据库进行实际更改。
这意味着如果您的方法抛出错误, 它可能会在客户端和服务器上失败。因此, ```ValidatedMethod``` 在Meteor中打开未记录的选项, 以避免在模拟抛出错误时调用服务器端实现。

虽然这种行为对于在某种方法肯定会失败的情况下保存服务器资源很有好处, 但在服务器方法成功的情况下, 确保模拟不会引发错误 
(例如, 如果您没有在该方法需要正确进行模拟的客户端加载一些数据)。在这种情况下, 您可以将 "仅服务器端执行" 逻辑包含在检查方法模拟的块中:

``` 
if (!this.isSimulation) {
  // Logic that depends on server environment here
}
```

##  从表单中调用方法

```ValidationError```约定所启用的主要功能是方法和调用它们的表单之间的简单集成。通常, 您的应用程序可能会在 UI 中对方法进行一对一的表单映射。
首先, 让我们为业务逻辑定义一个方法:

``` 
// This Method encodes the form validation requirements.
// By defining them in the Method, we do client and server-side
// validation in one place.
export const insert = new ValidatedMethod({
  name: 'Invoices.methods.insert',
  validate: new SimpleSchema({
    email: { type: String, regEx: SimpleSchema.RegEx.Email },
    description: { type: String, min: 5 },
    amount: { type: String, regEx: /^\d*\.(\d\d)?$/ }
  }).validator(),
  run(newInvoice) {
    // In here, we can be sure that the newInvoice argument is
    // validated.
    if (!this.userId) {
      throw new Meteor.Error('Invoices.methods.insert.not-logged-in',
        'Must be logged in to create an invoice.');
    }
    Invoices.insert(newInvoice)
  }
});
```

让我们定义一个简单的 HTML 表单:

``` 
<template name="Invoices_newInvoice">
  <form class="Invoices_newInvoice">
    <label for="email">Recipient email</label>
    <input type="email" name="email" />
    {{#each error in errors "email"}}
      <div class="form-error">{{error}}</div>
    {{/each}}
    <label for="description">Item description</label>
    <input type="text" name="description" />
    {{#each error in errors "description"}}
      <div class="form-error">{{error}}</div>
    {{/each}}
    <label for="amount">Amount owed</label>
    <input type="text" name="amount" />
    {{#each error in errors "amount"}}
      <div class="form-error">{{error}}</div>
    {{/each}}
  </form>
</template>
```

现在, 让我们编写一些 JavaScript 来处理这个表单:

``` 
import { insert } from '../api/invoices/methods.js';
Template.Invoices_newInvoice.onCreated(function() {
  this.errors = new ReactiveDict();
});
Template.Invoices_newInvoice.helpers({
  errors(fieldName) {
    return Template.instance().errors.get(fieldName);
  }
});
Template.Invoices_newInvoice.events({
  'submit .Invoices_newInvoice'(event, instance) {
    const data = {
      email: event.target.email.value,
      description: event.target.description.value,
      amount: event.target.amount.value
    };
    insert.call(data, (err, res) => {
      if (err) {
        if (err.error === 'validation-error') {
          // Initialize error object
          const errors = {
            email: [],
            description: [],
            amount: []
          };
          // Go through validation errors returned from Method
          err.details.forEach((fieldError) => {
            // XXX i18n
            errors[fieldError.name].push(fieldError.type);
          });
          // Update ReactiveDict, errors will show up in the UI
          instance.errors.set(errors);
        }
      }
    });
  }
});
```

正如您所看到的, 有相当数量的样板可以很好地在表单中处理错误, 但大多数都可以通过现成的表单框架或您自己设计的简单应用程序特定包装来轻松地抽象出来。

### 通过表单加载数据

由于方法可以作为通用的rpc使用, 所以它们也可以用来获取数据而不仅是发布(publication)。与通过发布(publication)加载数据相比, 
这种方法有一些优点和缺点, 我们建议您始终使用发布(publication)来加载数据。

方法可以帮助您从服务器上获取复杂计算的结果, 而不需要在服务器数据发生更改时进行更新。通过方法获取数据的最大缺点是, 
数据不会自动加载到 ```Minimongo``` (Meteor的客户端数据缓存) 中, 因此您需要手动管理该数据的生命周期。

**使用本地集合存储和显示从方法获取的数据**

集合是在客户端存储数据的一种非常方便的方法。如果您使用的是订阅以外的办法来获取数据, 则可以手动将其放入集合中。
让我们看一个例子, 我们有一个复杂的算法，从一系列的游戏中为一些球员计算平均分数。我们不希望使用发布来加载此数据, 
因为我们希望精确地控制它的运行时间, 并且不希望自动缓存数据。

首先, 您需要创建一个本地集合-这是一个仅在客户端上存在且不与服务器上的数据库集合绑定的集合。
在[集合文章中](http://guide.meteor.com/collections.html#local-collections)阅读更多内容。

``` 
// In client-side code, declare a local collection
// by passing `null` as the argument
ScoreAverages = new Mongo.Collection(null);
```

现在, 如果使用方法获取数据, 则可以将其放入此集合中:

``` 
import { calculateAverages } from '../api/games/methods.js';
function updateAverages() {
  // Clean out result cache
  ScoreAverages.remove({});
  // Call a Method that does an expensive computation
  calculateAverages.call((err, res) => {
    res.forEach((item) => {
      ScoreAverages.insert(item);
    });
  });
}
```

现在, 我们可以使用一个 UI 组件中的本地集合 ```ScoreAverages``` 中的数据, 与使用常规 MongoDB 集合的方式完全相同。
每次我们需要新的结果时，我们需要调用 ```updateAverages```，而不是自动更新。

##  高级概念

虽然您可以遵循流星入门教程,在简单的应用程序中轻松地使用方法(Method), 但是，了解他们是如何工作的, 才能在一个生产应用程序中更有效地使用它们。这是很重要的。
使用像Meteor这样的框架的一个缺点是，它为你做了很多事情, 你却并不是总能理解发生了什么, 所以学习一些核心概念是很好的。

### 方法调用生命周期

下面是在调用方法时，事情发生的先后顺序:

**1. 方法模拟在客户端运行**

如果我们在客户端和服务器代码中定义了此方法, 就像所有方法一样, 在调用它的客户机上执行方法模拟。

客户端进入一个特殊模式, 它跟踪对客户端集合所做的所有更改, 以便以后可以回滚。当这个步骤完成后, 
用户看到他们的UI被客户端数据库的新内容立即更新, 但服务器还没有收到任何数据。

如果从方法模拟中抛出异常 , 则默认情况Meteor会忽略它并继续执行步骤(2)。如果您使用的是 ```ValidatedMethod```
或通过一个特殊的 ```throwStubExceptions``` 选项 ```Meteor.apply```, 则从模拟中抛出的异常将停止服务器端方法的运行。

除非在调用方法时传递了 ```returnStubValue``` 选项, 否则将丢弃方法模拟的返回值, 在这种情况下, 它将返回给方法调用方。
默认情况下, ```ValidatedMethod``` 传递此选项。

**2. 方法将 DDP 消息发送到服务器**

流星客户端构造一个 DDP 消息发送到服务器。参数包括方法名称、参数以及表示此特定方法调用的自动生成的方法ID。

**3. 方法在服务器上运行**

当服务器收到消息时, 它将在服务器上再次执行方法代码。客户端版本是一个模拟, 将被回滚, 但这次是真实的版本, 是写到实际的数据库。
在服务器上运行实际的方法逻辑是非常重要的, 因为服务器是一个受信任的环境, 我们知道**安全关键**(security-critical)代码将按预期方式运行。

**4. 返回值被发送到客户端**

一旦该方法在服务器上运行完毕, 它就会向客户端发送一个结果消息, 以及在步骤2中生成的方法ID和返回值本身。客户端存储此项以备以后使用, 
但尚未调用方法回调。如果你把 ```onResultReceived``` 的选项传给```Meteor.apply```, 将触发该回调。

**5. 任何受该方法影响的DDP发布都将更新**

如果有页面上的发布都受到从该方法写入的数据库的影响, 则服务器会将相应的更新发送到客户端。请注意, 在下一步执行之前, 客户端数据系统不会向应用程序 UI 显示这些更新。

**6. 把更新消发送到客户端息, 用服务器结果替换数据, 触发方法回调**

将相关的数据更新发送到正确的客户端后, 服务器将返回方法生命周期中的最后一条消息——即相关方法ID的DDP更新消息。
客户端回滚在步骤1中的方法模拟中对客户端数据所做的任何更改, 并将其替换为步骤5中从服务器发送的实际更改。

最后, 回调传递到 ```Meteor.call```，实际上是从步骤4的返回值触发的。重要的是, 回调动作一直等到客户端是最新状态, 
这样您的方法回调就可以假定客户端状态反映了在该方法内所做的任何更改。

**错误案例**

在上面的列表中, 我们没有涉及到服务器上的方法执行引发错误的情况。在这种情况下, 没有返回值, 而客户端会得到一个错误。
将返回的错误作为第一个参数立即触发方法回调。错误处理的详细信息, 请参阅以下有关错误处理的部分。

### REST方法的好处

我们认为, "Methods" 为构建现代应用程序提供了比构建在 HTTP 上的 REST 端点更好的原始功能。
让我们来看看一些你使用"Methods"得到的东西, 你将不得不担心如果用HTTP会怎么样。本节的目的不是要说服你REST是坏的——它只是提醒你, 
你不需要在Meteor应用程序处理这些事情。

**方法使用同步模式的api, 但它是非阻塞的**

您可能注意到在上面的示例方法中, 我们不需要在与 MongoDB 交互时编写任何回调, 但是该方法仍然具有人们与 "Node.js" 
和回调样式代码关联的非阻塞属性。Meteor使用一个称为 [Fibres](https://github.com/laverdet/node-fibers) 的协同库, 
使您能够编写代码，使用返回值和抛出错误, 并避免处理大量嵌套回调。

**方法总是按顺序运行并返回**

当访问 ```REST API``` 时, 您有时会遇到一种情况, 即您按顺序发出两个请求, 但结果却是顺序不确定。
Meteor的底层机制确保这种情况不会发生。当从同一个客户端接收到多个方法调用时, Meteor运行完一个方法后才运行下一个。
如果您需要为一个特别长时间运行的方法禁用此功能, 则可以使用 ```this.unblock()``` 以允许在当前一个方法仍在进行时运行下一方式。
此外, 由于Meteor是基于```Websockets```而不是 ```HTTP```, 所有的方法调用和结果保证按他们发送的顺序到达。
你也可以通过一个特殊的选项```wait:true```给```Meteor.apply```以等待发送特定的"方法",直到所有其他方法返回, 而不是在这一方法返回后再发送任何其他方法。

**```Optimistic UI```的更改跟踪**

当方法模拟和服务器端程序运行时, Meteor会跟踪对数据库所产生的任何更改。这就是原因，能够让Meteor数据系统从方法模拟的变化回滚, 并代之以从服务器来的实际数据写入。
如果没有此自动数据库跟踪, 则很难实现正确的```Optimistic UI```系统。

### 从其他方法调用方法

有时, 您需要从其他方法调用方法。也许您已经实现了一些功能, 并且您希望添加一个自动填充某些参数的包装。
这是一个完全精细的模式, Meteor为你做了一些事:

在客户端方法模拟中, 调用另一种方法不会触发对服务器的额外请求——假定该方法的服务器端实现将执行此过程。但是, 它确实运行了被调用方法的模拟, 
因此客户端上的模拟将与服务器上发生的情况紧密匹配。

在服务器上的方法执行中, 调用另一个方法将运行该方法, 就好像它是由同一客户端调用一样。这意味着该方法照常运行, 
而上下文——userID、connection等 —— 取自原始方法调用。

