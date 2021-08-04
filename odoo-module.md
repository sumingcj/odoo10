## Manifest 显现 

清单文件用于将 python 包声明为 Odoo 模块 并指定模块元数据。 

这是一个名为 `__manifest__.py`并包含一个 Python 字典，其中每个键指定模块元数据。 

```
{ 
    'name' :  "A Module" , 
    'version' :  '1.0' , 
    'depends' :  [ 'base' ], 
    'author' :  "Author Name" , 
    'category' :  'Category' , 
    'description' :  """ 
    Description text
    """ , 
    # 数据文件总是在安装时加载 
    'data' :  [ 
        'views/mymodule_view.xml' , 
    ], 
    # 包含可选加载的演示数据和数据文件 
    'demo' :  [ 
        'demo/demo_data.xml' , 
    ], 
} 
```

可用的清单字段有： 

- `name` ( `str`， **required**） 

  模块的可读名称 

- `version` ( `str`) 

  此模块的版本，应遵循 [语义版本控制 ](http://semver.org)规则 

- `description` ( `str`) 

  模块的扩展描述，在 reStructuredText 中 

- `author` ( `str`) 

  模块作者的名字 

- `website` ( `str`) 

  模块作者的网站 URL 

- `license` ( `str`, **defaults**： `LGPL-3`) 

  模块的分发许可证 

- `category` ( `str`， **defaults**： `Uncategorized`) 

  Odoo 中的分类类别，模块的粗略业务领域。 尽管 使用 [现有类别 ](https://github.com/odoo/odoo/blob/master/odoo/addons/base/module/module_data.xml)建议 ，但该字段是  自由形式和未知类别是即时创建的。 类别  可以使用分隔符创建层次结构 `/`例如 `Foo / Bar` 将创建一个类别 `Foo`，一个类别 `Bar`作为孩子的类别 `Foo`，并将设置 `Bar`作为模块的类别。 

- `depends` ( `list(str)`) 

  必须在此之前加载的 Odoo 模块，要么是因为这 模块使用他们创建的功能或因为它改变了他们的资源 定义。 当一个模块被安装时，它的所有依赖项都会在之前安装 它。  同样，在加载模块之前加载依赖项。 

- `data` ( `list(str)`) 

  必须始终安装或更新的数据文件列表 模块。  模块根目录中的路径列表 

- `demo` ( `list(str)`) 

  仅在 安装或更新的数据文件列表 *演示中  模式* 

- `auto_install` ( `bool`，  **defaults**： `False`) 

  如果 `True`, 如果所有模块都安装 已安装依赖项。 一般用于实现协同集成的“链接模块” 在两个其他独立的模块之间。 例如 `sale_crm`取决于两者 `sale`和 `crm`并被设置 到 `auto_install`.  当两个 `sale`和 `crm`已安装，它 自动将 CRM 活动跟踪添加到销售订单，而无需 `sale`要么 `crm`彼此了解 

- `external_dependencies` ( `dict(key=list(str))`) 

  包含 python 和/或二进制依赖项的字典。 对于 python 依赖项， `python`必须为此定义键 应分配字典和要导入的 Python 模块列表 到它。 对于二进制依赖， `bin`必须为此定义键 字典和二进制可执行文件名称列表应该分配给它。 如果未安装 python 模块，则不会安装该模块 在主机或二进制可执行文件中找不到 主机的 PATH 环境变量。 

- `application` ( `bool`，  **defaults**： `False`) 

  该模块是否应被视为成熟的应用程序 ( `True`) 或者只是一个技术模块 ( `False`) 提供了一些 现有应用程序模块的额外功能。 

- `css` ( `list(str)`) 

  指定要导入的自定义规则的 css 文件，这些文件应该是 位于 `static/src/css`模块内部。 

- `images` ( `list(str)`) 

  指定模块要使用的图像文件。 

- `installable` ( `bool` **defaults**： `False`) 

  用户是否应该能够从 Web UI 安装模块。 

- `maintainer` ( `str`) 

  负责维护此模块的个人或实体，默认情况下 假设作者是维护者。 

- `{pre_init, post_init, uninstall}_hook` ( `str`) 

  用于模块安装/卸载的钩子，它们的值应该是 表示模块内部定义的函数名称的字符串 `__init__.py`. 

  - `pre_init_hook`以游标为唯一参数，这个函数是 在模块安装之前执行。 
  - `post_init_hook`将一个游标和一个注册表作为它的参数，这个 功能在模块安装后立即执行。 
  - `uninstall_hook`将一个游标和一个注册表作为它的参数，这个 功能在模块卸载后执行。 

​        这些钩子应该只在这个模块需要设置/清理时使用 通过 api 要么非常困难，要么不可能。 

