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

下面是在调用方法时，事情发生的顺序:



