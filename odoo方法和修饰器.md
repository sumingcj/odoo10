# 方法和修饰器

api 是命名修饰器，识别 cr,cursor, uid, user, user_id, id, ids, context

# @api.returns
返回指定模型的记录集

```python
@api.returns('res.partner')
def afun(self):
...
return x # a RecordSet
```

这样就返回合作伙伴记录集

# @api.one
返回当前记录，指明下面 self 是一条记录

```python
@api.one
def afun(self):
    self.name = 'toto'
```

# @api.multi
返回记录集，指明下面的 self 是记录集

```python
@api.multi
def afun(self):
    len(self)
```

# @api.model
保证兼容版本，指明下面的 self 是模型对象

```python
@api.model
def afun(self):
    pass
```

# @api.constrains
保证关系时的约束
# @api.depends
给定好依赖,表示依赖字段值变化，本字段值
也跟着变化

```python
@api.depends('name', 'an_other_field')
def afun(self):
    pass
```

# @api.onchange
监控字段的变化，然后操作响应

```python
@api.onchange('fieldx')
def do_stuff(self):
    if self.fieldx == x:
        self.fieldy = 'toto'
```





