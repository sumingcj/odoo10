QWeb是一个基于xml的模板引擎，用于生成HTML片段和页面，模板指令是写在xml标签中的以t-开头的属性，比如t-if
 如果要让一个标签不被渲染，可以采用t来包裹，这样会执行它里面的命令但是不产生任何输出

```xml
<t t-if="condition">
    <p>Test</p>
</t>
#condtition为true时上述代码会输出：<p>Test</p>

<div t-if="condition">
    <p>Test</p>
</div>

#condtition为true时上述代码会输出：
<div>
    <p>Test</p>
</div>
```

## 数据输出

Qweb有一个自动过滤xss和html的输出命令esc，它接受一个表达式，解析变输出结果：

```xml
<p><t t-esc="value"/></p>
#当value值为42时输出结果：
<p>42</p>
```

还有一个raw参数与esc类似，但不过滤html，用于显示处理好的html内容

## 条件语句

qweb有一个if条件判断指令，会自动解析其对应的属性值里的表达式：

```xml
<div>
    <t t-if="condition">
        <p>ok</p>
    </t>
</div>

#当condition是true的时候解析成：
<div>
    <p>ok</p>
</div>

#condition为false的时候解析成
<div>
</div>

#也可用下面的方法实现一样的功能
<div>
    <p t-if="condition">ok</p>
</div>
```

另外，`t-elif`和`t-else`可用于添加条件分支

```xml
<div>
    <p t-if="user.birthday == today()">Happy bithday!</p>
    <p t-elif="user.login == 'root'">Welcome master!</p>
    <p t-else="">Welcome!</p>
</div>
```

## 循环

Qweb有一个指令用于循环处理，t-foreach用来指定需要循环处理的数据，t-as提供的是在后面用于代表当前项目的变量名：

```xml
<t t-foreach="[1, 2, 3]" t-as="i">
    <p><t t-esc="i"/></p>
</t>
#上述语句输出：
<p>1</p>
<p>2</p>
<p>3</p>

#也可用下面的方法实现一样的功能
<p t-foreach="[1, 2, 3]" t-as="i">
    <t t-esc="i"/>
</p>
```

foreach可用于数组（当前项目即是值）、映射表（当前项目是key）、整形数字（相当于0-X的数组）

```bash
* $as_all - 被循环的对象
* `$as_value` - 当前循环的值，当处理列表和数字时与 `$as`是一样的，当处理映射表时它代表值，而`$as`代表的是键
* $as_index - 当前循环索引，第0开始计算
* $as_size  - 被循环对象的大小
* $as_first - 当前项目是否是第一个，相当于$as_index == 0
* $as_last - 当前项目是否是最后一个，相当于$as_index + 1 == $as_size
* $as_parity - 当前项目是奇数个还是偶数
* $as_even - 当前项目索引是否为奇数
* $as_odd - 当前项目索引是否为偶数
```

上述参数只在foreach里面可用，但可在循环的最后复制到全局环境中

```xml
<t t-set="existing_variable" t-value="False"/>
<!-- existing_variable now False -->

<p t-foreach="[1, 2, 3]" t-as="i">
    <t t-set="existing_variable" t-value="True"/>
    <t t-set="new_variable" t-value="True"/>
    <!-- existing_variable and new_variable now True -->
</p>

<!-- existing_variable always True -->
<!-- new_variable undefined -->
```

## 属性

qweb可以对属性进行实时计算并在输出时设置，通过t-attr来实现，有三种形式：

- `t-att-$name` 可以创建一个名为`$name`的属性，原属性的值会被解析为新生成的属性的值

```xml
<div t-att-a="42"/>  
#输出
<div a="42"></div>
```

- `t-attf-$name` 与第一个类似，但它的值是一个格式化字符串而不是表达式，一般用于字符+变量组合如：

```csharp
<t t-foreach="[1, 2, 3]" t-as="item">
    <li t-attf-class="row {{ item_parity }}"><t t-esc="item"/></li>
</t>

#输出：
<li class="row even">1</li>
<li class="row odd">2</li>
<li class="row even">3</li>
```

- `t-att=mapping` 如果参数是映射表，每个键值对会生成一个属性：

```xml
<div t-att="{'a': 1, 'b': 2}"/>
#输出
<div a="1" b="2"></div>
```

- `t-att=pair` 如果参数是元组或2个元素的数组，那么第一个项就作为属性名，第二个作为属性值

```xml
<div t-att="['a', 'b']"/>
#输出
<div a="b"></div>
```

## 设置变量

qweb允许在模板内设置变量（用于记住一个计算结果、或为数据定义一个更明确的名字）
 使用t-set来实现，它的值就是设置的变量名

- t-value属性是一个表达式，解析后的值作为新变量的值

```csharp
<t t-set="foo" t-value="2 + 1"/>
<t t-esc="foo"/>
#输出3
```

- 如果没有t-value，节点的内容会被渲染被设置成变量的值

```xml
<t t-set="foo">
    <li>ok</li>
</t>
<t t-esc="foo"/>
#输出结果
&lt;li&gt;ok&lt;/li&gt;

#内容被esc自动输义了
```

## 调用子模板

qweb可以用于最高级别的渲染，但也可以通过`t-call`来包含其他模板
 `<t t-call="other-template"/>`会调用指定名字的模板
 如果other-template是`<p><t t-value="var"/></p>` 得到的结果会是`<p/>`

```xml
<t t-set="var" t-value="1"/>
<t t-call="other-template"/>

#会输出
<p>1</p>
```

这个有一个问题，在t-call外其他位置会可见。在t-call内设置的内容会在调用子模板时先执行并更新到当前环境

```xml
<t t-call="other-template">
    <t t-set="var" t-value="1"/>
</t>
```

t-call内包含的内容可以通过一个0的魔术变量来传递给被调用的模板：

```xml
#other-template
<div>
    This template was called with content:
    <t t-raw="0"/>
</div>

#main
<t t-call="other-template">
    <em>content</em>
</t>

#output
<div>
    This template was called with content:
    <em>content</em>
</div>
```

## Python

### 专用指令：

#### 自动格式化记录

- `t-field`只能用于格式化记录字段（从browe函数获取到的），可以根据字段类型自动匹配格式；
- `t-options`只能用于自定义字段，最常用的是`widget`，其他的选项都是`field-xx`或`widget-xx`

#### 调试器（t-debug）

通过PDB 的set_trace 来进行调试，接收的参数需要是模块名字，set_trace()会在该模块上调用
 `<t t-debug="pdb"/>`相当于：importlib.import_module("pdb").set_trace()

### Helpers

#### 基于请求的Helper

python端一般是在controller里使用qweb，这样可以直接调用`odoo.http.HttpRequest.render()`来渲染保存在数据库中的模板

```bash
response = http.request.render('my-template', {
    'context_value': 42
})
#会直接从controller里返回一个响应对象
```

#### 基于视图的Helper

比上面的更深层次的helper是在`ir.ui.view:`中的render方法

- render(cr, uid, id[, values][, engine='ir.qweb][, context])
   通过view的数据库id来渲染一个qweb视图模板，模板在`ir.ui.view`记录会自动加载，它会为渲染环境设置一系列默认值
- request - 当前WebRequest对象
- debug - 当前请求是否是debug模式
- quote_plus - 是否进行url encode转义
- json - 相关的标准库
- time - 相关的标准库
- datetime - 相关的标准库
- relativedelta - model的时间处理属性
- keep_query - 一个keep_query函数，参数1：values-传递给qweb的上下文环境，参数2：engine (str) 用于qweb渲染的odoo模型名

#### API

可以直接使用ir.qweb模型来继承、扩展他的功能

## Javascript

### 专用指令

- 定义模板
   t-name只能放在模板文件的最外面，离template标签最近的地方

```xml
<templates>
    <t t-name="template-name">
        <!-- template code -->
    </t>
</templates>
```

它没有其他参数，但可以使用一个t标签，当使用t标签时它需要有单个子元素，模板名可以随便取，一般会用`.`分隔来表示继承关系

- 模板继承
   模板继承是用来修改已存在的模板，即给在其他模块定义的模板添加内容。通过`t-extend`来表示，它的值是被继承的模板名，通过t-jquery来进行修改

```xml
<t t-extend="base.template">
    <t t-jquery="ul" t-operation="append">
        <li>new element</li>
    </t>
</t>
```

`t-jquery`给出的是一个css选择器，用于选择需要改变的节点，并通过`t-operation`指定需要进行的操作

- append - 新节点的内容添加到原节点的后面（最后一个子节点后）
- prepend - 新节点内容添加到原节点前面（第一个子节点前）
- before - 新节点内容添加到原节点前
- after - 新节点内容添加到原节点后
- inner - 新节点内容替换原节点的子节点
- replace - 新节点内容直接替换原节点
   如果没有指定operation，那么模板内容会被解析成javascript节点，并将context节点设置为this

### 调试

qweb的javascript提供多种调试工具

#### t-log

接收一个表达式参数，在渲染时解析并将结果输出到控制台

```csharp
<t t-set="foo" t-value="42"/>
<t t-log="foo"/>
#在控制台输出42
```

#### t-debug

在渲染时触发一个调试断点

```xml
<t t-if="a_test">
    <t t-debug="">
</t>
```

#### t-js

该节点内容里的javascript代码会在渲染时执行，接收一个context参数，将当前的环境传给js

```csharp
<t t-set="foo" t-value="42"/>
<t t-js="ctx">
    console.log("Foo is", ctx.foo);
</t>
```

### Helpers

core.qweb(core是`web.core`模块):一个 QWeb2.Engine()实例，在这个实例中所有模块相关模板文件全部有加载，而且可使用`_`(underscore方法), `_t`(翻译方法) 和 JSON
 core.qweb.render 可以用于渲染基本模块的模板

## API

### class QWeb2.Engine()

QWeb渲染器，处理qweb的大部分逻辑如加载、解析、编译、渲染
 odoo在core模块中将user实例化，并输出到core.qweb，同时将所有模板文件加载到qweb实例中
 一个 QWeb2.Engine()同时也可以当成一个命名空间来看待

- QWeb2.Engine.render(template[, context])
   将一个模板渲染成字符串，使用context来查找渲染时所遇到的变量

> 参数：1.template (String) - 需要渲染的模板名
>  2.context (Object) - 渲染时需要用到的命名空间

### QWeb2.Engine.add_template(templates)

这个方法在某些情况下比较有用，用于自己定义一个命名空间并获得单独的 QWeb2.Engine()实例，这样不要担心与其他模块冲突。

> 该方法加载指定的模板，参数可以是以下几种：
>  1.XML字符串 -- Qweb会将它解析成xml文档并加载
>  2.URL  -- Qweb会下载对应的url内容，并加载获得的xml字符串
>  3.Document 或 Node -- qweb会将所有第一级的子节点过滤一遍，并加载命名为template或从template继承的部分

### QWeb2.Engine.prefix

在解析的时候用来识别指令的前缀，默认是`t`

### QWeb2.Engine.debug

布尔型标志表示引擎是否使用调试模式，默认情况下，qweb会把模板执行过程中产生的错误拦截，在调试模式下，所有异常都会被保留

### QWeb2.Engine.jQuery

在模板继承处理时所使用的jQuery实例，默认是`window.jQuery`

### QWeb2.Engine.preprocess_node

一个方法，当它出现时会在编译dom节点和template节点前调用，一般用于自动模板中的翻译文本内容或属性，默认是null

