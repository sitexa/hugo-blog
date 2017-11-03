+++
title = "Meteor(3):数据集(Collections)和架构(Schemas)"
description = "title:数据集(Collections)和架构(Schemas)"
tags = [
    "JavaScript",
    "NodeJs",
    "Web application",
    "Mobile application",
    "Platform",
    "MongoDB",
]
date = "2017-11-03"
categories = [
    "Development",
    "nodejs",
]
thumbnail = "images/mongo-db.png"
+++

本章介绍如何在Meteor中定义、使用和维护MongoDB的数据集。

<!--more-->

通过本章的学习，你会了解：

1.  Meteor中不同类型的 MongoDB 数据集(Collection), 以及如何使用它们。
2.  如何定义数据集(Collection)的架构(Schema)以控制其内容。
3.  定义数据集(Collection)的架构(Schemas)时应考虑的事项。
4.  如何在写入数据集(Collection)时确保其架构(Schema)正确。
5.  如何更改数据集(Collection)的架构(Schema)。
6.  如何处理记录(Record)之间的关联(Association)。

##  Meteor中的MongoDB数据集(Collection)

web应用程序为用户提供了观察数据的视图, 一种修改数据的方法。无论是管理Todos的任务列表, 还是提交一个滴滴订单, 
您都在与一个持久但不断变化的数据层进行交互。

在Meteor中, 该数据层通常存储在 MongoDB 中。MongoDB 中的一组相关数据称为 "数据集(Collection)"。
在Meteor中, 您可以通过[collections](http://docs.meteor.com/api/collections.html#Mongo-Collection)访问 MongoDB。MongoDB数据集是应用程序数据的主要存储机制。

但是, 数据集不仅仅是保存和检索数据的一种方法。它们还提供了用户希望从最佳实践中获得的交互式、用户体验的核心功能。
Meteor使这种用户体验易于实现。

在本文中, 我们将仔细研究数据集在不同场景下的工作方式, 以及如何从中获得帮助。

### 服务端数据集(Server-side Collections)

当你在服务器端创建一个数据集：

``` 
Todos = new Mongo.Collection('todos');
```

你就创建了一个MongoDB数据集，和一个使用数据集的接口。这是一个相当简单的层，它基于MongoDB的node.js driver,是一个同步的API:

``` 
// This line won't complete until the insert is done
Todos.insert({_id: 'my-todo'});
// So this line will return something
const todo = Todos.findOne({_id: 'my-todo'});
// Look ma, no callbacks!
console.log(todo);
```

### 客户端数据集(Client-side Collections)

在客户端，你写相同一行程序：

``` 
Todos = new Mongo.Collection('todos');
```

做的是完全不同的事情！

在客户端上, 没有与 MongoDB 数据库的直接连接, 事实上, 对数据库的同步 API 是不可能的 (也可能不是您想要的)。
相反, 在客户端, 数据集是数据库的客户端缓存(cache)。这是通过 [Minimongo](https://github.com/meteor/meteor/blob/master/packages/minimongo/README.md) 库实现的。Minimongo 库是一种全JS实现的、
运行于内存中的MongoDB API实现。

这意味着在客户端上, 当您编写下面的代码时:

``` 
// This line is changing an in-memory Minimongo data structure
Todos.insert({_id: 'my-todo'});
// And this line is querying it
const todo = Todos.findOne({_id: 'my-todo'});
// So this happens right away!
console.log(todo);
```

将数据从服务器(MongoDB支撑)集合移动到客户端(内存中支撑)集合的方式是[数据加载文章](https://guide.meteor.com/data-loading.html)的主题。
一般来说, 您订阅(subscribe)的是一个发布(publication), 它将数据从服务器推送到客户端。
通常, 您可以假定客户端包含完整 MongoDB 数据集的某些子集的最新副本。

若要将数据写回服务器, 请使用方法(Method) ([方法主题文章](https://guide.meteor.com/methods.html))。

### 本地数据集(Local Collections)

有第三种方法可以在Meteor中使用数据集。在客户端或服务器上, 如果用以下两种方式之一创建数据集:

``` 
SelectedTodos = new Mongo.Collection(null);
SelectedTodos = new Mongo.Collection('selectedtodos', {connection: null});
```

这将创建一个本地数据集。这是一个没有数据库连接的 Minimongo 数据集 (通常, 数据集要么直接连接到服务器上的数据库, 要么通过客户端上的订阅来访问服务器数据库)。

本地数据集是使用 Minimongo 库的强大能力进行内存存储的一种简便方法。例如, 如果需要对数据执行复杂查询, 则可以使用它而不是简单的数组。
或者你可能想利用本地数据集在客户端的**反应式能力(reactivity)**, 以一种在Meteor中感觉很自然的方式来驱动UI。

##  定义架构(Schema)

虽然 MongoDB 是一个无模式(schema-less)的数据库, 它允许在数据结构中实现最大的灵活性, 但通常良好的做法是使用架构来约束集合的内容, 使其符合已知的格式。
如果不这样做, 那么您往往需要编写验证代码来检查和确认数据的结构, 因为数据集是从数据库中出来, 而不是进入数据库。
在大多数情况下, 读数据比写数据的情况更多，因此在编写时使用模式通常更简单，错误更少。

在Meteor中, [aldeed:simple-schema](https://atmospherejs.com/aldeed/simple-schema)是很优秀的架构包。
它是一种表达能力强大、基于 MongoDB 的架构, 用于插入和更新文档。另一种选择是[jagi:astronomy](https://atmospherejs.com/jagi/astronomy), 
它是一个完整的对象模型 (OM) 层, 提供架构定义、服务器/客户端验证器、对象方法和事件处理程序。

假设我们有一个```Lists```数据集。若要使用```simple-schema```为该数据集定义架构, 可以简单地创建 SimpleSchema 类的实例并将其附加到```Lists```对象中:

``` 
Lists.schema = new SimpleSchema({
  name: {type: String},
  incompleteCount: {type: Number, defaultValue: 0},
  userId: {type: String, regEx: SimpleSchema.RegEx.Id, optional: true}
});
```

Todos应用程序的这个例子定义了一个架构, 其中有几个简单的规则:

1.  我们指定列表的 name 字段是必需的, 并且必须是字符串。
2.  我们指定 incompleteCount 是一个数字, 如果没有另外指定, 则在插入时将其设置为0。
3.  我们指定用户 id 是可选的, 它必须是一个字符串, 它看起来就像一个使用者文档的标识。

我们直接将架构附加到```Lists```的命名空间, 这方便我们可以在需要时直接检查对象, 例如在窗体或方法([Method](https://guide.meteor.com/methods.html))中。
在下一节中, 我们将了解如何在写入集合时自动使用此架构。

您可以看到, 使用很少的代码, 我们已经成功地限制了列表的格式。您可以阅读有关简单模式文档([Simple Schema docs](http://atmospherejs.com/aldeed/simple-schema))中的架构所能完成的更复杂的事情。

### 针对架构进行验证

现在我们有了一个模式, 我们如何使用它？

使用架构来验证文档非常简单。我们可以写:

``` 
const list = {
  name: 'My list',
  incompleteCount: 3
};
Lists.schema.validate(list);
```

在这种情况下, 由于该列表根据架构是有效的, 因此```validate()```行运行时不会出现问题。然而, 我们写到:

``` 
const list = {
  name: 'My list',
  incompleteCount: 3,
  madeUpField: 'this should not be here'
};
Lists.schema.validate(list);
```

这时, ```validate()```调用将抛出一个 ```ValidationError```, 其中包含有关```list```文档的错误的详细信息。

### ValidationError

什么是 ```ValidationError```？这是一个Meteor使用的特殊的错误, 以指示用户在修改集合时有输入错误。
通常, ```ValidationError``` 上的详细信息用于标记表单, 其中有有关输入与架构不匹配的内容。
在方法([Methods](https://guide.meteor.com/methods.html#validation-error))文章中, 我们将看到更多有关如何工作的信息。


##  定义数据架构

现在您已经熟悉了```Simple Schema```的基本 API, 因此值得考虑一些可能影响数据架构设计的Meteor数据系统的约束。
虽然一般来说, 你可以像建立任何 MongoDB 的数据模式一样建立一个Meteor数据模式, 但仍然有一些重要的细节要牢记在心。

最重要的考虑是有关Meteor的数据加载协议：DDP，在线文档通讯协议。该协议实现的关键是, DDP 在顶极文档字段级别发送对文档的更改。
这意味着, 如果你有大的和复杂的字段(fields)在文件上经常变化, DDP将发送不必要的变更信息。

例如, 在 "纯" MongoDB 中, 您可以设计如下架构, 以便每个列表文档都有一个称为```Todos```的字段, 它是一个 ```todo``` 项目的数组:

``` 
Lists.schema = new SimpleSchema({
  name: {type: String},
  todos: {type: [Object]}
});
```

此模式的问题是, 由于刚才提到的 DDP 行为, 每个对列表中的 todo 项的更改都需要通过网络发送该列表的整个Todos集合。
这是因为 DDP 没有 "更改字段中的第三项叫Todos的文本字段", 而是 "将Todos字段改变成一个全新的数组"。

###  规范化和多个集合

上面的含义是, 我们需要创建更多的**数据集**来包含子文档(sub-documents)。在Todos应用程序中, 
我们需要一个```Lists```数据集和一个```Todos```数据集来包含每个列表的 todo 项。因此, 我们需要做一些通常与 SQL 数据库关联的事情, 
比如使用外键 (todo. listId) 将一个文档与另一个文件相关联。

在Meteor里，与一个典型的 MongoDB 应用程序不一样，这往往出问题, 因为它很容易发布重叠的文件集 (我们可能需要一组用户来渲染一个屏幕, 而跟别的功能发生交叉影响), 
当我们在应用程序中移动时, 这组用户数据可能留在客户端上。因此, 在这种情况下, 将子文档与父文档分开是有好处的。

但是, 考虑到版本3.2 之前的 MongoDB 不支持对多个集合 ("join") 的查询, 因此我们通常不得不将一些数据重新规范化到父集合。
规范化是在数据库中多次存储同一信息的做法 (相对于非冗余 "正常" 形式)。MongoDB 提倡规范化的数据库, 并且优化了这种做法。

在Todos应用程序里, 我们要显示每个列表旁边未完成的任务(todos)的数量, 我们需要规范化```list.incompleteTodoCount```。
这是一个不便理解但通常相当容易做到的事情, 我们将在下面一节中看到[如何规范化](https://guide.meteor.com/collections.html#abstracting-denormalizers)。

这种体系结构需要的另一个规范化需求是从父文档到子文档的。例如, 在Todos中, 我们通过```list.userId```属性来确保任务列表(todo lists)的私密性, 
但我们分别发布Todos, 又可能要对```list.userId```进行规范化。要做到这一点, 在创建 todo 时, 我们需要谨慎地从列表中取得userId, 并在列表的userId更改时更新所有相关Todos。

### 为未来设计

应用程序 (尤其是 web 应用程序) 很少有完成后不再修改的时候, 所以在设计数据架构时要考虑将来可能发生的更改。
在大多数情况下, 在有实际需要之前添加字段并不是一个好主意 (通常你所预期的需求实际上并不会最终发生)。

但是, 最好提前考虑到着时间的推移数据架构(schema)可能会发生改变。例如, 您可能有一个文档中的字符串列表 (可能是一组标记)。
尽管将它们作为一个子字段放在文档中似乎挺好(假设它们不会有太大的变化), 但也有可能，他们最终会变得更加复杂 
(也许标签会有一个创建者, 或者有一个子标签呢？), 那么从长远考虑，一开始就做一个单独的数据集可能更容易。

你在模式设计中的远见将取决于你的应用程序的独特需求, 并且需要你来作出判断。

### 在写数据库时使用架构

虽然有多种方法可以在将数据发送到数据集之前通过**简单模式**(Simple Schema)来检验它 (例如, 您可以在每个方法调用中检查数据架构), 
但最简单、最可靠的方法是使用 [aldeed:collection2](https://atmospherejs.com/aldeed/collection2) 包来检查每一次更改(insert/update/upsert)调用。

我们通过```attachSchema()```来实现：

``` 
Lists.attachSchema(Lists.schema);
```

这意味着, 我们每次调用```Lists.insert(), Lists.update(), Lists.upsert()```时, 我们的文档或修饰符将根据架构自动检查 (方式不同,取决于具体的更改请求)。

### defaultValue和数据清理

Collection2 做的一件事是 ["清理"数据](https://github.com/aldeed/meteor-simple-schema#cleaning-data), 然后再将其发送到数据库。这包括但不限于:

1.  强制类型——将字符串转换为数字
2.  删除架构中不具有的属性
3.  根据架构定义中的defaultValue分配默认值

但是, 有时在将文档插入到数据集之前, 需要对其进行复杂的初始化。例如, 在Todos应用程序中, 我们希望将新列表的名称设置为 "List x", 其中 x 是下一个可用的唯一字母。

我们通过继承```Mongo.Collection```并重写```insert()```来实现：

``` 
class ListsCollection extends Mongo.Collection {
  insert(list, callback) {
    if (!list.name) {
      let nextLetter = 'A';
      list.name = `List ${nextLetter}`;
      while (!!this.findOne({name: list.name})) {
        // not going to be too smart here, can go past Z
        nextLetter = String.fromCharCode(nextLetter.charCodeAt(0) + 1);
        list.name = `List ${nextLetter}`;
      }
    }
    // Call the original `insert` method, which will validate
    // against the schema
    return super.insert(list, callback);
  }
}
Lists = new ListsCollection('lists');
```

### 与insert/update/remove挂勾

上面的技术也可以用来提供一个位置来 "挂钩" 额外的功能到集合中。例如, 在删除列表时, 我们总是希望同时删除它的所有Todos。

我们通过继承```Mongo.Collection```，覆盖```remove()```来实现：

``` 
class ListsCollection extends Mongo.Collection {
  // ...
  remove(selector, callback) {
    Package.todos.Todos.remove({listId: selector});
    return super.remove(selector, callback);
  }
}
```

这种技术有几个缺点:

1.  当你在很多地方挂勾插件(Mutators)时，会产生很大的时间消耗。
2.  有时, 单个功能可以在多个插件上传播。
3.  用完全通用的方式 (包括每个可能的选择器和修饰符) 来编写钩子可能是一个挑战, 而且它不一定是你的应用程序所必需的 (也许你只用一种方式调用那个插件)。

处理第1点和第2点是将钩子分别放在各自的模块, 简单地用插件作为一个点, 以一种合理的方式调用该模块。下面我们将看到一个例子。

第3点通常可以通过将钩子放在调用插件的方法上而不是插件本身上来解决。虽然这是一个不完美的办法(需要小心, 我们在未来可能会不断添加别的方法,调用插件), 
但总好过编写一堆代码却从来没有真正调用 (保证不会用到!), 也好过那些想象中很通用实际上并不通用的插件。

### Abstracting denormalizers（抽象反规范化）

反规范化可能需要在多个集合的各种插件上发生。因此, 在一个位置集中定义反规范化逻辑, 并将其用一行代码挂钩到每个更改点。
这种方法的优点是, 反规范化逻辑集中在一个地方, 而不是分散在许多文件上, 但您仍然可以检查每个集合的代码, 并完全了解每次更新时发生的情况。

在Todos示例应用程序中, 我们构建了一个 ```incompleteCountDenormalizer``` 来抽象列表中未完成的todos的计数。
此代码需要在插入、更新 (选中或取消选中) 或删除 todo 项时运行。该代码类似于:

``` 
const incompleteCountDenormalizer = {
  _updateList(listId) {
    // Recalculate the correct incomplete count direct from MongoDB
    const incompleteCount = Todos.find({
      listId,
      checked: false
    }).count();
    Lists.update(listId, {$set: {incompleteCount}});
  },
  afterInsertTodo(todo) {
    this._updateList(todo.listId);
  },
  afterUpdateTodo(selector, modifier) {
    // We only support very limited operations on todos
    check(modifier, {$set: Object});
    // We can only deal with $set modifiers, but that's all we do in this app
    if (_.has(modifier.$set, 'checked')) {
      Todos.find(selector, {fields: {listId: 1}}).forEach(todo => {
        this._updateList(todo.listId);
      });
    }
  },
  // Here we need to take the list of todos being removed, selected *before* the update
  // because otherwise we can't figure out the relevant list id(s) (if the todo has been deleted)
  afterRemoveTodos(todos) {
    todos.forEach(todo => this._updateList(todo.listId));
  }
};
```

然后, 我们可以象这样把 ```denormalizer``` 插入到todos的更改动作中:

``` 
class TodosCollection extends Mongo.Collection {
  insert(doc, callback) {
    doc.createdAt = doc.createdAt || new Date();
    const result = super.insert(doc, callback);
    incompleteCountDenormalizer.afterInsertTodo(doc);
    return result;
  }
}
```

请注意, 我们只处理在应用程序中实际使用的插件——我们不处理列表上的 todo 计数可能发生的所有可能更改的地方。
例如, 如果在 todo 项上更改了 listId, 则需要更改**两个**列表的 ```incompleteCount```。
然而, 由于我们的应用程序不这样做, 我们在 ```denormalizer``` 中并不处理它。

处理每一个可能的 MongoDB 操作是很难做到的, 因为 MongoDB 有一个丰富的修饰语。
相反, 我们关注的只是处理我们将在应用程序中看到的修饰符。如果这很困难, 那么最好将钩子的逻辑移到实际进行相关修改的方法中
(尽管您需要努力确保在所有相关的地方都这样做, 不管是现在，还是将来发生变化的时候)。

如果有这样的软件包，那可能是很有意义的：完全抽象出一些常见的反规范化技术, 并实际尝试处理所有可能的修改。
如果你写了这样一个软件包, 请悄悄地告诉!

##  向新架构迁移

正如我们前面所讨论的, 试图提前预测数据模式的所有未来需求是不可能的。随着项目的成熟, 不可避免地会出现需要更改数据库架构的时候。
您需要注意如何迁移到新的模式, 并确保您的应用程序在迁移期间和之后能顺利运行。

### 编写迁移程序

用于编写迁移程序的一个有用的包是[percolate:migrations](https://atmospherejs.com/percolate/migrations), 
它为不同版本的架构之间切换提供了一个很好的框架。

例如, 假设我们要添加一个 todoCount 字段, 并确保它是为所有现有列表设置的。然后, 我们在服务器端代码 (如/server/migrations.js)中编写以下内容:

``` 
Migrations.add({
  version: 1,
  up() {
    Lists.find({todoCount: {$exists: false}}).forEach(list => {
      const todoCount = Todos.find({listId: list._id}).count();
      Lists.update(list._id, {$set: {todoCount}});
    });
  },
  down() {
    Lists.update({}, {$unset: {todoCount: true}}, {multi: true});
  }
});
```

此迁移程序要求在数据库迁移时首先运行, 当调用时, 将使每个列表与当前的 todo 计数一起更新。

要了解有关迁移包的 API 的更多信息, 请参阅其[文档](https://atmospherejs.com/percolate/migrations)。

### 批量更改

如果您的迁移需要更改大量数据, 特别是当您需要在运行应用程序服务器时停止它, 则最好使用 [MongoDB 的批量操作](https://docs.mongodb.org/v3.0/core/bulk-write-operations/)。

批量操作的优点是它只需要进行一次 MongoDB 的写操作, 这通常意味着它的速度要快得多。缺点是, 如果您的迁移是复杂的 
(通常情况下, 如果您不是只做```.update(.., .., {multi: true})```), 则准备批量更新可能需要大量的时间。

这意味着, 如果用户正在访问的网站, 而更新正在准备, 它可能会过时! 此外, 批量更新将在应用时锁定整个集合, 
如果需要一段时间, 它可能会在用户体验中造成明显的影响。由于这些原因, 您经常需要停止您的服务器, 并让您的用户知道您正在执行维护, 请耐心等待！

我们可以这样写我们的上述迁移 (请注意, 您必须在 MongoDB 2.6 或更高版本中才能存在批量更新操作)。
我们可以通过 ```Collection#rawCollection()``` 访问本机 MongoDB API:

``` 
Migrations.add({
  version: 1,
  up() {
    // This is how to get access to the raw MongoDB node collection that the Meteor server collection wraps
    const batch = Lists.rawCollection().initializeUnorderedBulkOp();
    //Mongo throws an error if we execute a batch operation without actual operations, e.g. when Lists was empty.
    let hasUpdates = false;
    Lists.find({todoCount: {$exists: false}}).forEach(list => {
      const todoCount = Todos.find({listId: list._id}).count();
      // We have to use pure MongoDB syntax here, thus the `{_id: X}`
      batch.find({_id: list._id}).updateOne({$set: {todoCount}});
      hasUpdates = true;
    });
    if(hasUpdates){
      // We need to wrap the async function to get a synchronous API that migrations expects
      const execute = Meteor.wrapAsync(batch.execute, batch);
      return execute();
    }
    return true;
  },
  down() {
    Lists.update({}, {$unset: {todoCount: true}}, {multi: true});
  }
});
```

请注意, 通过使用聚合([Aggregation](https://docs.mongodb.org/v2.6/aggregation/))来收集初始的 todo 计数集, 
我们可以更快地进行迁移。

### 运行迁移程序

要对开发数据库运行迁移程序,使用Meteor脚本很容易做到:

``` 
// After running `meteor shell` on the command line:
Migrations.migrateTo('latest');
```

如果迁移程序将log输出到控制台, 您将在运行Meteor服务器的终端窗口中看到它。

要针对生产数据库运行迁移程序, 请在生产模式 (包括数据库设置和环境变量) 的本地运行应用程序, 并以同样的方式使用Meteor脚本。
这是针对您的生产数据库运行```up()```所有未完成的迁移的功能。在我们的例子中, 它应该确保所有的列表都有一个 todoCount 字段集。

这样做的一个方法是在接近你的数据库的地方建立一个虚拟机, 该数据库服务器上安装了Meteor有和SSH访问权限, 
进入服务器后运行迁移程序。这样, 您的计算机和数据库之间的任何延迟都将被消除, 但您仍然需要非常小心地观察迁移过程。

**请注意, 在运行任何迁移之前, 应始终执行数据库备份!**

### 中断架构更改

有时, 当我们更改应用程序的模式时, 我们会以中断的方式进行, 这样旧的模式就不能正常使用新的代码库。例如, 如果我们有一些用户界面代码, 
大量依赖于所有具有 todoCount 集的列表, 那么在迁移运行之前, 我们的应用程序的 ui 将在部署后中断。

解决此问题的简单方法是在部署和完成迁移之间的一段时间内暂停应用程序。这很不合适, 特别是考虑到一些迁移可能需要数小时才能运行 
(尽管使用[批量更新](https://guide.meteor.com/collections.html#bulk-data-changes)可能在这里有很大的帮助)。

更好的方法是多级部署。其基本思想是:

1.  部署可以处理旧模式和新架构的应用程序版本。在我们的例子中, 不期望 todoCount 存在, 但是在创建新的Todos时正确地更新了它。
2.  运行迁移。此时, 您应该确保所有列表都有一个 todoCount。
3.  部署依赖新架构的新代码, 不再知道如何处理旧模式。现在UI中我们可以放心地依赖```list.todoCount``` 。

还有一件事需要注意, 特别是在这种多阶段部署中, 准备回滚是很重要的! 因此, 迁移包允许您指定一个```down()```函数，
并调用```Migrations.migrateTo (x)``` 迁移回版本 x。

因此, 如果我们想逆转我们的迁移, 我们会运行:

``` 
// The "0" migration is the unmigrated (before the first migration) state
Migrations.migrateTo(0);
```

如果您发现需要回滚您的代码版本, 则需要担心数据, 用相反的部署步骤仔细地进行回滚操作。

### 警告

上面描述的迁移策略的某些方面可能不是最理想的方法 (尽管在许多情况下可能是合适的)。以下是其他一些需要注意的事项:

1.  通常情况下, 最好不要依赖迁移中的应用程序代码 (因为应用程序将随着时间的推移而变化, 而迁移没有)。例如, 
    让您的迁移程序通过您的 Collection2 集合 (从而检查架构、设置 autovalues 等)可能会中为，因为随着时间的推移您的架构发生了变化。

    避免此问题的一种方法是, 不要在数据库上运行旧的迁移。这办法虽然受点限制, 但可以工作。

2.  在本地计算机上运行迁移可能会使它花费更长的时间, 因为您的计算机不可能接近生产数据库。

将特殊的 "迁移应用程序" 部署到与实际应用程序相同的硬件可能是解决上述问题的最好方法。这是不可思议的, 如果用这样的应用程序跟踪迁移程序, 
使用日志和提供UI来检查和运行它们（迁移程序）。也许可以建立一个样板应用程序 (如果你这样做, 请让我们知道, 我们将链接到这里!）

### 集合之间的关联

正如我们前面讨论的, 在Meteor应用中在不同的集合存在有关联的文档是非常常见的。因此, 当您有感兴趣的文档时 (例如, 在单个列表中的所有todos), 
需要编写查询来获取相关文档也是很常见的。

为了让事情变容易, 我们可以将函数附加到属于给定集合的文档的原型, 从而给出文档的 "方法" (面向对象的意义)。然后, 
我们可以使用这些方法创建新的查询来查找相关文档。


### Collection helpers（数据集Helpers）


我们可以使用 [dburles:collection-helpers](https://atmospherejs.com/dburles/collection-helpers) 程序包来轻松地将这些方法 (或 "Helpers"") 附加到文档中。例如:

``` 
Lists.helpers({
  // A list is considered to be private if it has a userId set
  isPrivate() {
    return !!this.userId;
  }
});
```

将此帮助器附加到Lists集合后, 每次从数据库 (在客户端或服务器上) 获取列表时, 它都将具有```.isPrivate()```函数可用:

``` 
const list = Lists.findOne();
if (list.isPrivate()) {
  console.log('The first list is private!');
}
```

### Association helpers

现在我们可以将帮助程序附加到文档中, 定义一个读取相关文档的帮助器是很简单的:

```
Lists.helpers({
  todos() {
    return Todos.find({listId: this._id}, {sort: {createdAt: -1}});
  }
});
```

现在, 我们可以很容易地找到所有的Todos列表:

``` 
const list = Lists.findOne();
console.log(`The first list has ${list.todos().count()} todos`);
```

