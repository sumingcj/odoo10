# 翻译模块

## 导出可译术语

一个模块的有很多的内容都是可以翻译的，可通过导出功能来决定翻译哪些内容。
 在设置-翻译-导入/导出-导出翻译中：选择默认语言、po格式、需要翻译的模块，点击导出。
 会得到一个pot文件，用来放在`yourmodule/i18n/`文件夹中，这个文件决定了哪些字段被翻译，并用msginit生成po文件放在同个目录中，并命名为`language.po`，当odoo系统语言设置为对应的时候会自动加载。

## 隐式输出

odoo默认将data类型内容翻译输出：

- 在非qweb视图中，所有属性如string, help, sum, confirm ， placeholder都和内容一起被输出
- qweb模板中除了声明`t-translation="off"`外的所有文本节点都被输出
- 除了模型被标记为`_translate = False`的field，它们的string，help被输出，如果有selection而且是列表或元组也会被输出，如果translate 属性设置为True，它所有已存在的值都会被输出
- 在_constraints 和_sql_constraints 中的帮助或错误消息也会被输出

## 明确输出

当有时需要在python或javascript代码中使用的，odoo是不能自动输出的，可以通过调用方法来实现：

```bash
#在python中，方法是 odoo._()
title = _("Bank Accounts")

#在javascript中，方法是odoo.web._t()
title = _t("Bank Accounts")
```

>注意只有照原意的不带变量的才能被输出

```bash
# 这个可能不会被正确地翻译出来
_("Scheduled meeting with %s" % invitee.name)

# good
_("Scheduled meeting with %s") % invitee.name
```

# QWeb报表

odoo中报表也是使用qweb定义的，pdf导出是使用wkhtmltopdf来完成的。
 如果需要为一个模型创建报表，需要定义report及对应模板。如果有需要的话还可以指定特定的纸张格式，如果需要访问其他模型，就需要定义Custom Reports，这样才能在模板中获取其他模型的数据。

## Report

每个report必须用report action定义；为了方便，`report`标签可用于定义一个报表，而不用去定义对应action和其他参数。`<report>`标签可带有以下属性：

- id - 生成的数据的id
- name (必选) - 报表名用于查找及描述
- model (必选) - 报表所对应的模型
- report_type (必选)  - `qweb-pdf`: pdf | `qweb-html` : html
- report_name - 输出pdf时文件名
- groups - Many2many字段用于指定可以查看使用该报表的用户组
- attachment_use - 如果设置为true时，该报表会以记录的附件的形式保存，一般用于一次生成多次使用的报表
- attachment - 用于定义报表名的python表达式，记录可以通过object对象访问
- paperformat - 用于打印报表的文件格式的外部id（默认是公司的格式）

例：

```xml
<report
    id="account_invoices"
    model="account.invoice"
    string="Invoices"
    report_type="qweb-pdf"
    name="account.report_invoice"
    file="account.report_invoice"
    attachment_use="True"
    attachment="(object.state in ('open','paid')) and
        ('INV'+(object.number or '').replace('/','')+'.pdf')"
/>
```

## 报表模板

### 最简单的模板

```xml
<template id="report_invoice">
    <t t-call="report.html_container">
        <t t-foreach="docs" t-as="o">
            <t t-call="report.external_layout">
                <div class="page">
                    <h2>Report title</h2>
                    <p>This object's name is <span t-field="o.name"/></p>
                </div>
            </t>
        </t>
    </t>
</template>
```

通过调用`external_layout`来给报表添加默认的头部和尾部，pdf内容会是`<div class="page">`里的内容。模板id需与报表声明中一致，比如上面的`account.report_invoice`，由于这是qweb模板，可以在`docs`对象中取得字段内容。

在报表中有几个内置的变量：

- `docs` - 当前报表的数据
- `doc_ids` - docs记录里的id列表
- `doc_model` - docs记录对应的模型
- `time` - 指向python time库的引用
- `user` - 生成报表的res.user记录
- `res_company` - 生成报表用户的公司

如果需要在模板中使用其他的模型或记录，可使用自定义报表。

### 可翻译的模板

如果想要对报表进行翻译，需要定义两个模板：主报表模板、翻译文档
 然后可以在主报表模板中通过`t-lang`属性来设置语言代码并调用翻译文档，如果有用到可翻译的字段如国家名时，也需要在对应环境中看一下相关的记录有没有问题。

```xml
#订单报表
<!-- 主模板 -->
<template id="report_saleorder">
    <t t-call="report.html_container">
        <t t-foreach="docs" t-as="doc">
            <t t-call="sale.report_saleorder_document" t-lang="doc.partner_id.lang"/>
        </t>
    </t>
</template>

<!-- 翻译模板 -->
<template id="report_saleorder_document">
    <!-- 使用合作伙伴的语言环境检查一下 -->
    <t t-set="doc" t-value="doc.with_context({'lang':doc.partner_id.lang})" />
    <t t-call="report.external_layout">
        <div class="page">
            <div class="oe_structure"/>
            <div class="row">
                <div class="col-xs-6">
                    <strong t-if="doc.partner_shipping_id == doc.partner_invoice_id">Invoice and shipping address:</strong>
                    <strong t-if="doc.partner_shipping_id != doc.partner_invoice_id">Invoice address:</strong>
                    <div t-field="doc.partner_invoice_id" t-options="{"no_marker": True}"/>
                <...>
            <div class="oe_structure"/>
        </div>
    </t>
</template>
```

在上例中，所有销售订单报表会根据客户的语言来打印，如果只需要替换内容而文件头尾使用默认语言的话，可以在调用布局时使用：`<t t-call="report.external_layout" t-lang="en_US">`

### 二维码

二维码是由controller生成的图片，并且可以很容易的嵌入到报表中。
 `![]('/report/barcode/QR/%s' % 'My text in qr code')`
 还可以使用查询url来传多个参数：`<img t-att-src="'/report/barcode/? type=%s&value=%s&width=%s&height=%s'%('QR', 'text', 200, 200)"/>`

### 其他可能有用的

- bootstrap和fontawsome类可以用在报表模板中
- 本地css可以直接放在报表中
- 全局css可以被插入到主报表模板布局中并进行继承。

```xml
<template id="report_saleorder_style" inherit_id="report.style">
  <xpath expr=".">
    <t>
      .example-css-class {
        background-color: red;
      }
    </t>
  </xpath>
</template>
```

- 如果显示样式有问题，检查一下wkhtmltopdf 插件和代理问题

## 文件格式

文件格式是`report.paperformat`记录，有以下属性：

- name (必选) - 用于查找及区分的名字
- description - 格式的描述
- format - 一个预定义的格式如（A0-A9，B0-B10等）或自定义，默认是A4
- dpi - 输出的DPI，默认90
- margin_top, margin_bottom, margin_left, margin_right - mm为单位的margin值
- page_height, page_width - mm为单位的尺寸
- orientation - 横向或纵向 Landscape ， Portrait
- header_line - boolean，是否显示标题行
- header_spacing - mm为单位的头部空白

```xml
<record id="paperformat_frenchcheck" model="report.paperformat">
    <field name="name">French Bank Check</field>
    <field name="default" eval="True"/>
    <field name="format">custom</field>
    <field name="page_height">80</field>
    <field name="page_width">175</field>
    <field name="orientation">Portrait</field>
    <field name="margin_top">3</field>
    <field name="margin_bottom">3</field>
    <field name="margin_left">3</field>
    <field name="margin_right">3</field>
    <field name="header_line" eval="False"/>
    <field name="header_spacing">3</field>
    <field name="dpi">80</field>
</record>
```

## 自定义报表

报表模型默认有一个 get_html 方法用于查找 report.module.report_name名字的模型，如果存在会用它来调用qweb引擎，否则会用普通函数。如果想要在报表模板中添加一些其他的内容如其他模型的记录，可以将render_html 覆盖并通过docargs 传递对象

```python
from odoo import api, models

class ParticularReport(models.AbstractModel):
    _name = 'report.module.report_name'
    @api.model
    def render_html(self, docids, data=None):
        report_obj = self.env['report']
        report = report_obj._get_report_from_name('module.report_name')
        docargs = {
            'doc_ids': docids,
            'doc_model': report.model,
            'docs': self,
        }
        return report_obj.render('module.report_name', docargs)
```

## 报表和网页

报表是由报表模块动态生成的，而且可以通过url访问。
 如果想看一个销售订单报表，可以通过`http://<server-address>/report/html/sale.report_saleorder/38` 查看网页版，也可以通过`http://<server-address>/report/pdf/sale.report_saleorder/38`查看pdf版

# 工作流

在odoo中工作流用于人工管理与模型数据相关的一系列任务。另外也提供一个更高级的方式来管理作用于记录的任务。
 工作流是一个有方向的图表，每个节点叫活动，中间的线叫做转变。

- 活动定义了odoo服务需要做的事，比如改变数据的状态、发送邮件
- 转变 控制工作流如何从一个活动进行到下一个

在工作流定义中，可以添加条件、信号、转变的触发器，这样工作流的动作会依赖于用户操作、数据变化、或其他python代码。

总之，odoo的工作流提供了以下特性：

- 一条记录或文档随着时间演变的过程
- 在不同条件下的自动作用
- 管理公司角色和验证过程
- 管理对象之间的互动
- 很明显的显示文档在生命周期中的流动

> 例：一个基本的订单有以下流程：
>  生成 -> 确认  ->  关闭|取消

订单从生成状态开始，可由用户确认，然后 发货（关闭）或取消

一个公司可能想要为订单打折，销售人员可以自由决定折扣比例（最高15%），当高于15%时需要由管理人员决定，该工作流可以直接在界面上修改，不需要更改任何代码。
 由于活动可以处理各种各样的请求，验证过程中验证请求就可以自动发送到对应的员工那里。

## 基础

定义一个带有数据的工作流比较简单：给出一个拥有活动和转变的workflow记录。

```xml
<record id="test_workflow" model="workflow">
    <field name="name">test.workflow</field>
    <field name="osv">test.workflow.model</field>
    <field name="on_create">True</field>
</record>

<record id="activity_a" model="workflow.activity">
    <field name="wkf_id" ref="test_workflow"/>
    <field name="flow_start">True</field>
    <field name="name">a</field>
    <field name="kind">function</field>
    <field name="action">print_a()</field>
</record>
<record id="activity_b" model="workflow.activity">
    <field name="wkf_id" ref="test_workflow"/>
    <field name="flow_stop">True</field>
    <field name="name">b</field>
    <field name="kind">function</field>
    <field name="action">print_b()</field>
</record>

<record id="trans_a_b" model="workflow.transition">
    <field name="act_from" ref="activity_a"/>
    <field name="act_to" ref="activity_b"/>
    <field name="signal">signal_goto_b</signal>
</record>
```

一个工作流总是被定义在一个通过osv属性指定的model上，在activity和transition里定义的函数就是调用的该模型的。

> 上面例子定义的工作流中,定义了两个活动a和b，一个从a到b的转变，第一个活动通过flow_start 设置为true来告诉系统工作流从它开始。由于工作流的on_create属性为true，每条新创建的记录都会自动创建相应工作流，否则工作流需要用其他方式如python代码来创建。
>  1.当工作流开始时，从活动a执行，该活动是一个函数，所以会自动调用test.workflow上的print_a方法（cr, uid, ids, context参数会自动传过去）
>  2.a和b的转换定义了一个信号但没有条件，这就意味着只要收到signal_goto_b 信息，工作流就会自动转到b

## 活动

转换可以在工作流结构中看到，但活动是真正进行处理的地方。有多种活动类型： Dummy, Function, Subflow,  Stop all，在活动完成后每一种做的事不一样，除此之外，活动还有其他的属性

### 流程开始和结束

flow_start(boolean)属性用于指定活动是否在工作流实例化时处理。可以有多个活动同时设置为True，当工作流实例化时就将它们全部处理，并对他们下一步的转换进行评估。

flow_stop(boolean)属性用于指定在哪个活动中停止工作流实例。在所有的flow_stop为True的活动完成后，工作流才会被视为已完成。

一个活动可以是另外一个流程，也叫子流程，只有子流程处理完成，该活动才算处理完成。

### 子流程

一个活动可以是一个子流程，该流程通过`subflow_id`来实例化。

#### 从子流程发送信号

子流程可以通过一个signal_send属性来向上级流程发送信号，父流程实例中可以通过`subflow`来访问子流程的signal_send,所在当活动在子流程中运行时，上级流程可以知道当前运行状态。

### 服务端Action

一个活动可以通过定义一个`action_id`属性来指向一个服务端action

### Python action

一个活动可以通过`action`属性来指定执行一段python代码，它的执行环境与Transition Condition里的一致，本单下面会有。

### 分解模式

当一个活动被处理后，odoo自动进行转换并决定下一个活动，如果一个活动有多个转换条件，odoo需要决定接下来去执行哪一个或多个。选项由split_model的属性控制：

- XOR (默认) -默认情况下odoo会用按顺序用第一个符合条件的转换，其他的会被忽略
- OR - 在此模式下所有满足条件的转换同时被执行，不符合条件的被忽略
- AND - 此模式下odoo会等会有条件全满足后再一次全部转换

### 合并模式

与分解模式类似，多个转换可以同时指向同一个活动。`join_mode`属性可以有以下值：

- XOR (default) - 只要有一个转换满足就能触发活动的处理
- AND - 必须全部转换都满足才会处理活动

### 类型

活动类型定义了它可以执行什么样的任务

- Dummy (dummy, default)  - 调用一个服务端action或者什么也不做，一般用于调度
- Function (function) - 运行python代码，执行服务端action
- Stop all (stopall) - 将工作流实例停止并标记为已完成
- Subflow (subflow) - 开始运行另一个工作流，当新工作流完成后该活动就算处理完成。默认情况下子工作流会用原工作流的数据进行初始化，也可通过一个python表达式来给出新的记录id，子工作流就会使用新记录来实例化

## 转变

转变过程为工作流安排提供结构控制。当一个活动完成时，工作流引擎会从结束的活动出发转到下一个活动，在简单的形式中，工作流是顺序的，并且前面的一完成后续的就会执行。
 也可以不一下子全执行，可以在转变中进行等待，只在满足一些规则的时候才转到下一个。有条件、信号、触发器规则：

### 条件

当一个活动结束时，它的对应的转变过程会检查来决定是否可以转到下一个活动，当只定义了条件（没有信号和触发规则）时，odoo会评估条件，如果得到结果是True，该工作流就通过了这个转变过程，如果条件不满足的话它就会在对应值改变时自动重新评估或通过一个显示方法调用来评估。

默认情况下，`condition`属性（被用于评估的表达式）是True，如果表达式是多行，最后一行决定它的值。在条件被运行时，所有模型的列名和相关记录的属性都会被自动定义到odoo的`safe_eval`环境变量中

### 信号

在条件规则的基础上，一个转变过程可以指定一个信号规则。当声明了信号规则后，就算满足条件了，转变也会被冻结并等待唤醒。
 可以通过发送信号来唤醒对应的转变过程，通常是在用户界面里添加一个按钮，将name属性设置为信号名，一旦点击了按钮，信号就自动被发送到当前记录的工作流实例。

### 触发器

当条件不满足时，转变就不会发生。可以通过一个触发器来跳过转变触发下一个活动，当条件不满足时，触发器会自动保存到数据库，然后，就可以通过相应触发器来唤醒工作流，促使它们重新评估转换条件。这个机制使得唤醒所付出的代价更下，因为只需要监听有触发器的部分。

触发器在数据库中保存相应的记录id和工作流实例的引用，转变过程定义了一个模型名`trigger_model`、一个用于评估对应模型记录id的python表达式`trigger_expression`，任意一条记录都可以唤醒与其相关联的工作流。