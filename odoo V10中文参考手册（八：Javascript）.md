# Javascript

## Widgets

从`web.Widget`输出class Widget()，是所有可视组件的基类，相当于mvc的view层，提供一系列的处理页面的方法

- 处理多widget之间的继承、被继承关系
- 提供可扩展的生命周期安全管理（当父类被destruct时自动将对应子类清除）
- 自动使用qweb引擎渲染
- 与backbone兼容的快捷方法

### DOM根元素

Widget()负责的是根DOM下的一部分widget页面,可以通过两个属性来获取widget的DOM：

- Widget.el - widget对应的原始根DOM
- Widget.$el - 使用jQuery选择的el

有两种方法来定义生成DOM根元素：

- Widget.template - qweb的模板名，指定模板会在widget初始化之后、实际渲染之前渲染。该模板生成的根元素就会作为对应widget的DOM根元素
- Widget.tagName - 当没有设置模板名的时候使用，默认是`div`，它会被设置成widget的DOM根元素，可以通过以下属性来自定义对应DOM根元素：
- Widget.id - 在DOM根元素上生成一个id属性
- Widget.className - 在DOM根元素上生成一个class属性
- Widget.attributes - 属性映射表，会自动将里面的键值对设置为根元素的对应属性
- 当设置了模板名，上述参数将不能使用
- Widget.renderElement() - 可以通过此方法来渲染widget的根DOM并设置，使用的是template或tagName，并调用setElement() 来设置
- 可以覆盖renderElement() 方法来实现自定义的渲染，但是如果没有在里面调用`__super`的话必须要调用setElement() 方法

#### 使用widget

widget的生命周期分三个阶段：

- 1.创建并初始化
   Widget.init(parent) - widget的初始化方法，可以接收更多的参数来覆盖父级widget

> 参数：
>  parent (Widget()) - 新创建的widget的父级，如果某widget没有父级可传`null`

- 2.注入DOM并启动，通过调用以下方法中的一个来完成
- Widget.appendTo(element) - 渲染widget并用jquery的appendTo添加到对应DOM最后一个后元素前
- Widget.prependTo(element) 渲染widget并用jquery的prependTo插入到对应DOM第一个子元素前
- Widget.insertAfter(element) - 渲染widget并用jquery的insertAfter添加到对应dom之后
- Widget.insertBefore(element) - 渲染widget并用jquery的insertBefore添加到对应dom之前

上述方法接收的参数和对应jquery方法接收的参数一致，会返回一个 deferred延迟执行对象，并赋予三个任务：

> 1.使用renderElement()来渲染widget的根元素
>  2.用对应的jquery方法将widget插入到dom中
>  3.启动widget并将启动的结果返回
>  `Widget.start()`:当widget被插入到DOM之后异步启动，一般用于异步的rpc调用以获取远端数据用于widget中，完成后需要返回一个deferred对象。在start方法执行完成之前widget的功能不一定是完整的。

- 3.销毁并清除widget对象
   Widget.destroy() - 销毁它的子类，解绑所有事件，将它的根元素从DOM移除。当父widget被销毁时会自动调用，如果它没有父类 或需要将当前widget移除但保留父级widget时 就必须显示调用

与widget销毁相关的函数：

- `Widget.alive(deferred[, reject=false])`
   由于RPC调用一般比较耗时，可能在它执行完成的时候widget已经被销毁了，这时在会在一个无效的widget对象上做操作，alive可用于处理rpc调用，并保证RPC调用返回后只在有效的widget上执行对应操作：

```php
this.alive(this.model.query().all()).then(function (records) {
    // would break if executed after the widget is destroyed, wrapping
    // rpc in alive() prevents execution
    _.each(records, function (record) {
        self.$el.append(self.format(record));
    });
});
```

> 参数：
>  deferred  - deferred对象
>  reject  - 默认情况下如rpc调用完成后widget已被销毁的话对应的deferred对象只是被封锁了，如果设置为True的话会将其调用拒绝

- `Widget.isDestroyed()`
   如果widget已经被销销毁了，会返回true，否则返回false

#### 获取DOM内容

由于widget负责其DOM元素下的内容，可以用一个简便的方法去获取它DOM元素内的子片段：
 `Widget.$(selector)` 将css选择器应用到widget的根DOM上

```kotlin
this.$(selector);
相当于this.$el.find(selector);
```

#### 重置DOM根元素

`Widget.setElement(element)`
 将widget的根DOM设置为指定的DOM，参数element需为一个DOM元素或相应的jquery对象

### DOM事件处理

widget一般需要在相应页面内响应用户的动作，这需要通过将事件绑定到DOM元素上来实现。

- Widget.events
   事件是一个事件选择器（事件名和css选择器之间以空格分开）- 回调函数的映射，回调函数可以是widget内置函数或一个函数对象，this表示相应widget

```jsx
events: {
    'click p.oe_some_class a': 'some_method',
    'change input': function (e) {
        e.stopPropagation();
    }
},
```

回调函数只会被对应根DOM的匹配子元素触发。如果事件选择器留空的话，该事件是会被自动绑定到widget的根DOM上。

- Widget.delegateEvents()
   该方法用于将事件绑定到DOM，当设置好widget的根dom后会自动被调用，可以通过重写它来设置比events映射表指定的更为复杂的事件，但父级方法必须被显式调用，否则events不会被处理。
- Widget.undelegateEvents()
   用于在dom被销毁或重设时解绑events，当delegateEvents被覆盖时，它也需要进行覆盖。

> 该方法需要与backbone的delegateEvents相兼容

### Widget子类

可以通过extend来创建Widget()的子类，并提供了一些抽象方法和具体方法用于使用。

```kotlin
var MyWidget = Widget.extend({
    // 渲染对象时使用的qweb模板
    template: "MyQWebTemplate",
    events: {
        // 事件绑定示例
        'click .my-button': 'handle_click',
    },

    init: function(parent) {
        this._super(parent);
        // 在渲染之前执行的内容
        // initialization
    },
    start: function() {
        var sup = this._super();
        // 渲染初始化逻辑

        // 允许多重deferred对象
        return $.when(
            // 从父类获取异步信号
            sup,
            // 返回自己的异步信号
            this.rpc(/* … */))
    }
});


##使用
// 创建实例
var my_widget = new MyWidget(this);
// 渲染并插入到dom
my_widget.appendTo(".some-div");

##销毁
my_widget.destroy();
```

### 开发规范

- 避免使用id属性，用id会让部件重用变得很麻烦。可以使用class、dom节点或jquery来替代使用。如果一定要用的情况下，需要使用_.uniqueId()来特别声明：`this.id = _.uniqueId('my-widget-')`
- 避免使用很通用的css名如content、navigation，以防止命名冲突。一般以根据它对应的部分来命名。
- 避免使用全局选择器，因为某个组件可能在同个页面重复使用如仪表板，像$(selector) or document.querySelectorAll(selector) 可能会导致错误的操作，使用widget对应的`($el)`或`$()`来选择
- 不要认为你的部件拥有或控制它自己的`$el`
- html模板和渲染需要合用qweb
- 所有用于显示信息或处理事件的交互性质的组件必须继承自Widget() 并且使用它的api正确的实现

## RPC

为了进行显示和交互，需要使用rpc与odoo服务器通信，odoo提供两种api来处理：

- 底层基于JSON rpc与模块对应python片段通信
- 高级别的直接odoo模块调用
   所有api都是异步调用的，所以它们都会返回deferred延期执行对象

### 高级别API 直接调用odoo模块

通过`Model()`来访问odoo的对象方法，通过call方法(来自web.Model)和 query方法(来自web.DataModel)来访问odoo服务器对象

- `call()`是直接被映射到odoo服务端对象的同名方法的，用法跟odoo模型的api使用只有以下三点区别：
- 交互是异步的，所以在rpc中是返回的deferred延期对象（它们会自己处理对应rpc调用结果）
- 由于javascript规范没有`__getattr__`和`method_missing`特性，需要指定调度rpc的方法
- 没有池的概念，当需要用的时候就实例化model代理，而不是从另一个（如全局）中获取

```jsx
var Users = new Model('res.users');

Users.call('change_password', ['oldpassword', 'newpassword'],
                  {context: some_context}).then(function (result) {
    // do something with change_password result
});
```

- `query()`方法是搜索（odoo的search和read）的接口，返回一个`Query()`对象，该对象不可改变但是可以基于它创建新的query对象，添加新的属性和方法到原始对象。

```jsx
Users.query(['name', 'login', 'user_email', 'signature'])
     .filter([['active', '=', true], ['company_id', '=', main_company]])
     .limit(15)
     .all().then(function (users) {
    // do work with users records
});
```

query在调用`all()`或`first()`方法之前是不会实际执行的，这两个方法每次调用都会触发一个rpc请求。可以用来进行实时查询。

#### class Model(name)

- Model.name 对象所绑定的model名
- Model.call(method[, args][, kwargs]) 使用对应参数调用当前模型的对应方法

> 参数：

1. method (String)  通过rpc调用的模型方法
2. args (Array<>) 位置匹配的参数列表
3. kwargs (Object<>) 传递的关键字参数

- Model.query(fields)

> 参数：fields (Array<String>)  搜索时需要获取的字段列表

#### class odoo.web.Query(fields)

##### **第一部分方法是读取方法，它们使用所调用对象的数据来响应rpc请求**

- odoo.web.Query.all() - 读取当前Query对象所对应的数据，返回一个deferred的数组
- odoo.web.Query.first() - 读取当前Query对象对应的第一个 结果，如果没有结果返回null，有结果返回deferred对象
- odoo.web.Query.count() - 获取当前Query对象可得到的记录数量
- odoo.web.Query.group_by(grouping...) - 获取查询数据的分组，

> 参数：    grouping (Array<String>) - 分组列表
>  返回：    `Deferred<Array<odoo.web.QueryGroup>> | null`

##### **第二部分方法是设置方法，它们会创建一个新Query对象并对相关属性进行扩展或替换**

- odoo.web.Query.context(ctx) 添指定的环境变量添加到搜索中
- odoo.web.Query.filter(domain) 将指定的domain表达式添加到查询条件中，通过and与已存在的domain联接
- odoo.web.Query.offset(offset)  设置查询的起始位置，会将原有的offset替换
- odoo.web.Query.limit(limit) 设置查询的数据量，会将原来的limit替换
- odoo.web.Query.order_by(fields…) 覆盖原来的排序规则，像Django的QuerySet.order_by一样
- 接收多个排序字段，按重要性高到低排序，第一个优先级最高，字段以字符串提供
- 每个字段默认是按升序，可以在字段前加`-`表示倒序
   与django不同，它没有用?来进行随机乱序和对关联字段排序方法

**分组聚合 **
odoo有非常强大的分组运算功能，但是它是递归的，并且第N+1层依赖于第n层所提供的数据，所以当 odoo.models.Model.read_group()工作时这个api就不是那么直观。
odoo一般用Query()方法来代替 read_group()方法。

```jsx
some_query.group_by(['field1', 'field2']).then(function (groups) {
    // do things with the fetched groups
});
```

该方法可以接受一个字段列表参数、也可以不带参数执行，无参数时直接返回null而不是deferred对象
 当分组条件从其他地方来的时候，可以通过两种方法来测试：

- 对group_by所得到的结果进行检查：

```jsx
var groups;
if (groups = some_query.group_by(gby)) {
    groups.then(function (gs) {
        // groups
    });
}
// no groups
```

- 使用`when()`将返回值强制转换为deferred对象

```jsx
$.when(some_query.group_by(gby)).then(function (groups) {
    if (!groups) {
        // No grouping
    } else {
        // grouping, even if there are no groups (groups
        // itself could be an empty array)
    }
});
```

成功的情况下group_by返回的结果是一个 QueryGroup()数组

##### `class odoo.web.QueryGroup()` 返回分组的属性key，有以下几种

- grouped_on -- 基于哪个字段进行分组计算
- value -- 当前分组的grouped_on的值
- length -- 分组内的记录数量
- aggregates -- 分组聚合结果的 {field: value} 映射

##### `odoo.web.QueryGroup.query([fields...])` 相当于Model.query() ，但只包含当前分组内的记录，返回一个Query对象供后续使用

##### `odoo.web.QueryGroup.subgroups()` 返回一个指向当前的子QueryGroup()的数组的deferred对象

### 底层API：RPC调用python程序

`Session()`对象（通过web.Session实例化）的rpc方法提供了一个低级别api用来直接调用python程序，该方法接收一个完整URL、一个参数key=>value映射表 作为参数，并将对应获取的结果转换成json格式

```jsx
session.rpc('/web/dataset/resequence', {
    model: some_model,
    ids: array_of_ids,
    offset: 42
}).then(function (result) {
    // resequence didn't error out
}, function () {
    // an error occured during during call
});
```

## web client

### javascript模块系统

从odoo v8开始使用一套跟requirejs类似的js模块系统，它有以下优点：

- 依赖关系可以保证按顺序加载
- 更容易将文件分割成更小的逻辑单元
- 没有全局变量
- 很容易检查依赖关系，让重构变得容易很多

缺点：

- 如果要通过odoo交互就必须用模块系统加载，因为有很多对象只在模块系统中能用
- 不支持循环依赖

在这种模式下，通过require来导入需要的模块，并且显示声明所输出的对象。

```jsx
odoo.define('addon_name.service', function (require) {
    var utils = require('web.utils');
    var Model = require('web.Model');

    // do things with utils and Model
    var something_useful = 15;
    return  {
        something_useful: something_useful,
    };
});
```

上面的代码创建了一个名叫addon_name.service的模块，使用odoo.define函数定义。

> odoo.define函数有两个参数：
>  1.name - 新定义的模块名

2.function - 在里面定义该模块实际包含的内容，接收一个require参数，如果需要输出内容就需要有对应返回，require函数用于获取依赖的模块。

用javascript来导入需要的模块就声明输出内容，web客户端会自动进行加载。模块在文件中定义，一般最好一个文件对应一个模块。模块可以返回一个deferred对象，这样该模块只在deferred执行后才加载，而且模块可以被废弃，并在控制台记录对应信息：

> Missing dependencies - 该模块不会出现在页面中，可能是javascript文件不在页面中或模块名有错误

Failed modules - 有javascript错误
 Rejected modules - 模块返回的是一个废弃的deferred
 Rejected linked modules - 该模块依赖于已废弃的模块
 Non loaded modules - 模块所依赖的模块不存在或有错误

### Web client结构

- `framework/`文件夹包含所有底层的模块
- `web.ajax` 用于处理rpc调用
- `web.core` 核心模块，给出很多有用的对象如`qweb`,`_t`
- `web.Widget` 包含widget类
- `web.Model` 抽象化的`web.ajax`，用于直接调用服务端模型的方法
- `web.session` 如`odoo.session`
- `web.utils` 有用的代码段
- `web.time` 时间相关函数
- `views/`文件夹包含所有视图定义
- `widgets/`包含独立的部件
- `js/`文件夹包含一些重要的文件
- `action_manager.js` ActionManager 类
- `boot.js` 模块系统的入口
- `menu.js` 顶级菜单的定义
- `web_client.js` 部件WebClient
- `view_manager.js` 包含ViewManager

还有其他两个文件：`tour.js`用于tour，`compatibility.js`用于将旧系统与新系统兼容，在这个文件中每个模块名被输出到全局变量odoo中。理论上模块可以不通过变量odoo使用。

#### javascript习惯

- 在模块最前面声明所有依赖关系，一般按模块名字母顺序来排列
- 在最后声明所有输出
- 在模块开始时添加`use strict` 声明
- 以合适的名字命名模块如：`addon_name.description`
- 类名首字母大写如：ActionManager ，但其他的小写如web.ajax
- 每个文件只定义一个模块

## 在 Odoo Web 客户端中测试

### Javascript 单元测试 

Odoo Web 包括对 Odoo Web 的核心代码和您自己的 javascript 模块进行单元测试的方法。在 javascript  方面，单元测试基于 QUnit，带有许多帮助程序和扩展，以便更好地与 Odoo 集成。  要查看运行器的外观，请找到（或启动）启用了 Web 客户端的 Odoo 服务器，然后导航到 /web/tests  这将显示运行器选择器，其中列出了所有带有 javascript 单元测试的模块，并允许启动任何它们（或一次在所有模块中进行所有  javascript 测试）。

![img](https://www.odoo.com/documentation/10.0/_images/runner.png)

单击任何运行器按钮将在捆绑的 QUnit 运行器中启动相应的测试：

![img](https://www.odoo.com/documentation/10.0/_images/tests.png)

### 编写测试用例 

第一步是列出测试文件。这是通过 Odoo 清单的`test`键完成的，通过向其中添加 javascript 文件：

```json
{ 
    'name': "Demonstration of web/javascript tests",
    'category' :  'Hidden' , 
    'depends' :  [ 'web' ], 
    'test' :  [ 'static/test/demo.js' ], 
} 
```

并创建相应的测试文件 

>笔记
>
>不存在的测试文件将被忽略，如果一个模块的所有测试文件都被忽略（找不到），测试运行器将认为该模块没有javascript测试。

之后，刷新 runner 选择器将显示新模块 并允许运行其所有（到目前为止为 0）测试： 

![img](https://www.odoo.com/documentation/10.0/_images/runner2.png)

下一步是创建一个测试用例： 

```kotlin
odoo.testing.section('basic section', function (test) {
    test('my first test', function () {
        ok(false, "this test has run");
    });
});
```

所有测试助手和结构都位于 `odoo.testing` 模块中。 Odoo 测试存在于` section()` 中，它本身就是模块的一部分。节的第一个参数是节的名称，第二个是节体。 

由 `section()` 提供给回调的 `test` 用于注册给定的测试用例，只要测试运行器实际执行其工作，就会运行该测试用例。 Odoo Web 测试用例在其中使用`标准 QUnit 断言`。  

此时启动测试运行器将运行测试并显示相应的断言消息，红色表示测试失败： 

![img](https://www.odoo.com/documentation/10.0/_images/tests2.png)

修复测试（通过在断言中将 false 替换为 true）将使其通过：

![img](https://www.odoo.com/documentation/10.0/_images/tests3.png)

### 断言 

如上所述，Odoo Web 的测试使用 `qunit 断言`。它们是全局可用的（因此它们可以在不引用任何内容的情况下被调用）。以下列表可用：

- ok(*state*[, *message*])

​       检查状态是否为真（在 javascript 意义上）

- strictEqual(*actual*, *expected*[, *message*])

  检查实际值（由被测试的方法产生）和预期值是否相同（大致相当于 ok(actual === expected, message)）

  

- notStrictEqual(*actual*, *expected*[, *message*])

​       检查实际值和预期值是否*不相同* （大致相当于 `ok(actual !== expected, message)`) 

- `deepEqual(actual, expected[, message])`

​       实际和预期之间的深入比较：递归到 容器（对象和数组）以确保它们具有相同的 键/元素数量，并且值匹配。 

- `notDeepEqual(actual, expected[, message])`

​       逆运算 [`deepEqual()`](https://www.odoo.com/documentation/10.0/reference/javascript.html#deepEqual)

- `throws(block[, expected][, message])`

​       检查该块在调用时是否抛出错误。 （可选）根据预期验证该错误。

>- Arguments 
> - **block (Function)** --
> - **expected (Error | RegExp)** --如果是正则表达式，则检查抛出的错误消息是否与正则表达式匹配。如果是错误类型，则检查抛出的错误是否属于该类型。
>

- `equal(actual, expected[, message])`

​      使用 == 运算符及其强制规则检查实际和预期是否大致相等。 

- `notEqual(actual, expected[, message])`

​      逆运算 [`equal()`](https://www.odoo.com/documentation/10.0/reference/javascript.html#equal)

### 获取 Odoo 实例 

Odoo 实例是访问大多数 Odoo Web  模块行为（函数、对象等）的基础。因此，测试框架会自动构建一个，并加载被测试的模块及其内部的所有依赖项。这个新实例作为第一个位置参数提供给您的测试用例。让我们通过在测试模块中添加 javascript 代码（不是测试代码）来观察：

```python
{
    'name': "Demonstration of web/javascript tests",
    'category': 'Hidden',
    'depends': ['web'],
    'js': ['static/src/js/demo.js'],
    'test': ['static/test/demo.js'],
}
```

```js
// src/js/demo.js
odoo.web_tests_demo = function (instance) {
    instance.web_tests_demo = {
        value_true: true,
        SomeType: instance.web.Class.extend({
            init: function (value) {
                this.value = value;
            }
        })
    };
};
```

然后添加一个新的测试用例，它只是检查`instance`是否包含我们在模块中创建的所有预期内容：

```js
// test/demo.js
test('module content', function (instance) {
    ok(instance.web_tests_demo.value_true, "should have a true value");
    var type_instance = new instance.web_tests_demo.SomeType(42);
    strictEqual(type_instance.value, 42, "should have provided value");
});
```

### DOM 便笺簿 

与更广泛的客户端一样，在测试期间强烈不鼓励随意访问文档内容。但是仍然需要 DOM 访问，例如在测试之前完全初始化`widgets`。 

因此，在 jQuery 实例中，测试用例获取 DOM 暂存器作为其第二个位置参数。在每次测试之前，该暂存器都会被完全清理干净，只要它不在暂存器之外执行任何操作，您的代码就可以为所欲为：

```js
// test/demo.js
test('DOM content', function (instance, $scratchpad) {
    $scratchpad.html('<div><span class="foo bar">ok</span></div>');
    ok($scratchpad.find('span').hasClass('foo'),
       "should have provided class");
});
test('clean scratchpad', function (instance, $scratchpad) {
    ok(!$scratchpad.children().length, "should have no content");
    ok(!$scratchpad.text(), "should have no text");
});
```

> 笔记 
>
> 暂存器的顶级元素没有清理，测试用例可以添加文本或 DOM 子项，但不应更改 `$scratchpad` 本身。

### 加载模板 

为避免相应的处理成本，QWeb 中默认不加载模板。如果您需要渲染，例如使用 QWeb 模板的小部件，您可以通过`tempalte`选项请求将它们加载到`test case function`。

这将在运行测试用例之前自动加载实例的 qweb 中的所有相关模板：

```python
{
    'name': "Demonstration of web/javascript tests",
    'category': 'Hidden',
    'depends': ['web'],
    'js': ['static/src/js/demo.js'],
    'test': ['static/test/demo.js'],
    'qweb': ['static/src/xml/demo.xml'],
}
```

```xml
<!-- src/xml/demo.xml -->
<templates id="template" xml:space="preserve">
    <t t-name="DemoTemplate">
        <t t-foreach="5" t-as="value">
            <p><t t-esc="value"/></p>
        </t>
    </t>
</templates>
```

```js
// test/demo.js
test('templates', {templates: true}, function (instance) {
    var s = instance.web.qweb.render('DemoTemplate');
    var texts = $(s).find('p').map(function () {
        return $(this).text();
    }).get();

    deepEqual(texts, ['0', '1', '2', '3', '4']);
});
```



### 异步案例 

到目前为止的测试用例都是同步的，它们从第一行到最后一行执行，一旦最后一行执行完毕，测试就完成了。但是 Web 客户端充满了异步代码，因此测试用例需要`asynchronous code`。  这是通过从 case 回调中返回一个 `deferred` 来完成的：

```js
// test/demo.js
test('asynchronous', {
    asserts: 1
}, function () {
    var d = $.Deferred();
    setTimeout(function () {
        ok(true);
        d.resolve();
    }, 100);
    return d;
});
```

此示例还使用 `options parameter `来指定案例应该期望的断言数量，如果指定的断言更少或更多，则案例将被视为失败。  异步测试用例必须指定它们将运行的断言数量。这允许更容易地捕捉情况，例如测试架构没有被警告异步操作。

>笔记
>
>异步测试用例也有 2 秒的超时时间：如果测试没有在 2 秒内完成，则视为失败。这几乎总是意味着测试不会解决。此超时仅适用于测试本身，不适用于设置和拆卸过程。
>
>如果返回的延迟被拒绝，则测试将失败，除非将` fail_on_rejection` 设置为 ==false==。

### RPC 

异步测试用例的一个重要子集是需要执行（并在一定程度上链接）RPC 调用的测试用例

> 笔记 
>
> 因为它们是异步案例的子集，RPC 案例还必须提供有效的`assertions count`。

要启用模拟 RPC，请将 `rpc option`设置为`mock`。这将向测试用例回调添加第三个参数：

- mock(*rpc_spec*, *handler*)

     根据第一个参数的形状，可以以两种不同的方式使用： 

  - 如果它匹配模式 `model:method`（如果它包含一个冒号，本质上），该调用将设置直接到 Odoo 服务器（通过 XMLRPC）的 RPC 调用的模拟，如通过例如执行的那样`odoo.web.Model.call()`。  在这种情况下，`handler`应该是一个带有两个参数`args` 和 `kwargs` 的函数，匹配服务器端的相应参数，并且应该简单地返回值，就像它由 Python XMLRPC 处理程序返回一样：

  ```js
  test('XML-RPC', {rpc: 'mock', asserts: 3}, function (instance, $s, mock) {
      // set up mocking
      mock('people.famous:name_search', function (args, kwargs) {
          strictEqual(kwargs.name, 'bob');
          return [
              [1, "Microsoft Bob"],
              [2, "Bob the Builder"],
              [3, "Silent Bob"]
          ];
      });
  
      // actual test code
      return new instance.web.Model('people.famous')
          .call('name_search', {name: 'bob'}).then(function (result) {
              strictEqual(result.length, 3, "shoud return 3 people");
              strictEqual(result[0][1], "Microsoft Bob",
                  "the most famous bob should be Microsoft Bob");
          });
  });
  ```

  

  - 否则，如果它匹配绝对路径（例如 `/a/b/c`），它将模拟对 Web 客户端控制器的 JSON-RPC 调用，例如 ` /web/webclient/translations`。在这种情况下，处理程序采用单个 `params `参数，其中包含通过 JSON-RPC  提供的所有参数。  

    如前所述，处理程序应该简单地返回结果值，就像由原始 JSON-RPC 处理程序返回一样：

  ```js
  test('JSON-RPC', {rpc: 'mock', asserts: 3, templates: true}, function (instance, $s, mock) {
      var fetched_dbs = false, fetched_langs = false;
      mock('/web/database/get_list', function () {
          fetched_dbs = true;
          return ['foo', 'bar', 'baz'];
      });
      mock('/web/session/get_lang_list', function () {
          fetched_langs = true;
          return [['vo_IS', 'Hopelandic / Vonlenska']];
      });
  
      // widget needs that or it blows up
      instance.webclient = {toggle_bars: odoo.testing.noop};
      var dbm = new instance.web.DatabaseManager({});
      return dbm.appendTo($s).then(function () {
          ok(fetched_dbs, "should have fetched databases");
          ok(fetched_langs, "should have fetched languages");
          deepEqual(dbm.db_list, ['foo', 'bar', 'baz']);
      });
  });
  ```

>笔记
>
>模拟处理程序可以包含断言，这些断言应该是断言计数的一部分（如果对包含断言的处理程序进行多次调用，它会乘以有效的断言数）。

### API测试 

- odoo.testing.section(*name*, [*options*, ]*body*)

测试部分用作相关测试的共享命名空间（常量或值仅设置一次）。身体功能应该包含测试本身。  请注意，测试运行的顺序基本上是未定义的，不要依赖它。

>Arguments
>
>- **name** (`String`) -- 
>- **options** ([`TestOptions`](https://www.odoo.com/documentation/10.0/reference/javascript.html#TestOptions)) -- 
>- **body** (Function<[`case()`](https://www.odoo.com/documentation/10.0/reference/javascript.html#odoo.testing.case), void>) -- 

- odoo.testing.case(*name*, [*options*, ]*callback*)

  在测试运行器中注册一个测试用例回调，回调只会在运行器启动后运行（或者可能根本不运行，如果测试被过滤掉）。

  >Arguments
  >
  >- **name** (`String`) -- 
  >- **options** ([`TestOptions`](https://www.odoo.com/documentation/10.0/reference/javascript.html#TestOptions)) -- 
  >- **callback** (`Function<instance, $, Function<String, Function, void>>`) -- 

- *class* TestOptions()

  可以传递给 `section()` 或 `case() `的各种选项。除了`setup`和`teardown`外， `case() `上的选项将覆盖 `section() ` 上的相应选项，例如可以为 `section()` 设置` rpc`，然后为该 `section()` 的某些 `case()` 设置不同的设置

  - TestOptions.asserts

    一个整数，在正常执行测试期间应该运行的断言数。对于异步测试是必需的。

  - TestOptions.setup

  - 测试用例设置，在每个测试用例之前运行。如果指定了两个部分，则部分的` setup() `在案例自己的之前运行。

  - TestOptions.teardown

    测试用例拆解，如果两者都存在，则用例的`teardown() `在相应部分之前运行。

  - TestOptions.fail_on_rejection

    如果测试是异步的并且其结果承诺被拒绝，则测试失败。默认为 `true`，设置为` false` 以在拒绝的情况下不通过测试：

    ```js
    // test/demo.js
    test('unfail rejection', {
        asserts: 1,
        fail_on_rejection: false
    }, function () {
        var d = $.Deferred();
        setTimeout(function () {
            ok(true);
            d.reject();
        }, 100);
        return d;
    });
    ```

  - TestOptions.rpc

    在测试期间使用的 RPC 方法，`“mock”`或`“rpc”`之一。任何其他值都将禁用测试的 RPC（例如，如果套件启用了它们）。

  - TestOptions.templates

    在开始测试之前是否应将当前模块（及其依赖项）的模板加载到 QWeb 中。布尔值，默认为 `false`。

测试运行器还可以使用直接在 `window` 对象上设置的两个全局配置值：

- `oe_all_dependencies` 是一个包含所有具有 web 组件的模块的`Array`，按依赖关系排序（对于具有依赖关系 `A' `的模块`A`，`A'` 的任何模块必须在数组中出现在` A `之前）

### 通过 Python 运行 

Web 客户端包括在命令行（或在 CI 系统中）运行这些测试的方法，但在实际运行时，它非常简单，先决条件部分的设置有一些复杂性。      

1. 安装 `PhantomJS`。它是一个无头浏览器，允许自动运行和测试网页。 QUnitSuite 使用它来实际运行 `qunit` 测试套件。      

   `PhantomJS` 网站为某些平台提供了预构建的二进制文件，您的操作系统的包管理可能也提供了它。      

   如果你从源代码构建 `PhantomJS`，我建议你准备一些编织时间，因为它不是很快（它需要编译` Qt `和 `Webkit`，这两个都是相当大的项目）。

   >笔记
   >
   >因为 `PhantomJS` 是基于 webkit 的，所以它无法测试 Firefox、Opera 或 Internet Explorer  是否可以正确运行测试套件（并且它只是 Safari 和 Chrome 的近似值）。因此，建议偶尔在实际浏览器中运行测试套件。
   >
   >构建的 `PhantomJS`版本是 1.7，以前的版本应该可以工作，但实际上不受支持（并且当 `PhantomJS` 本身出现问题时往往只是段错误，因此调试起来很麻烦）。

2. 安装一个包含所有相关模块的新数据库（所有模块至少有一个 web 组件），然后重新启动服务器 

   > 笔记  
   >
   > 对于某些测试，需要复制源数据库。此操作要求与被复制的数据库没有连接，但 Odoo 当前不会中断现有/未完成的连接，因此重新启动服务器是确保一切都处于正确状态的最简单方法。

3. 使用指定的正确插件路径启动 `oe run-tests -d $DATABASE -mweb`（并用您在上面创建的源数据库替换 `$DATABASE`）

   >笔记  
   >
   >如果您省略 `-mweb`，运行程序将尝试运行所有模块中的所有测试，这可能会也可能不会。

如果一切顺利，您现在应该会看到一个测试列表，在他们的名字旁边（希望）ok，最后会报告运行的测试数量和花费的时间： 

```kotlin
test_empty_find (odoo.addons.web.tests.test_dataset.TestDataSetController) ... ok
test_ids_shortcut (odoo.addons.web.tests.test_dataset.TestDataSetController) ... ok
test_regular_find (odoo.addons.web.tests.test_dataset.TestDataSetController) ... ok
web.testing.stack: direct, value, success ... ok
web.testing.stack: direct, deferred, success ... ok
web.testing.stack: direct, value, error ... ok
web.testing.stack: direct, deferred, failure ... ok
web.testing.stack: successful setup ... ok
web.testing.stack: successful teardown ... ok
web.testing.stack: successful setup and teardown ... ok

[snip ~150 lines]

test_convert_complex_context (odoo.addons.web.tests.test_view.DomainsAndContextsTest) ... ok
test_convert_complex_domain (odoo.addons.web.tests.test_view.DomainsAndContextsTest) ... ok
test_convert_literal_context (odoo.addons.web.tests.test_view.DomainsAndContextsTest) ... ok
test_convert_literal_domain (odoo.addons.web.tests.test_view.DomainsAndContextsTest) ... ok
test_retrieve_nonliteral_context (odoo.addons.web.tests.test_view.DomainsAndContextsTest) ... ok
test_retrieve_nonliteral_domain (odoo.addons.web.tests.test_view.DomainsAndContextsTest) ... ok

----------------------------------------------------------------------
Ran 181 tests in 15.706s

OK
```

恭喜，您刚刚成功执行了 OpenERP Web 测试套件的“离线”运行。

>笔记 
>
>请注意，这将运行 web 模块的所有 Python 测试，但运行所有 Odoo 的所有 `web` 测试。这可能令人惊讶。

