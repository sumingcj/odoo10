# Odoo10开发教程一（构建模块）

> 警告
>  本教程需要已经安装odoo


#### 启动/停止Odoo服务器

Odoo采用C/S架构，客户端通过Web浏览器访问服务端，遵从RPC协议。业务逻辑和扩展通常在服务端执行，而只有添加客户端支持的新特征才会在客户端添加代码（例如，交互过程中新数据的映射表示）。启动服务器，只需要在shell中调用命令odoo-bin，或者完整的路径名调用：

```
odoo-bin
```

通过`Ctrl-c`或杀死相应的系统进程来停止Odoo服务。

#### 构建一个Odoo模块

服务端扩展和客户端扩展都被封装为模块，这些模块可选择性的被安装，安装完成后通过数据库来加载。模块即可以是全新的业务逻辑，也可以是更改和扩展已有的业务逻辑。比如创建一个中国会计模块，将中国的会计准则添加到Odoo的通用会计中，也可以创建一个全新的实时可视化管理车队的模块。Odoo中的所有功能都是包含在模块中。

##### 模块的组成

Odoo模块包含多个部分：
**业务对象**
 　Python类，这些类会被Odoo框架自动持久化，持久化的方式决定于类的定义。
**数据文件**
 　包括视图、菜单、动作、工作流、权限、演示数据等，以XML或CSV文件定义。
**Web控制器**
 　处理Web浏览器的请求
**静态页面数据**
 　网站或界面使用的图片、CSS或JavaScript文件

##### 模块结构

每个模块都是模块目录中的一个子目录。可以通过`--addons-path`选项指定模块目录的路径。

> 提示
>  大多数命令行选项可以通过配置文件进行设置

Odoo模块由清单文件进行声明。查看清单文件文档了解详细信息。模块是一个包含`__init__.py`文件的的Python包，`__init__.py`文件包含了模块需要的导入的各Python文件。
 例如，如果模块中包含`mymodule.py`文件，`__init__.py`应该这样写：

```
from . import mymodule
```

Odoo提供了脚手架机制来快速创建新模块，`odoo-bin`子命令`scaffold`用来创建一个空模块

```js
$ odoo-bin scaffold <模块名> <模块放置路径>
```

该命令为模块创建一个子目录，并自动为模块创建一些标准文件。这些文件大多只包含被注释的代码和XML元素。后面将解释这些文件的含义。

> **练习创建模块**
>  使用上面的命令行创建一个空模块*Open Academy*，并将其安装在Odoo中。
>
> 1. 调用命令`odoo-bin scaffold openacademy addons`
> 2. 修改模块中的相关文件
> 3. 不要修改其它文件

```
openacademy/__manifest__.py
# -*- coding: utf-8 -*-
{
    'name': "Open Academy",

    'summary': """Manage trainings""",
    
    'description': """
        Open Academy module for managing trainings:
            - training courses
            - training sessions
            - attendees registration
    """,
    
    'author': "My Company",
    'website': "http://www.yourcompany.com",
    
    # Categories can be used to filter modules in modules listing
    # Check https://github.com/odoo/odoo/blob/master/odoo/addons/base/module/module_data.xml
    # for the full list
    'category': 'Test',
    'version': '0.1',
    
    # any module necessary for this one to work correctly
    'depends': ['base'],
    
    # always loaded
    'data': [
        # 'security/ir.model.access.csv',
        'templates.xml',
    ],
    # only loaded in demonstration mode
    'demo': [
        'demo.xml',
    ],
}
openacademy/__init__.py
# -*- coding: utf-8 -*-
from . import controllers
from . import models
openacademy/controllers.py
# -*- coding: utf-8 -*-
from odoo import http

# class Openacademy(http.Controller):
#     @http.route('/openacademy/openacademy/', auth='public')
#     def index(self, **kw):
#         return "Hello, world"

#     @http.route('/openacademy/openacademy/objects/', auth='public')
#     def list(self, **kw):
#         return http.request.render('openacademy.listing', {
#             'root': '/openacademy/openacademy',
#             'objects': http.request.env['openacademy.openacademy'].search([]),
#         })

#     @http.route('/openacademy/openacademy/objects/<model("openacademy.openacademy"):obj>/', auth='public')
#     def object(self, obj, **kw):
#         return http.request.render('openacademy.object', {
#             'object': obj
#         })
openacademy/demo.xml
<odoo>
    <data>
        <!--  -->
        <!--   <record id="object0" model="openacademy.openacademy"> -->
        <!--     <field name="name">Object 0</field> -->
        <!--   </record> -->
        <!--  -->
        <!--   <record id="object1" model="openacademy.openacademy"> -->
        <!--     <field name="name">Object 1</field> -->
        <!--   </record> -->
        <!--  -->
        <!--   <record id="object2" model="openacademy.openacademy"> -->
        <!--     <field name="name">Object 2</field> -->
        <!--   </record> -->
        <!--  -->
        <!--   <record id="object3" model="openacademy.openacademy"> -->
        <!--     <field name="name">Object 3</field> -->
        <!--   </record> -->
        <!--  -->
        <!--   <record id="object4" model="openacademy.openacademy"> -->
        <!--     <field name="name">Object 4</field> -->
        <!--   </record> -->
        <!--  -->
    </data>
</odoo>
openacademy/models.py
# -*- coding: utf-8 -*-

from odoo import models, fields, api

# class openacademy(models.Model):
#     _name = 'openacademy.openacademy'

#     name = fields.Char()
openacademy/security/ir.model.access.csv
id,name,model_id/id,group_id/id,perm_read,perm_write,perm_create,perm_unlink
access_openacademy_openacademy,openacademy.openacademy,model_openacademy_openacademy,,1,0,0,0
openacademy/templates.xml
<odoo>
    <data>
        <!-- <template id="listing"> -->
        <!--   <ul> -->
        <!--     <li t-foreach="objects" t-as="object"> -->
        <!--       <a t-attf-href="{{ root }}/objects/{{ object.id }}"> -->
        <!--         <t t-esc="object.display_name"/> -->
        <!--       </a> -->
        <!--     </li> -->
        <!--   </ul> -->
        <!-- </template> -->
        <!-- <template id="object"> -->
        <!--   <h1><t t-esc="object.display_name"/></h1> -->
        <!--   <dl> -->
        <!--     <t t-foreach="object._fields" t-as="field"> -->
        <!--       <dt><t t-esc="field"/></dt> -->
        <!--       <dd><t t-esc="object[field]"/></dd> -->
        <!--     </t> -->
        <!--   </dl> -->
        <!-- </template> -->
    </data>
</odoo>
```

#### 对象关系映射

Odoo的关键组件是ORM层。这个层避免了大量手写SQL，并提供扩展性和安全性。业务对象被声明为`Model`类的扩展类，并自动将它们集成到持久层中。可以通过定义时设置属性来配置模型。最重要的属性是`_name`,必填属性，它定义了模块在Odoo系统中的名称。一个最简单的模型定义：

```
from odoo import models
class MinimalModel(models.Model):
    _name = 'test.model'
```

#### 模型字段

字段定义了模型中需要存储的内容和存储的位置。字段通过类的属性来定义：

```
from odoo import models, fields

class LessMinimalModel(models.Model):
    _name = 'test.model2'

    name = fields.Char()
```

**通用属性**
 和模型一样，字段也可以配置，字段通过属性参数的方式来配置：

```
name = field.Char(required=True)
```

一些属性可以用于所有字段，以下是最常见的属性：
`string(unicode,default: field's name)`
 　用户界面中的字段标签（用户可见）
`required（bool,default:False）`
 　如果为True,这个字段不能为空，它必须有一个默认值或者在创建记录时总是给定一个值。
`help (unicode, default: '')`
 　长格式，在用户界面上显示的提示。
`index (bool, default: False)`
 　请求Odoo在列上创建数据库索引。

**简单字段**
 有两大类字段：简单字段和关联字段，简单字段的值是存储在模型表中的原子值，而关联字段是指向模型（相同模型或不同模型）的记录行。
 简单字段的例子如：Boolean、Date、Char
 关联字段的例子如：Many2One、One2Many、Many2Many

**保留字段**
 Odoo在所有模型中都创建几个固定字段，这些字段由系统管理，用户程序不应写入。但是可以在用户程序中读取：
`id（Id）`
 　模型中记录的唯一标识符
`create_date(Datetime)`
 　记录的创建日期
`create_uid(Many2one)`
 　创建记录的用户
`write_date(Datetime)`
 　记录的最后修改时间
`write_uid(Many2one)`
 　上次修改记录的用户

**特殊字段**
 默认情况下，Odoo的name在所有模型上还需要一个字段，用于显示和搜索。用于这些目的的字段可以通过设置字段`_rec_name`来实现。

> 练习定义模型，在openacademy模块中定义新的数据模型*课程*,每门课程包含两个字段，标题和描述，其中标题是必填字段。编辑文件`openacademy/models/models.py`以包含`Course`类。

```
openacademy/models.py
from odoo import models, fields, api

class Course(models.Model):
    _name = 'openacademy.course'

    name = fields.Char(string="Title", required=True)
    description = fields.Text()
```

#### 数据文件

Odoo是一个高度数据驱动的系统，虽然行为是通过Python代码制定的，但一些模块的值是在加载时通过数据文件设置的。

> 提示：
>  一些模块的作用仅仅是为了将数据添加到Odoo中

模块数据通过带有`<record>`元素的XML数据文件来声明。每个`<record>`元素创建或更新数据库中的一个记录行。

```
<odoo>
    <data>
        <record model="{model name}" id="{record identifier}">
            <field name="{a field name}">{a value}</field>
        </record>
    </data>
</odoo>
```

- `model`是在记录行中记录的Odoo模型名称

- `id`是外部标识符，它允许引用记录（而不必知道其在数据库中的标识符）

- `<field>`，每个`<field>`对应数据行中的一个字段，name属性是字段名（例如*description*）,而`body`就是字段的值。
   数据文件通过在manifest文件声明而被载入，他们既可以在`data`列表声明（始终载入）也可以在`demo`列表声明（仅在演示模式下载入）

  > 练习定义演示数据，添加演示数据以填充*Course*模型的数据，编辑文件`openacademy/demo/demo.xml`来添加演示数据

```
openacademy/demo/demo.xml
<odoo>
    <data>
        <record model="openacademy.course" id="course0">
            <field name="name">Course 0</field>
            <field name="description">Course 0's description

Can have multiple lines
            </field>
        </record>
        <record model="openacademy.course" id="course1">
            <field name="name">Course 1</field>
            <!-- no description for this one -->
        </record>
        <record model="openacademy.course" id="course2">
            <field name="name">Course 2</field>
            <field name="description">Course 2's description</field>
        </record>
    </data>
</odoo>
```

#### 操作和菜单

操作和菜单是数据库中的常规数据，通常以数据文件声明。操作可以通过三种方式触发：
 1.点击菜单项（链接到特定操作）
 2.点击视图中的按钮（如果这些按钮已经被连接到操作）
 3.作为对象的上下文操作
 因为菜单的声明相对复杂，所以有个一个`<menuitem>`的快捷方式来声明`ir.ui.menu`菜单对象，并将其更容易的连接到对应的操作。

```
<record model="ir.actions.act_window" id="action_list_ideas">
    <field name="name">Ideas</field>
    <field name="res_model">idea.idea</field>
    <field name="view_mode">tree,form</field>
</record>
<menuitem id="menu_ideas" parent="menu_root" name="Ideas" sequence="10"
          action="action_list_ideas"/>
```

> 危险
>  操作必须在XML文件中对应的菜单之前声明.数据文件是按顺序执行的，操作的`id`必须在对应的菜单建立之前就存在于数据库中。
>
> 练习定义新菜单项，在开放学院菜单项下定义新菜单项来访问课程。用户应该能够：
>
> - 显示所有课程的列表
> - 建立或编辑课程
>    1.建立`openacademy/views/openacademy.xml`以创建操作和能够触发操作的菜单项。
>    2.添加这个文件到`openacademy/__manifest__.py`下的`data`列表。

```
openacademy/__manifest__.py
 'data': [
        # 'security/ir.model.access.csv',
        'templates.xml',
        'views/openacademy.xml',
    ],
    # only loaded in demonstration mode
    'demo': [
openacademy/views/openacademy.xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <data>
        <!-- window action -->
        <!--
            The following tag is an action definition for a "window action",
            that is an action opening a view or a set of views
        -->
        <record model="ir.actions.act_window" id="course_list_action">
            <field name="name">Courses</field>
            <field name="res_model">openacademy.course</field>
            <field name="view_type">form</field>
            <field name="view_mode">tree,form</field>
            <field name="help" type="html">
                <p class="oe_view_nocontent_create">Create the first course
                </p>
            </field>
        </record>

        <!-- top level menu: no parent -->
        <menuitem id="main_openacademy_menu" name="Open Academy"/>
        <!-- A first level in the left side menu is needed
             before using action= attribute -->
        <menuitem id="openacademy_menu" name="Open Academy"
                  parent="main_openacademy_menu"/>
        <!-- the following menuitem should appear *after*
             its parent openacademy_menu and *after* its
             action course_list_action -->
        <menuitem id="courses_menu" name="Courses" parent="openacademy_menu"
                  action="course_list_action"/>
        <!-- Full id location:
             action="openacademy.course_list_action"
             It is not required when it is the same module -->
    </data>
</odoo>
```

# Odoo10开发教程二（基本视图）

### 基本视图


视图定义了模型数据的呈现方式。不同的视图类型决定了数据的可视化方式（记录行列表、图形化聚合）。视图可以通过类型（比如*partners*列表）或id被请求。对于一般请求，将被对应类型的最低优先级视图响应（每个类型的最低优先级视图是该类型的默认视图）。视图继承允许更改在其他地方声明的视图（添加或删除内容）。

#### 通用视图声明

视图通过一个`ir.ui.view`的模型记录来声明。视图类型由`arch`字段的根元素隐含定义：

```
<record model="ir.ui.view" id="view_id">
    <field name="name">view.name</field>
    <field name="model">object_name</field>
    <field name="priority" eval="16"/>
    <field name="arch" type="xml">
        <!-- view content: <form>, <tree>, <graph>, ... -->
    </field>
</record>
```

> 危险
>  因为视图的内容是XML，所以`arch`字段必须被声明为`type="xml"`以正确解析。

#### 树视图

树视图也被称为列表视图，以表格形式显示记录。根元素是`<tree>`.最简单的树视图是在表格中列出所有字段（每列对应一个字段）:

```
<tree string="Idea list">
    <field name="name"/>
    <field name="inventor_id"/>
</tree>
```

#### 表单视图

表单视图通常用来建立和编辑单条记录。根元素是`<form>`,由结构元素(groups,notebooks)和交互元素(button,fields)组成。

```
<form string="Idea form">
    <group colspan="4">
        <group colspan="2" col="2">
            <separator string="General stuff" colspan="2"/>
            <field name="name"/>
            <field name="inventor_id"/>
        </group>

        <group colspan="2" col="2">
            <separator string="Dates" colspan="2"/>
            <field name="active"/>
            <field name="invent_date" readonly="1"/>
        </group>
    
        <notebook colspan="4">
            <page string="Description">
                <field name="description" nolabel="1"/>
            </page>
        </notebook>
    
        <field name="state"/>
    </group>
</form>
```

> 练习使用XML定制窗体视图
>  建立课程对象的表单视图，显示课程的名称和描述字段。

```
openacademy/views/openacademy.xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <data>
        <record model="ir.ui.view" id="course_form_view">
            <field name="name">course.form</field>
            <field name="model">openacademy.course</field>
            <field name="arch" type="xml">
                <form string="Course Form">
                    <sheet>
                        <group>
                            <field name="name"/>
                            <field name="description"/>
                        </group>
                    </sheet>
                </form>
            </field>
        </record>

        <!-- window action -->
        <!--
            The following tag is an action definition for a "window action",
```

> 练习notebook结构元素
>  在课程的表单视图中，将描述字段放在一个选项卡中，然后再添加选项卡放置其它字段。修改后的课程表单视图如下：

```
openacademy/views/openacademy.xml
                    <sheet>
                        <group>
                            <field name="name"/>
                        </group>
                        <notebook>
                            <page string="Description">
                                <field name="description"/>
                            </page>
                            <page string="About">
                                This is an example of notebooks
                            </page>
                        </notebook>
                    </sheet>
                </form>
            </field>
```

表单视图也可以使用纯HTML来进行更灵活的布局

```
<form string="Idea Form">
    <header>
        <button string="Confirm" type="object" name="action_confirm"
                states="draft" class="oe_highlight" />
        <button string="Mark as done" type="object" name="action_done"
                states="confirmed" class="oe_highlight"/>
        <button string="Reset to draft" type="object" name="action_draft"
                states="confirmed,done" />
        <field name="state" widget="statusbar"/>
    </header>
    <sheet>
        <div class="oe_title">
            <label for="name" class="oe_edit_only" string="Idea Name" />
            <h1><field name="name" /></h1>
        </div>
        <separator string="General" colspan="2" />
        <group colspan="2" col="2">
            <field name="description" placeholder="Idea description..." />
        </group>
    </sheet>
</form>
```

#### 搜索视图

搜索视图可对列表视图（或者其它聚合视图）中的字段进行搜索。搜索视图的根元素是`<search>`，内容包含所有可以搜索的字段。

```
<search>
    <field name="name"/>
    <field name="inventor_id"/>
</search>
```

如果在模型中没有定义搜索视图，Odoo会生成一个只包含`name`字段的搜索视图。

> 练习搜索课程，通过标题和描述来搜索课程。

```
openacademy/views/openacademy.xml
</field>
        </record>

        <record model="ir.ui.view" id="course_search_view">
            <field name="name">course.search</field>
            <field name="model">openacademy.course</field>
            <field name="arch" type="xml">
                <search>
                    <field name="name"/>
                    <field name="description"/>
                </search>
            </field>
        </record>
    
        <!-- window action -->
        <!--
            The following tag is an action definition for a "window action",
```



# Odoo10开发教程三（模型关联）

### 模型关联

一个模型中的记录可能关联到另一个模型中的记录。例如，销售订单记录会关联到一个包含客户数据的客户记录中；同时销售订单记录也会关联到销售订单明细记录。

> 练习建立一个授课模型
>  在开放学院模块中，我们考虑一个授课模型：一个授课是在给定的时间中对给定的受众教授指定的课程。为授课建立模型，授课包括名称、开始时间、持续时间和席位数。添加操作和菜单项来显示新的模型。
>
> - 在`openacademy/models/models.py`文件中创建*Session*类
> - 在`openacademy/view/openacademy.xml`文件中添加访问授课对象的菜单和操作

```
openacademy/models.py
    name = fields.Char(string="Title", required=True)
    description = fields.Text()

class Session(models.Model):
    _name = 'openacademy.session'

    name = fields.Char(required=True)
    start_date = fields.Date()
    duration = fields.Float(digits=(6, 2), help="Duration in days")
    seats = fields.Integer(string="Number of seats")
openacademy/views/openacademy.xml
        <!-- Full id location:
             action="openacademy.course_list_action"
             It is not required when it is the same module -->

        <!-- session form view -->
        <record model="ir.ui.view" id="session_form_view">
            <field name="name">session.form</field>
            <field name="model">openacademy.session</field>
            <field name="arch" type="xml">
                <form string="Session Form">
                    <sheet>
                        <group>
                            <field name="name"/>
                            <field name="start_date"/>
                            <field name="duration"/>
                            <field name="seats"/>
                        </group>
                    </sheet>
                </form>
            </field>
        </record>
    
        <record model="ir.actions.act_window" id="session_list_action">
            <field name="name">Sessions</field>
            <field name="res_model">openacademy.session</field>
            <field name="view_type">form</field>
            <field name="view_mode">tree,form</field>
        </record>
    
        <menuitem id="session_menu" name="Sessions"
                  parent="openacademy_menu"
                  action="session_list_action"/>
    </data>
</odoo>
```

> 注意
> `digits=(6,2)`指定浮点数的精度：6是数字的总和，而2是小数位长度，这同时表明整数位的最大长度是4

#### 关联字段

关联字段链接同一模型（不同层次结构）或者不同模型之间的记录。关联字段有三种类型：
`Many2one(other_model, ondelete='set null')`
 一个链接到其它对象的简单示例是这样的：

```
print foo.other_id.name
```

`One2many(other_model, related_field)`
 这是一个虚拟的关联，是`Many2one`的逆，`One2many`作为记录的容器，访问它将返回一个记录集（也可能是一个空记录集）：

```
for other in foo.other_ids:
    print other.name
```

> 警告
>  因为`One2many`是一个虚拟关联，所以必须有一个`Many2one`字段存在于`other_model`,其名称也必须是`related_field`

`Many2many(other_model)`
 双向多对多关联，一方的任一记录可以与另一方的任意数量记录关联。作为记录的容器，访问它也可能导致返回空记录集

```
for other in foo.other_ids:
    print other.name
```

> 练习*Many2one*关联
>  编辑*Course*和*Session*模型以反映他们与其它模型的关联：
>
> - 课程有一个负责的用户；该字段的值是内置模型`res.users`的记录
> - 一个授课有一个教师；该字段的值是内置模型`res.partner`的记录
> - 授课与课程相关；该字段的值是`openacademy.course`模型的记录，并且是必填项
> - 在模型中添加`Many2one`关联，并在视图显示

```
openacademy/models.py
    name = fields.Char(string="Title", required=True)
    description = fields.Text()

    responsible_id = fields.Many2one('res.users',
        ondelete='set null', string="Responsible", index=True)

class Session(models.Model):
    _name = 'openacademy.session'
    start_date = fields.Date()
    duration = fields.Float(digits=(6, 2), help="Duration in days")
    seats = fields.Integer(string="Number of seats")

    instructor_id = fields.Many2one('res.partner', string="Instructor")
    course_id = fields.Many2one('openacademy.course',
        ondelete='cascade', string="Course", required=True)
openacademy/views/openacademy.xml
                    <sheet>
                        <group>
                            <field name="name"/>
                            <field name="responsible_id"/>
                        </group>
                        <notebook>
                            <page string="Description">
            </field>
        </record>

        <!-- override the automatically generated list view for courses -->
        <record model="ir.ui.view" id="course_tree_view">
            <field name="name">course.tree</field>
            <field name="model">openacademy.course</field>
            <field name="arch" type="xml">
                <tree string="Course Tree">
                    <field name="name"/>
                    <field name="responsible_id"/>
                </tree>
            </field>
        </record>
    
        <!-- window action -->
        <!--
            The following tag is an action definition for a "window action",
                <form string="Session Form">
                    <sheet>
                        <group>
                            <group string="General">
                                <field name="course_id"/>
                                <field name="name"/>
                                <field name="instructor_id"/>
                            </group>
                            <group string="Schedule">
                                <field name="start_date"/>
                                <field name="duration"/>
                                <field name="seats"/>
                            </group>
                        </group>
                    </sheet>
                </form>
            </field>
        </record>
    
        <!-- session tree/list view -->
        <record model="ir.ui.view" id="session_tree_view">
            <field name="name">session.tree</field>
            <field name="model">openacademy.session</field>
            <field name="arch" type="xml">
                <tree string="Session Tree">
                    <field name="name"/>
                    <field name="course_id"/>
                </tree>
            </field>
        </record>
    
        <record model="ir.actions.act_window" id="session_list_action">
            <field name="name">Sessions</field>
            <field name="res_model">openacademy.session</field>
```

> 练习逆关联*One2many*
>  使用逆关联字段*one2many*，编辑模型以反映课程和授课之间的关系。
>
> - 编辑`Course`类，并且加入字段到它的表单视图

```
openacademy/models.py
    responsible_id = fields.Many2one('res.users',
        ondelete='set null', string="Responsible", index=True)
    session_ids = fields.One2many(
        'openacademy.session', 'course_id', string="Sessions")

class Session(models.Model):
openacademy/views/openacademy.xml
                            <page string="Description">
                                <field name="description"/>
                            </page>
                            <page string="Sessions">
                                <field name="session_ids">
                                    <tree string="Registered sessions">
                                        <field name="name"/>
                                        <field name="instructor_id"/>
                                    </tree>
                                </field>
                            </page>
                        </notebook>
                    </sheet>
```

> 练习多对多关联*many2many*
>  在授课模型中添加关联字段*many2many*,将每次授课和参与的听众做关联，听众来自于内置模型`res.partner`。相应的调整对应的视图。
>
> - 修改`Session`类并且加入字段到它的表单视图中

```
openacademy/models.py
    instructor_id = fields.Many2one('res.partner', string="Instructor")
    course_id = fields.Many2one('openacademy.course',
        ondelete='cascade', string="Course", required=True)
    attendee_ids = fields.Many2many('res.partner', string="Attendees")
openacademy/views/openacademy.xml
                                <field name="seats"/>
                            </group>
                        </group>
                        <label for="attendee_ids"/>
                        <field name="attendee_ids"/>
                    </sheet>
                </form>
            </field>
```



# Odoo10开发教程四（继承）

### 继承

#### 模型继承

Odoo提供两种继承机制，以模块化方式扩展现有模型。
 第一种继承机制允许一个模块修改另一个模块中定义的模型的行为。

- 给模型添加字段
- 覆盖模型现有字段
- 给模型添加约束
- 给模型添加方法
- 覆盖模型现有方法

第二种继承机制（委托）允许将模型的每条记录链接到父模型的记录，并且提供对父记录的透明访问。

#### 视图继承

Odoo不是通过覆盖来修改现有视图，而是通过视图继承。子视图不仅能够修改继承至父视图的自身内容，而且能够修改和删除父视图中的内容。
 扩展视图使用`inherit_id`字段引用其父代，而不是单个视图，其`arch`字段由任意数量的`xpath`元素组成，选择和更改其父视图的内容：

```
<!-- improved idea categories list -->
<record id="idea_category_list2" model="ir.ui.view">
    <field name="name">id.category.list2</field>
    <field name="model">idea.category</field>
    <field name="inherit_id" ref="id_category_list"/>
    <field name="arch" type="xml">
        <!-- find field description and add the field
             idea_ids after it -->
        <xpath expr="//field[@name='description']" position="after">
          <field name="idea_ids" string="Number of ideas"/>
        </xpath>
    </field>
</record>
```

*expr*
 在父视图中选者单个元素的*XPath*表达式。如果没有匹配到元素或者匹配到多个元素则引发错误。
*position*
 对匹配到的元素进行操作。
*inside*
 　在匹配元素的末尾追加
*before*
 　作为匹配元素的同级元素添加在其后面
*after*
 　作为匹配元素的同级元素添加在其前面
*replace*
 　替换匹配的元素
*attributes*
 　使用新的属性替换匹配元素的属性

> 提示
>  当匹配单个元素时，`position`可以直接在匹配的元素上设置属性。下面的两个继承将给出相同的结果：

```
<xpath expr="//field[@name='description']" position="after">
    <field name="idea_ids" />
</xpath>

<field name="description" position="after">
    <field name="idea_ids" />
</field>
```

> 练习更改现有内容
>
> - 使用模型继承，修改现有*partner*模型，添加`instructor`布尔字段，以及对应表示"授课-讲师"关联的*many2many*字段
>
> - 使用视图继承在*partner*的表单视图中显示这个字段
>
>   > 注意，这里是通过开发人员模式来查找视图外部ID并放置新字段的。
>
> - 创建文件`openacademy/models/partner.py`并将其导入`__init__.py`
>
> - 创建文件`openacademy/views/partner.xml`并将其添加到`__manifest__.py`

```
openacademy/__init__.py
# -*- coding: utf-8 -*-
from . import controllers
from . import models
from . import partner
openacademy/__manifest__.py
        # 'security/ir.model.access.csv',
        'templates.xml',
        'views/openacademy.xml',
        'views/partner.xml',
    ],
    # only loaded in demonstration mode
    'demo': [
openacademy/partner.py
# -*- coding: utf-8 -*-
from odoo import fields, models

class Partner(models.Model):
    _inherit = 'res.partner'

    # Add a new column to the res.partner model, by default partners are not
    # instructors
    instructor = fields.Boolean("Instructor", default=False)
    
    session_ids = fields.Many2many('openacademy.session',
        string="Attended Sessions", readonly=True)
openacademy/views/partner.xml
<?xml version="1.0" encoding="UTF-8"?>
 <odoo>
    <data>
        <!-- Add instructor field to existing view -->
        <record model="ir.ui.view" id="partner_instructor_form_view">
            <field name="name">partner.instructor</field>
            <field name="model">res.partner</field>
            <field name="inherit_id" ref="base.view_partner_form"/>
            <field name="arch" type="xml">
                <notebook position="inside">
                    <page string="Sessions">
                        <group>
                            <field name="instructor"/>
                            <field name="session_ids"/>
                        </group>
                    </page>
                </notebook>
            </field>
        </record>

        <record model="ir.actions.act_window" id="contact_list_action">
            <field name="name">Contacts</field>
            <field name="res_model">res.partner</field>
            <field name="view_mode">tree,form</field>
        </record>
        <menuitem id="configuration_menu" name="Configuration"
                  parent="main_openacademy_menu"/>
        <menuitem id="contact_menu" name="Contacts"
                  parent="configuration_menu"
                  action="contact_list_action"/>
    </data>
</odoo>
```

#### Domain

Odoo中，Domain代表记录集的条件表达式。Domain是定义模型子集的规则集合。每个规则是一个包含名称、操作和值的三元组。例如，下面是*Product*模型子集的Domain表达式，“单价大于1000且类型为服务”的记录集：

```
[('product_type', '=', 'service'), ('unit_price', '>', 1000)]
```

多个规则组合时，默认条件组合方式是AND。逻辑运算符`&(AND)`,`|(OR)`,`!(NOT)`可以用来显示的组合多个规则。它们在前缀位置使用（操作符在参数之前，而不是中间）。例如下面的Domain表达式，含义是"类型为服务或者单价不介于1000和2000之间"

```
['|',
    ('product_type', '=', 'service'),
    '!', '&',
        ('unit_price', '>=', 1000),
        ('unit_price', '<', 2000)]
```

当在客户端界面选择记录集时，`domain`参数可以添加到关联字段上，以限制只显示有效的关联字段。

> 练习在关联字段上使用Domain，当为授课选择讲师时，只有`instructor`值为`True`的讲师会被显示出来。

```
openacademy/models.py
    duration = fields.Float(digits=(6, 2), help="Duration in days")
    seats = fields.Integer(string="Number of seats")

    instructor_id = fields.Many2one('res.partner', string="Instructor",
        domain=[('instructor', '=', True)])
    course_id = fields.Many2one('openacademy.course',
        ondelete='cascade', string="Course", required=True)
    attendee_ids = fields.Many2many('res.partner', string="Attendees")
```

> 注意
>  声明为文字列表的domain会在服务端进行计算，不会出现在右侧的动态列表中，而声明为字符串的domain是在客户端进行计算的，字段名将出现在右侧列表。
>
> 练习更复杂的domain，创建新的*partner*类别*Techer/Level1*和*Techer/Level2*.一个授课的教授人可以是讲师或者任意级别的教师。
>
> - 修改*Session*模型的domain
> - 修改`openacademy/view/partner.xml`以获得访问*Partner*类别的入口。

```
openacademy/models.py
    seats = fields.Integer(string="Number of seats")

    instructor_id = fields.Many2one('res.partner', string="Instructor",
        domain=['|', ('instructor', '=', True),
                     ('category_id.name', 'ilike', "Teacher")])
    course_id = fields.Many2one('openacademy.course',
        ondelete='cascade', string="Course", required=True)
    attendee_ids = fields.Many2many('res.partner', string="Attendees")
openacademy/views/partner.xml
        <menuitem id="contact_menu" name="Contacts"
                  parent="configuration_menu"
                  action="contact_list_action"/>

        <record model="ir.actions.act_window" id="contact_cat_list_action">
            <field name="name">Contact Tags</field>
            <field name="res_model">res.partner.category</field>
            <field name="view_mode">tree,form</field>
        </record>
        <menuitem id="contact_cat_menu" name="Contact Tags"
                  parent="configuration_menu"
                  action="contact_cat_list_action"/>
    
        <record model="res.partner.category" id="teacher1">
            <field name="name">Teacher / Level 1</field>
        </record>
        <record model="res.partner.category" id="teacher2">
            <field name="name">Teacher / Level 2</field>
        </record>
    </data>
</odoo>
```

# Odoo10开发教程五（计算字段和默认值）

### 计算字段和默认值


到目前为止，我们接触的字段都是存储在数据库中并直接从数据库检索。字段也可以通过计算获得。在这种情况下，字段的值不是直接检索自数据库，而是通过调用模型的方法来实时计算获得。要创建计算字段，需要设置它的`compute`属性为方法名。这个计算方法通过计算`self`的每条记录来设置字段的值。

> 注意
> `self`是一个记录的有序集合，它支持标准的Python集合操作，如`len(self)`和`iter(self)`，加上额外的集合操作`recs1 + recs2`。迭代过程逐个提供`self`记录，其中每个记录本身是大小为1的集合。你可以通过点记号来访问/分配单个记录上的字段`record.name`。

```
import random
from odoo import models, fields, api

class ComputedModel(models.Model):
    _name = 'test.computed'

    name = fields.Char(compute='_compute_name')
    
    @api.multi
    def _compute_name(self):
        for record in self:
            record.name = str(random.randint(1, 1e6))
```

#### 依赖

计算字段的值通常取决于所在记录行的其它字段的值。ORM层期望开发人员使用`depends()`装饰器来指定计算方法的依赖性。当某些依赖关系被修改后，ORM层通过给定的依赖关系来触发字段的重新计算。

```
from odoo import models, fields, api

class ComputedModel(models.Model):
    _name = 'test.computed'

    name = fields.Char(compute='_compute_name')
    value = fields.Integer()
    
    @api.depends('value')
    def _compute_name(self):
        for record in self:
            record.name = "Record with value %s" % record.value
```

> 练习计算字段
>
> - 加入座席占用百分比字段到授课模型。
> - 在列表视图和表单视图中显示这个字段
> - 以进度条的方式显示这个字段

```
openacademy/models.py
    course_id = fields.Many2one('openacademy.course',
        ondelete='cascade', string="Course", required=True)
    attendee_ids = fields.Many2many('res.partner', string="Attendees")

    taken_seats = fields.Float(string="Taken seats", compute='_taken_seats')
    
    @api.depends('seats', 'attendee_ids')
    def _taken_seats(self):
        for r in self:
            if not r.seats:
                r.taken_seats = 0.0
            else:
                r.taken_seats = 100.0 * len(r.attendee_ids) / r.seats
openacademy/views/openacademy.xml
                                <field name="start_date"/>
                                <field name="duration"/>
                                <field name="seats"/>
                                <field name="taken_seats" widget="progressbar"/>
                            </group>
                        </group>
                        <label for="attendee_ids"/>
                <tree string="Session Tree">
                    <field name="name"/>
                    <field name="course_id"/>
                    <field name="taken_seats" widget="progressbar"/>
                </tree>
            </field>
        </record>
```

#### 默认值

任何字段都可以给出默认值。在字段定义中，添加选项`default=x`,x可以是Python字面值（bool，int，float，string），也可以是一个有返回值的方法。

```
name = fields.Char(default="Unknown")
user_id = fields.Many2one('res.users', default=lambda self: self.env.user)
```

> 注意
> `self.env` 对象给出了访问请求参数和其他有用的信息：
>
> - `self.env.cr` 或者 `self._cr`是数据库游标对象，通常用于查询数据库
> - `self.env.uid`或者`self._uid`是当前用户的数据库ID
> - `self.env.user`是当前用户记录
> - `self.env.ref(xml_id)`返回XML ID对应的记录
> - `self.env[model_name]`返回给定模型的实例
>
> 练习默认值
>
> - 定义start_date默认值为今天
> - 在授课类添加字段`active`，并且设置其默认值为`True`

```
openacademy/models.py
    _name = 'openacademy.session'

    name = fields.Char(required=True)
    start_date = fields.Date(default=fields.Date.today)
    duration = fields.Float(digits=(6, 2), help="Duration in days")
    seats = fields.Integer(string="Number of seats")
    active = fields.Boolean(default=True)
    
    instructor_id = fields.Many2one('res.partner', string="Instructor",
        domain=['|', ('instructor', '=', True),
openacademy/views/openacademy.xml
                                <field name="course_id"/>
                                <field name="name"/>
                                <field name="instructor_id"/>
                                <field name="active"/>
                            </group>
                            <group string="Schedule">
                                <field name="start_date"/>
```

> 注意
>  Odoo 有内置规则：`active`字段值为`False`时记录不可见

#### Onchange机制

"onchange"机制为客户端界面提供了一种方法，当用户在字段中填写了值，不需要向数据库保存任何内容，就可以更新表单。例如，假设模型有三个字段`amount`,`unit_price`和`price`，当数量和单价改变时，自动重新计算价格，并在表单界面更新。要实现这个需求，需要定义一个方法，并使用`onchange()`装饰器，`onchange()`的参数指定了在那个字段改变时，触发方法。其中`self`代表表单视图中的记录，你所做的任何更改，`self`都将立刻反应在表单上。

```
<!-- content of form view -->
<field name="amount"/>
<field name="unit_price"/>
<field name="price" readonly="1"/>
# onchange handler
@api.onchange('amount', 'unit_price')
def _onchange_price(self):
    # set auto-changing field
    self.price = self.amount * self.unit_price
    # Can optionally return a warning and domains
    return {
        'warning': {
            'title': "Something bad happened",
            'message': "It was very bad indeed",
        }
    }
```

对于计算字段的值，系统内置了`onchange()`装饰器。通过授课表单我们可以观察到：当改变座席数和参与者人数，`taken_seats`进度条会自动更新。

> 练习
>  通过"onchange"机制显示的验证无效值，例如座位数为负数或者座位数多与参与者。

```
openacademy/models.py
                r.taken_seats = 0.0
            else:
                r.taken_seats = 100.0 * len(r.attendee_ids) / r.seats

    @api.onchange('seats', 'attendee_ids')
    def _verify_valid_seats(self):
        if self.seats < 0:
            return {
                'warning': {
                    'title': "Incorrect 'seats' value",
                    'message': "The number of available seats may not be negative",
                },
            }
        if self.seats < len(self.attendee_ids):
            return {
                'warning': {
                    'title': "Too many attendees",
                    'message': "Increase seats or remove excess attendees",
                },
            }
```

#### 模型约束

odoo提供两种方式实现自动验证，`python constraints`和`sql constraints`
 Python约束通过方法装饰器`constraints()`来定义，并在记录集上调用这个方法。装饰器参数指定了约束涉及的字段，当涉及的字段中任一发生改变时触发方法执行。如果不满足约束条件，该方法将引发异常：

```
from odoo.exceptions import ValidationError

@api.constrains('age')
def _check_something(self):
    for record in self:
        if record.age > 20:
            raise ValidationError("Your record is too old: %s" % record.age)
    # all records passed the test, don't return anything
```

> 练习添加Python约束，讲师不能在自己的授课出席人中

```
openacademy/models.py
# -*- coding: utf-8 -*-

from odoo import models, fields, api, exceptions

class Course(models.Model):
    _name = 'openacademy.course'
                    'message': "Increase seats or remove excess attendees",
                },
            }

    @api.constrains('instructor_id', 'attendee_ids')
    def _check_instructor_not_in_attendees(self):
        for r in self:
            if r.instructor_id and r.instructor_id in r.attendee_ids:
                raise exceptions.ValidationError("A session's instructor can't be an attendee")
```

SQL约束通过模型属性`_sql_constraints`进行定义。它是一个三元素的元组的列表(name, sql_definition, message)，其中`name`是SQL约束名称，`sql_definition`是约束规则，`message`是违反约束规则时的警告信息。

> 练习添加SQL约束,在Postgre SQL文档帮助下，添加下列约束：
>
> - 验证课程描述与课程标题不能完全一样
> - 验证课程名是唯一的

```
openacademy/models.py
    session_ids = fields.One2many(
        'openacademy.session', 'course_id', string="Sessions")

    _sql_constraints = [
        ('name_description_check',
         'CHECK(name != description)',
         "The title of the course should not be the description"),
    
        ('name_unique',
         'UNIQUE(name)',
         "The course title must be unique"),
    ]

class Session(models.Model):
    _name = 'openacademy.session'
```

> 练习添加重复项，因为我们为课程名称添加了唯一性约束，所以不能再使用"复制"功能(表单->复制)。重写"复制"方法,允许复制课程对象，将原始名称更改为"原始名称的副本"。

```
openacademy/models.py
    session_ids = fields.One2many(
        'openacademy.session', 'course_id', string="Sessions")

    @api.multi
    def copy(self, default=None):
        default = dict(default or {})
    
        copied_count = self.search_count(
            [('name', '=like', u"Copy of {}%".format(self.name))])
        if not copied_count:
            new_name = u"Copy of {}".format(self.name)
        else:
            new_name = u"Copy of {} ({})".format(self.name, copied_count)
    
        default['name'] = new_name
        return super(Course, self).copy(default)
    
    _sql_constraints = [
        ('name_description_check',
         'CHECK(name != description)',
```

![img](https://cover.kancloud.cn/hx78/odoo_10!middle

# Odoo10开发教程六（高级视图）

#### 高级视图

##### 树视图

树视图可以采用辅助属性来进一步自定义其行为：
`decoration-{$name}`
 　允许根据对应记录属性修改行的文本风格。对于每个记录，将使用记录的属性作为上下文来计算表达式，如果值为`true`,则将相应的样式应用于行。其他上下文值为`uid`（当前用户的标识）和`current_date`（`yyyy-MM-dd`格式的当前日期字符串）。`{$name}`可以是`bf(font-weight:bold)`、`it(font-style:italic)`或任何bootstrap上下文颜色（`danger,info,muted,primary,success,warning`）

```
<tree string="Idea Categories" decoration-info="state=='draft'"
    decoration-danger="state=='trashed'">
    <field name="name"/>
    <field name="state"/>
</tree>
```

`editable`
`top`和`bottom`使树视图可直接编辑(而不需要通过表单视图)，其值就是新行出现的位置。

> 练习列表着色
>  编辑授课的树视图，使得持续时间少于5天的授课以蓝色显示，持续时间超过15天的授课以红色显示。编辑授课的树视图：

```
openacademy/views/openacademy.xml
            <field name="name">session.tree</field>
            <field name="model">openacademy.session</field>
            <field name="arch" type="xml">
                <tree string="Session Tree" decoration-info="duration<5" decoration-danger="duration>15">
                    <field name="name"/>
                    <field name="course_id"/>
                    <field name="duration" invisible="1"/>
                    <field name="taken_seats" widget="progressbar"/>
                </tree>
            </field>
```

##### 日历

将记录显示为日历活动，通过将根元素设置为`<calendar>`，主要的属性有：
`color`
 　字段的名称通过颜色来区分。颜色会自动分配给事件，但相同颜色定义的事件（`@color`属性有相同值的记录）将被使用相同的颜色。
`date_start`
 　记录中用于保存事件开始日期/时间的字段。
`date_stop`(可选)
 　记录中用于保存时间结束日期/时间的字段。

为每个日历事件定义标签的字段

```
<calendar string="Ideas" date_start="invent_date" color="inventor_id">
    <field name="name"/>
</calendar>
```

> 练习日历视图
>  给授课模型添加一个日历视图，使用户可以查看与开放学院相关联的事件。
>
> - 添加一个计算字段`end_date`，通过`start_date`和`duration`计算获得。
> - 反函数使字段可写，并允许在日历视图中移动授课（通过拖放操作）
> - 向授课模型添加日历视图
> - 添加日历视图到授课模型的动作中

```
openacademy/models.py
# -*- coding: utf-8 -*-

from datetime import timedelta
from odoo import models, fields, api, exceptions

class Course(models.Model):
    attendee_ids = fields.Many2many('res.partner', string="Attendees")

    taken_seats = fields.Float(string="Taken seats", compute='_taken_seats')
    end_date = fields.Date(string="End Date", store=True,
        compute='_get_end_date', inverse='_set_end_date')
    
    @api.depends('seats', 'attendee_ids')
    def _taken_seats(self):
                },
            }
    
    @api.depends('start_date', 'duration')
    def _get_end_date(self):
        for r in self:
            if not (r.start_date and r.duration):
                r.end_date = r.start_date
                continue
    
            # Add duration to start_date, but: Monday + 5 days = Saturday, so
            # subtract one second to get on Friday instead
            start = fields.Datetime.from_string(r.start_date)
            duration = timedelta(days=r.duration, seconds=-1)
            r.end_date = start + duration
    
    def _set_end_date(self):
        for r in self:
            if not (r.start_date and r.end_date):
                continue
    
            # Compute the difference between dates, but: Friday - Monday = 4 days,
            # so add one day to get 5 days instead
            start_date = fields.Datetime.from_string(r.start_date)
            end_date = fields.Datetime.from_string(r.end_date)
            r.duration = (end_date - start_date).days + 1
    
    @api.constrains('instructor_id', 'attendee_ids')
    def _check_instructor_not_in_attendees(self):
        for r in self:
openacademy/views/openacademy.xml
            </field>
        </record>

        <!-- calendar view -->
        <record model="ir.ui.view" id="session_calendar_view">
            <field name="name">session.calendar</field>
            <field name="model">openacademy.session</field>
            <field name="arch" type="xml">
                <calendar string="Session Calendar" date_start="start_date"
                          date_stop="end_date"
                          color="instructor_id">
                    <field name="name"/>
                </calendar>
            </field>
        </record>
    
        <record model="ir.actions.act_window" id="session_list_action">
            <field name="name">Sessions</field>
            <field name="res_model">openacademy.session</field>
            <field name="view_type">form</field>
            <field name="view_mode">tree,form,calendar</field>
        </record>
    
        <menuitem id="session_menu" name="Sessions"
```

##### 搜索视图

搜索视图的`<field>`元素可以使用`@filter_domain`覆盖为在给定字段上搜索而生成的域。在给定的域中，`self`表示用户输入的值。在下面的示例中，它用于搜索两个字段`name`和`description`。搜索视图还可以包含`<filter>`元素，这些元素用作预定义搜索的切换。过滤器必须具有以下属性之一：
`domain`
 　给搜索指定domain表达式
`context`
 　给搜索指定上下文；使用`group_by`对结果进行分组。

```
<search string="Ideas">
    <field name="name"/>
    <field name="description" string="Name and description"
           filter_domain="['|', ('name', 'ilike', self), ('description', 'ilike', self)]"/>
    <field name="inventor_id"/>
    <field name="country_id" widget="selection"/>

    <filter name="my_ideas" string="My Ideas"
            domain="[('inventor_id', '=', uid)]"/>
    <group string="Group By">
        <filter name="group_by_inventor" string="Inventor"
                context="{'group_by': 'inventor_id'}"/>
    </group>
</search>
```

对于非默认的搜索视图，使用`search_view_id`字段。而通过`context`字段为搜索字段设置默认值：`search_default_field_name`表单的上下文关键字将初始化field_name的值。搜索过滤器必须有`@name`选项，并且其值是布尔类型的（只能在默认情况可用）

> 练习搜索视图
>
> - 在课程搜索视图中添加按钮，用以筛选当前用户负责的课程，并且作为默认选择。
> - 再添加一个分组按钮，用于对当前用户负责的课程进行分组。

```
openacademy/views/openacademy.xml
                <search>
                    <field name="name"/>
                    <field name="description"/>
                    <filter name="my_courses" string="My Courses"
                            domain="[('responsible_id', '=', uid)]"/>
                    <group string="Group By">
                        <filter name="by_responsible" string="Responsible"
                                context="{'group_by': 'responsible_id'}"/>
                    </group>
                </search>
            </field>
        </record>
            <field name="res_model">openacademy.course</field>
            <field name="view_type">form</field>
            <field name="view_mode">tree,form</field>
            <field name="context" eval="{'search_default_my_courses': 1}"/>
            <field name="help" type="html">
                <p class="oe_view_nocontent_create">Create the first course
                </p>
```

##### 甘特图

水平条状的甘特图通常用于显示项目计划和进度，根元素是`<gantt>`。

```
<gantt string="Ideas"
       date_start="invent_date"
       date_stop="date_finished"
       progress="progress"
       default_group_by="inventor_id" />
```

> 练习甘特图
>  添加甘特图使用户可以查看授课的日程排期，授课将按讲师分组。
>
> - 创建一个计算字段，表示以小时计算的授课持续时间
> - 添加甘特图，并且将甘特图添加到授课模型的action上。

```
openacademy/models.py
    end_date = fields.Date(string="End Date", store=True,
        compute='_get_end_date', inverse='_set_end_date')

    hours = fields.Float(string="Duration in hours",
                         compute='_get_hours', inverse='_set_hours')
    
    @api.depends('seats', 'attendee_ids')
    def _taken_seats(self):
        for r in self:
            end_date = fields.Datetime.from_string(r.end_date)
            r.duration = (end_date - start_date).days + 1
    
    @api.depends('duration')
    def _get_hours(self):
        for r in self:
            r.hours = r.duration * 24
    
    def _set_hours(self):
        for r in self:
            r.duration = r.hours / 24
    
    @api.constrains('instructor_id', 'attendee_ids')
    def _check_instructor_not_in_attendees(self):
        for r in self:
openacademy/views/openacademy.xml
            </field>
        </record>

        <record model="ir.ui.view" id="session_gantt_view">
            <field name="name">session.gantt</field>
            <field name="model">openacademy.session</field>
            <field name="arch" type="xml">
                <gantt string="Session Gantt" color="course_id"
                       date_start="start_date" date_delay="hours"
                       default_group_by='instructor_id'>
                    <field name="name"/>
                </gantt>
            </field>
        </record>
    
        <record model="ir.actions.act_window" id="session_list_action">
            <field name="name">Sessions</field>
            <field name="res_model">openacademy.session</field>
            <field name="view_type">form</field>
            <field name="view_mode">tree,form,calendar,gantt</field>
        </record>
    
        <menuitem id="session_menu" name="Sessions"
```

##### 图形视图

图形视图用来表示对模型的概述和分析，根元素是`<graph>`。

> 注意
>  多维表的核心视图（根元素`<pivot>`）允许选择文件管理器以获得正确的图形数据库，然后再转到更多的图形视图。核心视图与图形视图共享相同的内容定义。

聚合视图有4种显示模式，通过`@type`属性定义。
**Bar(默认值)**
 　条形图，第一个维度用于在水平轴上定义组，其它维度定义每个组的聚合条。默认情况下，条是并排的，也可以通过`<graph>`的`@stacked="True"`来让条堆叠。
**Line**
 　2维折线图
**Pie**
 　2维饼图
 图形视图包含的`<field>`元素有`@type`属性定义值：
`row(默认值)`
 　该字段是聚合的
`measure`
 　该字段是分组后聚合的

```
<graph string="Total idea score by Inventor">
    <field name="inventor_id"/>
    <field name="score" type="measure"/>
</graph>
```

> 警告
>  图形视图只能对数据库字段进行聚合，不能对不存储在数据库的计算字段进行聚合。
>
> 练习图形视图
>  在授课对象中添加图形视图，为每个课程在条形视图下显示出席人数。
>
> - 添加字段将出席人数这计算字段存储在数据库
> - 添加相关图形视图

```
openacademy/models.py
    hours = fields.Float(string="Duration in hours",
                         compute='_get_hours', inverse='_set_hours')

    attendees_count = fields.Integer(
        string="Attendees count", compute='_get_attendees_count', store=True)
    
    @api.depends('seats', 'attendee_ids')
    def _taken_seats(self):
        for r in self:
        for r in self:
            r.duration = r.hours / 24
    
    @api.depends('attendee_ids')
    def _get_attendees_count(self):
        for r in self:
            r.attendees_count = len(r.attendee_ids)
    
    @api.constrains('instructor_id', 'attendee_ids')
    def _check_instructor_not_in_attendees(self):
        for r in self:
openacademy/views/openacademy.xml
            </field>
        </record>

        <record model="ir.ui.view" id="openacademy_session_graph_view">
            <field name="name">openacademy.session.graph</field>
            <field name="model">openacademy.session</field>
            <field name="arch" type="xml">
                <graph string="Participations by Courses">
                    <field name="course_id"/>
                    <field name="attendees_count" type="measure"/>
                </graph>
            </field>
        </record>
    
        <record model="ir.actions.act_window" id="session_list_action">
            <field name="name">Sessions</field>
            <field name="res_model">openacademy.session</field>
            <field name="view_type">form</field>
            <field name="view_mode">tree,form,calendar,gantt,graph</field>
        </record>
    
        <menuitem id="session_menu" name="Sessions"
```

##### 看板视图

看板视图用于任务组织、生产进度等，根元素是`<kanban>`。看板视图显示一组可按列分组的卡片。每个卡片表示一个记录，每列都显示聚合字段的值。例如项目任务可以按阶段（每列是一个阶段）分组或者按负责人（每列是一个用户）分组。看板视图将每个卡的结构定义为表单元素（包括基本HTML）和QWeb的混合。

> 练习看板视图
>  添加显示按课程分组的授课看板视图（列是课程）
>
> - 授课模型中添加整型字段`color`
> - 添加看板视图并更新action

```
openacademy/models.py
    duration = fields.Float(digits=(6, 2), help="Duration in days")
    seats = fields.Integer(string="Number of seats")
    active = fields.Boolean(default=True)
    color = fields.Integer()

    instructor_id = fields.Many2one('res.partner', string="Instructor",
        domain=['|', ('instructor', '=', True),
openacademy/views/openacademy.xml
        </record>

        <record model="ir.ui.view" id="view_openacad_session_kanban">
            <field name="name">openacad.session.kanban</field>
            <field name="model">openacademy.session</field>
            <field name="arch" type="xml">
                <kanban default_group_by="course_id">
                    <field name="color"/>
                    <templates>
                        <t t-name="kanban-box">
                            <div
                                    t-attf-class="oe_kanban_color_{{kanban_getcolor(record.color.raw_value)}}
                                                  oe_kanban_global_click_edit oe_semantic_html_override
                                                  oe_kanban_card {{record.group_fancy==1 ? 'oe_kanban_card_fancy' : ''}}">
                                <div class="oe_dropdown_kanban">
                                    <!-- dropdown menu -->
                                    <div class="oe_dropdown_toggle">
                                        <i class="fa fa-bars fa-lg"/>
                                        <ul class="oe_dropdown_menu">
                                            <li>
                                                <a type="delete">Delete</a>
                                            </li>
                                            <li>
                                                <ul class="oe_kanban_colorpicker"
                                                    data-field="color"/>
                                            </li>
                                        </ul>
                                    </div>
                                    <div class="oe_clear"></div>
                                </div>
                                <div t-attf-class="oe_kanban_content">
                                    <!-- title -->
                                    Session name:
                                    <field name="name"/>
                                    <br/>
                                    Start date:
                                    <field name="start_date"/>
                                    <br/>
                                    duration:
                                    <field name="duration"/>
                                </div>
                            </div>
                        </t>
                    </templates>
                </kanban>
            </field>
        </record>
    
        <record model="ir.actions.act_window" id="session_list_action">
            <field name="name">Sessions</field>
            <field name="res_model">openacademy.session</field>
            <field name="view_type">form</field>
            <field name="view_mode">tree,form,calendar,gantt,graph,kanban</field>
        </record>
    
        <menuitem id="session_menu" name="Sessions"
                  parent="openacademy_menu"
```

# Odoo10开发教程七（工作流和安全）


#### 工作流

工作流是与动态业务对象相关联的模型。工作流也用于跟踪动态演进的进程。

> 练习伪工作流
>  在授课模型上添加一个字段`state`，用于定义一个工作流程。授课存在三个可能的状态：Draft（草稿，默认值）、Confirmed（已确认）、Done（已完成）。在授课的form视图中，添加一个只读字段用于显示课程状态，并可以通过按钮来改变状态。有效的状态值迁移包括：
>
> - Draft->Confirmed
> - Confirmed->Draft
> - Confirmed->Done
> - Done->Draft

1. 添加一个新的字段`state`

2. 添加状态迁移方法，这个方法可以被form表单的按钮所调用，用以更改授课的状态

3. 将相关按钮添加到授课的form视图
   `openacademy/models.py`

```
    attendees_count = fields.Integer(
        string="Attendees count", compute='_get_attendees_count', store=True)
       
    state = fields.Selection([
        ('draft', "Draft"),
        ('confirmed', "Confirmed"),
        ('done', "Done"),
    ], default='draft')
       
    @api.multi
    def action_draft(self):
        self.state = 'draft'
       
    @api.multi
    def action_confirm(self):
        self.state = 'confirmed'
       
    @api.multi
    def action_done(self):
        self.state = 'done'
       
    @api.depends('seats', 'attendee_ids')
    def _taken_seats(self):
        for r in self:
```

   `openacademy/views/openacademy.xml`

   ```
            <field name="model">openacademy.session</field>
            <field name="arch" type="xml">
                <form string="Session Form">
                    <header>
                        <button name="action_draft" type="object"
                                string="Reset to draft"
                                states="confirmed,done"/>
                        <button name="action_confirm" type="object"
                                string="Confirm" states="draft"
                                class="oe_highlight"/>
                        <button name="action_done" type="object"
                                string="Mark as done" states="confirmed"
                                class="oe_highlight"/>
                        <field name="state" widget="statusbar"/>
                    </header>
       
                    <sheet>
                        <group>
                            <group string="General">
   ```

工作流可以与Odoo中的任何对象关联，并且可完全定制化。工作流用于构造和管理业务对象和文档的生命周期，并且通过图形化工具定义转换器、触发器等。工作流、动作（节点或操作）和迁移（条件）通常以XML记录行的形式定义。在工作流进行导航的标签称为工作项。

> 警告
>  与模型相关的工作流仅在创建模型记录时被创建。因此，在工作流定义之前创建的授课实例是没有与之对应的工作流实例的。
>
> 练习工作流
>  使用真正的授课工作流替换之前的伪工作流。修改授课的form视图，按钮将调用工作流而不是调用模型的方法。

   ```
openacademy/__manifest__.py
        'templates.xml',
        'views/openacademy.xml',
        'views/partner.xml',
        'views/session_workflow.xml',
    ],
    # only loaded in demonstration mode
    'demo': [
openacademy/models.py
        ('draft', "Draft"),
        ('confirmed', "Confirmed"),
        ('done', "Done"),
    ])

    @api.multi
    def action_draft(self):
openacademy/views/openacademy.xml
            <field name="arch" type="xml">
                <form string="Session Form">
                    <header>
                        <button name="draft" type="workflow"
                                string="Reset to draft"
                                states="confirmed,done"/>
                        <button name="confirm" type="workflow"
                                string="Confirm" states="draft"
                                class="oe_highlight"/>
                        <button name="done" type="workflow"
                                string="Mark as done" states="confirmed"
                                class="oe_highlight"/>
                        <field name="state" widget="statusbar"/>
openacademy/views/session_workflow.xml
<odoo>
    <data>
        <record model="workflow" id="wkf_session">
            <field name="name">OpenAcademy sessions workflow</field>
            <field name="osv">openacademy.session</field>
            <field name="on_create">True</field>
        </record>

        <record model="workflow.activity" id="draft">
            <field name="name">Draft</field>
            <field name="wkf_id" ref="wkf_session"/>
            <field name="flow_start" eval="True"/>
            <field name="kind">function</field>
            <field name="action">action_draft()</field>
        </record>
        <record model="workflow.activity" id="confirmed">
            <field name="name">Confirmed</field>
            <field name="wkf_id" ref="wkf_session"/>
            <field name="kind">function</field>
            <field name="action">action_confirm()</field>
        </record>
        <record model="workflow.activity" id="done">
            <field name="name">Done</field>
            <field name="wkf_id" ref="wkf_session"/>
            <field name="kind">function</field>
            <field name="action">action_done()</field>
        </record>
    
        <record model="workflow.transition" id="session_draft_to_confirmed">
            <field name="act_from" ref="draft"/>
            <field name="act_to" ref="confirmed"/>
            <field name="signal">confirm</field>
        </record>
        <record model="workflow.transition" id="session_confirmed_to_draft">
            <field name="act_from" ref="confirmed"/>
            <field name="act_to" ref="draft"/>
            <field name="signal">draft</field>
        </record>
        <record model="workflow.transition" id="session_done_to_draft">
            <field name="act_from" ref="done"/>
            <field name="act_to" ref="draft"/>
            <field name="signal">draft</field>
        </record>
        <record model="workflow.transition" id="session_confirmed_to_done">
            <field name="act_from" ref="confirmed"/>
            <field name="act_to" ref="done"/>
            <field name="signal">done</field>
        </record>
    </data>
</odoo>
   ```

> 提示
>  要检查授课实例对应的工作流实例是否被正确创建，可以：**设置->技术->工作流->实例**
>
> 练习自动状态迁移
>  当超过一半座席被保留时，自动将授课的状态从*Draft*迁移到*Confirmed*。

```
openacademy/views/session_workflow.xml
            <field name="act_to" ref="done"/>
            <field name="signal">done</field>
        </record>

        <record model="workflow.transition" id="session_auto_confirm_half_filled">
            <field name="act_from" ref="draft"/>
            <field name="act_to" ref="confirmed"/>
            <field name="condition">taken_seats > 50</field>
        </record>
    </data>
</odoo>
```

> 练习服务器动作
>  用服务器动作替换用于同步授课状态的Python方法。工作流和服务器动作都可以从UI创建。

```
openacademy/views/session_workflow.xml
            <field name="on_create">True</field>
        </record>

        <record model="ir.actions.server" id="set_session_to_draft">
            <field name="name">Set session to Draft</field>
            <field name="model_id" ref="model_openacademy_session"/>
            <field name="code">
model.search([('id', 'in', context['active_ids'])]).action_draft()
            </field>
        </record>
        <record model="workflow.activity" id="draft">
            <field name="name">Draft</field>
            <field name="wkf_id" ref="wkf_session"/>
            <field name="flow_start" eval="True"/>
            <field name="kind">dummy</field>
            <field name="action"></field>
            <field name="action_id" ref="set_session_to_draft"/>
        </record>

        <record model="ir.actions.server" id="set_session_to_confirmed">
            <field name="name">Set session to Confirmed</field>
            <field name="model_id" ref="model_openacademy_session"/>
            <field name="code">
model.search([('id', 'in', context['active_ids'])]).action_confirm()
            </field>
        </record>
        <record model="workflow.activity" id="confirmed">
            <field name="name">Confirmed</field>
            <field name="wkf_id" ref="wkf_session"/>
            <field name="kind">dummy</field>
            <field name="action"></field>
            <field name="action_id" ref="set_session_to_confirmed"/>
        </record>

        <record model="ir.actions.server" id="set_session_to_done">
            <field name="name">Set session to Done</field>
            <field name="model_id" ref="model_openacademy_session"/>
            <field name="code">
model.search([('id', 'in', context['active_ids'])]).action_done()
            </field>
        </record>
        <record model="workflow.activity" id="done">
            <field name="name">Done</field>
            <field name="wkf_id" ref="wkf_session"/>
            <field name="kind">dummy</field>
            <field name="action"></field>
            <field name="action_id" ref="set_session_to_done"/>
        </record>

        <record model="workflow.transition" id="session_draft_to_confirmed">
```

##### 安全

安全是为了实现统一的安全策略而进行配置的访问控制机制。
**基于组的访问控制机制**
 组是通过在`res.groups`模型的记录行来创建的，并通过菜单访问权限来限制权限。但是，即使没有菜单，对象仍然可以间接被访问，因此必须为组定义实际的对象级权限（读取、写入、创建、取消关联）。一般通过模块中的CSV文件插入。也可以通过字段的`groups`属性来限制对视图或对象上特定字段的访问。
**访问权限**
 访问权限通过`ir.model.access`的记录行来定义。每个访问权限与模型、组（或者非全局访问的组）相关联，并且是一组权限：读取、写入、创建、取消关联。这些访问权限一般通过`ir.model.access.csv`这个CSV文件创建。

```
id,name,model_id/id,group_id/id,perm_read,perm_write,perm_create,perm_unlink
access_idea_idea,idea.idea,model_idea_idea,base.group_user,1,1,1,0
access_idea_vote,idea.vote,model_idea_vote,base.group_user,1,1,1,0
```

> 练习
>  通过Odoo界面添加访问控制权限
>  建立一个新用户`John Smith`，然后建立`OpenAcademy/Session Read`组，并赋予这个组对*授课*模型的读权限。
>
> 1. 建立一个新用户`John Smit`通过 **设置->用户->用户**
> 2. 建立一个新组`session_read`通过 **设置->用户->组**,这个组拥有对*授课*模型的读权限
> 3. 编辑`John Smith`用户，把他加入到`session_read`组
> 4. 以`John Smith`身份登录系统，检查权限是否正确。

------

> 练习
>  通过数据文件添加访问控制权限：
>
> - 建立一个组`OpenAcademy / Manager`，这个组对开放学院的所有模型都有完全权限。
> - 让`Session`和`Course`对所有用户可读
> - 建立新的文件`openacademy/security/security.xml`用来定义`OpenAcademy Manager`组
> - 编辑文件`openacademy/security/ir.model.access.csv`来添加对模型的访问权限
> - 最后更新`openacademy/__manifest__.py`来添加新的数据文件

```
openacademy/__manifest__.py
    # always loaded
    'data': [
        'security/security.xml',
        'security/ir.model.access.csv',
        'templates.xml',
        'views/openacademy.xml',
        'views/partner.xml',
openacademy/security/ir.model.access.csv
id,name,model_id/id,group_id/id,perm_read,perm_write,perm_create,perm_unlink
course_manager,course manager,model_openacademy_course,group_manager,1,1,1,1
session_manager,session manager,model_openacademy_session,group_manager,1,1,1,1
course_read_all,course all,model_openacademy_course,,1,0,0,0
session_read_all,session all,model_openacademy_session,,1,0,0,0
openacademy/security/security.xml
<odoo>
    <data>
        <record id="group_manager" model="res.groups">
            <field name="name">OpenAcademy / Manager</field>
        </record>
    </data>
</odoo>
```

**记录规则**
 记录规则限制对给定模型的记录子集的访问权限。一个规则就是`ir.rule`模型的一行记录，并且将其关联到模型、数组（many2many字段）、或domain。domain指定了对那些记录有访问权限。
 这是一个记录规则的例子，这个规则防止非`cancel`状态的负责人被删除。请注意，`groups`字段的值必须遵守与ORM的`write()`方法一样的规则。

```
<record id="delete_cancelled_only" model="ir.rule">
    <field name="name">Only cancelled leads may be deleted</field>
    <field name="model_id" ref="crm.model_crm_lead"/>
    <field name="groups" eval="[(4, ref('sales_team.group_sale_manager'))]"/>
    <field name="perm_read" eval="0"/>
    <field name="perm_write" eval="0"/>
    <field name="perm_create" eval="0"/>
    <field name="perm_unlink" eval="1" />
    <field name="domain_force">[('state','=','cancel')]</field>
</record>
```

> 练习记录规则
>  为*授课*模型和*OpenAcademy / Manager*组添加记录规则，这个记录规则限制只有课程负责人可以对课程进行`write`和`unlink`操作，如果课程还没有负责人，这个组的所有用户都可以编辑它。在`openacademy/security/security.xml`文件中创建新的规则：
> `openacademy/security/security.xml`
>
> ```
> OpenAcademy / Manager
> ```

```
    <record id="only_responsible_can_modify" model="ir.rule">
        <field name="name">Only Responsible can modify Course</field>
        <field name="model_id" ref="model_openacademy_course"/>
        <field name="groups" eval="[(4, ref('openacademy.group_manager'))]"/>
        <field name="perm_read" eval="0"/>
        <field name="perm_write" eval="1"/>
        <field name="perm_create" eval="0"/>
        <field name="perm_unlink" eval="1"/>
        <field name="domain_force">
            ['|', ('responsible_id','=',False),
                  ('responsible_id','=',user.id)]
        </field>
    </record>
</data>
```

```
min_weight 
max_weight 
min_length
max_length
min_width
max_width 
min_height
max_height 
min_girth 
max_girth
```

