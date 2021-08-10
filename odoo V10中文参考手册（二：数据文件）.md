## 结构

odoo通过xml文件来定义数据，xml文件内容由odoo标签包含的元素组成

```xml
<!-- the root elements of the data file -->
<odoo>
  <operation/>
  ...
</odoo>
```

## 核心操作

### record

定义或更新一条数据库记录，有以下属性：

定义或更新一条数据库记录，有以下属性：

> model (required) -- 需要创建或更新的model名称
>  id -- 记录的唯一识别符
>  context -- 新增记录时使用的环境
>  forcecreate -- 在更新模式下如果记录不存在自动创建，需要指定id，默认该值为True

### field

每条记录可由多个field标签组成，当新增记录时用来定义值。如果一条记录没有任何field，在进行更新时是不触发任何操作的，在新增记录时会全部使用默认值。每个field必须有name属性，指定对应哪个字段。
 如果field的值被设置为空，会自动给一个False，来清空该字段或者避免使用默认值

- search 对于关系模型字段，取值需要一个基于对应模型的domain表达式，并使用它来搜索对应字段的值，如果是Many2one关系只返回第一条记录
- ref 如果提供了ref属性，它的取值必须是另一个模型的唯一id，用于查找设置field的值，一般用在Many2one 和 Reference字段上
- type 该属性用于解析field对应的内容，内容为文件时也可以通过使用file属性来实现解析
- xml, html 提供field的子元素作为一个单独的文档，将其中 %(external_id)s. %% 对应内容的解析出来
- file 确保内容是一个有效的文件路径
- char 将字段的值直接填充在field里
- base64 将field对应内容进行base64-encode，当图片作为附件时可与file属性结合使用
- int 将field对应值转换为整形填充在field里
- float 将field对应值转换为浮点型填充在field里
- list, tuple 需要包含与field的值完全对应的元素，每个元素决定生成的值，并填充到field
- eval 当目前有的方法都无法达到目的的时候，可以使用eval来执行一个python函数，并将函数返回值填充到field。函数执行环境会自动加载time, datetime, timedelta, relativedelta模型、一个决定引用模型的方法(ref) 以及一个当前模型的对象(obj)

### delete

delete标签可以删除所定义的任意数量的记录，有以下几个属性：

- model (required) -- 指定从哪个数据模型中删除记录
- id -- 需要删除的数据id
- search -- 用于搜索需要删除的数据的domain表达式，和id不同时定义

### function

function标签会调用model的指定方法，有两个必选参数：model（模型名）和methon(函数名)

### workflow

workflow标签会给已存在的工作流传递一个信号，可以通过ref属性指定对应id的一个工作流，或一个value属性返回一个工作流的id，该标签也有两个必选参数：model（与工作流相关联的模型）和action（发送给工作流的信号）

## 快捷方法

由于odoo的部分重要构造的方法比较复杂难懂，数据文件可以在record里通过比较简便的的方式来使用它们

### menuitem -- 定义一条ir.ui.menu记录

- 父级Menu
- 如果设置了parent属性，它的值必须指向另一条menuitem记录的id
- 如果没有设置parent属性，会将name属性值解析为以/分隔的菜单名称并插入到当前menu中
- 否则该菜单会被定义成一个顶级菜单
- 菜单名 - 如果name属性没有设置，那么会通过对应的触发动作找到菜单名，如果没有的话会使用对应记录的id
- 分组 - groups属性由一系列关联到res.groups模型的id组成，以逗号分隔，如果某个group前面是-号，代表该组从menu的分组中删除
- action - 如果指定的话，当打开该菜单时，会自动执行它所指向的action id
- id - 菜单的id

### template

创建一个QWeb视图，只需要一个arch元素并包含以下的属性：

- id -- 视图的id
- name, inherit_id, priority 与ir.ui.view的一致
- primary -- 设置为True并与inherit_id一起使用时，设置为主视图
- groups -- 以逗号分隔的分组id
- page -- 设置为True时，该页面为网页
- optional -- enabled 或 disabled，在用户界面中是否可以被禁用，默认是可以禁用

### report

使用默认值创建一条ir.actions.report.xml记录，大部分情况下只代替ir.actions.report.xml对应字段的属性，同时也会自动在more菜单里创建一个指向report对应model的项目

## CSV数据文件

XML文件比较易读，但是在用来在单个模型中创建大量记录时会变得很冗长，所以数据文件也可以使用csv文件来定义，一般用来定义模型的约束

>文件名是model_name.csv，第一行列出被改变的字段名（须有一个id作唯一识别），每行创建一条新记录，如：

```bash
#res.country.state.csv

"id","country_id:id","name","code"
state_au_1,au,"Australian Capital Territory","ACT"
state_au_2,au,"New South Wales","NSW"

#第一列是唯一的id，第二列是关联的country对象的id，第三四列是state模型的name和code
```
