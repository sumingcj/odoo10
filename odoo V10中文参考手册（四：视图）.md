## 公共结构

- name (必选) 用于通过名字查找
- model 与view相关联的model
- priority 当搜索查找view时，优先级最低的view会被返回
- arch 视图layout的描述
- groups_id 指定可查看、使用视图的用户组id，many2many关系
- inherit_id 当前视图的父级视图
- mode 继承模式，当inherit_id没有设置时，它的值是primary,当设置了inherit_id后，它默认值是extension，可手动设置为primary
- application 定义哪些视图可以被切换，默认情况下所有视图都可以

## 继承

### 视图匹配

- 当通过(model, type)来请求视图时，与model、type匹配且mode=primary 优先级最低的视图会被返回
- 当通过id请求视图时，如果它的模型不是primary，那么最取他的最近的mode=primary的父级视图

### 视图解析

解析符合mode=primary的视图并得到arch内容：

- 如果当前视图有一个父视图，且父视图是完全确定的，直接应用当前视图的继承规范
- 如果当前视图没有父视图，那么arch将直接被使用
- 查找当前视图的extensino模式子视图，使用深度优先算法应用它们的继承规范

应用到子视图之后的结果产生最终的arch

### 继承规范

继承规范由一个定位元素组成，用来匹配父视图中被继承的元素、和 子视图中会被用来修改的继承元素
 一共有三种用来匹配目标元素的定位元素：

- 带有expr属性的xpath元素 ，expr是一个用在arch中的xpath表达式，找到的第一个节点就是匹配结果
- 带有name属性的field元素，匹配第一个一样name的field元素，其他的属性在匹配时被忽略
- 其他的元素：匹配第一个拥有一样的name及其他属性的元素（忽略position,version属性）

继承规范通过可选的position属性来指定如何修改匹配的节点

- inside（默认） - 添加到匹配的节点前
- replace - 替换匹配的节点
- after - 添加到匹配的节点的父节点之后
- before - 添加到匹配的节点的父节点之前
- attributes - 继承的内容是一系列拥有name属性的attribute 元素，且有可选的内容主体
   1.如果attribute有内容主体，就在匹配的节点上添加以name命名的、以内容主体为值的属性
   2.如果attribute没有内容主体，就将匹配节点上名字为name的属性删除，如果没有对应的属性，抛出一个错误

## 列表视图

列表视图的根元素是`<tree>`,它可以有以下几种属性：

### editable

默认情况下选择单行记录时会打开对应记录的表单，该属性让数据可以在列表内进行编辑，有效的值是top和bottom，可让新的记录出现在列表的顶部或底部

### default_order

重定义视图的排序规则，以逗号分隔多个字段，可使用desc来进行倒序`<tree default_order="sequence,name desc">`

### decoration-name

可以根据值来改变字段的显示，$name可为bf (font-weight: bold), it (font-style: italic)或其他bootstrap样式如danger, info, muted, primary, success,warning，取值为python表达式，对每条记录执行相应表达式，当结果为true的时候将对应的样式应用

### create, edit, delete

可以通过将它们设置为false来禁用视图中的对应操作

### on_write

只当启用editable时有用，在调用时会传给函数新增或修改后的记录，该函数需要返回一个用于更新列表的记录id列表

### button

在一个列表单元格中显示按钮

> 属性列表：
>  1.icon -- 用来展示按钮的图标
>  2.string -- 当没有icon的时候，button显示的文字，有icon的时候、相当于alt属性值
>  3.type -- 按钮类型，表示点击它之后如何影响系统
>
> > 1)workflow（默认）：将按钮name作为信号发送给工作流，记录的内容作为参数
> >  2)object ： 调用当前数据列表模型的方法，方法名是按钮的name，调用时带有记录id和当前上下文环境
> >  3)action ： 加载ir.actions，按钮name是该action在数据库的id，上下文环境扩展到列表的model(作为active_model)、当前记录(active_id)、所有当前加载记录的id(active_ids)

> 4.name,args 与type一样
>  5.attrs 基于记录值的动态属性，将domain表达式应用在记录上，当返回值为True的时候设置相应的属性，一般用于invisible （隐藏按钮）、readonly （禁用按钮但显示）这两种属性
>  6.states invisible属性attrs的简写，给出一个以逗号分隔的state列表，需要模型有一个对应的state属性，可以将不在state列表中的记录的按钮隐藏
>  7.context 当响应odoo的调用时，合并到视图的上下文环境中
>  8.confirm 当点击按钮时给出的确认消息

### field

定义一个所有记录都需要展示的列

> 属性列表：
>  1.name  需要显示的字段名
>  2.string 该列的名称
>  3.invisible 查询而且保存该字段但不显示
>  4.groups 可以看到该字段的用户组列表
>  5.widget 用来展示该字段的可选形式
>
> > progressbar 进度条用于展示浮点数
> >  many2onebutton当关联字段值存在时显示勾，不存在显示X
> >  handle对于排序字段，直接显示向上向下箭头
> >  sum, avg 在底部显示基于当前页面数据的计算
> >  attrs 基于记录值的动态属性，只对当前栏有效，即可以第一条记录中该字段显示，第二条隐藏

## 表单

表单视图用于展示单条数据，根元素是form，由常规html和构造部分、语义部分组成

### 构造部分

构造部分提供了结构和可视特性，以元素或者元素的子元素的形式应用到表单视图的元素中

#### 1.notebook

定义一个tab块，每一个tab通过一个page子元素定义，每个page可以有以下属性：

- string (required) --tab标签的名称
- accesskey --html accesskey
- attrs --基于记录值的动态属性

#### 2.group

用于定义栏目在表单中布局，默认情况下一个group定义两个列，并且每个最直接的子元素占用一个列，field类型的元素默认显示一个标签
 group占用的列数是可以通过col属性自定义的，默认2个；其他元素可以通过**colspan**属性来定义占的列数，子元素是横向布局的，可以通过设置string 属性来定义group所展示的标题

#### 3.newline

只在group元素里才有用，代表开启新的行

#### 4.separator

一条水平线，可以通过string属性来设置该区域的标题

#### 5.sheet

可以用作form的子元素用来表示更加狭义的表单

#### 6.header

与sheet一起使用，显示在sheet的上方的一个条，一般用于显示工作流和状态栏

### 语义部分

语义部分用于与odoo系统交互

#### 1.button 与列表的button一致

#### 2.field

展示当前记录的某个字段，有以下属性：

- name (必选) -- 用于展示字段名
- widget -- 每个字段根据其数据类型有一个默认的展示方式，widget属性可指定用一个别的方式来展示
- options -- 用于指定widget字段配置的json对象
- class -- 用于设置当前元素的html class属性：

> oe_inline -  防止它自动将之后的字段换行
>  oe_left, oe_right - 相当于css的float
>  oe_read_only, oe_edit_only - 只在相应的模式下展示该字段
>  oe_no_button - 不为many2one字段显示导航按钮
>  oe_avatar - 当该字段为图片时，将它展示为头像（90*90的正方形）

- groups - 只将该字段展示给指定用户组
- on_change - 在字段值改变时调用对应方法，从8.0开始改用模型中的 odoo.api.onchange()
- attrs - 基于记录值的动态参数
- domain - 当以选择的方式显示关联字段时，用过过滤数据
- context - 用于关联字段，显示数据时提供上下文环境
- readonly - 该字段可在读和编辑模式下展示，但是永远是不能编辑的
- required - 当该值没有设置就保存时给出一个错误提示并阻止保存
- nolabel - 不显示字段的标签，只有在该字段是group子元素时用意义
- placeholder - 字段值为空时展示的提示
- mode - 对于one2many字段，用于展示其关联的记录的形式，有tree, form, kanban , graph，默认是tree
- help - 当将鼠标放在字段或标签时显示的提示
- filename - 对于二进制的字段，相关字段给出文件名
- password - 表示该字段是一个密码，不明文展示

### 业务视图规则

业务视图是指向普通用户的，像：机会、产品、合作伙伴、任务、项目等

![img](https:////upload-images.jianshu.io/upload_images/4048165-3b79fe4269595169.png?imageMogr2/auto-orient/strip|imageView2/2/w/1131)

一般情况下，业务视图由以下元素组成：

- 展示在顶部的业务流程的状态按钮
- 中间展示一个表单的表格
- 底部展示评论和历史操作记录

```xml
<form>
    <header> ... content of the status bar  ... </header>
    <sheet>  ... content of the sheet       ... </sheet>
    <div class="oe_chatter"> ... content of the bottom part ... </div>
</form>
```

#### 1.状态条

用于展示当前记录的状态和相应的动作按钮

![img](https:////upload-images.jianshu.io/upload_images/4048165-d333dc32a6b362e9.png?imageMogr2/auto-orient/strip|imageView2/2/w/890)

- 按钮
   按钮的顺序与业务流程的顺序一致，例如在销售流程中，流程如下：发送询价单->确认询价->创建发货单->发货
   高亮的按钮强调下一步的流程，用于提示用户，通常放在第一个。另外取消按钮一般被设置成灰色，如在发货里退款按钮不会被设置为高亮。通过设置oe_highlight的class属性来将按钮元素高亮显示
   `<button class="oe_highlight" name="..." type="..." states="..."/>`
- 状态
   使用statusbar 部件，并将当前的状态标红。普通的状态会一直显示出来，异常或依赖于其他流程的状态只有在当前状态下才会显示。states是根据字段值对应的顺序来展示的，一直显示的状态可通过statusbar_visible属性指定：
   `<field name="state" widget="statusbar" statusbar_visible="draft,sent,progress,invoiced,done" />`

#### 4.表格

业务视图展示出来要像一张表格一样

![img](http://www.odoo.com/documentation/10.0/_images/sheet.png)

- 在page或form里的元素不会自动分组，它们会根据普通的html规则来布局，可以通过group或div标签来进行分组展示
- 默认情况下group标签定义两列，可以通过col="n"来指定多列，并且每列的宽度是一样的。
- 可以给group标签添加string属性来给该区块内容添加标题
   `<group string="Time-sensitive operations">`
- 不在group内的field标签默认是不生成label的，可以通过<label for="field_name>来指定label

1.表格头
 某些表格是它们只在编辑模式下才显示字段的标签

![img](https:////upload-images.jianshu.io/upload_images/4048165-97f5749161967be2.png?imageMogr2/auto-orient/strip|imageView2/2/w/365)



![img](https:////upload-images.jianshu.io/upload_images/4048165-3c42f4c2eb38bf18.png?imageMogr2/auto-orient/strip|imageView2/2/w/554)

使用oe_edit_only 的class属性来指定label只在编辑模式下才展示，oe_inline 属性将多字段设置展示到单行中

```csharp
<label for="name" class="oe_edit_only"/>
<h1><field name="name"/></h1>

<label for="planned_revenue" class="oe_edit_only"/>
<h2>
    <field name="planned_revenue" class="oe_inline"/>
    <field name="company_currency" class="oe_inline oe_edit_only"/> at
    <field name="probability" class="oe_inline"/> % success rate
</h2>
```

2.按钮
 一个表单中可以放置多个相关的操作按钮，如果商机表单里有安排一个电访、安排一次会面，可直接显示在表单中

```jsx
<div class="oe_button_box" name="button_box">
    <button string="安排电访" name="..." type="action"/>
    <button string="安排会面" name="action_makeMeeting" type="object"/>
</div>
```

3.分组和标题
 为了方便视图扩展，需要给group指定一个name属性，这样在扩展视图中可以更容易的找到添加字段的正确位置

##### 特殊案例：小计

某些class是用来展示小计时使用的，比如发货单：



![img](https:////upload-images.jianshu.io/upload_images/4048165-7f4b1ba70217a5f1.png?imageMogr2/auto-orient/strip|imageView2/2/w/1028)



```csharp
<group class="oe_subtotal_footer">
    <field name="amount_untaxed"/>
    <field name="amount_tax"/>
    <field name="amount_total" class="oe_subtotal_footer_separator"/>
    <field name="residual" style="margin-top: 10px"/>
</group>
```

4.placeholder和行内输入框
 有时候字段的标签把表单搞的太复杂了，可以将标签隐藏而采用placeholder来提示用户该输入框需要输入什么，同时可以通过div包裹多个inline输入框来让它们像是单个字段一样显示，例：

```xml
<group>
    <label for="street" string="Address"/>
    <div>
        <field name="street" placeholder="Street..."/>
        <field name="street2"/>
        <div>
            <field name="zip" class="oe_inline" placeholder="ZIP"/>
            <field name="city" class="oe_inline" placeholder="City"/>
        </div>
        <field name="state_id" placeholder="State"/>
        <field name="country_id" placeholder="Country"/>
    </div>
</group>
```

5.图片
 图片一般在表单的右边显示
 `<field name="product_image" widget="image" class="oe_avatar oe_right"/>`

6.标签
 大多数多对多关系字段，如分类，一般使用多标签来展示
 `<field name="category_id" widget="many2many_tags"/>`

### 配置表单指导原则

一般不需要header（因为没有状态、工作流、按钮），不需要表格，如：阶段配置

![img](https:////upload-images.jianshu.io/upload_images/4048165-0952afac48e93573.png?imageMogr2/auto-orient/strip|imageView2/2/w/1128)

### 弹出框表单指导原则

例：商机的安排电话访问表单



![img](https:////upload-images.jianshu.io/upload_images/4048165-c868de21b4a0744d.png?imageMogr2/auto-orient/strip|imageView2/2/w/1077)

1.避免使用分隔线
 2.不使用取消按钮（因为一般用户会直接关闭弹框来替代）
 3.操作钮钮必须被高亮显示
 4.如果是文本框，使用placeholder来代替label
 5.将按钮放在header元素中

### 配置向导指导原则

例：设置-配置-销售
 1.单行显示（没有弹出框）2.没有表格 3.保留取消按钮 4.保存按钮标红

## 图表

图表是用于将数据进行聚合显示用的，它的根标签是gragh，有以下几种属性：

- type - bar 柱形图（默认）、line 线形图 、 pie 扇形图
- stacked - 只在柱形图里使用，设置为True的时候，会在将柱形图排到group里

图表里唯一能插入的元素就是field，它有以下几种属性

- name 必选 -- 用在图表中的字段名
- type -- 指定该字段是进行分类统计还是总计

> row 默认 -- 根据指定字段分组，所有图表至少支持一级分组，有些支持多级，在pivot视图中，每个分组拿自己的记录
>  col -- 只在pivot表中使用，创建一个列优先的分组集合
>  measure -- 分组中用来聚合的字段

- interval -- 在基于date或datetime字段进行分组统计时使用day, week, month, quarter , year来计算

### pivot

pivot视图使用 pivot表来展示汇总数据，根标签是pivot，有以下属性：

- disable_linking - 设置为True时取消单元格和数据列表视图之间的链接
- display_quantity - 设置为True时以默认方式显式数量

pivot跟其他图表一样也只允许field元素

## 看板视图

看板视图展示出一个看板图，由多个卡片构成，可以是对列分组展示或直接展示，它的根标签是kanban，有以下属性

- default_group_by -- 当action或search没有进行分组时，视图是否需要进行分组，取值为用于进行分组的字段名
- default_order -- 当用户没有对记录进行排序时卡片中所使用的排序字段
- class -- 添加看板视图根html的类属性
- quick_create -- 是否可以在不切换到表单视图的情况下直接创建记录，当看板视图是经过分组的时候它是启用的，否则不启用，可通过设置True来强制启用，False强制禁用

子元素可以是以下几种

#### field

定义用来集合计算或用在看板视图逻辑中的字段，如果某字段仅用于在看板视图中展示，它不需要预先进行定义，有以下属性：

- name (required) -- 用于获取数据的字段名
- sum, avg, min, max, count -- 在看板视图最上方展示对应的计算后的值，每个字段只支持一个

#### templates

定义一个QWeb模板列表，卡片可以分割成多个模板，但看板视图至少需要定义一个kanban-box标签，每条记录会执行一次，看板视图用的是严格的qweb javascript，有以下几个环境变量：

- instance -- 当前qweb实例
- widget -- 可用来获取元数据信息，
- record -- 一个带有所有被请求字段的对象，每个字段有value和raw_value两个属性，value遵循当前用户格式，raw_value是直接读取出来的数据
- formats -- 用于操纵和转换值的web.formats()模块
- read_only_mode -- 只读

### 按钮和字段

由于看板模板是标准的qweb，看板视图有特殊的处理field、button、a标签的方式

- 默认情况下字段值显示的是格式化之后的，除非它匹配了对应的视图widget
- 拥有type属性的按钮和链接会转换成odoo相关的操作：

> 1.action,object  -- 与odoo按钮的属性一致

2.open --  在只读模式下打开卡片的视图
 3.edit --  在编辑模式下打开卡片视图
 4.delete -- 删除卡片的记录且移除卡片

### javascript API

class KanbanRecord()

- Widget() 将单条记录解析到卡片中
- kanban_color(raw_value) 将一个color片段转换成oe_kanban_color_color_index
- kanban_getcolor(raw_value) 将color片段转换为color_index
- kanban_image(model, field, id[, cache][, options]) 将指定字段转换成图片URL

> model -- 保存图片的model， field -- 保存图片数据的字段名 ， id -- 需要展示图片的记录id，cache--图片在浏览器的缓存时间（秒），0表示不缓存

- kanban_text_ellipsis(string[, size=160]) 将比较长的内容提取一部分显示

## 日历视图

日历视图按天、周、月来显示数据，根元素是calendar，有以下属性：

- date_start (必选) -- 储存开始时间的字段
- date_stop -- 储存结束时间的字段，当提供了该字段时记录可以直接在视图中删除
- date_delay -- 与date_stop类似，表示的是该事件的持续时间
- color -- 用于定义颜色的字段，颜色字段值相同的记录会在视图中以相同的颜色显示
- event_open_popup -- 以弹框代替表单来打开事件，默认是禁用的
- quick_add -- 允许快速添加事件，只需要提供name就行，当创建失败时会转到一个完整的表单弹出框
- display -- 将字段名用[]包裹展示
- all_day -- 布尔型，用来定义对应事件是否是全天有效
- mode -- 默认的显示模式：day, week, month

## 甘特图

甘特图用于展示甘特图表如流程，根元素是gantt，没有子元素，但可以有以下属性：

- date_start (必选) -- 储存开始时间的字段
- date_stop -- 提供结束时间的字段，可以用date_delay来实现同样的作用，两者必须提供一个，如果该字段被设置为False，那该事件的开始时间和结束时间是同个时间点
- date_delay -- 提供事件持续时间的字段
- duration_unit -- 持续时间的单位，minute, hour (默认), day, week, month, year
- default_group_by -- 任务分组的依据字段
- type -- gantt（默认） 传统甘特图、 consolidate(首个child的值被合并甘特图任务中)、planning(children会自动显示到甘特图任务中）
- consolidation -- 在记录单元格中用于显示合并值的字段名
- consolidation_max -- 数据字典，表示超过一定的值会标红显示 ，如：`{"user_id": 100}`
- string -- 展示在合并值旁边的字符，如果没设置会自动取对应字段的label
- fold_last_level -- 如果设置了该属性，最后一个分组级别会被折叠
- round_dnd_dates -- 开始和结束时间取整
- drag_resize -- 任务调整，默认True

## 示意图

示意图可用来展示原来就是图表的记录，根元素是diagram，没有属性，有几种子元素：

### node（必选）

定义图表的节点，有以下属性：

- object -- 节点对应的model
- shape -- 就像列表视图的颜色、字体一样的形状表示，唯一可选的取值是rectangle （长方形），默认无
- bgcolor -- 用来表示节点的背景颜色，默认是白色，可取值grey

### arrow （必选）

用于定义图表的箭头，有以下属性：

- object（必选） -- 箭头对应的model
- source (必选) -- model的Many2one字段，用于指向箭头的源节点数据
- destination (必选) -- model的Many2one字段，指向箭头的目标节点数据
- label -- python格式的属性列表，相应的属性值会用作箭头的label显示

### label

用于解释示意图的节点，string属性定义的是节点的内容，每个label带有编号显示在示意图头部

## 搜索视图

搜索视图与其他视图不同，因为它不实际显示内容，它用于过滤其他视图的内容，定义的形式却一样。它的根元素是search，没有属性。可以有以下几中子元素：

### field

field使用用户提供的值来定义domain表达式和上下文环境，当产生搜索domain表达式后，会与field提供的表达式使用and进行合并作用，可有以下几种属性：

- name -- 需要过滤的字段名
- string -- 字段的label
- operator -- 默认情况下field会生成[(name, operator, provided_value)]格式的表达式，其中name是字段名，provided_value是用户提供的值，operator属性可以重写默认的运算符（默认情况下是根据字段类型分配，数字型是=，字符型是ilike）
- filter_domain -- 用于搜索的完整的domain表达式，可以用self变量来将提供的值注入，当operator和filter_domain同时赋值时，filter_domain有最高优先级
- context -- 允许添加上下文的值
- groups -- 使该字段只对某些用户组可用
- widget -- 使用指定的搜索部件（唯一的用例是V8的many2one字段选择插件）
- domain -- 如果字段提供自动完成时（many2one），过滤出可能的自动完成结果

### filter

过滤器搜索视图里是被预定义的，只能被启用或禁用。主要用于将数据添加到搜索的上下文环境或者添加新的片段到搜索filter，有以下属性：

- string (required) -- 过滤器的label
- domain -- 一个domain表达式，被添加到action的domain表达式中，作为搜索的domain表达式一部分
- context -- 一个python格式数据字典，被合并到action的domain表达式中，用于生成搜索的domain表达式
- name -- 过滤器的逻辑名
- help -- 过滤器的描述文字
- groups -- 指定过滤器可用的用户组

```xml
<filter domain="[('state', '=', 'draft')]"/>
<filter domain="[('state', '=', 'done')]"/>
#上述代码当两个都选的时候是以or联接的

<filter domain="[('state', '=', 'draft')]"/>
<separator/>
<filter domain="[('delay', '<', 15)]"/>
#上述代码当两个都选的时候用and联接
```

### separator

用于将多组过滤器分开，一般用于很简单的视图里

### group

也是用于分离多组过滤器，在复杂的视图中比separator更加易读

### 默认搜索

搜索字段和过滤条件可以通过action的context使用search_default_name 配置，对于字段就是需要搜索的值，对过滤器它是一个布尔值，例：

```bash
{
  'search_default_foo': 'acro',
  'search_default_bar': 1
}
#自动激活bar过滤器，并在foo字段搜索acro
```

## QWeb视图

QWeb视图是标准的qweb模板，嵌入在视图的arch标签中，没有指定的根标签
 每个qweb视图只能包含一个模板，模板名必须与视图的id完全一致

