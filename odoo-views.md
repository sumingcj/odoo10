

[TOC]



## 通用结构 

视图对象公开了许多字段，除非指定，否则它们是可选的 否则。 

- `name`**（mandatory）** 

  仅在查找时用作视图的助记符/描述 某种列表 

- `model`

  链接到视图的模型（如果适用）（不适用于 QWeb 视图） 

- `priority`

  客户端程序可以通过以下方式请求视图 `id`，或由 `(model, type)`.  为了 后者，将搜索正确类型和模型的所有视图， 和最低的 `priority`号码将被返回（它是 “默认视图”）。 `priority`还定义了[视图继承](https://www.odoo.com/documentation/10.0/reference/views.html#reference-views-inheritance) 期间的应用顺序

- `arch`

  视图布局的描述 

- `groups_id`

  [`Many2many`](https://www.odoo.com/documentation/10.0/reference/orm.html#odoo.fields.Many2many)允许查看/使用的组的字段 当前视图 

- `inherit_id`

  当前视图的父视图，参见 [继承 ](https://www.odoo.com/documentation/10.0/reference/views.html#reference-views-inheritance)，  默认取消设置 

- `mode`

  继承模式，参见 [继承 ](https://www.odoo.com/documentation/10.0/reference/views.html#reference-views-inheritance)。 如果  `inherit_id`是取消设置 `mode`只能是 `primary`.  如果 `inherit_id`设置， `extension`默认情况下，但可以明确设置 到 `primary`

- `application`

  定义可切换视图的网站功能。  默认情况下，视图总是 已申请 



## 继承

### 视图匹配 

- 如果视图被请求 `(model, type)`，具有正确模型的视图 并键入， `mode=primary`并且匹配最低优先级 
- 当视图被请求时 `id`, 如果它的模式不是 `primary`它的  *最接近的* 父模式 `primary`匹配 

### 视图方案 （resolution）

方案生成最终 `arch`对于请求/匹配 `primary` 看法： 

1. 如果视图有父视图，则父视图完全解析，然后当前 应用视图的继承规范 
2. 如果视图没有父视图，则其 `arch`按原样使用 
3. 当前视图的子视图模式 `extension`被抬头和他们的 继承规范应用深度优先（应用子视图，然后 它的孩子，然后是它的兄弟姐妹） 

应用子视图的结果产生最终 `arch`

### 继承规范 

继承规范由一个元素定位器组成，以匹配 父视图中的继承元素和子元素 将用于修改继承的元素。 

有三种类型的元素定位器用于匹配目标元素： 

- 一个 `xpath`元素与 `expr`属性。 `expr`是一个 [XPath](http://en.wikipedia.org/wiki/XPath) 表达式应用于当前 `arch`，第一个节点 它发现匹配 
- 一种 `field`元素与 `name`属性，匹配第一个 `field` 与相同 `name`.  匹配过程中忽略所有其他属性 
- 任何其他元素：具有相同名称且相同的第一个元素 属性（忽略 `position`和 `version`属性）匹配 

继承规范可能有一个可选的 `position`属性指定 应如何更改匹配的节点： 

- `inside`**（default）** 

  继承规范的内容被附加到匹配的节点 

- `replace`

  继承规范的内容替换匹配的节点。 任何文本节点只包含 `$0`在规范的内容内 被匹配节点的完整副本替换，有效地包装 匹配的节点。 

- `after`

  继承规范的内容被添加到匹配节点的 父节点，在匹配节点之后 

- `before`

  继承规范的内容被添加到匹配节点的 父节点，在匹配节点之前 

- `attributes`

  继承规范的内容应该是 `attribute`元素 与 `name`属性和可选主体： 如果 `attribute`元素有一个主体，一个名为的新属性 在其之后 `name`在匹配的节点上创建 `attribute`元素的文本作为值 如果 `attribute`元素没有主体，属性命名为 它的 `name`从匹配的节点中删除。  如果没有这样的属性 存在，引发错误 

视图的规范按顺序应用。 



## 列表 

列表视图的根元素是 `<tree>`. 列表视图的  root 可以具有以下属性： 

- `editable`

  默认情况下，选择列表视图的行会打开相应的  [表单视图 ](https://www.odoo.com/documentation/10.0/reference/views.html#reference-views-form)。 这 `editable`属性使 列表视图本身可就地编辑。 有效值为 `top`和 `bottom`, *新* 出现 记录  分别位于列表的顶部或底部。 内联 的架构 [表单视图 ](https://www.odoo.com/documentation/10.0/reference/views.html#reference-views-form)是  派生自列表视图。 对 有效的大多数属性 [表单视图 ](https://www.odoo.com/documentation/10.0/reference/views.html#reference-views-form)的字段和按钮 因此被列表接受  视图，尽管如果列表视图是，它们可能没有任何意义  不可编辑 

- `default_order`

  覆盖视图的顺序，替换模型的默认顺序。 该值是一个以逗号分隔的字段列表，后缀为 `desc`到 倒序排序： `<tree  default_order= "sequence,name desc" >  `

- `colors`

  自 9.0 版后已弃用： 替换为 `decoration-{$name}`

- `fonts`

  自 9.0 版后已弃用： 替换为 `decoration-{$name}`

- `decoration-{$name}`

  允许根据相应的行更改文本的样式 记录的属性。 值是 Python 表达式。  对于每条记录，计算表达式 将记录的属性作为上下文值，如果 `true`， 这 相应的样式应用于该行。  其他上下文值是 `uid`（当前用户的ID）和 `current_date`（当前日期 作为表单的字符串 `yyyy-MM-dd`). `{$name}`可 `bf` ( `font-weight: bold`),  `it` ( `font-style: italic`)，或任何 [引导上下文颜色 ](http://getbootstrap.com/components/#available-variations)( `danger`, `info`,  `muted`,  `primary`,  `success`要么 `warning`). 

- `create`,  `edit`,  `delete`

  允许 *DIS* 通过设置abling在视图中对应的动作  对应的属性 `false`

- `on_write`

  只对一个有意义 `editable`列表。  应该是方法名 在列表的模型上。  该方法将被调用 `id`记录的 在创建或编辑该记录（在数据库中）之后。 该方法应返回要加载或更新的其他记录的 id 列表。 

- `string`

  视图的替代可翻译标签 8.0 版后已弃用： 不再显示 

列表视图的可能子元素是： 

- `button`

  在列表单元格中显示一个按钮

  - `icon`

    用于显示按钮的图标 

  - `string` 

    - 如果没有`icon`, 按钮的文本 
    - 如果有 `icon`,  `alt`图标的文字 
    
  - `type`

    按钮的类型，指示点击它如何影响 Odoo： 

    - `workflow`**（default）** 

      向工作流发送信号。  按钮的 `name`是个 工作流信号，该行的记录作为参数传递给 信号 

    - `object`
    
      调用列表模型上的方法。  按钮的 `name`是个 方法，该方法使用当前行的记录 ID 和 当前上下文。
    
    - `action`
    
      加载一个执行 `ir.actions`，按钮的 `name`是个 操作的数据库 ID。  上下文用列表的 模型（如 `active_model`)，当前行的记录 ( `active_id`) 以及列表中当前加载的所有记录 ( `active_ids`, 可能只是数据库记录的一个子集 匹配当前搜索） 
    
  - `name`看 `type`
  
    - `args`看 `type`
  
    - `attrs`
  
    基于记录值的动态属性。 
  
    属性到域的映射，域在 当前行记录的上下文，如果 `True`相应的 属性设置在单元格上。 
  
    可能的属性是 `invisible`（隐藏按钮）。
  
    - `states`
  
    简写 `invisible` `attrs`: 状态列表，逗号分隔， 要求模型具有 `state`领域，它是 视图中使用。 制作按钮 `invisible`如果记录 *不在* 其中之一  列出的州 
  
  >Danger
  >
  >Using `states` in combination with `attrs` may lead to unexpected results as domains are combined with a logical AND.
  >危险 
  >使用 `states`结合 `attrs`可能会导致 意外的结果，因为域与逻辑 AND 组合在一起。
  >
  >
  
    -  `context`执行按钮的 Odoo 调用时合并到视图的上下文中
  
    -  `confirm`之前显示的确认消息（并让用户接受） 执行按钮的 Odoo 调用 

- `field`

  定义应显示相应字段的列 每条记录。  可以使用以下属性：

  -  `name`

    要在当前模型中显示的字段的名称。  一个给定的名字 每个视图只能使用一次 

  - `string`

    字段列的标题（默认情况下，使用 `string`的 模型的字段） 

  - `invisible`

    获取并存储该字段，但不显示该列 桌子。  对于不应显示但显示的字段是必需的 例如使用 `@colors`

  - `groups`

    列出应该能够看到该字段的组 

  - `widget`字段显示的替代表示。  可能的列表视图 值是： 

    - `progressbar`显示 `float`字段作为进度条。 
    - `many2onebutton`如果字段是，则用复选标记替换 m2o 字段的值 填充，如果没有，则打叉 
    - `handle`为了 `sequence`字段，而不是显示字段的值 只显示一个拖放图标 

  - `sum`,  `avg`在列底部显示相应的聚合。 这  聚合仅在 上计算 *当前显示的* 记录 。 这  聚合操作必须匹配相应字段的  `group_operator`

  - `attrs`基于记录值的动态属性。  只影响当前 领域，所以例如 `invisible`将隐藏该字段但保留相同 其他记录的字段可见，它不会隐藏列本身 

>Note
>if the list view is editable, any field attribute from the form view is also valid and will be used when setting up the inline form view
>笔记 
>如果列表视图是 `editable`, 来自表单视图也是有效的并且将  在设置内联表单视图时使用 



## 表单

表单视图用于显示来自单个记录的数据。  他们的根元素是 `<form>`. 它们由常规 [HTML ](http://en.wikipedia.org/wiki/HTML)和附加  结构和语义成分。 

### 结构件 

结构组件提供结构或“视觉”特征，几乎没有 逻辑。  它们用作表单视图中的元素或元素集。 

- `notebook`

  定义选项卡式部分。  每个选项卡都通过一个 `page`孩子 元素。  页面可以具有以下属性： 

  - `string`（必需的） 选项卡的标题 
  - `accesskey`一个 HTML 访问 [键 ](http://www.w3.org/TR/html5/editing.html#the-accesskey-attribute)
  - `attrs`基于记录值的标准动态属性 

- `group`

  用于定义表单中的列布局。  默认情况下，组定义 2 列 并且大多数组的直接子级采用单列。 `field`直接的 组的子级默认显示标签，标签和字段 本身的 colspan 为 1。 

  a中的列数 `group`可以使用 `col` 属性，元素采用的列数可以使用自定义 `colspan`. 孩子们水平排列（尝试在之前填充下一列 改变行）。 

  组可以有一个 `string`属性，显示为组的 标题 

- `newline`

  只在内部有用 `group`元素，提前结束当前行并 立即切换到新行（不填充任何剩余的列 预先） 

- `separator`

  小的水平间距，具有 `string`属性表现为一个部分 标题 

- `sheet`

  可以用作直接子项 `form`用于更窄、更灵敏的 表单布局 

- `header`

  结合 `sheet`, 在工作表上方提供全宽位置 本身，通常用于显示工作流按钮和状态小部件 

### 语义成分 

语义组件与 Odoo 相关联并允许与 Odoo 交互 系统。  可用的语义组件有： 

- `button`

  调用 Odoo 系统，类似于 [列表视图按钮 ](https://www.odoo.com/documentation/10.0/reference/views.html#reference-views-list-button)

- `field`

  呈现（并可能允许编辑）当前的单个字段 记录。  可能的属性有：

  - `name`（强制的） 要呈现的字段的名称
  -  `widget`字段具有基于其类型的默认呈现 （例如 [`Char`](https://www.odoo.com/documentation/10.0/reference/orm.html#odoo.fields.Char), [`Many2one`](https://www.odoo.com/documentation/10.0/reference/orm.html#odoo.fields.Many2one)）。  这 `widget`属性允许使用 不同的渲染方法和上下文。
  -  `options`指定字段小部件的配置选项的 JSON 对象 （包括默认小部件） 
  - `class`在生成的元素上设置的 HTML 类，常见的字段类有：
    - `oe_inline`防止在字段后面出现通常的换行符 
    - `oe_left`,  `oe_right`[浮动 ](https://developer.mozilla.org/en-US/docs/Web/CSS/float)将字段 到相应的方向 
    - `oe_read_only`,  `oe_edit_only`只在对应的表单模式下显示字段 
    - `oe_no_button`避免在 a 中显示导航按钮 [`Many2one`](https://www.odoo.com/documentation/10.0/reference/orm.html#odoo.fields.Many2one)
    - `oe_avatar`对于图像字段，将图像显示为“头像”（方形，最大 90x90 大小，一些图像装饰） 
  - `groups`仅显示特定用户的字段 
  - `on_change`编辑此字段的值时调用指定的方法，可以生成 更新其他字段或为用户显示警告 8.0 版后已弃用： 使用 [`odoo.api.onchange()`](https://www.odoo.com/documentation/10.0/reference/orm.html#odoo.api.onchange)在模型上 
  - `attrs`基于记录值的动态元参数 
  - `domain`仅适用于关系字段，在显示现有字段时应用的过滤器 选择记录 
  - `context`仅适用于关系字段，获取可能值时传递的上下文 
  - `readonly`以只读和编辑模式显示该字段，但永远不要让它 可编辑
  - `required`如果该字段没有，则生成错误并阻止保存记录 有一个价值 
  - `nolabel`不自动显示字段的标签，只有在 field 是 a 的直接孩子 `group`元素 `placeholder`帮助消息显示在 *空白* 字段中。 可以替换字段标签  复杂的形式。 *不应* 作为数据示例，因为用户有责任  将占位符文本与填充字段混淆 
  - `mode`为了 [`One2many`](https://www.odoo.com/documentation/10.0/reference/orm.html#odoo.fields.One2many), 显示模式（视图类型）用于 字段的链接记录。  之一 `tree`,  `form`,  `kanban`要么 `graph`.  默认是 `tree`（列表显示） 
  - `help`悬停字段或其标签时为用户显示的工具提示 `filename`对于二进制字段，提供相关字段的名称 文件 
  - `password`表明一个 [`Char`](https://www.odoo.com/documentation/10.0/reference/orm.html#odoo.fields.Char)字段存储密码和 不应该显示其数据 

### 业务视图指南 

业务视图针对的是普通用户，而不是高级用户。  例子 是：机会、产品、合作伙伴、任务、项目等。 

![img](D:\typora\Typora\pictures\oppreadonly.png)

一般来说，业务视图由 

1. 顶部的状态栏（带有技术或业务流程）， 
2. 中间的一张纸（表格本身）， 
3. 带有历史记录和评论的底部。 

从技术上讲，新的表单视图在 XML 中的结构如下： 

```xml
<form> 
    <header> ... 状态栏内容 ... </header> 
    <sheet> 表格 ... 内容 ... </sheet> 
    <div  class= "oe_chatter" > ... content底部... </div> 
</form> 
```

#### 状态栏 

状态栏的作用是显示当前记录的状态和 操作按钮。 

![img](D:\typora\Typora\pictures\status.png)

##### 按钮 

按钮的顺序遵循业务流程。  例如，在销售订单中， 逻辑步骤是： 

1. 发送报价单 
2. 确认报价 
3. 创建最终发票 
4. 发送货物 

突出显示的按钮（默认为红色）强调逻辑下一步，以  帮助用户。 它通常是第一个活动按钮。 另一方面，  取消 按钮 *必须* 保持灰色（正常）。 例如，在  发票 按钮 退款 绝不能为红色。 

从技术上讲，通过添加类“oe_highlight”来突出显示按钮： 

```xml
<button  class= "oe_highlight"  name= "..."  type= "..."  states= "..." /> 
```

##### 状态 

使用 `statusbar`小部件，并以红色显示当前状态。  状态 所有流程通用（例如，销售订单以报价单开始，然后我们 发送它，然后它变成一个完整的销售订单，最后它完成）应该是 始终可见，但异常或状态取决于特定的子流 应该只在当前时可见。 

![img](D:\typora\Typora\pictures\status1.png)![img](https://www.odoo.com/documentation/10.0/_images/status2.png)

状态按照字段中使用的顺序显示（列表中的 选择字段等）。  始终可见的状态用 属性 `statusbar_visible`. 

```xml
<field  name= "state"  widget= "statusbar" 
    statusbar_visible= "draft,sent,progress,invoiced,done"  /> 
```

#### 工作表 

所有业务视图都应该看起来像一张打印纸： 

![img](D:\typora\Typora\pictures\sheet.png)

1. 里面的元素 `<form>`要么 `<page>`不定义组、元素 它们内部按照正常的 HTML 规则布局。  他们的内容可以 使用明确分组 `<group>`或定期 `<div>`元素。 

2. 默认情况下，元素 `<group>`在里面定义两列，除非 属性 `col="n"`用来。  列具有相同的宽度（1/n 组的宽度）。  用一个 `<group>`元素以生成一列字段。 

3. 要为某个部分指定标题，请添加 `string`归因于 `<group>`元素： 

   ```xml
   <group  string= "Time-sensitive operations" > 
   ```

   这取代了以前的使用 `<separator string="XXX"/>`. 

4. 这 `<field>`元素不产生标签，除非作为直接子元素 的 `<group>`元素。 采用 `<label for="field_name>`生成一个字段的标签。 

##### 工作表标题 

一些工作表的标题包含一个或多个字段，以及这些字段的标签 字段仅在编辑模式下显示。 

| 查看模式                                                     | 编辑模式                                      |
| ------------------------------------------------------------ | --------------------------------------------- |
| ![img](https://www.odoo.com/documentation/10.0/_images/header.png) | ![img](D:\typora\Typora\pictures\header2.png) |

使用 HTML 文本， `<div>`,  `<h1>`,  `<h2>`... 生成漂亮的标题，以及 `<label>`与班级 `oe_edit_only`只显示字段的标签 在编辑模式。  班上 `oe_inline`将使字段内联（而不是 块）：字段后面的内容将显示在同一行而不是 比在它下面的线上。  上面的表单由以下 XML 生成： 

```xml
<label  for= "name"  class= "oe_edit_only" /> 
<h1><field  name= "name" /></h1> 

<label  for= "planned_revenue"  class= "oe_edit_only" /> 
<h2> 
    <field  name = "planned_revenue"  class= "oe_inline" /> 
    <field  name= "company_currency"  class= "oe_inline oe_edit_only" /> at 
     <field  name= "probability"  class= "oe_inline" /> % success rate
 </h2> 
```

##### 按钮盒 

表单中可以显示许多相关的操作或链接。  例如，在 机会窗体，“安排通话”和“安排会议”操作具有 CRM 使用的重要场所。  而不是将它们放在 “更多”菜单，将它们作为按钮（在顶部）直接放在工作表中以制作 它们更显眼，更容易访问。 

![img](D:\typora\Typora\pictures\header3.png)

从技术上讲，按钮放置在一个 `<div>`将它们分组为 挡在工作表的顶部。 

```xml
<div  class= "oe_button_box"  name= "button_box" > 
    <button  string= "Schedule/Log Call"  name= "..."  type= "action" /> 
    <button  string= "Schedule Meeting"  name= "action_makeMeeting"  type= "对象" /> 
</div> 
```

##### 组和标题 

现在生成一列字段 `<group>`元素，具有 可选标题。 

![img](D:\typora\Typora\pictures\screenshot-03.png)

```xml
<group  string= "Payment Options" > 
    <field  name= "writeoff_amount" /> 
    <field  name= "payment_option" /> 
</group> 
```

建议在表单上有两列字段。  为此，只需 放在 `<group>`包含顶级内部字段的元素 `<group>`元素。 

为了使 [视图扩展 ](https://www.odoo.com/documentation/10.0/reference/views.html#reference-views-inheritance)更简单，它是  建议放一个 `name`属性在 `<group>`元素，所以新的领域 可以很容易地添加到正确的位置。 

##### 特殊情况：小计 

定义了一些类来呈现小计，如发票形式： 

![img](D:\typora\Typora\pictures\screenshot-00.png)

```xml
<group  class= "oe_subtotal_footer" > 
    <field  name= "amount_untaxed" /> 
    <field  name= "amount_tax" /> 
    <field  name= "amount_total"  class= "oe_subtotal_footer_separator" /> 
    <field  name= "residual"  style= " margin-top: 10px" /> 
</group> 
```

##### 占位符和内联字段 

有时字段标签会使表单过于复杂。 可以省略字段标签，  而是在字段内放置一个占位符。 占位符文本是  仅在字段为空时可见。 占位符应该告诉你要做什么  放在领域内，它 *不能* 作为一个例子，因为他们经常被混淆  带有填充数据。 

还可以通过将字段“内联”呈现在一个 显式块元素，如 `<div>`.  这允许在语义上分组 相关字段就好像它们是单个（复合）字段一样。 

以下示例取自“ *潜在客户”* 表单，显示了占位符和  内联字段（邮编和城市）。 

| 编辑模式                                          | 查看模式                                                     |
| ------------------------------------------------- | ------------------------------------------------------------ |
| ![img](D:\typora\Typora\pictures\placeholder.png) | ![img](https://www.odoo.com/documentation/10.0/_images/screenshot-01.png) |

```xml
<group> 
    <label  for= "street"  string= "Address" /> 
    <div> 
        <field  name= "street"  placeholder= "Street..." /> 
        <field  name= "street2" /> 
        <div> 
            < field  name= "zip"  class= "oe_inline"  placeholder= "ZIP" /> 
            <field  name= "city"  class= "oe_inline"  placeholder= "City" /> 
        </div> 
        <field  name= "state_id"  placeholder= "State" /> 
        <field  name= "country_id"  placeholder= "Country" /> 
    </div> 
</group> 
```

##### 图片 

图像（如头像）应显示在工作表的右侧。  这 产品形式如下： 

![img](D:\typora\Typora\pictures\screenshot-02.png)

上面的表单包含一个以以下内容开头的 <sheet> 元素： 

```xml
<field  name= "product_image"  widget= "image"  class= "oe_avatar oe_right" /> 
```

##### 标签 

最多 [`Many2many`](https://www.odoo.com/documentation/10.0/reference/orm.html#odoo.fields.Many2many)领域，如类别，更好 呈现为标签列表。  使用小部件 `many2many_tags`为了这： 

![img](D:\typora\Typora\pictures\screenshot-04.png)

```xml
<field  name= "category_id"  widget= "many2many_tags" /> 
```

### 配置表格指南 

配置表单示例：阶段、休假类型等。这涉及所有 每个应用程序的配置下的菜单项（如销售/配置）。 

![img](D:\typora\Typora\pictures\nosheet.png)

1. 没有标题（因为没有状态，没有工作流程，没有按钮） 
2. 没有表 

### 对话框表单指南 

示例：从机会“安排通话”。 

![img](D:\typora\Typora\pictures\wizard-popup.png)

1. 避免分隔符（标题已经在弹出标题栏中，所以另一个 分隔符不相关） 
2. 避免取消按钮（用户通常关闭弹出窗口以获得相同的 影响） 
3. 动作按钮必须突出显示（红色） 
4. 当有文本区域时，使用占位符代替标签或 分隔器 
5. 就像在常规表单视图中一样，将按钮放在 <header> 元素中 

### 配置向导指南 

示例：设置/配置/销售。 

1. 始终在线（无弹出窗口） 
2. 没有表 
3. 保留取消按钮（用户无法关闭窗口） 
4. “应用”按钮必须是红色的 



## 图表 

图形视图用于可视化多个记录的聚合或 记录组。  它的根元素是 `<graph>`可以采取以下措施 属性： 

- `type`

  `bar`**（default）**之一 ， `pie`和 `line`，要使用的图形类型 

- `stacked`

  仅用于 `bar`图表。  如果存在并设置为 `True`, 堆叠条形 组内 

图形视图中唯一允许的元素是 `field`哪个可以有 以下属性： 

- `name`**（required）** 

  要在图形视图中使用的字段的名称。  如果用于分组（而不是 比聚合） 

- `type`

  指示该字段应用作分组标准还是用作 组内的聚合值。  可能的值为： 

  - `row`（默认） 按指定字段分组。  所有图形类型至少支持一层 分组，有些可能支持更多。  对于枢轴视图，每个组都获得其 自己的行。 
  - `col`仅由数据透视表使用，创建按列分组 
  - `measure`在组内聚合的字段 

- `interval`

  在日期和日期时间字段上，按指定的时间间隔 ( `day`, `week`,  `month`,  `quarter`要么 `year`) 而不是分组 特定日期时间（固定秒分辨率）或日期（固定日期分辨率）。 

>Warning
>graph view aggregations are performed on database content, non-stored function fields can not be used in graph views
>警告 
>图视图聚合是对数据库内容执行的，非存储功能字段不能在图形视图中使用 



### 支点 

透视视图用于将聚合可视化为 [透视表 ](http://en.wikipedia.org/wiki/Pivot_table)。 它的根  元素是 `<pivot>`它可以采用以下属性： 

- `disable_linking`

  设置 `True`删除表格单元格到列表视图的链接。 

- `display_quantity`

  设置 `true`默认显示数量列。 

枢轴视图中允许的元素与图形视图相同。 



## Kanban (看板)

看板视图是 [看板 ](http://en.wikipedia.org/wiki/Kanban_board)可视化：它将记录显示为  “卡片”，介于 [列表视图 ](https://www.odoo.com/documentation/10.0/reference/views.html#reference-views-list)和  不可编辑的 [表单视图 ](https://www.odoo.com/documentation/10.0/reference/views.html#reference-views-form)。 记录可以分组  在用于工作流可视化或操作的列中（例如任务或  工作进度管理）或未分组（仅用于可视化记录）。 

看板视图的根元素是 `<kanban>`，它可以使用以下 属性： 

- `default_group_by`

  如果未指定分组，则看板视图是否应分组通过 操作或当前搜索。  应该是要分组的字段的名称 by 当没有另外指定分组时 

- `default_order`

  如果用户尚未对记录进行排序（通过 列表视图） 

- `class`

  将 HTML 类添加到看板视图的根 HTML 元素 

- `group_create`

  “添加新列”栏是否可见。  默认值：真。 

- `group_delete`

  是否可以通过上下文菜单删除组。  默认值：真。 

- `group_edit`

  是否可以通过上下文菜单编辑组。  默认值：真。 

- `quick_create`

  是否应该可以在不切换到 表格视图。  默认情况下， `quick_create`当看板视图处于启用状态时 分组，不分组时禁用。 设置 `true`始终启用它，并 `false`总是禁用它。 

视图元素的可能子元素是： 

- `field`

  声明要在看板 使用的字段 *逻辑中* 。 如果该字段只是显示在  看板视图，不需要预先声明。 

  可能的属性有：

  - `name`**（required） **

    要获取的字段的名称 

- `templates`

  定义了一个 列表 [QWeb ](https://www.odoo.com/documentation/10.0/reference/qweb.html#reference-qweb)模板 。 卡片定义可能是  为清晰起见，拆分为多个模板，但看板视图 *必须* 在  至少一个根模板 `kanban-box`, 将为每个渲染一次 记录。 
  
  看板视图主要使用标准的 [javascript qweb ](https://www.odoo.com/documentation/10.0/reference/qweb.html#reference-qweb-javascript)并提供以下上下文变量： 
  
  - `instance`当前的 [Web 客户端 ](https://www.odoo.com/documentation/10.0/reference/javascript.html#reference-javascript-client)实例 `widget`当前 [`KanbanRecord()`](https://www.odoo.com/documentation/10.0/reference/views.html#KanbanRecord), 可以用来获取一些 元信息。  这些方法也可以直接在 模板上下文，不需要通过访问 `widget`
  - `record`具有所有请求字段作为其属性的对象。  每个字段都有 两个属性 `value`和 `raw_value`，前者被格式化 根据当前用户参数，后者是来自的直接值 一种 [`read()`](https://www.odoo.com/documentation/10.0/reference/orm.html#odoo.models.Model.read)（日期和日期时间字段除外  这是 [根据用户的语言环境格式化 ](https://github.com/odoo/odoo/blob/a678bd4e/addons/web_kanban/static/src/js/kanban_record.js#L102)）
  -  `formats`这 `web.formats()`操作和转换值的模块 
  - `read_only_mode`不言自明 

### 按钮和字段 

虽然大多数看板模板都是标准的 [QWeb ](https://www.odoo.com/documentation/10.0/reference/qweb.html#reference-qweb)，  看板视图流程 `field`,  `button`和 `a`元素特别： 

- 默认情况下，字段被它们的格式化值替换，除非它们 匹配特定的看板视图小部件 

- 按钮和链接 `type`属性变成执行 Odoo 相关 操作而不是它们的标准 HTML 函数。  可能的类型有：
  - `action`,  `object`标准行为 [Odoo 按钮的 ](https://www.odoo.com/documentation/10.0/reference/views.html#reference-views-list-button)，大多数与标准相关的属性  可以使用 Odoo 按钮。 
  - `open`以只读模式在表单视图中打开卡片记录
  - `edit`以可编辑模式在表单视图中打开卡的记录 
  - `delete`删除卡的记录并移除卡 

### Javascript API 

###### `class KanbanRecord()`

[`Widget()`](https://www.odoo.com/documentation/10.0/reference/javascript.html#Widget)处理单个记录到一个记录的渲染 卡片。  在它自己的渲染中可用 `widget`在模板中 语境。 

###### `kanban_color(raw_value)`

将颜色分割值转换为看板颜色类 `oe_kanban_color_*color_index*`.  内置 CSS 提供类 最多一个 `color_index`9. 

###### `kanban_getcolor(raw_value)`

将颜色分割值转换为颜色索引（在 0 到 9 之间，由 默认）。  颜色分割值可以是数字或字符串。 

###### `kanban_image(model, field, id[, cache][, options])`

生成指定字段的 URL 作为图像访问。 

Arguments (参数): 

- **model** （ `String`) -- 托管图像的模型 
- **filed** （ `String`) -- 保存图像数据的字段的名称 
- **id** -- 包含要显示的图像的记录的标识符 
- **cache** ( `Number`) -- 浏览器的缓存持续时间（以秒为单位） 默认值应该被覆盖。 `0`禁用 完全缓存 

Returns (返回):  一个图片网址 

> 警告 
>
> `kanban_text_ellipsis`已在 Odoo 9 中删除。 CSS `text-overflow`应该改用。 

## 日历 

日历视图将记录显示为每天、每周或每月的事件 日历。  它们的根元素是 `<calendar>`.  上的可用属性 日历视图是： 

- `date_start`（必需的） 

  保存事件开始日期的记录字段的名称 

- `date_stop`

  保存事件结束日期的记录字段的名称，如果 `date_stop`提供记录变得可移动（通过拖放） 直接在日历中 

- `date_delay`

  替代 `date_stop`, 提供事件的持续时间而不是 它的结束日期 

- `color`

  用于 的记录字段的名称 *颜色分割* 。 记录在  相同的颜色段在日历中分配相同的高亮颜色，  颜色是半随机分配的。 

- `event_open_popup`

  在对话框中打开事件而不是切换到表单视图，禁用 默认情况下 

- `quick_add`

  在点击时启用快速事件创建：只要求用户输入 `name`并尝试使用该事件和单击事件创建一个新事件 时间。  如果快速创建失败，则回退到完整表单对话框 

- `display`

  用于事件显示的格式字符串，字段名称应在括号内 `[`和 `]`

- `all_day`

  记录上布尔字段的名称，指示相应的 事件被标记为全天（并且持续时间无关紧要） 

- `mode`

  加载日历时的默认显示模式。 可能的属性有： `day`,  `week`,  `month`



## 甘特图 

甘特视图适当地显示甘特图（用于调度）。 

甘特视图的根元素是 `<gantt/>`，它没有孩子，但可以 取以下属性： 

- `date_start`**（reqiured）** 

  提供每个事件的开始日期时间的字段名称 记录。 

- `date_stop`

  提供每个事件的结束持续时间的字段名称 记录。  可以换成 `date_delay`.  一个（并且只有一个） `date_stop`和 `date_delay`必须提供。 如果该字段是 `False`作为记录，它被假定为“点事件” 并且结束日期将设置为开始日期 

- `date_delay`

  提供事件持续时间的字段名称 

- `duration_unit`

  之一 `minute`,  `hour`**（default）** ， `day`,  `week`,  `month`,  `year`

- `default_group_by`

  用于对任务进行分组的字段名称 

- `type`

  - `gantt`经典甘特图视图**（default）**  
  - `consolidate`第一个孩子的价值观在甘特图的任务中得到巩固 
  - `planning`孩子们显示在甘特图的任务中 

- `consolidation`

  字段名称以在记录单元格中显示合并值 

- `consolidation_max`

  以“group by”字段为键和最大合并的字典 在以红色显示单元格之前可以达到的值 （例如 `{"user_id": 100}`) 警告 字典定义必须使用双引号， `{'user_id': 100}`是 不是有效值 

- `string`

  显示在合并值旁边的字符串，如果未指定，则为标签 将使用合并字段的 

- `fold_last_level`

  如果设置了值，则折叠最后一个分组级别 

- `round_dnd_dates`

  允许将任务的开始和结束日期四舍五入到最近的刻度线 

- `drag_resize`

  调整任务大小，默认为 `true`



## 图表 

图表视图可用于显示记录的有向图。  根 元素是 `<diagram>`并且不带任何属性。 

图表视图的可能子项是： 

- `node`**（reqiured）**

  定义图的节点。  它的属性是：

  - `object`节点的 Odoo 模型
  - `shape`类似于 颜色和字体的条件形状映射 [列表视图 ](https://www.odoo.com/documentation/10.0/reference/views.html#reference-views-list)。 唯一有效的形状是 `rectangle`（这 默认形状是省略号） 
  - `bgcolor`如同 `shape`，但有条件地为 节点。  默认背景色为白色，唯一有效的替代 是 `grey`. 

- `arrow`**（reqiured）**

  定义图的有向边。  它的属性是：

  - `object`**（reqiured）** Edge 的 Odoo 模型
  -  `source`**（reqiured）** [`Many2one`](https://www.odoo.com/documentation/10.0/reference/orm.html#odoo.fields.Many2one)边缘模型指向的字段 边缘的源节点记录 
  - `destination`**（reqiured）** [`Many2one`](https://www.odoo.com/documentation/10.0/reference/orm.html#odoo.fields.Many2one)边缘模型指向的字段 边缘的目的节点记录 
  - `label`Python 属性列表（作为带引号的字符串）。  相应的 属性的值将被连接并显示为边缘的 标签 

- `label`

  图表的解释性说明， `string`属性定义 笔记的内容。  每个 `label`输出为图中的一个段落 标题，很容易看到，但没有任何特别的强调。 



## 搜索 

搜索视图与以前的视图类型不同，因为它们不显示  *content* ：虽然它们适用于特定模型，但它们用于过滤  其他视图的内容（通常是聚合视图  例如 [列表 ](https://www.odoo.com/documentation/10.0/reference/views.html#reference-views-list)或 [图表 ](https://www.odoo.com/documentation/10.0/reference/views.html#reference-views-graph)）。 除此之外  用例不同，它们的定义方式相同。 

搜索视图的根元素是 `<search>`.  它不需要任何属性。 

搜索视图的可能子元素是： 

- `field`

  字段使用用户提供的值定义域或上下文。 搜索时  域被生成，域域相互组合并且  使用 过滤器 **AND** 。 字段可以具有以下属性：

  - `name`要过滤的字段的名称 

  - `string`字段的标签

  - `operator`

    默认情况下，字段生成表单域 `[(*name*, *operator*, *provided_value*)]`在哪里 `name`是字段的名称和 `provided_value`是用户提供的值，可能  过滤或转换（例如，用户应提供  *标签* 选择字段值的 ，而不是值本身）。 

    这 `operator`属性允许覆盖默认运算符， 这取决于字段的类型（例如 `=`对于浮动字段，但是 `ilike`用于字符字段） 

  - `filter_domain`

    用作字段搜索域的完整域，可以使用 `self`变量以在自定义中注入提供的值 领域。  可用于生成更加灵活的域 比 `operator`单独（例如一次搜索多个字段） 

    如果两者 `operator`和 `filter_domain`提供， `filter_domain`优先。

  - `context`

    允许添加上下文键，包括用户提供的值（其中 至于 `domain`可作为 `self`多变的）。 

    默认情况下， 字段不生成域。 

    >笔记 
    >
    >域和上下文是包含性的，两者都是生成的 如果一个 `context`被指定。  
    >
    >只生成上下文 值，设置 `filter_domain`到一个空列表： `filter_domain="[]"

  - `groups`使该字段仅对特定用户可用 `widget`为该字段使用特定的搜索小部件（在 标准的 Odoo 8.0 是一个 `selection`小部件 [`Many2one`](https://www.odoo.com/documentation/10.0/reference/orm.html#odoo.fields.Many2one)字段）
  - `domain`如果该字段可以提供自动完成 （例如 [`Many2one`](https://www.odoo.com/documentation/10.0/reference/orm.html#odoo.fields.Many2one))，过滤可能的 完成结果。

- `filter`

  过滤器是搜索视图中的预定义切换，只能启用 或禁用。  

  它的主要目的是将数据添加到搜索上下文（ 上下文传递给数据视图以进行搜索/过滤），或追加新的 部分到搜索过滤器。 

  过滤器可以具有以下属性： 

  - `string`（必需的） 过滤器的标签 
  - `domain`一个 Odoo [域 ](https://www.odoo.com/documentation/10.0/reference/orm.html#reference-orm-domains)，将被附加到  操作域作为搜索域的一部分 
  - `context`一个 Python 字典，合并到操作的域中以生成 搜索域
  -  `name`过滤器的逻辑名称，可用于 [默认启用它 ](https://www.odoo.com/documentation/10.0/reference/views.html#reference-views-search-defaults)，也可用作  [继承钩子 ](https://www.odoo.com/documentation/10.0/reference/views.html#reference-views-inheritance)
  - `help`过滤器的较长说明文本，可以显示为 工具提示 
  - `groups`使过滤器仅对特定用户可用 

  >提示
  >
  > 7.0 版中的新功能。 
  >
  >过滤器序列（没有非过滤器将它们分开）被处理 作为包容性复合：它们将与 `OR`相当 比平时 `AND`，例如
  >
  >```xml
  ><filter  domain= "[('state', '=', 'draft')]" />  
  ><filter  domain= "[('state', '=', 'done')]" /> 
  >```
  >
  >如果两个过滤器都被选中，将选择其记录 `state` 是 `draft`要么 `done`， 但
  >
  >```xml
  ><filter  domain= "[('state', '=', 'draft')]" />  
  ><separator/>  
  ><filter  domain= "[('delay', '<', 15)]" />  
  >```
  >
  > 如果两个过滤器都被选中，将选择其记录 `state` 是 `draft` **和**  `delay`低于 15。 

- `separator`

  可用于在简单的搜索视图中分隔过滤器组 

- `group`

  可用于分隔过滤器组，比 `separator`在复杂的搜索视图中 



### 搜索默认值 

搜索字段和过滤器可以通过操作的 `context` 使用 `search_default_*name*`键。  对于字段，该值应为 要在字段中设置的值，对于过滤器，它是一个布尔值。  例如， 假设 `foo`是一个字段并且 `bar`是一个过滤器一个动作上下文： 

```python
{ 
  'search_default_foo' ：  'acro' ， 
  'search_default_bar' ：  1 
} 
```

将自动启用 `bar`过滤和搜索 `foo`字段为  *阿克罗* 。 



## QWeb 

QWeb 视图是 标准 [QWeb ](https://www.odoo.com/documentation/10.0/reference/qweb.html#reference-qweb)视图内部的 模板 `arch`.  它们没有特定的根元素。 

一个 QWeb 视图只能包含一个模板 [4 ](https://www.odoo.com/documentation/10.0/reference/views.html#template-inherit)，并且  模板的名称 *必须* 与视图的完整匹配（包括模块名称）  [外部标识 ](https://www.odoo.com/documentation/10.0/glossary.html#term-external-id)。 

[模板 ](https://www.odoo.com/documentation/10.0/reference/data.html#reference-data-template)应该用作定义 QWeb 的快捷方式  意见。 

[[1\] ](https://www.odoo.com/documentation/10.0/reference/views.html#id3)出于向后兼容的原因 

[[2\] 增加 ](https://www.odoo.com/documentation/10.0/reference/views.html#id1)了扩展功能，让QWeb中的匹配更简单  意见： `hasclass(*classes)`如果上下文节点有 所有指定的类 

[[3\] ](https://www.odoo.com/documentation/10.0/reference/views.html#id2)由于历史原因，它起源于树型视图  后来重新用于更多表格/列表类型的显示 

[[4\] ](https://www.odoo.com/documentation/10.0/reference/views.html#id4)或者没有模板，如果它是一个继承的视图，那么 [它   应该只包含 xpath 元素 ](https://www.odoo.com/documentation/10.0/reference/views.html#reference-views-inheritance)

​                                 

>来源 https://www.odoo.com/documentation/10.0/reference/views.html