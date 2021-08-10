## 记录集

model的数据是通过数据集合的形式来使用的，定义在model里的函数执行时它们的self变量也是一个数据集合

```python
class AModel(models.Model):
    _name = 'a.model'

    def a_method(self):
        # self can be anywhere between 0 records and all records in the database
        self.do_operation()

    def do_operation(self):
    print self # => a.model(1, 2, 3, 4, 5)
    for record in self:
        print record # => a.model(1), then a.model(2), then a.model(3), ...
```

获取有关联关系的字段(one2many,many2one,many2many)也是返回一个数据集合，如果字段为空则返回空的集合。

> 每个赋值语句都会触发数据库字段更新，同时更新多个字段时可使用或者更新多条记录时使用write函数

```python
# 3 * len(records) database updates
for record in records:
    record.a = 1
    record.b = 2
    record.c = 3

# len(records) database updates
for record in records:
    record.write({'a': 1, 'b': 2, 'c': 3})

# 1 database update
records.write({'a': 1, 'b': 2, 'c': 3})
```

- 数据缓存和预读取
   odoo会为记录保留一份缓存，它有一种内置的预读取机制，通过缓存来提升性能。

- 集合运算符

  - record in set返回record是否在set中，record须为单条记录，record not in set反之
  - set1 <= set2 返回set1是否为set2的子集
  - set1 >= set2 返回set2是否为set1的子集
  - set1 | set2 返回set1和set2的并集
  - set1 & set2 返回set1和set2的交集
  - set1 - set2 返回在集合set1中但不在set2中的记录

- 其他集合运算

  - filtered() 返回满足条件的数据集,如

    ```python
    # only keep records whose company is the current user's
    records.filtered(lambda r: r.company_id == user.company_id)
    
    # only keep records whose partner is a company
    records.filtered("partner_id.is_company")
    ```

  - sorted() 返回根据提供的键排序之后的结果

    ```python
    # sort records by name
    records.sorted(key=lambda r: r.name)
    ```

  - mapped() 返回应用了指定函数之后的结果集

    ```python
    #returns a list of summing two fields for each record in the set
    records.mapped(lambda r: r.field1 + r.field2)
    
    #函数也可以是字符串 对应记录的字段
    # returns a list of names
    records.mapped('name')
    
    # returns a recordset of partners
    record.mapped('partner_id')
    ```

    ## 运行环境

    运行环境保存了很多ORM相关的变量：数据库查询游标、当前用户、元数据，还存有缓存。所有的model数据集都有不可改变的环境变量，可使用`env`来访问,如records.env.user,[records.env.cr](http://records.env.cr),records.env.context,运行环境还可用于为其他模型初始化一个空的集合并对该模型进行查询

    ```python
    self.env['res.partner'].search([['is_company', '=', True], ['customer', '=', True]])
    #res.partner(7, 18, 12, 14, 17, 19, 8, 31, 26, 16, 13, 20, 30, 22, 29, 15, 23, 28, 74)
    ```

- 更改运行环境
   可以基于一个运行环境量自定义，以得到拥有新运行环境的数据集

  - sudo() 使用现有数据集创建一个新运行环境，得到一个基于新运行环境的数据集的拷贝

```python
# create partner object as administrator
env['res.partner'].sudo().create({'name': "A Partner"})

# list partners visible by the "public" user
public = env.ref('base.public_user')
env['res.partner'].sudo(public).search([])
```

- with_context()
   一个参数时可用于替换当前运行环境的context，多个参数时通过keyword添加到当前运行环境context或单参数时设置的context
- with_env() 完整替换当前运行环境

## 常用ORM函数

- search() 接收domain表达式参数，返回符合条件的数据集合，可以通过limit,offset参数返回一个子集，还可通过order参数对数据排序

```python
>>> self.search([('is_company', '=', True), ('customer', '=', True)])
res.partner(7, 18, 12, 14, 17, 19, 8, 31, 26, 16, 13, 20, 30, 22, 29, 15, 23, 28, 74)
>>> self.search([('is_company', '=', True)], limit=1).name
'Agrolait'
```

> 如果只需要知道满足条件的数据数量，可以使用search_count()函数python

- create() 接收多个字段、值的组合，返回新创建的数据集

  ```python
  >>> self.create({'name': "New Name"})
  res.partner(78)
  ```

- write() 接收多个字段、值组合，会对指定数据集的所有记录进行修改，不返回

  ```python
  self.write({'name': "Newer Name"})
  ```

- browse() 根据数据的id或者一组id来查找，返回符合条件的数据集合

  ```
  >>> self.browse([7, 18, 12])
  res.partner(7, 18, 12)
  ```

- exists() 得到某个数据集中保留在数据库中的那部分，或在对一个数据集进行处理后重新赋值

```python
if not record.exists():
    raise Exception("The record has been deleted")

records.may_remove_some()
# only keep records which were not deleted
records = records.exists()
```

- ref() 运行环境函数根据提供的external id返回对应的数据记录

  ```python
  >>> env.ref('base.group_public')
  res.groups(2)
  ```

- ensure_one() 检验某数据集是否只包含单条数据，如果不是则报错

  ```python
  records.ensure_one()
  # 和下面的语句效果相同
  assert len(records) == 1, "Expected singleton"
  ```

## 创建模型

模型字段就是定义的模型的属性，默认情况下字段名称就是属性名的大写，也可通过string参数指定

```python
from odoo import models, fields
class AModel(models.Model):
    _name = 'a.model.name'

    field1 = fields.Char()
    field2 = fields.Integer(string="an other field")
```

可以通过default参数设置字段的默认值，默认值可以是特定的值，也可以指向一个函数

```python
a_field = fields.Char(default="a value")

def compute_default_value(self):
    return self.get_value()
a_field = fields.Char(default=compute_default_value)
```

- 实时计算的字段
   字段也可以是通过实时计算得来的，指定compute参数，并且当用到其他字段时需要使用depends声明

```python
from odoo import api
total = fields.Float(compute='_compute_total')

@api.depends('value', 'tax')
def _compute_total(self):
    for record in self:
        record.total = record.value + record.value * record.tax
```

1.依赖的字段如果是子集里的字段，可用.表示

```python
@api.depends('line_ids.value')
def _compute_total(self):
    for record in self:
        record.total = sum(line.value for line in record.line_ids)
```

2.默认情况下实时计算的字段是不保存到数据库的，可以通过store=True参数来设置保存且可以搜索
 3.可以为实时计算字段设置search参数来使其可搜索，参数的值须是一个返回domain表达式的函数

```python
upper_name = field.Char(compute='_compute_upper', search='_search_upper')

def _search_upper(self, operator, value):
    if operator == 'like':
        operator = 'ilike'
    return [('name', operator, value)]
```

4.实时计算的字段也可以通过inverse参数来赋值，通过一个反转compute的函数来设置相关的字段值

```python
document = fields.Char(compute='_get_document', inverse='_set_document')

def _get_document(self):
    for record in self:
        with open(record.get_document_path) as f:
            record.document = f.read()
def _set_document(self):
    for record in self:
        if not record.document: continue
        with open(record.get_document_path()) as f:
            f.write(record.document)
```

5.多个字段可以同时使用同一个方法计算而来

```python
discount_value = fields.Float(compute='_apply_discount')
total = fields.Float(compute='_apply_discount')

@depends('value', 'discount')
def _apply_discount(self):
    for record in self:
        # compute actual discount from discount percentage
        discount = record.value * record.discount
        record.discount_value = discount
        record.total = record.value - discount
```

6.关联字段
 关联字段是实时计算字段的一个特例，它会给出子集对应的值，通过related参数来定义，就像普通字段一样可以保存到数据库
`nickname = fields.Char(related='user_id.partner_id.name', store=True)`

- onchange：实时更新用户界面
   当用户在表单中更改某个字段的值时，其他相关字段可以在不需保存的情况下实时更新

  ```python
  @api.onchange('field1', 'field2') # 当这两个字段值改变时调用该函数
  def check_change(self):
    if self.field1 < self.field2:
        self.field3 = True
  ```

  实时计算字段和onchange方法会自动在客户端调用，不需要在视图里做特别声明，但可以在视图中通过on_change="0"参数来阻止onchange函数的自动调用
  `<field name="name" on_change="0"/>`

- 低级SQL
   运行环境的cr属性指向当前数据库查询的游标，在需要进行ORM没有提供的复杂查询或性能优化时，可以通过它直接执行查询
  `self.env.cr.execute("some_sql", param1, param2, param3)`
   但由于运行环境变量里同时存有大量缓存，所以在进行create/update,delete操作后需要使用invalidate_all()方法清除缓存: `self.env.invalidate_all()`,select方法是不需要更新缓存的，因为没有对数据进行更改

## 模型使用

odoo的模型都是从class odoo.models.Model(pool, cr)继承而来
 模型的属性结构：
 1._name 业务对象的名称
 2._rec_name 可选的name字段名称，供osv的name_get()方法使用，默认值name
 3._inherit 如果设置了**name属性，它的取值是单个或多个父级的模型名称；没有设置**name属性时，只能是单个模型名称
 4._order 在搜索的时候的默认排序，默认值是id
 5._auto 指定该表是否需要创建，默认值是True，如果设置成False需要重写init方法来创建表
 6._table 当_auto设置成false时，该值为创建的表名；默认情况下会自动生成一个
 7._inherits 定义上级模型关联使用的外键

```python
_inherits = {
    'a.model': 'a_field_id',
    'b.model': 'b_field_id'
}
```

8._sql_constraints 通过一个(name, sql_definition, message)的三元组列表定义表的sql级别约束
 9._parent_store 与 parent_left ， parent_right 一起使用，可得到一个嵌套的集合，默认是不启用的

### CRUD

- create(vals)
   使用传递的vals参数为模型生成一条新记录，参数示例：`{'field_name': field_value, ...}`，该方法返回该新创建的记录

- browse([ids])
   返回满足条件的记录集合，参数可以为空或单个id或一系列id

- unlink() 删除当前集合的数据

- write(vals) 使用提供的数据更新当前集合里的所有记录，参数示例：`{'foo': 1, 'bar': "Qux"}`

  - int或float型字段，给定的值须为对应的整型或浮点型

  - 布尔型字段，对应的值须为bool型

  - Selection字段，给定的值须符合条件

  - Many2one字段，给定的值须与对应的数据库记录相符

  - 其他无关联关系的字段使用字符串作值

  - One2many和Many2many字段通过一个特殊的格式命令来操纵对应字段值，通过一系列三元组按顺序来对数据进行操作，下面是一些常用的：

    ```json
    (0, _, values) 为指定的value字典添加一条新记录
    (1, id, values) 更新一条现有记录，条件是id为指定id且value在指定values中，不能在create方法里使用
    (2, id, _) 将指定id的数据从数据集中删除并从数据库删除，不能在create里使用
    (3, id, _) 将指定id的数据从数据集中删除但不从数据库删除，不能用在One2many关系及create里
    (4, id, _) 将指定id的数据添加到数据集中，不能用在One2many关系上
    (5, _, _) 将集合的所有数据删除，相当于当3作用于每条记录上
    (6, _, ids) 使用ids列表里匹配的所有数据替换当前记录，相当于先执行5再循环执行4
    ```

- read([fields]) 从self里读取指定的字段，专供rpc使用

- read_group(domain, fields, groupby, offset=0, limit=None, orderby=False, lazy=True) 得到一个通过groupby参数分组后的记录的列表

  ```json
  domain 搜索条件的domain表达式列表 [['field_name', 'operator', 'value'], ...]
  fields (list)  需要展示出来的字段列表
  groupby (list) 用来分组的表达式列表，分组表达式可以是单个字段或者一个函数如：'field:groupby_function'，目前函数只支持 'day', 'week', 'month', 'quarter' or 'year'并且只能作用在date和datetime字段上
  offset (int) 可选参数，代表从哪个条记录开始取
  limit (int)  可选参数，代表取出多少条记录
  orderby (list) 可选参数，和search函数的orderby参数一样
  lazy (bool) 值为true时，记录只会根据第一个groupby值进行分组，后面的groupby参数会被存在__context 中；值为false时会把所有分组条件一起执行
  ```

### Searching

- search(args[, offset=0][, limit=None][, order=None][, count=False])
   根据args参数里的domain表达式来搜索所有记录，参数列表：
   1.args domain表达式，为空时返回所有记录
   2.offset (int) 从第几条记录开始取
   3.limit (int) 返回记录行数的最大值
   4.order (str) 排序的字段
   5.count (bool) 当值为True的时候只返回匹配记录的条数

- search_count(args)
   返回根据给定domain表达式参数查询所得到的记录条数

- name_search(name='', args=None, operator='ilike', limit=100)
   返回根据name条件来查询，并满足args指定的domain表达式的记录集合

  ```json
  name (str) -- 用来匹配的name字符串
  args (list) -- domain表达式列表
  operator (str) --  用来匹配的操作符，如： 'like' ， '='.
  limit (int) -- 可选参数，最多返回的记录行数
  ```

### 记录集合操作

- ids 得到当前记录集合的id列表
- ensure_one() 验证一个记录集合是否只包含一条记录
- exists() 返回当前记录集中真正存在的子集，并把缓存中未删除的部分做标记，可用于判断`if record.exists():`
- filtered(func) 返回满足func参数内条件的记录集合，参数可以是一个函数或者用.分隔的字段列表
- sorted(key=None, reverse=False) 返回按key排序之后的记录集，key参数可以是一个返回单个key的函数或字段名称或为空，reverse参数为True时即为倒序
- mapped(func) 将func函数应用到所有记录上，并返回记录列表或集合

### 环境切换

- sudo([user=SUPERUSER]) 返回通过指定用户得到的新记录集，默认会返回SUPERUSER的记录集（前提是权限没有问题）

- with_context([context][, **overrides])
   返回当前记录集在扩展环境下的新记录集，扩展环境可以由指定环境和overrides参数合并而成、或由当前环境和overrides参数合并而成

  ```python
  # current context is {'key1': True}
  r2 = records.with_context({}, key2=True)
  # -> r2._context is {'key2': True}
  r2 = records.with_context(key2=True)
  # -> r2._context is {'key1': True, 'key2': True}
  ```

- with_env(env) 返回在指定环境下的新版记录集合

### 字段和视图查询

- fields_get([fields][, attributes])
   以数据字典的形式返回字段的定义，通过继承得来的字段也会在其中，string/help/selection属性会自动被翻译

  - fields参数是字段列表、为空或不传返回所有字段
  - attributes 可指定字段的属性、为空或不传时返回全部的

- fields_view_get([view_id | view_type='form'])
   返回指定视图的具体组成如：字段，模型，视图结构

  > 参数列表：
  >  view_id 视图的id或None
  >  view_type 当view_id参数为空时指定视图类型如form,tree等
  >  toolbar 参数为true时将上下文动作包含在内

### 其他方法

- default_get(fields) 获取指定字段的默认值
- name_get() 以列表形式返回每条记录的描述，默认是display_name字段
- name_create(name) 相当于调用create方法创建一条新记录而只设置一个display_name

### 内置字段

- id 数据识别字段
- _log_access 决定记log的字段（created_date,write_uid..)是否创建，默认值True
- create_date 记录创建的时间
- create_uid 创建人的id，关联到res.users
- write_date 记录最近的修改时间
- write_uid 最近修改记录的用户id，关联到res.users

### 保留字段

一些字段名称是给model保留的，用来实现一些预定义的功能。当需要实现对应功能是需要对相应的保留字段进行定义

- name(Char) -- _rec_name的默认值，在需要用来展示的时候使用
- active(Boolean) -- 设置记录的全局可见性，当值为False时通过search和list是获取不到的
- sequence(Integer) -- 可修改的排序，可以在列表视图里通过拖拽进行排序
- state(Selection) -- 对象的生命周期阶段，通过fileds的states属性使用
- parent_id(Many2one) -- 用来对树形结构的记录排序，并激活domain表达式的child_of运算符
- parent_left,parent_right -- 与 _parent_store结合使用，提供更好的树形结构数据读取

## 装饰器函数

模块提供了两种api形式，在传统形式中，所有参数明确地传给方法；还有一种记录行形式，提供了更加面向对象化的操作方式

```python
#传统方式：
model = self.pool.get(MODEL)
ids = model.search(cr, uid, DOMAIN, context=context)
for rec in model.browse(cr, uid, ids, context=context):
    print rec.name
model.write(cr, uid, ids, VALUES, context=context)

#新的记录行方式
env = Environment(cr, uid, context) # cr, uid, context wrapped in env
model = env[MODEL]                  # retrieve an instance of MODEL
recs = model.search(DOMAIN)         # search returns a recordset
for rec in recs:                    # iterate over the records
    print rec.name
recs.write(VALUES)                  # update all records in recs
```

在传统方式下，通过某些参数自动应用了装饰方法

- odoo.api.multi(method)
   在记录行方式下装饰一个对记录进行操作的方法

```python
@api.multi
def method(self, args):
...

#传统方式下使用方式
# recs = model.browse(cr, uid, ids, context)
recs.method(args)

model.method(cr, uid, ids, args, context=context)
```

- odoo.api.model(method)
   在记录行方式下装饰一个内容不明确、但模型明确的方法

```python
@api.model
def method(self, args):
    ...

#传统方式
# recs = model.browse(cr, uid, ids, context)
recs.method(args)

model.method(cr, uid, args, context=context)
```

- odoo.api.depends(*args)
   返回为compute方法指定依赖字段的装饰器，每个参数必须是字符串

```python
name = fields.Char(compute='_compute_pname')

@api.one
@api.depends('partner_id.name', 'partner_id.is_company')
def _compute_pname(self):
    if self.partner_id.is_company:
        self.pname = (self.partner_id.name or "").upper()
    else:
        self.pname = self.partner_id.name
```

- odoo.api.constrains(*args)
   装饰一个约束检查方法，每个参数必须是需要检查的字段

```python
@api.one
@api.constrains('name', 'description')
def _check_description(self):
    if self.name == self.description:
        raise ValidationError("Fields name and description must be different")
```

> 在检验失败时抛出ValidationError错误，且不支持关联字段检验

- odoo.api.onchange(*args)
   返回一个监控指定字段的onchange方法的装饰器，每个参数必须是字段名称

  ```python
  @api.onchange('partner_id')
  def _onchange_partner(self):
    self.message = "Dear %s" % (self.partner_id.name or "")
  ```

  > 该函数可能会返回 以数据字典形式组装的当前更改字段的domain表达式和一个警告消息，不支持关联字段处理
  
  ```python
  return {
  'domain': {'other_id': [('partner_id', '=', partner_id)]},
  'warning': {'title': "Warning", 'message': "What is this?"},
  }
  ```
  
- odoo.api.returns(model, downgrade=None, upgrade=None)
   返回一个获取model实例的方法的装饰器

  > 参数列表
  >  model 模型名称，self代表当前模型
  >  downgrade 一个将value值从记录形式转化为传统形式的方法：downgrade(self, value, *args, **kwargs)
  >  upgrade 一个将value从传统形式转化为记录形式的方法：upgrade(self, value, *args, **kwargs)

self,*args,**kwargs是在传统形式下需要传的参数

```python
该装饰器将函数的输出变成api形式：传统形式下返回id/ids/false,记录形式下返回记录集合
@model
@returns('res.partner')
def find_partner(self, arg):
    ...     # return some record

# output depends on call style: traditional vs record style
partner_id = model.find_partner(cr, uid, arg, context=context)

# recs = model.browse(cr, uid, ids, context)
partner_record = recs.find_partner(arg)
```

- odoo.api.one(method)
   装饰一个需要将self作为单例模式使用的记录形式方法，该方法自动对记录进行循环并将结果组织成列表，当记录行被returns方法装饰过时，该方法会将对应的实例组织起来，从9.0版本开始不用了
- odoo.api.v7(method_v7) 用于装饰支持老版api的方法
- odoo.api.v8(method_v8) 用于装饰支持新版api的方法

```python
@api.v8
def foo(self):
...

@api.v7
def foo(self, cr, uid, ids, context=None):
    ...
```

## 字段

### 基本字段

- class odoo.fields.Field(string=, **kwargs)

  > 参数列表：
  >  string(string) -- 用户能看到的字段的标签
  >  help(string) -- 用户能看到的关于该字段的提示
  >  readonly(boolean) -- 字段是否设置为只读，默认为False
  >  required(boolean) -- 字段是否为必须，默认False
  >  index(boolean) -- 字段是否作为索引保存在数据库中，默认False
  >  default -- 字段的默认值，可以是一个特定的值或者一个有返回值的函数，可使用default=None来忽略字段的default设置
  >  states -- 用数据字典封装视图里的属性-值对，如'readonly', 'required', 'invisible'
  >  groups -- 用逗号分隔的xml id列表，可以限制用户对字段的访问
  >  copy(boolean) -- 指定当数据行被复制时该字段是否被复制，默认是True，实时计算字段和one2many字段默认为False
  >  oldname(string) -- 之前的字段名称，在做数据迁移的时候orm可以自动进行重命名

- 实时计算字段

可定义一个字段，它的值通过指定函数实时计算得来，定义实时计算字段只需要指定compute属性即可，它有以下几种参数：

> compute -- 用于计算的函数名称
>  inverse -- 逆向计算函数的函数名，可选
>  search -- 实现该字段search方法的函数名
>  store -- 是否在数据库存储该字段值，默认False
>  compute_sudo -- 是否需要使用超级管理员对该字段进行重新计算

例：

```python
upper = fields.Char(compute='_compute_upper',
                    inverse='_inverse_upper',
                    search='_search_upper')

@api.depends('name')
def _compute_upper(self):
    for rec in self:
        rec.upper = rec.name.upper() if rec.name else False

def _inverse_upper(self):
    for rec in self:
        rec.name = rec.upper.lower() if rec.upper else False

def _search_upper(self, operator, value):
    if operator == 'like':
        operator = 'ilike'
    return [('name', operator, value)]
```

实时计算的方法必须用  odoo.api.depends()方法装饰以决定其依赖于哪些字段，且同一个计算方法可用于多个计算字段，只需要在方法里将对应的字段赋值就行，search方法的返回值须是一个与条件对应的domain表达式，在对模型执行搜索处理domain表达式时调用

- 关联字段
   关联字段的值是通过一系列的外键字段并通过其关联的模型读取，参数：`related`
   属性(string, help, readonly, required,groups, digits, size, translate,  sanitize, selection, comodel_name, domain,  context)只要没有被重定义会自动从源字段复制过来，默认情况下关联字段是不保存到数据库的，就像实时计算字段一样，可以通过指定store=True来指定保存

- 依赖于Company的字段
   假如一个用户属于多个公司，那么他在不同记录条件下得到的该字段值是不同的，参数`company_dependent -- boolean(默认False)`

- sparse 字段
   sparse字段一般是不为null的，大部分这类字段用于序列化存储，参数`sparse -- 该字段值的存储位置`

- 增加的定义
   子类可以重定义与父类同名同类型的字段，字段属性也会从父类继承过来并且可以被重定义

  > 例：第二个子类只为state字段添加提示

```python
class First(models.Model):
    _name = 'foo'
    state = fields.Selection([...], required=True)

class Second(models.Model):
    _inherit = 'foo'
    state = fields.Selection(help="Blah blah blah")
```

### 常用字段

- class odoo.fields.Char(string=, **kwargs) 字符串字段，可指定长度，一般在客户端以单行显示

  > 参数
  >  size (int) -- 值的最大长度
  >  translate -- 启用字段的翻译

- class odoo.fields.Boolean(string=, **kwargs) 布尔类型

- class odoo.fields.Integer(string=, **kwargs) 整型

- class odoo.fields.Float(string=, digits=, **kwargs) 浮点型，可接受digits 参数`(total, decimal)`指定位数

- class odoo.fields.Text(string=, **kwargs) Text类型，用于储存较多的内容

- class odoo.fields.Selection(selection=, string=, **kwargs)

  > 参数
  >  selection -- 指定该字段的取值列表，为(value,string)列表或一个模型的方法或方法名
  >  selection_add -- 当该字段来自重定义时，它提供selection参数的扩展，为(value,string)列表

- class odoo.fields.Html(string=, **kwargs) 储存html内容

- class odoo.fields.Date(string=, **kwargs) date类型

  > 1.static context_today(record, timestamp=None)
  >  返回客户端时区的当前日期，可以接收一个datetime格式的参数
  >  2.static from_string(value) 将ORM的值转换为date的值
  >  3.static to_string(value) 将date格式的值转换为ORM的值
  >  4.static today(*args) 以ORM值的格式返回当前日期

- class odoo.fields.Datetime(string=, **kwargs)

  > 1.static context_timestamp(record, timestamp) 一般用fields.datetime.now()替代
  >  2.static from_string(value) 将ORM格式的值转换为datetime格式
  >  3.static now(*args) 获取ORM格式的当前时间
  >  4.static to_string(value) 将datetime格式的值转换为ORM接受的格式

### 关系模型字段

- ==class odoo.fields.Many2one(comodel_name=, string=, **kwargs)==
   该字段的获取到的集合的记录数量只会是0（无记录）或1（单条记录）

  > 参数列表：
  >  comodel_name(string) -- 目标模型名称，除非是关联字段否则该参数必选
  >  domain -- 可选，用于在客户端筛选数据的domain表达式
  >  context -- 可选，用于在客户端处理时使用
  >  ondelete -- 当所引用的数据被删除时采取的操作，取值：'set null', 'restrict', 'cascade'
  >  auto_join -- 在搜索该字段时是否自动生成JOIN条件，默认False
  >  delegate -- 设置为True时可以通过当前model访问目标model的字段，与_inherits功能相同

- ==class odoo.fields.One2many(comodel_name=, inverse_name=, string=, **kwargs)==
   该字段的值是目标model的所有记录

  > 参数列表：
  >  comodel_name -- 目标模型名称，
  >  inverse_name -- 在comodel_name 中对应的Many2one字段
  >  domain -- 可选，用于在客户端筛选数据的domain表达式
  >  context -- 可选，用于在客户端处理时使用
  >  auto_join -- 在搜索该字段时是否自动生成JOIN条件，默认False
  >  limit(integer) -- 可选，在读取时限制数量

注：除非是关联字段，否则comodel_name和inverse_name是必选参数

- ==class odoo.fields.Many2many(comodel_name=, relation=, column1=, column2=, string=, **kwargs)==
   该字段的值为一个数据集合

  > 参数：
  >  comodel_name -- 目标模型名称，除非是关联字段否则该参数必选
  >  relation -- 可选，关联的model在数据库存储的表名，默认采用comodel_name获取数据
  >  column1 -- 可选，与relation表记录相关联的列名
  >  column2 -- 可选，与relation表记录相关联的列名
  >  domain -- 可选，用于在客户端筛选数据的domain表达式
  >  context -- 可选，用于在客户端处理时使用
  >  limit(integer) -- 可选，在读取时限制数量

- ==class odoo.fields.Reference(selection=, string=, **kwargs)==
   基于odoo.fields.Selection

## 继承和扩展

odoo有三种模块化的模型继承机制：

- 根据原有模型创建一个全新的模型，并基于新创建的模型修改，新模型与已存在的视图兼容，并保存在同一张表中
- 从其他模块中扩展模型，并进行替换，一般用于复制，已存在的视图会忽略新建的模型，数据保存在新的数据表中
- 通过代理访问其他模型的字段，可以同时继承多个模型，数据保存在新的数据表中，新的模型会包含一个嵌入的原模型，并且该模型数据是同步的

### 传统继承

当_inherit和_name属性一起使用时，odoo基于原有模型创建一个新模型，新的模型会自动继承原模型的字段、方法等

```python
class Inheritance0(models.Model):
    _name = 'inheritance.0'

    name = fields.Char()

    def call(self):
        return self.check("model 0")

    def check(self, s):
        return "This is {} record {}".format(s, self.name)

class Inheritance1(models.Model):
    _name = 'inheritance.1'
    _inherit = 'inheritance.0'

    def call(self):
        return self.check("model 1")
```

### 扩展

当只使用_inherit属性时，新的模型会替代已存在的模型，当需要给模型添加字段、方法、重置属性时比较有用

```python
class Extension0(models.Model):
    _name = 'extension.0'

    name = fields.Char(default="A")

class Extension1(models.Model):
    _inherit = 'extension.0'

    description = fields.Char(default="Extended")
```

### 代理

代理模式使用_inherits属性来指定一个模型当找不到指定字段时直接去对应的子模型查找

```python
class Child0(models.Model):
    _name = 'delegation.child0'

    field_0 = fields.Integer()

class Child1(models.Model):
    _name = 'delegation.child1'

    field_1 = fields.Integer()

class Delegating(models.Model):
    _name = 'delegation.parent'

    _inherits = {
        'delegation.child0': 'child0_id',
        'delegation.child1': 'child1_id',
    }

    child0_id = fields.Many2one('delegation.child0', required=True, ondelete='cascade')
    child1_id = fields.Many2one('delegation.child1', required=True, ondelete='cascade')

#use
env = self.env
record = env['delegation.parent'].create({
    'child0_id': env['delegation.child0'].create({'field_0': 0}).id,
```

## Domain表达式

domain表达式是由多个(field_name, operator, value)元组构成的列表或数组

> field_name -- 字段名或者用.号分隔的Many2one关系的字段如：'street' , 'partner_id.country'
>  operator(str) -- 用于对字段值和给定值进行比较的运算符：
>
> > =,!=,>,>=,<,<=,
> > =?(值为false或none时返回true，否则与=效果一致)
> > =like()将字段数据与value进行匹配，_代表匹配单个字符、%匹配0或多个字符
> > like() 将字段数据与%value% 进行匹配,
> > not like 不与%value%匹配
> > ilike 忽略大小写的like函数
> > not ilike 忽略大小写的not like
> > =ilike 忽略大小写的=like
> > in 与value的任意值相等，value须为值列表
> > not in 与value的任意值都不相等
> > child_of 是否由value派生而来

>value 对应值，必须与相应条件对应
>
>> 多个domain表达式可用运算符进行连接，运算符写在两个表达式之前。
>>
>> > & 逻辑与 ，| 逻辑或，！逻辑非

```python
#例：
[('name','=','ABC'),
 ('language.code','!=','en_US'),
 '|',('country_id.code','=','be'),
     ('country_id.code','=','de')]
```

## Porting from the old API to the new API

这个基本不需要用

------

译自odoo官方文档：http://www.odoo.com/documentation/10.0/reference/orm.html
