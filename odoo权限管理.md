# 权限管理

# 权限管理的四个层次

## 菜单级别

不属于指定菜单所包含组的用户看不到该菜单，不客全，只是隐藏菜单，若知道菜单 ID，仍然可以通过指定 URL 访问

## 对象级别

对某个对角是否有'创建，读取，修改，删除'的权限，可以简单理解为表对象

## 记录级别

对对象表中的数据的访问权限，比如访问“客户”对象，业务员只能对自己创建的客户有访问权限，而经理可以访问其管辖的业务员所有的“客户”对象

## 字段级别

一个对象或表上的某些字段的访问权限，

比如产品的成本字段只有经理有读权限

```js
'name':fields.char('Name',size=128,required=True,select=True,write=['base.group_admin']read=['base.group_admin'])
```

定义 name 字段只能超级用户组可读写

# 权限组

权限管理核心是权限组，每个权限组，可以设置权限组的 Menus,Access Right,Record Rule

## Menus

定义该权限组可以访问哪些菜单，若该权限组可以访问某父菜单，父菜单对应的子菜单会显示出来若不想显示其子菜单，可以把其子菜单加入 "Useablity/No One" 权限组。

## Access Right

定义该权限组可以访问哪些对象，以及拥有 增、查、改、删的哪个权限 (create,read,write,unlink)

## Record Rule

定义该权限组可以访问对象中的哪些记录，以及拥有 增、查、改、删的哪个权限 ，Access Right 是对对象中的所有记录赋权限，Record Rule 则通过定义 domain 过滤指定某些记录赋权限

```python
['&',('department','=',user.context_department_id.id),('state','=','pr_draft')]
```

申购单的部门等于当前用户的部门，且申购单的状态是草稿状态

# 基于组的访问控制

## 视图中

运用 group_id

```xml
<record id="view_order_form_editable_list" model="ir.ui.view">
    <field name="name">sale.order.form.editable.list</field>
    <field name="model">sale.order</field>
    <field name="inherit_id" ref="sale.view_order_form" />
    <field name="group_id" eval="[(6,0,[ref('product.group.uos'),ref('product.group_stock_packaging'),ref('sale.group_mrp_properties')])]" />
    <field name="arch" type="xml">
    <xpath expr="//field[@name='order_line]/tree" position="before"
           <attribute name="editable" />
        </xpath>
    </field>
</record>
```



- eval

  把 eval 的值通过作为 python 运算返回该属性

- ref

视图的方法，根据 module_name.xml_id 返回数据库 id[(6,0,[xx,yy])](0,_ ,{'field': value}) 这将创建一个新的记录并连接它

```txt
(1,id,{'field': value}): 这是更新一个已经连接了的记录的值
(2,id,_) 这是删除或取消连接某个已经连接了的记录
(3,id,_) 这是取消连接但不删除一个已经连接了的记录
(4,id,_) 连接一个已经存在的记录
(5,_,_) 取消连接但不删除所有已经连接了的记录
(6,_,[ids]) 用给出的列表替换掉已经连接了的记录

这里的下划线一般是 0 或 False
```



## 访问权限管理

对于其内的数据访问权限管理有两种机制:

第一种是模型访问权限管理 (accessrule)；

第二种是记录规则管理 (record rule)。

record rule 是对 accessrule 的细化如果不为模块设置规则，默认只有 Administator 才能访问这个模型的数据

record rule 对 Administator 用户是无效的，而 access rule 还是有效

### access rule

权限对象模型是` ir.model.access.csv`

一般是放在` security` 文件夹下的 `ir.model.access.csv `文件来管理的文件表头如下：`id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink`

来一个例子：

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlinkaccess_todo_task_group_user,todo.task.user,model_todo_task,base.group126_user,1,1,1,1
```

```json
id:权限的 ID 不可重复
name 描述
model_id:id：对象，命名是 model_xxx
group_id:id 组名称
下面的，0 表示无权限， 1 表示有权限
perm_read 只读
perm_write 修改
perm_create 创建
perm_unlink 删除
```



### record rule

一般是放在` security` 文件夹下的 模块名`_record_rules.xml` 文件来管理的

```xml
<?xml version="1.0" encoding="utf-8”?>
<openerp>
    <data noupdate="1">
        <record id="todo_task_user_rule" model="ir.rule">
            <field name="name">ToDo Tasks only for owner</field>
            <field name="model_id" ref="model_todo_task"/>
            <field name="domain_force">[(’create_uid’,’=’,user.id)]</field>
            <field name="groups" eval="[(4,ref(’base.group_user’))]"/>
            <field name="perm_read" eval="1" />
            <field name="perm_write" eval="1" />
            <field name="perm_create" eval="1" />
            <field name="perm_unlink" eval="1" />
        </record>
    </data>
</openerp>
```

```js
record rule 记录是 ir.rule 模型， 存在 public.ir_rule 表格中
model_id 作用于哪个模型domain_force 对该模型中所有记录进行某种过滤操作
noupdate 值为 1 表示升级模块不会更新本数据
base.group_user 是人力资源 / 雇员 
```

# 完整的例子

## 建立组

```xml
<record id="group_department_project_admin" model="res.groups">
    <field name="name">A</field>
    <fieldname="category_id" ref="B"/>
    <field name="users" eval="[(4, ref('base.user_root'))]"/> //把 admin 用户加
入该组中
</record>
```

```json
@ name 组名称
@ category_id 属于哪个应用程序，或者哪个模块
@ users 组里面的用户
```

这样 B 应用程序就建立了一个名叫 A 的组。并且初始化了 A 组的一个用户 admin  

## 组控制菜单显示

A

```xml
<record model="ir.ui.menu" id=" memu_id1">
    <field name="name">menu1</field>
    <field name="groups_id" eval="[(6,0,[ref('A'),ref('B')]),]"/>
    <field name="sequence">1</field>
</record>
```

```json
@name 菜单名称
@groups_id 哪些组可以访问该菜单
@sequence 该菜单的序号
这样 A 组与 B 组的成员都可以访问 menu1 菜单，menu1 菜单的显示顺序为 1
注：eval 后面解释，多个组访问用“，”隔开
```

```xml
<menuitem id="menu_id2 " name="menu2" parent="menu_id1"sequence="1" groups="A,B "/>
```

```json
@ name 菜单名称
@ parent 父类菜单 如果没有可以不写 parent
@ groups 哪些组可以访问该菜单
这样 menu1 的子菜单 menu2 可以被 A 组合 B 组的成员访问
```

## 权限规则

```xml
<record model="ir.rule" id="rule1">
    <field name="name">rule1</field>
    <field name="model_id" ref="model_model1"/>
    <field name="global" eval="True"/>
    <field name="domain_force">[1,’=’,1]</field>
    <field name="groups" eval="[(4,ref('A'))]"/>
</record>
```

```json
@name 规则名称
@model_id 依赖的模块
@global 是否是全局
@domain_force 过滤条件
@groups 属于哪个组这样 A 组的成员就可以取到 model_model1 的所有数据
```



## ir.model.access.csv

```json
@id 随便取
@name 随便取
@model_id:id 这个就是你所定义的对象了
@group_id:哪个组
@perm_read","perm_write","perm_create","perm_unlink" 增删改查权限了。1 代表有权限 
```

# Eval

- many2many

```json
(0,0,{values}) 根据 values 里面的信息新建一个记录。
(1,ID,{values})更新 id=ID 的记录（写入 values 里面的数据）
(2,ID) 删除 id=ID 的数据（调用 unlink 方法，删除数据以及整个主从数据链接关系）
(3,ID) 切断主从数据的链接关系但是不删除这个数据
(4,ID) 为 id=ID 的数据添加主从链接关系。
(5) 删除所有的从数据的链接关系就是向所有的从数据调用(3,ID)
(6,0,[IDs]) 用 IDs 里面的记录替换原来的记录（就是先执行(5)再执行循环 IDs 执行
（4,ID））
```

- one2many

```json
(0, 0,{ values })
(1,ID,{values}) 
(2,ID)
```

- many2one

many2one 的字段比较简单，直接填入已经存在的数据的 id 或者填入 False 删除原来
的记录。

# 隐藏的常用技巧

* 直接隐藏

  ```xml
  <group name="owner" position="attributes">
      <attribute name="invisible">True</attribute>
  </group>
  ```

*  满足某些条件的隐藏

  ```xml
  <xpath expr="//field[@name='parent_id']" position='attributes'>
      <attribute name="attrs">{'invisible': [('passenger','=',True)]}</attribute>
  </xpath>
  <group col="4" string='旅客信息' attrs="{'invisible': [('supplier','=',True)]}">
  </group>
  ```

*  通过组来隐藏

```xml
<xpath expr="//field[@name='type']" position="attributes">
    <attribute name="groups">base.group_no_one</attribute>
</xpath>
```

* 菜单的隐藏

```xml
<record model="ir.ui.menu" id="crm.menu_crm_opportunities">
    <field eval="[(6,0, [ref('base.group_no_one'),])]"name="groups_id"/></record> 
```

