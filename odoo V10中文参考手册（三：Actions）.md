## action简介

actions定义了系统对于用户的操作的响应：登录、按钮、选择项目等，action可以保存在数据库或在按钮方法中以数据字典的形式返回。每个action有两个必选属性：

- type -- 响应动作的类型，决定使用哪个字段或动作的响应方式
- name -- 在用户界面中显示给用户的易读的动作描述

客户端可以通过4种形式来获取action

- False - 会自动关闭所有的对话框
- String - 如果有相匹配的客户端动作，会被解释为相应动作的标签，否则会当成Number处理
- Number - 从数据库读取相应的action记录，一般是数据库标识id
- Dict - 当成客户端动作来执行

## 窗口Action(ir.actions.act_window )

最常用的action类型，用于将model的数据展示出来

>字段列表：
> 1.res_model -- 需要在view里显示数据的model
> 2.views -- 一个(view_id, view_type) 列表，view_type代表视图类型如：form,tree,gragh...，view_id是可选的数据库id或False，如果没有指定id，客户端会自动用fields_view_get()获取相应类型的默认视图，type参数列表的第一个会被默认用来展示
> 3.res_id (可选) -- 当默认的视图类型是form时，可用于指定加载的数据
> 4.search_view_id (可选) -- (id, name)，id是储存在数据库的搜索视图，默认会读取model的默认搜索视图
> 5.target (可选) -- 定义视图是 在当前视图上打开(current)、使用全屏模式(fullscreen)、使用弹出框(new)、可使用main代替current来清除面包屑导航
> 6.context (可选) -- 额外的需要传给视图的环境数据
> 7.domain (可选) -- 自动添加到视图搜索中的查询
> 8.limit (可选) -- 在客户端显示的数据量，默认80
> 9.auto_search(可选) -- 搜索是否在加载默认视图后立即执行，默认True

```php
#用列表和表单视图来打开customer按钮
{
    "type": "ir.actions.act_window",
    "res_model": "res.partner",
    "views": [[False, "tree"], [False, "form"]],
    "domain": [["customer", "=", true]],
}

#在新的对话框中打开一个指定产品的表单
{
    "type": "ir.actions.act_window",
    "res_model": "product.product",
    "views": [[False, "form"]],
    "res_id": a_product_id,
    "target": "new",
}
```

保存在数据库里窗口action有一些不同的需要被客户忽略的字段，大多数情况下用来组成视图列表

- view_mode -- 以逗号分隔的视图类型列表，所有类型的视图会被展示出来
- view_ids -- 视图对象的一系列的字段，用于定义视图的默认内容
- view_id -- 将指定的view加入到视图中，以防不被view_ids所包含
   上述参数一般在使用数据文件定义action的时候使用：

```xml
<record model="ir.actions.act_window" id="test_action">
    <field name="name">A Test Action</field>
    <field name="res_model">some.model</field>
    <field name="view_mode">graph</field>
    <field name="view_id" ref="my_specific_view"/>
</record>
#默认使用my_specific_view，即使它不是对应模型的默认视图
```

服务端组合视图的步骤：

- 依次获取view_ids的(id, type)
- 如果定义了view_id而且它的类型没有被包含在其中，将它加到最前面
- 对于所有没有指定的view_mode，加一个(False,type)

## 链接Action(ir.actions.act_url)

可以通过odoo的链接打开一个网站页面，可通过两个字段来自定义：

- url -- 当激活action时所打开的链接
- target -- new：在新窗口打开，self：替换当前页面内容，默认new

```json
{
    "type": "ir.actions.act_url",
    "url": "http://odoo.com",
    "target": "self",
}
```

## 服务器Action (ir.actions.server)

可以通过action定位来触发复杂的服务端代码，需要两个与客户端相关的字段：

- id -- 服务端action在数据库存储的id
- context (可选) -- 执行服务端action的上下文环境
   储存在数据库中的action可以基于state执行一些特别的动作，部分字段在state之间是相互共享的
- model_id -- 与action相关联的model，在 evaluation contexts中可用
- condition (可选) -- 使用服务端的  evaluation contexts 来执行python代码，如果是False则阻止action执行，默认值是True

动作类型是可以随意扩展的，默认的动作类型：

- code --  当调用action时执行的python代码

```csharp
<record model="ir.actions.server" id="print_instance">
    <field name="name">Res Partner Server Action</field>
    <field name="model_id" ref="model_res_partner"/>
    <field name="code">
        raise Warning(object.name)
    </field>
</record>

# 在code片段中可以定义一个action变量，会被返回给客户端用于指定下一个执行的action
<field name="code">
    if object.some_condition():
        action = {
            "type": "ir.actions.act_window",
            "view_mode": "form",
            "res_model": object._name,
            "res_id": object.id,
        }
</field>
```

- object_create -- 使用钩子创建一条新记录（通过create或copy方法）
- use_create
   1.new - 基于指定的 model_id创建一条记录
   2.new_other - 基于指定的crud_model_id创建一条记录
   3.copy_current - 复制action所引用的记录
   4.copy_other - 复制一个通过ref_object获得的记录
- fields_lines --当创建或复制记录时需要修改的字段，One2many 会有以下字段：
   1.col1 -- 在use_create里所包含的需要被重赋值的ir.model.fields
   2.value -- 字段对应的值，基于type进行解析
   3.type -- 取值value:就是value字段的值，取值equation：value字段会当成python来解析
- crud_model_id -- 当use_create为new_other时，表示用于创建新记录的model id
- ref_object -- 当use_create为copy_other时用于指定创建记录时引用的记录
- link_new_record -- 是否用用link_field_id将新记录和当前记录进行many2one关联，默认False
- link_field_id -- 指定当前记录与新记录进行many2one关联的字段
- object_write -- 与object_create相似，只是只修改当前记录而不创建新记录
- use_create
   1.current - 修改更新到当前记录
   2.other - 修改更新到通过crud_model_id 或 ref_object指定的新记录
   3.expression - 修改更新到通过crud_model_id 以及 write_expression筛选过后的记录
- write_expression - 返回一条记录或对象id的python表达式
- fields_lines,crud_model_id,ref_object与object_create一致
- multi
   将通过child_ids many2many关系定义的action一个个执行，如果有action自己返回action，最后一个action被返回给客户端作为将前multi action的下一个action
- trigger 发送一个信号给工作流
- wkf_transition_id - 用于触发的与workflow.transition有Many2one关系的id
- use_relational_model - 如果是base（默认），则触发当前记录的维护信号；如果是relational，则触发通过wkf_model_id 和 wkf_field_id筛选出来的当前记录的字段
- client_action -- 返回通过action_id定义的action

### 上下文环境

有些键在上下文环境和服务端action里是可用的：

- model -- 通过model_id与action关联的model
- object, obj -- 只在有active_model 和active_id才可用，给出经过active_id过滤的记录
- pool -- 当前数据库注册
- datetime, dateutil, time -- python模块
- cr -- 当前查询游标
- user -- 当前用户记录
- context -- 执行上下文环境
- Warning -- 警告异常的构造器

## 报表Action (ir.actions.report.xml)

此action为打印报表的触发器

- name(必选) -- 在一个列表里进行查找时使用
- model (必选) -- 报表所反映的数据来源model
- report_type  (必选) -- qweb-pdf | qweb-html
- report_name -- 报表命名，用于输出的pdf文件名
- groups_id -- 可以读取或使用当前报表的用户组，Many2many字段
- paperformat_id -- 报表所使用的纸张格式，默认使用公司的格式，Many2one字段
- attachment_use -- 当取值true的时候只在第一次请求时生成报表，之后直接从保存的报表打印，可用于生成后不会有改变的报表
- attachment -- 使用python表达式来定义报表名字，该记录可用变量object访问

## 客户端Actions (ir.actions.client)

触发一个完全在客户端实现的action

- tag -- action在客户端的标识符，一般是一个专用的字符串
- params (可选) -- 用来传给客户端的python数据字典格式数据
- target (可选) -- current:当前内容区打开action,fullscreen:以全屏模式打开，new：以弹出框打开

```bash
#例：打开一个pos界面，不需要服务端知道它是如何运行的
{
    "type": "ir.actions.client",
    "tag": "pos.ui"
}
```

