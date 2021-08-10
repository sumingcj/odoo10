Odoo 实现了一些有用的类和 mixin，使您可以轻松添加 对象上的常用行为。 本指南将详细介绍其中的大部分内容，包括 示例和用例。



## 消息功能 

### 消息整合 

#### 基本消息系统 

将消息传递功能集成到您的模型中非常容易。只需继承`mail.thread `模型并将消息传递字段（及其相应的小部件）添加到您的表单视图中，您就可以立即启动并运行。

>例子  
>
>让我们创建一个表示商务旅行的简单模型。由于组织这种旅行通常涉及很多人和很多讨论，所以我们在模型上添加对消息交换的支持。
>
>```python
>class BusinessTrip(models.Model):
>    _name = 'business.trip'
>    _inherit = ['mail.thread']
>    _description = 'Business Trip'
>
>    name = fields.Char()
>    partner_id = fields.Many2one('res.partner', 'Responsible')
>    guest_ids = fields.Many2many('res.partner', 'Participants')
>```
>
>在表单视图中： 
>
>```python
><record id="businness_trip_form" model="ir.ui.view">
>    <field name="name">business.trip.form</field>
>    <field name="model">business.trip</field>
>    <field name="arch" type="xml">
>        <form string="Business Trip">
>            <!-- Your usual form view goes here
>            ...
>            Then comes chatter integration -->
>            <div class="oe_chatter">
>                <field name="message_follower_ids" widget="mail_followers"/>
>                <field name="message_ids" widget="mail_thread"/>
>            </div>
>        </form>
>    </field>
></record>
>```

在模型上添加聊天支持后，用户可以轻松地在模型的任何记录上添加消息或内部注释；

每个人都会发送通知（向所有关注者发送消息，向员工（base.group_user）用户发送内部注释）。

如果您的邮件网关和通用地址配置正确，这些通知将通过电子邮件发送，并且可以直接从您的邮件客户端回复；

自动路由系统会将答案路由到正确的线程。 

在服务器端，一些辅助函数可以帮助您轻松发送消息并管理记录中的关注者：

##### 发布消息 

###### message_post(*self*, *body=''*, *subject=None*, *message_type='notification'*, *subtype=None*, *parent_id=False*, *attachments=None*, *content_subtype='html'*, ***kwargs*)

在现有线程中发布新消息，返回新的 mail.message ID。

>Parameters：
>
>- **body** ([`str`](https://docs.python.org/2/library/functions.html#str)) -- 消息的正文，通常是将被清理的原始 HTML
>- **message_type** ([`str`](https://docs.python.org/2/library/functions.html#str)) -- 请参阅 mail_message.type 字段
>- **content_subtype** ([`str`](https://docs.python.org/2/library/functions.html#str)) -- 如果纯文本：将正文转换为 html
>- **parent_id** ([`int`](https://docs.python.org/2/library/functions.html#int)) -- 在私人讨论的情况下，通过将父伙伴添加到消息来处理对先前消息的回复
>- **attachments** (`list(tuple(str,str))`) -- 形式（名称，内容）中的附件元组列表，其中内容不是 base64 编码的
>- ***\*kwargs** -- 额外的关键字参数将用作新 mail.message 记录的默认列值
>
>Returns：新创建的 mail.message 的 ID
>
>Return type：[int

###### message_post_with_view(views_or_xmlid, **kwargs)

使用 view_id 发送邮件/发布消息的助手方法，以使用 ir.qweb 引擎进行渲染。这种方法是独立的，因为模板和作曲家中没有任何东西可以批量处理视图。当模板处理 ir ui 视图时，此方法可能会消失。

>Parameters：
>
>**or ir.ui.view record** ([`str`](https://docs.python.org/2/library/functions.html#str)) -- 应发送的视图的外部 ID 或记录

###### message_post_with_template(*template_id*, ***kwargs*)

使用模板发送邮件的辅助方法

>Parameters：
>
>- **template_id** -- 要呈现以创建消息正文的模板的 id
>- ***\*kwargs** -- 用于创建 mail.compose.message 向导的参数（从 mail.message 继承）

##### 接收消息 

当邮件网关处理新电子邮件时，将调用这些方法。这些电子邮件可以是新线程（如果它们通过别名到达），也可以只是来自现有线程的回复。覆盖它们允许您根据电子邮件本身的某些值（即更新日期或电子邮件地址，将 CC 的地址添加为关注者等）在线程记录上设置值。

###### message_new(*msg_dict*, *custom_values=None*)

如果消息不属于现有线程，则在接收到给定线程模型的新消息时由 `message_process `调用。  

默认行为是创建相应模型的新记录（基于从消息中提取的一些非常基本的信息）。可以通过覆盖此方法来实现其他行为。

>Parameters：
>
>- **msg_dict** ([`dict`](https://docs.python.org/2/library/stdtypes.html#dict)) -- 包含电子邮件详细信息和附件的地图。有关详细信息，请参阅 `message_process `和` mail.message.parse`
>- **custom_values** ([`dict`](https://docs.python.org/2/library/stdtypes.html#dict)) -- 创建新线程记录时传递给 create() 的附加字段值的可选字典；请注意，这些值可能会覆盖来自消息的任何其他值
>
>Return type：[int](https://docs.python.org/2/library/functions.html#int)
>
>Returns：新创建的线程对象的 id

###### message_update(*msg_dict*, *update_vals=None*)

当接收到现有线程的新消息时，由 `message_process` 调用。默认行为是使用从传入电子邮件中获取的 `update_vals `更新记录。  

可以通过覆盖此方法来实现其他行为。

>Parameters：
>
>- **msg_dict** ([`dict`](https://docs.python.org/2/library/stdtypes.html#dict)) -- 包含电子邮件详细信息和附件的地图；有关详细信息，请参阅 `message_process` 和 `mail.message.parse()`。
>- **update_vals** ([`dict`](https://docs.python.org/2/library/stdtypes.html#dict)) -- 一个包含值的字典，用于根据其 ID 更新记录；如果 dict 为 None 或 void，则不执行写操作
>
>Returns：True

##### 追随者管理 

###### message_subscribe(*partner_ids=None*, *channel_ids=None*, *subtype_ids=None*, *force=True*)

将合作伙伴添加到记录关注者。

>Parameters：
>
>- **partner_ids** (`list(int)`) -- 将订阅记录的合作伙伴的 ID
>- **channel_ids** (`list(int)`) -- 将订阅记录的频道的 ID
>- **subtype_ids** (`list(int)`) -- 频道/合作伙伴将订阅的子类型的 ID（如果`None`，则默认为默认子类型）
>- **force** -- 如果为 True，则在使用参数中给出的子类型创建新关注者之前删除现有关注者
>
>Returns：Success/Failure
>
>Return type：[bool](https://docs.python.org/2/library/functions.html#bool)

###### message_subscribe_users(*user_ids=None*, *subtype_ids=None*)

message_subscribe 上的包装器，使用用户而不是合作伙伴。

>Parameters：
>
>- **user_ids** (`list(int)`) -- 将订阅记录的用户的 ID；如果没有，则改为订阅当前用户。
>- **subtype_ids** (`list(int)`) -- 频道/合作伙伴将订阅的子类型的 ID
>
>Returns：Success
>
>Return type：[bool](https://docs.python.org/2/library/functions.html#bool)

###### message_unsubscribe(*partner_ids=None*, *channel_ids=None*)

从记录的关注者中删除合作伙伴。

>Parameters：
>
>- **partner_ids** (`list(int)`) -- 将订阅记录的合作伙伴的 ID
>- **channel_ids** (`list(int)`) -- 将订阅记录的频道的 ID
>
>Returns：True
>
>Return type：[bool](https://docs.python.org/2/library/functions.html#bool)

###### `message_unsubscribe_users(*user_ids=None*)`

message_subscribe 上的包装器，使用用户。

> Parameters：
>
> **user_ids** (`list(int)`) -- 将取消订阅记录的用户的 ID；如果`None`，则取消订阅当前用户。
>
> Returns：True
>
> Return type：[bool](https://docs.python.org/2/library/functions.html#bool)

#### 记录更改 

`mail`模块在字段上添加了一个强大的跟踪系统，允许您在记录的聊天记录中记录对特定字段的更改。  

要将跟踪添加到字段，只需添加带有值 `onchange`（如果它应该仅在字段更改时在通知中显示）

或`always`（如果即使此特定字段未更改，值也应始终显示在更改通知中 - 例如，通过始终添加名称字段使通知更具解释性很有用）。

> 例子 
>
> 让我们跟踪我们出差的名称和责任人的变化：
>
> ```python
> class BusinessTrip(models.Model):
>     _name = 'business.trip'
>     _inherit = ['mail.thread']
>     _description = 'Business Trip'
> 
>     name = fields.Char(track_visibility='always')
>     partner_id = fields.Many2one('res.partner', 'Responsible',
>                                  track_visibility='onchange')
>     guest_ids = fields.Many2many('res.partner', 'Participants')
> ```
>
> 从现在开始，每次更改旅行名称或责任人都会在记录中记录注释。`name`字段也将显示在通知中，以提供有关通知的更多上下文（即使名称没有更改）。

#### 亚型 

子类型使您可以更精细地控制消息。子类型充当通知的分类系统，允许文档订阅者自定义他们希望接收的通知子类型。  

子类型在您的模块中作为数据创建；该模型具有以下字段：

- `name`（**mandatory**） - [`Char`](https://www.odoo.com/documentation/10.0/reference/orm.html#odoo.fields.Char)

  子类型的名称将显示在通知自定义弹出窗口中

- `description` -  [`Char`](https://www.odoo.com/documentation/10.0/reference/orm.html#odoo.fields.Char)

  将添加到此子类型发布的消息中的说明。如果为空，则将添加名称

- `internal` -  [`Boolean`](https://www.odoo.com/documentation/10.0/reference/orm.html#odoo.fields.Boolean)

  具有内部子类型的消息将仅对员工可见，即 `base.group_user` 组的成员

- `parent_id` -  [`Many2one`](https://www.odoo.com/documentation/10.0/reference/orm.html#odoo.fields.Many2one)

  自动订阅的链接子类型；例如，项目子类型通过此链接链接到任务子类型。当某人订阅项目时，他将订阅此项目的所有任务，并使用父子类型找到子类型

- `relation_field` -  [`Char`](https://www.odoo.com/documentation/10.0/reference/orm.html#odoo.fields.Char)

  例如，链接项目和任务子类型时，“关系”字段是任务的“项目id”字段

- `res_model` -  [`Char`](https://www.odoo.com/documentation/10.0/reference/orm.html#odoo.fields.Char)

  子类型应用于的模型；如果为False，则此子类型适用于所有模型

- `default` -  [`Boolean`](https://www.odoo.com/documentation/10.0/reference/orm.html#odoo.fields.Boolean)

  订阅时是否默认激活子类型

- `sequence` -  [`Integer`](https://www.odoo.com/documentation/10.0/reference/orm.html#odoo.fields.Integer)

  用于在通知自定义弹出窗口中排序子类型

- `hidden` -  [`Boolean`](https://www.odoo.com/documentation/10.0/reference/orm.html#odoo.fields.Boolean)

  子类型是否隐藏在通知自定义弹出窗口中

通过将子类型与字段跟踪接口，可以根据用户可能感兴趣的内容订阅不同类型的通知。为此，可以重写`_track_subtype（）`函数：

###### _track_subtype(*init_values*)

根据已更新的值给出由记录上的更改触发的子类型。

```
Parameters：init_values (dict) —— 记录的原始值；dict中仅存在修改的字段
Returns：子类型的完整外部id，如果未触发子类型，则为False
```

>例子 
>
>让我们在示例类中添加一个`state`字段，并在该字段更改值时触发具有特定子类型的通知。 
>
>首先，让我们定义我们的子类型：
>
>```xml
><record id="mt_state_change" model="mail.message.subtype">
>    <field name="name">Trip confirmed</field>
>    <field name="res_model">business.trip</field>
>    <field name="default" eval="True"/>
>    <field name="description">Business Trip confirmed!</field>
></record>
>```
>
>然后，我们需要重写`track_subtype（）`函数。此函数由跟踪系统调用，以根据当前应用的更改确定应使用的子类型。在我们的例子中，当`state`字段从draft更改为confirmed时，我们希望使用新的子类型：
>
>```python
>class BusinessTrip(models.Model):
>    _name = 'business.trip'
>    _inherit = ['mail.thread']
>    _description = 'Business Trip'
>
>    name = fields.Char(track_visibility='onchange')
>    partner_id = fields.Many2one('res.partner', 'Responsible',
>                                 track_visibility='onchange')
>    guest_ids = fields.Many2many('res.partner', 'Participants')
>    state = fields.Selection([('draft', 'New'), ('confirmed', 'Confirmed')],
>                             track_visibility='onchange')
>
>    def _track_subtype(self, init_values):
>        # init_values contains the modified fields' values before the changes
>        #
>        # the applied values can be accessed on the record as they are already
>        # in cache
>        self.ensure_one()
>        if 'state' in init_values and self.state == 'confirmed':
>            return 'my_module.mt_state_change'  # Full external id
>        return super(BusinessTrip, self)._track_subtype(init_values)
>```

#### 自定义通知 

向关注者发送通知时，在模板中添加按钮以允许直接从电子邮件执行快速操作非常有用。即使是一个简单的按钮直接链接到记录的表单视图也会很有用；但是，在大多数情况下，您不希望向门户用户显示这些按钮。 

通知系统允许通过以下方式自定义通知模板： 

- 显示 *访问按钮* ：这些按钮在通知电子邮件的顶部可见，允许收件人直接访问记录的表单视图
- 显示 *跟随按钮* ：这些按钮允许收件人直接从记录中快速订阅
- 显示 *取消关注按钮* ：这些按钮允许收件人直接快速取消订阅记录
- 显示 *自定义操作按钮* ：这些按钮是对特定路线的调用，允许您通过电子邮件直接执行一些有用的操作（例如，将潜在客户转换为商机、为费用经理验证费用表等）

这些按钮设置可以应用于不同的组，您可以通过覆盖函数`_notification_recipients`来定义这些组。

###### _notification_recipients(*message*, *groups*)

根据已更新的值给出由记录上的更改触发的子类型。

>Parameters：
>
>- **message** (`record`) -- 当前正在发送的`mail.message`记录
>
>- **groups** (`list(tuple)`) -- 表单的元组列表（组名称、组函数、组数据），其中：
>
>  - group_name
>
>    是仅用于重写和操作组的标识符。默认组为`user`（链接到员工用户的收件人）、`portal`（链接到门户用户的收件人）和`customer`（未链接到任何用户的收件人）。覆盖使用的一个示例是添加一个链接到资源组的组，如Hr专员，以设置特定的操作按钮。
>
>  - group_func
>
>    是以合作伙伴记录为参数的函数指针。此方法将应用于收件人，以了解他们是否属于给定的组。只保留第一个匹配组。评估顺序是列表顺序。
>
>  - group_data
>
>    是包含通知电子邮件参数的dict，具有以下可能的键-值：
>
>    - has_button_access
>
>      是否在电子邮件中显示Access<Document>。默认情况下，对于新组为True，对于门户/客户为False。
>
>    - button_access
>
>      使用此按钮编辑所选文本。使用按钮的url和标题记录
>
>    - has_button_follow
>
>      是否在电子邮件中显示跟踪（如果收件人当前未跟踪该线程）。默认情况下，对于新组为True，对于门户/客户为False。
>
>    - button_follow
>
>      使用按钮的URL和标题将URL复制到剪贴簿
>
>    - has_button_unfollow
>
>      是否在电子邮件中显示Unfollow（如果收件人当前正在跟踪线程）。默认情况下，对于新组为True，对于门户/客户为False。
>
>    - button_unfollow
>
>      使用此按钮编辑所选文本。使用按钮的url和标题记录
>
>    - actions
>
>      要在通知电子邮件中显示的操作按钮列表。每个动作都是一个dict，包含按钮的url和标题。
>
>Returns：子类型的完整外部id，如果未触发子类型，则为False。

可以通过调用`_notification_link_helper（）`函数自动生成操作列表中的URL：

###### _notification_link_helper(*self*, *link_type*, ***kwargs*)

在当前记录上为给定类型生成链接（如果设置了kwargs模型和res_id，则在特定记录上生成链接）。

>Parameters：
>
>**link_type** ([`str`](https://docs.python.org/2/library/functions.html#str)) -- 
>
>要生成的链接类型；可以是以下任一值：
>
>- `view`
>
>  链接到记录的窗体视图
>
>- `assign`
>
>  将记录的用户分配到记录的`user_id`字段（如果存在）
>
>- `follow`
>
>  不言自明
>
>- `unfollow`
>
>  不言自明
>
>- `workflow`
>
>  触发工作流信号；信号的名称应作为kwarg `signal`提供
>
>- `method`
>
>  调用记录上的方法；该方法的名称应作为kwarg `method`提供
>
>- `new`
>
>  打开新记录的空表单视图；您可以通过在kwarg `action_id`中提供其id（数据库id或完全解析的外部id）来指定特定操作
>
>Returns：为记录选择的类型的链接
>
>Return type：[str](https://docs.python.org/2/library/functions.html#str)

>例子
>
>让我们为商务旅行状态更改通知添加一个自定义按钮； 此按钮会将状态重置为草稿，并且仅对成员可见 （假想的）组旅行经理（ `business.group_trip_manager`) 
>
>```python
>class BusinessTrip(models.Model):
>    _name = 'business.trip'
>    _inherit = ['mail.thread', 'mail.alias.mixin']
>    _description = 'Business Trip'
>
>    # Pevious code goes here
>
>    def action_cancel(self):
>        self.write({'state': 'draft'})
>
>    def _notification_recipients(self, message, groups):
>        """ Handle Trip Manager recipients that can cancel the trip at the last
>        minute and kill all the fun. """
>        groups = super(BusinessTrip, self)._notification_recipients(message, groups)
>
>        self.ensure_one()
>        if self.state == 'confirmed':
>            app_action = self._notification_link_helper('method',
>                                method='action_cancel')
>            trip_actions = [{'url': app_action, 'title': _('Cancel')}]
>
>        new_group = (
>            'group_trip_manager',
>            lambda partner: bool(partner.user_ids) and
>            any(user.has_group('business.group_trip_manager')
>            for user in partner.user_ids),
>            {
>                'actions': trip_actions,
>            })
>
>        return [new_group] + groups
>```
>
>请注意，我可以在这个方法之外定义我的求值函数，并定义一个全局函数来代替lambda，但是为了在这些文档文件中更简短、更不冗长（有时会很无聊），我选择前者而不是后者。

#### 覆盖默认值 

有几种方法可以自定义`mail.thread`模型的行为，包括（但不限于）：

- `_mail_post_access` -  [`Model`](https://www.odoo.com/documentation/10.0/reference/orm.html#odoo.models.Model)属性 

  能够在模型上发布消息所需的访问权限；默认情况下，需要`write`访问权限，也可以设置为`read`

- 上下文键： 

  这些上下文键可用于在调用`create（）`或`write（）`期间（或任何其他可能有用的方法）控制`mail.thread`功能，如自动订阅或字段跟踪。

  - `mail_create_nosubscribe`: 在创建或消息发布时，不要将当前用户订阅到记录线程
  - `mail_create_nolog`: 在创建时，不要记录自动“<Document>created”消息
  - `mail_notrack`: 在创建和写入时，不要执行创建消息的值跟踪
  - `tracking_disable`:  在创建和写入时，不执行邮件线程功能（自动订阅、跟踪、发布等）
  - `mail_auto_delete`: 自动删除邮件通知；默认为True
  - `mail_notify_force_send`:如果要发送的电子邮件通知少于50封，请直接发送，而不是使用队列；默认为True
  - `mail_notify_user_signature`:在电子邮件通知中添加当前用户签名；默认为True



### 邮件别名 

别名是可配置的电子邮件地址，链接到特定记录（通常继承`mail.alias.mixin`模型），通过电子邮件联系时将创建新记录。它们是一种从外部访问系统的简单方法，允许用户或客户在数据库中快速创建记录，而无需直接连接到Odoo。

#### 别名与传入邮件网关 

有些人使用传入邮件网关也是出于同样的目的。您仍然需要一个正确配置的邮件网关来使用别名，但是一个catchall域就足够了，因为所有路由都将在Odoo内部完成。别名与邮件网关相比有几个优点：

- 更容易配置 
  - 单个传入网关可由多个别名使用；这避免了必须在您的域名上配置多封电子邮件（所有配置都在Odoo中完成）
  - 配置别名不需要系统访问权限
- 更连贯 
  - 可在相关记录上配置，而不是在设置子菜单中
- 更容易覆盖服务器端 
  - Mixin模型从一开始就可以扩展，允许您比使用邮件网关更容易地从传入的电子邮件中提取有用的数据。

#### 别名支持集成 

别名通常在父模型上配置，然后通过电子邮件联系时，父模型将创建特定记录。例如，Project使用别名创建任务或问题，销售团队使用别名生成潜在客户。

>注
>
>将由别名创建的模型`must`继承`mail_thread`模型。

通过继承`mail.Alias.mixin`添加别名支持；此mixin将为创建的父类的每个记录生成一个新的`mail.alias`记录（例如，在创建时初始化了其`mail.alias`记录的每个`project.project`记录）。

>注 
>
>别名也可以手动创建，并由简单的`Many2One`字段支持。本指南假设您希望与自动创建别名、记录特定默认值等进行更完整的集成

与`mail.thread`继承不同，`mail.alias.mixin `**requires** 一些特定的重写才能正常工作。这些覆盖将指定所创建别名的值，如它必须创建的记录类型，以及这些记录可能具有的某些默认值，具体取决于父对象：

###### get_alias_model_name(*vals*)

返回别名的模型名称。未回复现有记录的传入电子邮件将导致创建此别名模型的新记录。该值可能取决于`vals`，`vals`是创建此模型的记录时传递给`create`的值的dict。

>Parameters：**dict** (`vals`) -- 将保存别名的新创建记录的值
>
>Returns：模型名称
>
>Return type：[str](https://docs.python.org/2/library/functions.html#str)

###### `get_alias_values()`

返回值以创建别名，或在别名创建后写入别名。虽然不是完全强制性的，但通常需要通过在alias的alias_defaults字段中设置默认值字典来确保新创建的记录将链接到alias的父项（即在正确的项目中创建的任务）。

>Returns：将写入新别名的值的措辞
>
>Return type：[dict](https://docs.python.org/2/library/stdtypes.html#dict)

`get_alias_values（）`覆盖特别有趣，因为它允许您轻松修改别名的行为。在可以在别名上设置的字段中，以下字段特别重要：

- `alias_name` -  [`Char`](https://www.odoo.com/documentation/10.0/reference/orm.html#odoo.fields.Char)

  电子邮件别名的名称，如“jobs”，如果您想捕获的电子邮件<jobs@example.odoo.com>

- `alias_user_id` -  [`Many2one`](https://www.odoo.com/documentation/10.0/reference/orm.html#odoo.fields.Many2one) ( `res.users`) 

  在此别名上接收电子邮件时创建的记录的所有者；如果未设置此字段，系统将尝试根据发件人（发件人）地址查找正确的所有者，或者如果找不到该地址的系统用户，则将使用管理员帐户

- `alias_defaults` -  [`Text`](https://www.odoo.com/documentation/10.0/reference/orm.html#odoo.fields.Text)

  为该别名创建新记录时，将对其进行评估以提供默认值的Python字典

- `alias_force_thread_id` -  [`Integer`](https://www.odoo.com/documentation/10.0/reference/orm.html#odoo.fields.Integer)

  线程（记录）的可选ID，所有传入消息将附加到该线程（记录），即使它们没有回复该线程（记录）；如果设置，将完全禁用新记录的创建

- `alias_contact` -  [`Selection`](https://www.odoo.com/documentation/10.0/reference/orm.html#odoo.fields.Selection)

  使用mailgateway在文档上发布消息的策略 

  - 每个人：每个人都可以发帖 
  - 合作伙伴：仅经过身份验证的合作伙伴 
  - 追随者：仅相关文档的追随者或以下渠道的成员

请注意，别名使用`委托继承`，这意味着当别名存储在另一个表中时，您可以直接从父对象访问所有这些字段。这使您可以从记录的表单视图轻松配置别名。

>例子
>让我们在BusinessTrip类中添加别名，以便通过电子邮件动态创建费用。
>
>```python
>class BusinessTrip(models.Model):
>    _name = 'business.trip'
>    _inherit = ['mail.thread', 'mail.alias.mixin']
>    _description = 'Business Trip'
>
>    name = fields.Char(track_visibility='onchange')
>    partner_id = fields.Many2one('res.partner', 'Responsible',
>                                 track_visibility='onchange')
>    guest_ids = fields.Many2many('res.partner', 'Participants')
>    state = fields.Selection([('draft', 'New'), ('confirmed', 'Confirmed')],
>                             track_visibility='onchange')
>    expense_ids = fields.One2many('business.expense', 'trip_id', 'Expenses')
>    alias_id = fields.Many2one('mail.alias', string='Alias', ondelete="restrict",
>                               required=True)
>
>    def get_alias_model_name(self, vals):
>    """ Specify the model that will get created when the alias receives a message """
>        return 'business.expense'
>
>    def get_alias_values(self):
>    """ Specify some default values that will be set in the alias at its creation """
>        values = super(BusinessTrip, self).get_alias_values()
>        # alias_defaults holds a dictionnary that will be written
>        # to all records created by this alias
>        #
>        # in this case, we want all expense records sent to a trip alias
>        # to be linked to the corresponding business trip
>        values['alias_defaults'] = {'trip_id': self.id}
>        # we only want followers of the trip to be able to post expenses
>        # by default
>        values['alias_contact'] = 'followers'
>        return values
>
>class BusinessExpense(models.Model):
>    _name = 'business.expense'
>    _inherit = ['mail.thread']
>    _description = 'Business Expense'
>
>    name = fields.Char()
>    amount = fields.Float('Amount')
>    trip_id = fields.Many2one('business.trip', 'Business Trip')
>    partner_id = fields.Many2one('res.partner', 'Created by')
>```
>
>我们希望我们的别名能够从商务旅行的表单视图中轻松配置，因此，让我们在表单视图中添加以下内容：
>
>```xml
><page string="Emails">
>    <group name="group_alias">
>        <label for="alias_name" string="Email Alias"/>
>        <div name="alias_def">
>            <!-- display a link while in view mode and a configurable field
>            while in edit mode -->
>            <field name="alias_id" class="oe_read_only oe_inline"
>                    string="Email Alias" required="0"/>
>            <div class="oe_edit_only oe_inline" name="edit_alias"
>                 style="display: inline;" >
>                <field name="alias_name" class="oe_inline"/>
>                @
>                <field name="alias_domain" class="oe_inline" readonly="1"/>
>            </div>
>        </div>
>        <field name="alias_contact" class="oe_inline"
>                string="Accept Emails From"/>
>    </group>
></page>
>```
>
>现在，我们可以直接从表单视图更改别名地址，并更改可以向别名发送电子邮件的用户。 然后，我们可以覆盖费用模型上的`message_new（）`，以便在创建费用时从电子邮件中获取值：
>
>```python
>class BusinessExpense(models.Model):
>    # Previous code goes here
>    # ...
>
>    def message_new(self, msg, custom_values=None):
>        """ Override to set values according to the email.
>
>        In this simple example, we simply use the email title as the name
>        of the expense, try to find a partner with this email address and
>        do a regex match to find the amount of the expense."""
>        name = msg_dict.get('subject', 'New Expense')
>        # Match the last occurence of a float in the string
>        # Example: '50.3 bar 34.5' becomes '34.5'. This is potentially the price
>        # to encode on the expense. If not, take 1.0 instead
>        amount_pattern = '(\d+(\.\d*)?|\.\d+)'
>        expense_price = re.findall(amount_pattern, name)
>        price = expense_price and float(expense_price[-1][0]) or 1.0
>        # find the partner by looking for it's email
>        partner = self.env['res.partner'].search([('email', 'ilike', email_address)],
>                                                 limit=1)
>        defaults = {
>            'name': name,
>            'amount': price,
>            'partner_id': partner.id
>        }
>        defaults.update(custom_values or {})
>        res = super(BusinessExpense, self).message_new(msg, custom_values=defaults)
>        return res
>```

## 网站特色 

### 访客追踪 

可以使用`utm.mixin`类通过指向指定资源的链接中的参数来跟踪在线营销/传播活动。mixin向模型中添加3个字段：

- `campaign_id`: `utm.campaign`对象的`Many2one`字段（例如圣诞特价、秋季系列等）
- `source_id`: `utm.campaign`对象的`Many2one`字段（例如搜索引擎、邮件列表等） 
- `medium_id`:  `utm.campaign`对象的`Many2one`字段（例如蜗牛邮件、电子邮件、社交网络更新等） 

这些模型只有一个字段`name`（即它们只是用来区分活动，但没有任何特定行为）。

一旦客户使用url中设置的这些参数访问您的网站（例如 http://www.odoo.com/?campaign_id=mixin_talk&source_id=www.odoo.com&medium_id=website)，在访问者网站中为这些参数设置了三个cookie。一旦从网站创建了一个继承utm.mixin的对象（即lead表单、job  application等），utm.mixin代码就会生效，并从cookie中获取值，以在新记录中设置它们。完成此操作后，在定义报告和视图（分组依据等）时，您可以将活动/源/媒体字段用作任何其他字段。 要扩展此行为，只需向简单模型添加一个关系字段（该模型应支持快速创建（即使用单个名称值调用create（）），并扩展函数tracking_fields（）：

要扩展此行为，只需向简单模型添加一个关系字段（该模型应支持快速创建（即使用单个`name`值调用`create（）`），并扩展函数`tracking_fields（）`：

```python
class UtmMyTrack(models.Model):
    _name = 'my_module.my_track'
    _description = 'My Tracking Object'

    name = fields.Char(string='Name', required=True)


class MyModel(models.Models):
    _name = 'my_module.my_model'
    _inherit = ['utm.mixin']
    _description = 'My Tracked Object'

    my_field = fields.Many2one('my_module.my_track', 'My Field')

    @api.model
    def tracking_fields(self):
        result = super(MyModel, self).tracking_fields()
        result.append([
        # ("URL_PARAMETER", "FIELD_NAME_MIXIN", "NAME_IN_COOKIES")
            ('my_field', 'my_field', 'odoo_utm_my_field')
        ])
        return result
```

这将告诉系统创建一个名为*odoo_utm_my_field* 的cookie，其值位于url参数==my_field==中；

通过从网站表单调用创建此模型的新记录后，==utm.mixin==的==create()==方法的通用重写将从cookie中获取此字段的默认值（如果尚不存在，则将动态创建==my_module.my_track==）。

您可以在以下模型中找到具体的集成示例：

- `crm.lead`在 CRM ( *crm* ) 应用程序中 
- `hr.applicant`在Recruitment Process  ( *hr_recruitment* ) 应用程序中 
- `helpdesk.ticket`在Helpdesk （ **helpdesk* * - 仅限 Odoo Enterprise）应用程序中 



### 网站可见性 

您可以很容易地在任何记录上添加网站可见性切换。虽然这个mixin很容易手动实现，但它是继`mail.thread`继承之后最常用的；证明它有用的证据。此mixin的典型用例是任何具有前端页面的对象；通过控制页面的可见性，您可以在编辑页面时花点时间，并且只在您满意的情况下发布页面。

要包含Functionality，您只需继承`website.published.mixin`：

```python
class BlogPost(models.Model):
    _name = "blog.post"
    _description = "Blog Post"
    _inherit = ['website.published.mixin']
```

这个 mixin 在你的模型上添加了 2 个字段： 

- `website_published`:  [`Boolean`](https://www.odoo.com/documentation/10.0/reference/orm.html#odoo.fields.Boolean)字段，表示发布状态
- `website_url`:  [`Char`](https://www.odoo.com/documentation/10.0/reference/orm.html#odoo.fields.Char)字段，表示访问对象的URL

请注意，最后一个字段是计算字段，必须为您的类实现：

```python
def _compute_website_url(self):
    for blog_post in self:
        blog_post.website_url = "/blog/%s" % (log_post.blog_id)
```

一旦机制到位，您只需调整前端和后端视图，使其可访问。在后端，在按钮框中添加按钮通常是一种方法：

```xml
<button class="oe_stat_button" name="website_publish_button"
    type="object" icon="fa-globe">
    <field name="website_published" widget="website_button"/>
</button>
```

在前端，需要进行一些安全检查，以避免向网站访问者显示“编辑”按钮：

```html
<div id="website_published_button" class="pull-right"
     groups="base.group_website_publisher"> <!-- or any other meaningful group -->
    <t t-call="website.publish_management">
      <t t-set="object" t-value="blog_post"/>
      <t t-set="publish_edit" t-value="True"/>
      <t t-set="action" t-value="'blog.blog_post_action'"/>
    </t>
</div>
```

请注意，必须将对象作为变量`object`传递给模板；在本例中，`blog.post`记录作为`blog_post`变量传递给`qweb`渲染引擎，需要将其指定给发布管理模板。`publish_edit`变量允许前端按钮链接到后端（允许您轻松地从前端切换到后端，反之亦然）；如果已设置，则必须在`action`变量中指定要在后端调用的操作的完整外部id（请注意，模型必须存在表单视图）。

` action  website_publish_button`在mixin中定义，并根据您的对象调整其行为：如果类具有有效的`website_url`计算函数，则用户单击按钮时会重定向到前端；然后，用户可以直接从前端发布页面。这确保了不会发生意外的在线发布。如果没有计算函数，只会触发布尔值 `website_published`。

### 网站元数据 

这个简单的mixin让您可以轻松地在前端页面中插入元数据。

```python
class BlogPost(models.Model):
    _name = "blog.post"
    _description = "Blog Post"
    _inherit = ['website.seo.metadata', 'website.published.mixin']
```

这个 mixin 在你的模型上添加了 3 个字段： 

- `website_meta_title`:  [`Char`](https://www.odoo.com/documentation/10.0/reference/orm.html#odoo.fields.Char)属性允许您为页面设置附加标题
- `website_meta_description`:  [`Char`](https://www.odoo.com/documentation/10.0/reference/orm.html#odoo.fields.Char)属性包含页面的简短描述（有时用于搜索引擎结果）
- `website_meta_keywords`:  [`Char`](https://www.odoo.com/documentation/10.0/reference/orm.html#odoo.fields.Char)属性包含一些关键字，以帮助您的页面被搜索引擎更精确地分类；“升级”工具将帮助您轻松选择词汇相关的关键字

这些字段可以在前端使用编辑器工具栏中的“升级”工具进行编辑。设置这些字段可以帮助搜索引擎更好地索引页面。请注意，搜索引擎的结果并不仅仅基于这些元数据；最好的SEO实践仍然应该是获得可靠来源的参考。



## 其他 

### 客户评分 

mixin允许发送电子邮件询问客户评级，在看板流程中自动转换，并汇总评级统计数据。

#### 为您的模型添加评分 

要添加评级支持，只需继承 `rating.mixin`模型： 

```python
class MyModel(models.Models):
    _name = 'my_module.my_model'
    _inherit = ['rating.mixin', 'mail.thread']

    user_id = fields.Many2one('res.users', 'Responsible')
    partner_id = fields.Many2one('res.partner', 'Customer')
```

mixin 的行为适应您的模型： 

- 这 `rating.rating`记录将链接到 `partner_id`你的领域 模型（如果该字段存在）。 
  - 如果使用的字段不是`partner_id`，则可以使用函数`rating_get_partner_id(）`覆盖此行为
- 这`rating.rating`记录将链接到您模型的`user_id`字段的合作伙伴（如果该字段存在）（即被评级的合作伙伴）
  - 如果使用`user_id`以外的其他字段，则可以使用函数`rating_get_rated_partner_id（）`覆盖此行为（请注意，函数必须返回`res.partner`，对于`user_id`，系统自动获取用户的合作伙伴）
- 聊天记录将显示评级事件（如果您的模型继承自 `mail.thread`) 

#### 通过电子邮件发送评级请求 

如果您希望发送电子邮件请求评级，只需生成一封包含评级对象链接的电子邮件即可。非常基本的电子邮件模板可能如下所示：

```xml
<record id="rating_my_model_email_template" model="mail.template">
            <field name="name">My Model: Rating Request</field>
            <field name="email_from">${object.rating_get_rated_partner_id().email or '' | safe}</field>
            <field name="subject">Service Rating Request</field>
            <field name="model_id" ref="my_module.model_my_model"/>
            <field name="partner_to" >${object.rating_get_partner_id().id}</field>
            <field name="auto_delete" eval="True"/>
            <field name="body_html"><![CDATA[
% set access_token = object.rating_get_access_token()
<p>Hi,</p>
<p>How satsified are you?</p>
<ul>
    <li><a href="/rating/${access_token}/10">Satisfied</a></li>
    <li><a href="/rating/${access_token}/5">Not satisfied</a></li>
    <li><a href="/rating/${access_token}/1">Very unsatisfied</a></li>
</ul>
]]></field>
</record>
```

然后，您的客户将收到一封电子邮件，其中包含一个简单网页的链接，允许他们提供与您的用户交互的反馈（包括一条免费文本反馈消息）。

然后，通过定义评级操作，您可以非常轻松地将评级与表单视图集成：

```xml
<record id="rating_rating_action_my_model" model="ir.actions.act_window">
    <field name="name">Customer Ratings</field>
    <field name="res_model">rating.rating</field>
    <field name="view_mode">kanban,pivot,graph</field>
    <field name="domain">[('res_model', '=', 'my_module.my_model'), ('res_id', '=', active_id), ('consumed', '=', True)]</field>
</record>

<record id="my_module_my_model_view_form_inherit_rating" model="ir.ui.view">
    <field name="name">my_module.my_model.view.form.inherit.rating</field>
    <field name="model">my_module.my_model</field>
    <field name="inherit_id" ref="my_module.my_model_view_form"/>
    <field name="arch" type="xml">
        <xpath expr="//div[@name='button_box']" position="inside">
            <button name="%(rating_rating_action_my_model)d" type="action"
                    class="oe_stat_button" icon="fa-smile-o">
                <field name="rating_count" string="Rating" widget="statinfo"/>
            </button>
        </xpath>
    </field>
</record>
```

请注意，评级有默认视图（看板、枢轴、图表），允许 您可以快速鸟瞰您的客户评分。 

您可以在以下模型中找到具体的集成示例： 

- `project.task`在项目 ( *rating_project* ) 应用程序中 
- `helpdesk.ticket`在helpdesk（*helpdesk* - 仅限 Odoo Enterprise）应用程序中 

