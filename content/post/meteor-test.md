+++
title = "Meteor(7):测试(testing)"
description = "How to test your Meteor application."
tags = [
    "JavaScript",
    "NodeJs",
    "Web application",
    "Mobile application",
    "Platform",
    "MongoDB",
]
date = "2017-11-13"
categories = [
    "Development",
    "nodejs",
]
thumbnail = "images/meteor-guide/meteor-test.jpg"
+++

测试Meteor应用程序。

<!--more-->

##  介绍

测试允许您确保您的应用程序按您认为的方式工作, 特别是代码库随着时间的推移而变化。如果你有好的测试, 你就有信心重构和重写代码。
测试也是应用程序的最具体的文档形式, 因为其他开发人员可以通过阅读测试来了解如何使用您的代码。

自动化测试非常重要, 因为它允许您更频繁地运行更大的测试集, 从而使您能够立即捕获回归错误。

### 测试类型

整本Meteor指南书都是基于测试来写的, 所以我们将简单地介绍一下测试的一些基础知识。
编写测试时要考虑的重要事项是您要测试应用程序的哪个部分, 以及如何验证这种工作。

-   单元测试。如果您正在测试应用程序的一个小模块, 就是在写单元测试。为了隔离每个测试, 您需要对模块通常利用的其他模块进行存根和模拟。
    您通常还需要监视该模块来验证它们是否发生希望的行为。
-   集成测试。如果您正在测试多个模块的行为是否正常, 则您在写集成测试。此类测试要复杂得多, 并且可能需要在客户端和服务器上运行代码, 
    以验证跨该鸿沟的通信是否按预期方式工作。通常, 集成测试仍将隔离整个应用程序的一部分, 并直接验证代码中的结果。
-   验收测试。如果要编写测试, 测试应用程序的任何运行版本, 在浏览器级别进行验证, 在按下按钮时发生正确的事情, 这就是验收测试 (有时称为 "结束测试")。
    这些测试通常尝试尽可能少地挂钩到应用程序中, 而不是设置正确的数据来运行测试。
-   负载测试。最后, 您可能希望测试您的应用程序在典型的负载下工作情况, 或者在它崩溃之前查看它可以处理多少负载。这称为负载测试或压力测试。
    此类测试可能具有挑战性, 通常不会经常运行, 但在大型产品发布之前都要进行这种测试, 这是非常重要的。

### Meteor测试的挑战

在大多数情况下, 测试Meteor应用程序与测试任何其他的全栈JavaScript应用程序没有什么不同。然而, 与传统的后端或前端的框架相比, 
有两个因素可以使测试更具挑战性:

-   客户端/服务端数据。Meteor的数据系统简化了客户端与服务器之间的差距, 并且允许您构建应用程序时不考虑数据的移动方式。
    因而测试您的代码在该间隙中是否确实正确工作变得至关重要。在传统的框架中, 您花费大量的时间考虑客户端和服务器之间的接口, 
    您通常可以隔离测试接口的两边。但是, Meteor的完整应用程序测试模式使得编写覆盖完整堆栈的集成测试变得很容易。
    另一个挑战是在客户端上下文中创建测试数据;我们将在下面的小节中讨论如何生成测试数据。
-   响应式。Meteor的响应系统是 "最终一致的", 这意味着，当你改变一个输入时, 你要过一段时间后才会看到用户界面的变化。
    这在测试时可能是一个挑战, 但是有一些方法可以等到这些更改发生后再来验证结果, 例如使用```Tracker.afterFlush()```。

### ```meteor test```命令

在Meteor中测试应用程序的主要方法是```meteor test```命令。

这将以特殊的 "测试模式" 加载您的应用程序。它的目的是:

1.  不象Meteor应用程序通常那样急加载程序代码。
2.  急加载应用程序中的任何类似```*.test[s].*, or *.spec[s].*```的文件(包括在```imports/```文件夹中的文件)。
3.  设置```Meteor.isTest``标签为```true```。
4.  启动测试驱动程序包（参考下面部分）。

>   ```meteor build```和```meteor test```命令忽略```tests/```目录中的任何文件。这允许您将测试放在这个目录中,您可以在流星的内置测试工具之外使用一个测试运行程序, 并且仍然没有在您的应用程序中加载这些文件。请参阅Meteor的默认文件加载顺序规则。

这意味着您可以在具有特定文件名模式的文件中编写测试, 并且知道它们不会包括在应用程序的正常生成中。当您的应用程序在测试模式下运行时, 
这些文件将被加载 (而其他的都不会), 并且它们可以导入您要测试的模块。我们将看到这是单元测试和简单集成测试的理想选择。

此外, Meteor提供了一个 "完整应用程序" 测试模式。你可以用Meteor测试来运行 ```meteor test --full-app```。

这与测试模式类似, 主要区别如下:

1.  只加载名如```*.app-test[s].* and *.app-spec[s].*```的文件。
2.  象Meteor应用程序那样急加载应用程序代码。
3.  设置```Meteor.isAppTest```标签为```true```，而不是```Meteor.isTest```标签为```true```。

这意味着整个应用程序 (包括 web 服务器和客户端路由器) 都被加载, 并且将以正常方式运行。这使您可以编写更复杂的集成测试, 还可以为验收测试加载其他文件。

请注意, 在流星工具中还有另一个测试命令;```meteor test-packages```是一种测试Atmosphere包的方法, 这是在编写软件包文章中讨论的。

### 驱动程序包

运行```meteor test```命令时, 必须提供```--driver-package```参数。测试驱动程序是一个微型应用程序, 它运行在您的应用程序的位置, 
并运行每个已定义的测试, 同时在某种用户界面中报告结果。

有两种主要的测试驱动程序包:

-   web报告: Meteor应用, 显示一个特殊的测试报告的web用户界面, 您可以在上面查看测试结果。
-   控制台报告: 这是在命令行状态下用于自动化测试和如连续集成的。

### 推荐：摩卡(Mocha)

在这篇文章中, 我们将使用流行的[Mocha](https://mochajs.org/)测试器与[Chai](http://chaijs.com/)断言库来测试我们的应用程序。
为了在摩卡中编写和运行测试, 我们需要添加一个合适的测试驱动程序包。

有几个选项。选择那些对你的应用程序有意义的选项。您可以依赖不同的情况设置不同的测试命令。

-   [practicalmeteor:mocha](https://atmospherejs.com/practicalmeteor/mocha):
    运行客户端和服务端包或应用程序测试, 并在浏览器中显示所有结果。
    使用[spacejam](https://www.npmjs.com/package/spacejam)以支持命令行/CI。
-   [meteortesting:mocha](https://atmospherejs.com/meteortesting/mocha):
    运行客户端和/或服务端包或应用程序测试, 并报告服务器控制台中的所有结果。支持用于运行客户端测试的各种浏览器, 
    包括 PhantomJS、Selenium ChromeDriver 和 Electron。可用于在 CI 服务器上运行测试。有观察模式。

这些软件包在开发或生产模式下不做任何事情。他们声明自己是```testOnly```, 所以他们甚至在测试之外没有加载。
但是当我们的应用程序在测试模式下运行时, 测试驱动程序包将接管, 在客户端和服务器上执行测试代码, 并将结果呈现给浏览器。

下面是我们如何可以添加 ```practicalmeteor:mocha``` 包到我们的应用程序:

```
meteor add practicalmeteor:mocha
```

##  测试文件

通常情况下，测试文件本身(例如, 名为```todos-items.test.js```或```routing.app-specs.coffee```的文件)可以注册自己并由测试驱动程序运行。
对于mocha, 用```describe```和```it```来注册:

``` 
describe('my module', function () {
  it('does something that should be tested', function () {
    // This code will be executed by the test driver when the app is started
    // in the correct mode
  })
}
```

请注意, 不鼓励在mocha中使用箭头功能。    

##  测试数据

当您的应用程序在测试模式下运行时, 它将使用干净的测试数据库进行初始化。

如果您运行的测试依赖于使用数据库, 特别是数据库的内容, 则需要在测试中执行一些设置步骤, 以确保数据库处于所期望的状态。
您可以使用一些工具来完成这种工作。

为确保数据库干净, [xolvio:cleaner](https://atmospherejs.com/xolvio/cleaner)软件包非常有用。
您可以使用它在 ```beforeEach``` 块中重置数据库:

``` 
import { resetDatabase } from 'meteor/xolvio:cleaner';
describe('my module', function () {
  beforeEach(function () {
    resetDatabase();
  });
});
```

此技术只会在服务器上工作。如果需要从客户端测试重置数据库, 则可以使用方法执行此操作:

``` 
import { resetDatabase } from 'meteor/xolvio:cleaner';
// NOTE: Before writing a method like this you'll want to double check
// that this file is only going to be loaded in test mode!!
Meteor.methods({
  'test.resetDatabase': () => resetDatabase(),
});
describe('my module', function (done) {
  beforeEach(function (done) {
    // We need to wait until the method call is done before moving on, so we
    // use Mocha's async mechanism (calling a done callback)
    Meteor.call('test.resetDatabase', done);
  });
});
```

当我们将代码放在测试文件中时, 它不会在正常的开发或生产模式下加载 (这将是一个非常糟糕的事情!）如果创建具有类似功能的Atmosphere包,
则应将其标记为 ```testOnly```, 并只在测试模式中加载它。

### 生成测试数据

通常, 创建一组数据来运行测试是明智的。您可以对集合使用标准的```insert()```调用来实现此目的, 但通常更容易创建工厂方法, 
这有助于生成随机测试数据。一个非常好的软件包[dburles:factory](https://atmospherejs.com/dburles/factory)可用来做到这一点。

在Todos示例应用程序中, 我们定义了一个工厂来描述如何使用```faker npm```包创建一个测试todo项:

``` 
import faker from 'faker';
Factory.define('todo', Todos, {
  listId: () => Factory.get('list'),
  text: () => faker.lorem.sentence(),
  createdAt: () => new Date(),
});
```

要在测试中使用工厂, 我们只需调用```Factory.create```:

``` 
// This creates a todo and a list in the database and returns the todo.
const todo = Factory.create('todo');
// If we have a list already, we can pass in the id and avoid creating another:
const list = Factory.create('list');
const todoInList = Factory.create('todo', { listId: list._id });
```

### 模拟数据库

当```Factory.create```直接将文档插入到集合中, 并传递到```Factory.define```函数中时, 在客户端上使用它可能存在问题。
但有一个漂亮的隔离技巧, 你可以用一个模拟本地集合取代服务器支持的Todos客户端集合, 
这个是[hwillson:stub-collections](https://atmospherejs.com/hwillson/stub-collections)。

``` 
import StubCollections from 'meteor/hwillson:stub-collections';
import { Todos } from 'path/to/todos.js';
StubCollections.stub(Todos);
// Now Todos is stubbed to a simple local collection mock,
//   so for instance on the client we can do:
Todos.insert({ a: 'document' });
// Restore the `Todos` collection
StubCollections.restore();
```

在摩卡测试中, 通常是在 ```beforeEach/afterEach``` 块中使用```stub-collections```的。

##  单元测试

单元测试是一个过程, 它隔离了一段代码, 然后测试该部分的内部是否按预期工作。因为我们已经将我们的代码分割成了ES2015的模块, 
所以很自然地应当一次测试一个模块。

通过隔离一个模块并简单地测试其内部功能, 我们可以编写快速、准确的测试——他们可以快速地告诉您应用程序中的问题所在。
但请注意, 不完整的单元测试通常会隐藏 bug, 因为它们会将依赖项排除掉。因此, 将单元测试与较慢的 (可能不太常见) 集成测试和验收测试结合起来是很有用的。

### 简单的Blaze单元测试

在Todos示例应用程序中, 由于我们已经将用户界面分成了智能和可重用的组件, 所以很自然地要对我们的一些可重用组件进行单元测试 (下面我们将看到如何集成测试我们的智能组件)。

[imports/ui/test-helpers.js](https://github.com/meteor/todos/blob/master/imports/ui/test-helpers.js):

``` 
import { _ } from 'meteor/underscore';
import { Template } from 'meteor/templating';
import { Blaze } from 'meteor/blaze';
import { Tracker } from 'meteor/tracker';
const withDiv = function withDiv(callback) {
  const el = document.createElement('div');
  document.body.appendChild(el);
  try {
    callback(el);
  } finally {
    document.body.removeChild(el);
  }
};
export const withRenderedTemplate = function withRenderedTemplate(template, data, callback) {
  withDiv((el) => {
    const ourTemplate = _.isString(template) ? Template[template] : template;
    Blaze.renderWithData(ourTemplate, data, el);
    Tracker.flush();
    callback(el);
  });
};
```

要测试的可重用组件的一个简单示例是```Todos_item``` 模板。这里是一个单元测试的样子 (你可以在应用程序库中看到一些其他的)。

[imports/ui/components/client/todos-item.tests.js](https://github.com/meteor/todos/blob/master/imports/ui/components/client/todos-item.tests.js):

``` 
/* eslint-env mocha */
/* eslint-disable func-names, prefer-arrow-callback */
import { Factory } from 'meteor/dburles:factory';
import { chai } from 'meteor/practicalmeteor:chai';
import { Template } from 'meteor/templating';
import { $ } from 'meteor/jquery';
import { Todos } from '../../../api/todos/todos';
import { withRenderedTemplate } from '../../test-helpers.js';
import '../todos-item.js';
describe('Todos_item', function () {
  beforeEach(function () {
    Template.registerHelper('_', key => key);
  });
  afterEach(function () {
    Template.deregisterHelper('_');
  });
  it('renders correctly with simple data', function () {
    const todo = Factory.build('todo', { checked: false });
    const data = {
      todo: Todos._transform(todo),
      onEditingChange: () => 0,
    };
    withRenderedTemplate('Todos_item', data, el => {
      chai.assert.equal($(el).find('input[type=text]').val(), todo.text);
      chai.assert.equal($(el).find('.list-item.checked').length, 0);
      chai.assert.equal($(el).find('.list-item.editing').length, 0);
    });
  });
});
```

这种测试的特别点如下:

**Importing**

当我们在测试模式下运行我们的应用程序时, 只有我们的测试文件将被急加载。这意味着, 为了使用我们的模板, 我们需要导入它们!
在这个测试中, 我们导入```todos-item.js```, 它本身导入```todos.html```(是的, 你需要导入你的Blaze模板的html文件!)。

**Stubbing**

作为单元测试, 我们必须将模块的依赖项排除掉。在这种情况下, 由于我们将代码隔离到可重用组件中的方式, 没有太多的事情可做;
主要是我们需要把由```tap:i18n```系统生成的```{{_}}```帮助类(helper)屏蔽掉。
请注意, 我们在```beforeEach```屏蔽它, 并在```afterEach```恢复它的。

如果要测试使用全局变量的代码, 则需要导入这些全局变量。例如, 如果您有一个全局Todos集合并要测试此文件:

``` 
// logging.js
export function logTodos() {
  console.log(Todos.findOne());
}
```

于是, 您需要在该文件和测试中导入Todos:

``` 
// logging.js
import { Todos } from './todos.js'
export function logTodos() {
  console.log(Todos.findOne());
}
```

``` 
// logging.test.js
import { Todos } from './todos.js'
Todos.findOne = () => {
  return {text: "write a guide"}
}
import { logTodos } from './logging.js'
// then test logTodos
...
```

**创建数据**

我们可以使用工厂包的```.build()``` API 来创建一个测试文档, 而不将其插入到任何集合中。因此我们要非常小心, 
不直接在可重用组件中调用任何集合, 我们可以将构建的 todo 文档直接传递到模板中。

### 一个简单的```React```单元测试

我们也可以应用相同的结构来测试```React```组件, 并推荐[Enzyme](https://github.com/airbnb/enzyme)包, 它模拟反应组件的环境, 并允许您使用 CSS 选择器来查询它。
Todos应用程序的```React```分支中提供了更大的测试套件, 但现在我们来看一个简单的示例:

``` 
import { Factory } from 'meteor/dburles:factory';
import React from 'react';
import { shallow } from 'enzyme';
import { chai } from 'meteor/practicalmeteor:chai';
import TodoItem from './TodoItem.jsx';
describe('TodoItem', () => {
  it('should render', () => {
    const todo = Factory.build('todo', { text: 'testing', checked: false });
    const item = shallow(<TodoItem todo={todo} />);
    chai.assert(item.hasClass('list-item'));
    chai.assert(!item.hasClass('checked'));
    chai.assert.equal(item.find('.editing').length, 0);
    chai.assert.equal(item.find('input[type="text"]').prop('defaultValue'), 'testing');
  });
});
```

这个测试比上面的Blaze版本稍微简单一些, 因为React样例应用程序没有进行国际化。否则, 它在概念上是相同的。
我们使用酶的```shallow```函数来渲染 TodoItem 的成分, 以及用生成的对象来查询文档, 也可以模拟用户交互。
下面是一个模拟用户检查 todo 项的示例:

``` 
import { Factory } from 'meteor/dburles:factory';
import React from 'react';
import { shallow } from 'enzyme';
import { sinon } from 'meteor/practicalmeteor:sinon';
import TodoItem from './TodoItem.jsx';
import { setCheckedStatus } from '../../api/todos/methods.js';
describe('TodoItem', () => {
  it('should update status when checked', () => {
    sinon.stub(setCheckedStatus, 'call');
    const todo = Factory.create('todo', { checked: false });
    const item = shallow(<TodoItem todo={todo} />);
    item.find('input[type="checkbox"]').simulate('change', {
      target: { checked: true },
    });
    sinon.assert.calledWith(setCheckedStatus.call, {
      todoId: todo._id,
      newCheckedStatus: true,
    });
    setCheckedStatus.call.restore();
  });
});
```

在这个案例中, TodoItem组件在用户单击时调用一个Meteor方法 ```setCheckedStatus```, 但这是一个单元测试, 因此没有服务器运行。
所以我们用[Sinon](http://sinonjs.org/)来屏蔽它。在模拟单击后, 我们验证是否用了正确的参数调用存根。最后, 我们清理存根并恢复原始方法行为。

### 运行单元测试

为了运行我们的应用程序定义的测试, 我们在测试模式下运行我们的应用程序:

``` 
meteor test --driver-package practicalmeteor:mocha
```

正如我们已经定义了一个测试文件 (```imports/todos/todos.tests.j```), 这意味着上面的文件将被热加载, 
把```builds correctly from factory```测试添加到摩卡注册表中。

要运行测试, 请在浏览器中访问 ```http://localhost:3000```。这就启动了```practicalmeteor:mocha```, 
它在浏览器和服务器上运行您的测试。它在mocha测试报告的浏览器中显示测试结果。

通常, 在开发应用程序时, 在第二个端口 (如 3100) 上运行Meteor测试, 同时在一个单独的进程中运行主应用程序:

``` 
# in one terminal window
meteor
# in another
meteor test --driver-package practicalmeteor:mocha --port 3100
```

然后, 您可以打开两个浏览器窗口来查看应用程序的操作, 同时确保在进行更改时不中断任何测试。

### 隔离技术

在上面的单元测试中, 我们看到了一个非常有限的示例, 说明如何从较大的应用程序中隔离模块。这对于正确的单元测试至关重要。其他一些实用工具和技术包括:

-   [velocity:meteor-stubs](https://atmospherejs.com/velocity/meteor-stubs)包, 为大多数Meteor核心对象创建简单的存根。
-   或者, 您也可以使用像```Sinon```这样的工具直接建存根, 就像我们在简单的集成测试中看到的那样。
-   上面提到的[hwillson:stub-collections](https://atmospherejs.com/hwillson/stub-collections)软件包。

还有一些很好的隔离和测试应用程序。

**测试应用程序**

使用[johanbrook:publication-collector](https://atmospherejs.com/johanbrook/publication-collector)包, 
您可以在不创建传统订阅的情况下测试单个发布的输出:

``` 
describe('lists.public', function () {
  it('sends all public lists', function (done) {
    // Set a user id that will be provided to the publish function as `this.userId`,
    // in case you want to test authentication.
    const collector = new PublicationCollector({userId: 'some-id'});
    // Collect the data published from the `lists.public` publication.
    collector.collect('lists.public', (collections) => {
      // `collections` is a dictionary with collection names as keys,
      // and their published documents as values in an array.
      // Here, documents from the collection 'Lists' are published.
      chai.assert.typeOf(collections.Lists, 'array');
      chai.assert.equal(collections.Lists.length, 3);
      done();
    });
  });
});
```

请注意, 用户文档 (通常使用 ```Meteor.users.find()``` 查询的文件) 将通过关键字"user"从```PublicationCollector.collect()```调用中传递过来。
有关详细信息, 请参阅包中的"测试"部分。

##  集成测试

集成测试是一种跨越模块边界的测试。在最简单的情况下, 这仅仅意味着一些非常类似于单元测试的东西, 你在多个模块周围进行隔离, 创建一个奇异的 "测试系统"。

虽然在概念上不同于单元测试, 但通常不需要对单元测试运行任何不同的测试, 并且可以使用与单元测试相同的Meteor测试模式和隔离技术。

但是, 一个集成测试跨越了一个Meteor应用程序的客户端-服务器边界 (在测试的模块跨越该边界), 需要一个不同的测试基础结构, 
即Meteor的 "完整应用程序(full app))" 测试模式。

让我们来看看这两种测试的例子。

### 简单集成测试

我们的可重用组件是天然适合单元测试的组件;同样, 我们的智能组件往往需要一个集成测试才能真正正确地运行, 
因为智能组件的工作是将数据汇集在一起, 并将其提供给一个可重用的组件。

在Todos示例应用程序中, 我们对```Lists_show_page```智能组件进行了集成测试。此测试仅确保在数据库中存在正确的数据时, 
模板正确呈现, 即它正在收集正确的数据 (如我们所料)。它将渲染树(rendering tree)与Meteor栈(Meteor stack)中更复杂的数据订阅部分隔离。
如果我们想要测试的是与智能组件协同工作的订阅方面的功能, 我们需要编写一个完整的应用程序集成测试。

[imports/ui/components/client/todos-item.tests.js](https://github.com/meteor/todos/blob/master/imports/ui/components/client/todos-item.tests.js):

``` 
/* eslint-env mocha */
/* eslint-disable func-names, prefer-arrow-callback */
import { Meteor } from 'meteor/meteor';
import { Factory } from 'meteor/dburles:factory';
import { Random } from 'meteor/random';
import { chai } from 'meteor/practicalmeteor:chai';
import StubCollections from 'meteor/hwillson:stub-collections';
import { Template } from 'meteor/templating';
import { _ } from 'meteor/underscore';
import { $ } from 'meteor/jquery';
import { FlowRouter } from 'meteor/kadira:flow-router';
import { sinon } from 'meteor/practicalmeteor:sinon';
import { withRenderedTemplate } from '../../test-helpers.js';
import '../lists-show-page.js';
import { Todos } from '../../../api/todos/todos.js';
import { Lists } from '../../../api/lists/lists.js';
describe('Lists_show_page', function () {
  const listId = Random.id();
  beforeEach(function () {
    StubCollections.stub([Todos, Lists]);
    Template.registerHelper('_', key => key);
    sinon.stub(FlowRouter, 'getParam', () => listId);
    sinon.stub(Meteor, 'subscribe', () => ({
      subscriptionId: 0,
      ready: () => true,
    }));
  });
  afterEach(function () {
    StubCollections.restore();
    Template.deregisterHelper('_');
    FlowRouter.getParam.restore();
    Meteor.subscribe.restore();
  });
  it('renders correctly with simple data', function () {
    Factory.create('list', { _id: listId });
    const timestamp = new Date();
    const todos = _.times(3, i => Factory.create('todo', {
      listId,
      createdAt: new Date(timestamp - (3 - i)),
    }));
    withRenderedTemplate('Lists_show_page', {}, el => {
      const todosText = todos.map(t => t.text).reverse();
      const renderedText = $(el).find('.list-items input[type=text]')
        .map((i, e) => $(e).val())
        .toArray();
      chai.assert.deepEqual(renderedText, todosText);
    });
  });
});
```

特别关注以下几点：

**Importing**

由于我们将以与单元测试相同的方式运行此测试, 因此需要以与单元测试相同的方式导入测试中的相关模块。

**Stubbing**

由于在我们的集成测试中测试的系统有更大的表面积, 我们需要将更多的存根(stub)与栈(stack)的其余部分结合起来。
这里特别感兴趣的是我们使用 [hwillson:stub-collections](https://guide.meteor.com/testing.html#mocking-the-database) 包
和 [Sinon](http://sinonjs.org/) 导出(stub out)流路由器(Flow Router)和我们的订购。

**创建数据**

在这个测试中, 我们使用了工厂包的```.create()``` API, 它将数据插入到实际的集合中。
但是, 因为我们将所有的 "Todos & List Collection" 方法代理到本地集合中 (这就是```hwillson:stub-collections```执行的操作),
我们不会遇到从客户端执行插入的任何问题。

此集成测试可以与我们在上面运行单元测试的方式完全相同。

### 全应用程序集成测试

在Todos示例应用程序中, 我们有一个集成测试, 它确保我们在路由到它时看到列表的全部内容, 这说明了一些集成测试技术。

[imports/startup/client/routes.app-test.js](https://github.com/meteor/todos/blob/master/imports/startup/client/routes.app-test.js):

``` 
/* eslint-env mocha */
/* eslint-disable func-names, prefer-arrow-callback */
import { Meteor } from 'meteor/meteor';
import { Tracker } from 'meteor/tracker';
import { DDP } from 'meteor/ddp-client';
import { FlowRouter } from 'meteor/kadira:flow-router';
import { assert } from 'meteor/practicalmeteor:chai';
import { Promise } from 'meteor/promise';
import { $ } from 'meteor/jquery';
import { denodeify } from '../../utils/denodeify';
import { generateData } from './../../api/generate-data.app-tests.js';
import { Lists } from '../../api/lists/lists.js';
import { Todos } from '../../api/todos/todos.js';
// Utility -- returns a promise which resolves when all subscriptions are done
const waitForSubscriptions = () => new Promise(resolve => {
  const poll = Meteor.setInterval(() => {
    if (DDP._allSubscriptionsReady()) {
      Meteor.clearInterval(poll);
      resolve();
    }
  }, 200);
});
// Tracker.afterFlush runs code when all consequent of a tracker based change
//   (such as a route change) have occured. This makes it a promise.
const afterFlushPromise = denodeify(Tracker.afterFlush);
if (Meteor.isClient) {
  describe('data available when routed', () => {
    // First, ensure the data that we expect is loaded on the server
    //   Then, route the app to the homepage
    beforeEach(() => generateData()
      .then(() => FlowRouter.go('/'))
      .then(waitForSubscriptions)
    );
    describe('when logged out', () => {
      it('has all public lists at homepage', () => {
        assert.equal(Lists.find().count(), 3);
      });
      it('renders the correct list when routed to', () => {
        const list = Lists.findOne();
        FlowRouter.go('Lists.show', { _id: list._id });
        return afterFlushPromise()
          .then(waitForSubscriptions)
          .then(() => {
            assert.equal($('.title-wrapper').html(), list.name);
            assert.equal(Todos.find({ listId: list._id }).count(), 3);
          });
      });
    });
  });
}
```

注意：

-   在运行之前, 每个测试都使用 ```generateData``` 帮助器设置所需的数据 
    (请参阅有关创建集成测试数据的部分, 以了解更多详细信息), 然后转到主页。
-   虽然```Flow Router```不执行回调, 但我们可以使用```Tracker.afterFlush```来等待所有的响应式结果。
-   在这里, 我们编写了一个小实用程序 (可以抽象成一个通用包), 等待由路由更改创建的所有订阅 (在本例中为```todos.inList```订阅), 以便在检查其数据之前准备就绪。

### 运行全运用(full-app)测试

通过下面的命令运行全应用测试：

``` 
meteor test --full-app --driver-package practicalmeteor:mocha
```

当我们在浏览器中连接到测试实例时, 我们希望呈现一个测试用户界面, 而不是我们的应用程序界面, 因此, ```mocha-web-reporter```
程序包将隐藏我们的应用程序的任何界面, 并将其自己覆盖其上。但是, 该应用程序仍然正常运行, 所以我们可以测试路由和检查数据加载的正确性。

### 创建数据

要在 ```full-app``` 测试模式下创建测试数据, 通常是创建一些特殊的测试方法，然后从客户端调用它。
在测试 "完整应用程序" 时, 我们要确保发布的数据是正确的 (正如我们在本测试中所做的), 因此不只是将集合中的数据进行存根(stub out), 并在其中放置合成信息。
相反, 我们希望在服务器上创建实际数据, 并将其发布。

与我们使用上面的 "测试数据" 部分中的 ```beforeEach``` 中的清除数据库方法类似, 我们可以在运行测试之前调用一个方法来执行此项工作。
在我们的路由测试的案例中, 我们使用了一个名为 [imports/api/generate-data.app-tests.js](https://github.com/meteor/todos/blob/master/imports/api/generate-data.app-tests.js)折文件, 
它定义了这个方法 (并且只会在完整的应用程序测试模式下加载, 所以一般不可用!）：

``` 
// This file will be auto-imported in the app-test context,
// ensuring the method is always available
import { Meteor } from 'meteor/meteor';
import { Factory } from 'meteor/dburles:factory';
import { resetDatabase } from 'meteor/xolvio:cleaner';
import { Random } from 'meteor/random';
import { _ } from 'meteor/underscore';
import { denodeify } from '../utils/denodeify';
const createList = (userId) => {
  const list = Factory.create('list', { userId });
  _.times(3, () => Factory.create('todo', { listId: list._id }));
  return list;
};
// Remember to double check this is a test-only file before
// adding a method like this!
Meteor.methods({
  generateFixtures() {
    resetDatabase();
    // create 3 public lists
    _.times(3, () => createList());
    // create 3 private lists
    _.times(3, () => createList(Random.id()));
  },
});
let generateData;
if (Meteor.isClient) {
  // Create a second connection to the server to use to call
  // test data methods. We do this so there's no contention
  // with the currently tested user's connection.
  const testConnection = Meteor.connect(Meteor.absoluteUrl());
  generateData = denodeify((cb) => {
    testConnection.call('generateFixtures', cb);
  });
}
export { generateData };
```

请注意, 我们导出了一个客户端符号 ```generateData```, 它是方法调用的```promisified```版本, 这使得在测试中按顺序使用它变得更加简单。

还要注意的是我们使用第二个DDP连接到服务器的方式, 这是为了便于发送这些测试"控制"方法的调用。

##  验收测试(Acceptance testing)

验收测试是采用未经修改的应用程序版本并从 "外部" 进行测试的过程, 以确保它我们期望的方式运行。通常, 如果一个应用程序通过验收测试, 
说明我们从产品的角度正确地完成了我们的工作。

当验收测试以一般方式在浏览器的完整上下文中测试应用程序的行为时, 您可以使用一系列工具来指定和运行此类测试。
在本指南中, 我们将演示使用[Chimp](https://chimp.readme.io/), 一个具有齐备的```Meteor-specific```功能的验收测试工具, 
这个使得该工具易于使用。

Chimp需要Node 4 或 5。您可以通过运行下面的命令来检查Node版本:

``` 
node -v
```

我们可以使用```brew install node```来安装Node新版本，然后安装全局Chimp:

``` 
npm install --global chimp
```

请注意, 您也可以将```Chimp```作为 ```devDependency``` 包安装在您的```package.json```中, 但您可能会在部署应用程序时遇到问题,
因为它包括二进制依赖项。您可以通过运行 ```Meteor npm prune```来避免这些问题, 以便在部署之前删除非生产依赖项。

Chimp有多种设置选项, 我们可以添加一些npm脚本,运行我们在Chimp的两种主要模式中定义的测试。我们可以将它们添加到我们的```package.json```:

``` 
{
  "scripts": {
    "chimp-watch": "chimp --ddp=http://localhost:3000 --watch --mocha --path=tests",
    "chimp-test": "chimp --mocha --path=tests"
  }
}
```

Chimp将在```tests/``` (否则被Meteor工具忽略) 文件夹中查找所定义的验收测试文件。在Todos示例应用程序中, 
我们定义了一个简单的测试, 确保我们可以单击 "创建列表" 按钮。

[tests/lists.js](https://github.com/meteor/todos/blob/master/tests/lists.js):

``` 
/* eslint-env mocha */
/* eslint-disable func-names, prefer-arrow-callback */
// These are Chimp globals
/* globals browser assert server */
function countLists() {
  browser.waitForExist('.list-todo');
  const elements = browser.elements('.list-todo');
  return elements.value.length;
};
describe('list ui', function () {
  beforeEach(function () {
    browser.url('http://localhost:3000');
    server.call('generateFixtures');
  });
  it('can create a list @watch', function () {
    const initialCount = countLists();
    browser.click('.js-new-list');
    assert.equal(countLists(), initialCount + 1);
  });
});
```

### 运行验收测试

要运行验收测试, 我们只需要像往常一样启动我们的Meteor应用程序, 并将Chimp指向它。

在一个终端运行：
``` 
meteor
```

在另一个端终运行：
``` 
meteor npm run chimp-watch
```

然后, ```chimp-watch```命令将在浏览器中运行该测试, 并在更改测试或应用程序时继续进行重新运行。(请注意, 测试假定我们在端口3000上运行应用程序)。

因此, 这是一个很好的开发测试的方法。这为什么Chimp会有一个特性的原因, 我们可以用 "@watch" 来标记要调用的测试(在一个大的应用程序中运行我们的整个验收测试套件可能会耗费大量时间)。

```chimp-test```命令将只运行一次所有的测试。这对测试很有好处, 不管我们的套件是通过手动步骤测试，或作为连续集成过程的一部分。

### 创建数据

虽然我们可以在"纯"Meteor应用程序的上运行验收测试, 正如我们在上面所做的。
但启动一个```meteor server``` 和一个特殊的测试驱动程序(test driver),```tmeasday:acceptance-test-driver```，这种做法更好。
(您需要运行```Meteor add```将其添加到您的应用程序):

``` 
meteor test --full-app --driver-package tmeasday:acceptance-test-driver
```

在(运行于完整应用程序测试模式的)应用程序上运行验收测试套件的优势是, (我们已经创建的)所有的数据生成方法仍然可用。
否则,```acceptance-test-driver```不做任何事情。

在Chimp测试中, 您有一个在服务器上可用的DDP连接(到服务器)变量。因此, 您可以使用 "server.call()" (在Chimp测试中被包装为同步) 来调用这些方法。
这是在验收测试和集成测试之间共享数据准备代码的一种简便方法。

##  持续集成

持续集成测试是在项目的每个提交上运行测试的过程。

有两种主要的方法: 在开发人员的计算机上, 在允许他们将代码推送到中央存储库之前；在专用的CI服务器上, 在每次推送代码之后。
这两种技术都很有用, 而且都要求以命令行的方式运行测试。

### 命令行

我们已经看到了一个使用```meteor npm run chimp-tes```模式在命令行上运行测试的例子。

我们也可以使用Mocha的命令行驱动 [meteortesting:mocha](https://atmospherejs.com/meteortesting/mocha) 在命令行运行我们的标准测试。

添加和使用包非常简单:

``` 
meteor add meteortesting:mocha
meteor test --once --driver-package meteortesting:mocha
```

(```--once```参数确保在测试完成后Meteor过程停止)。

我们还可以将该命令添加到我们的 "package.json" 中作为测试脚本:

``` 
{
  "scripts": {
    "test": "meteor test --once --driver-package meteortesting:mocha"
  }
}
```

现在我们可以使用命令行```meteor npm test```运行测试了。

### CircleCI

[CircleCI](https://circleci.com/) 是一个很棒的持续集成服务, 它允许我们在每次推送到存储库 (如 GitHub) 上时运行 (可能耗时) 的测试。
要使用上面定义的命令测试, 我们可以按照他们的[标准入门教程](https://circleci.com/docs/getting-started), 并使用类似于下面的```circle.yml```文件:

``` 
machine:
  node:
    version: 0.10.43
dependencies:
  override:
    - curl https://install.meteor.com | /bin/sh
    - npm install
checkout:
  post:
    - git submodule update --init
```

