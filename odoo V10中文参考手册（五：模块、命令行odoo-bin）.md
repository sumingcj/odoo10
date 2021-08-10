# Manifest

manifest文件用于将python包定义成odoo的模块，并且声明模块的元数据，文件名为__ manifest __.py并包含一个python格式的数据字典

```python
{
    'name': "A Module",
    'version': '1.0',
    'depends': ['base'],
    'author': "Author Name",
    'category': 'Category',
    'description': """
    Description text
    """,
    # data files always loaded at installation
    'data': [
        'mymodule_view.xml',
    ],
    # data files containing optionally loaded demonstration data
    'demo': [
        'demo_data.xml',
    ],
}
```

可用的manifest字段：

### name (str, required)

模块名称

### version (str)

模块的版本号

### description (str)

模块的描述

### author (str)

模块作者

### website (str)

模块作者的网站地址

### license (str, defaults: LGPL-3)

模块遵循的发布协议，默认是LGPL-3

### category (str, default: Uncategorized)

在odoo中的分类类目，官方建议使用已存在的分类，但也可以通过该字段取值实时创建，并可用斜杠来划分等级如Foo/Bar创建一个Foo的分类，再在其下创建一个bar的分类，最后会设置给该模块bar分类

### depends (list(str))

声明哪些模块需要在加载本模块之间加载，因为这个模块用到了其他模块里的内容。当安装一个模块时，它所依赖的模块会自动全部安装上去。

### data (list(str))

每次加载模块时都需要加载的数据文件列表，每个文件名是相对模块目录的路径

### demo (list(str))

只在演示模式下才加载的数据文件

### auto_install (bool, default: False)

是否自动安装，默认是False，设置为True时，只要所依赖的模块安装了该模块会自动安装。一般用于协作模块如：sale_crm 依赖于sale和crm

> odoo系统已存在的分类列表：[https://github.com/odoo/odoo/blob/master/odoo/addons/base/module/module_data.xml](https://link.jianshu.com?t=https://github.com/odoo/odoo/blob/master/odoo/addons/base/module/module_data.xml)

# 运行

- -d `database`, --database `database` 指定安装或更新模块时使用的数据库
- -i `modules`, --init `modules` 以逗号分隔的需要在服务端运行之前安装的模块列表，需指定-d参数
- -u `modules`, --update `modules` 在服务端运行之前需要更新的模块列表，以逗号分隔
- --addons-path `directories` 模块储存的文件目录，以逗号分隔，运行时会扫描这些指定的目录
- --workers `count` 默认0，如果设置会开启多进程并运行指定数量的worker（用于处理http和rpc请求）

> 下面是一系列用于限制和回收worker的选项

- --limit-request `limit` 每个worker在生命周期内可处理的最多请求数，默认8196
- --limit-memory-soft `limit` 每个worker允许占用的最大内存，如果内存占用超过了，在当前请求完成后会被关闭并回收，默认640M
- --limit-memory-hard   `limit` 硬性的内存约束，一旦超过直接关闭不等请求完成，默认768M
- --limit-time-cpu `limit` 限制worker处理每个请求所占用的cpu秒数，超过则直接关闭，默认60
- --limit-time-real `limit` 限制workder处理请求所使用的时间，超过直接关闭，该时间包含数据库查询等时间在内，默认120
- --max-cron-threads `count` 专注于计划任务的worker数量，默认是2，在多线程下，worker是线程；在多进程下worker是进程，且这个是与http进程数分开的
- -c `config`, --config `config` 提供一下可选的配置文件
- -s, --save 保存服务器配置文件到当前配置文件，默认是HOME/.odoorc,当指定-c参数时是对应文件
- --proxy-mode 开启X-Forwarded-*头
- --test-enable 当安装模块后运行测试
- --dev `feature,feature,...,feature`

> 参数：

- all - 所有指定特性被激活
- xml - 直接从xml文件读取qweb模板，一旦模板在数据库中有修改，它就不会从xml中读取了，直接下次更新模块或重初始化
- reload - 当python文件有修改时自动重启服务（在用文本编辑器时可能没用。。。）
- qweb - 在解析qweb模板时，如果一个节点有`t-debug='debugger'`，暂停解析
- (i)p(u)db 开启指定的python调试器，当碰到错误时直接返回

## 数据库参数

- -r `<user>`, --db_user `<user>` 数据库用户名，用于连接postgresql数据库
- -w `<password>`, --db_password `<password>` 当使用密码验证模式时的数据库密码
- --db_host `<hostname>` 数据库的主机地址
- --db_port `<port>` 数据库监听的端口，默认5432
- --db-filter `<filter>` 隐藏不满足指定条件的数据库，filter参数是一个正则表达式，%h表示被请求的主机名，%d表示被请求的子域名([www.test.com](https://link.jianshu.com?t=http://www.test.com)和[test.com](https://link.jianshu.com?t=http://test.com)都会匹配到test数据库)
- --db-template `<template>` 当在数据库管理界面创建数据库时，使用指定的模板数据库，默认是template1

## 内置HTTP

- --no-xmlrpc 不开启http和长连接worker，但可能还是会开启计划任务worker
- --xmlrpc-interface `<interface>` HTTP服务器所监听的IP地址，默认是0.0.0.0
- --xmlrpc-port `<port>` HTTP服务器所监听的端口，默认8069
- --longpolling-port `<port>` 在多进程或gevent模式下长连接的端口，默认是8072，在默认多线程模式下不可用

## 日志

默认情况下odoo记录info级别的日志（工作流只记录warning)，且日志直接输出，可以用选项来指定日志的记录方式

- --logfile `<file>` 将日志输出保存到指定文件。在unix类系统中文件可以用其他的循环日志管理程序处理，当文件内容被替换时会自动重新打开
- --logrotate 启用每天循环日志记录，保存30份备份，它的循环频率和备份数量不能修改
- --syslog 记录到系统日志中
- --log-db `<dbname>` 使用ir.logging模型(ir_logging表)记录到指定数据库中
- --log-handler `<handler-spec>` LOGGER:LEVEL - 指定日志记录级别，LOGGER省略时使用默认的handler，LEVEL省略时会自动用INFO,可以重复handler来指定多个
   `odoo-bin --log-handler :DEBUG --log-handler werkzeug:CRITICAL --log-handler odoo.fields:WARNING`
- --log-request 开启RPC请求的DEBUG日志，相当于--log-handler=odoo.http.rpc.request:DEBUG
- --log-response 开启RPC输出的DEBUG日志，相当于--log-handler=odoo.http.rpc.response:DEBUG
- --log-web 开启HTTP请求和输出的DEBUG日志，相当于--log-handler=odoo.http:DEBUG
- --log-sql 开启数据库查询的DEBUG日志，相当于--log-handler=odoo.sql_db:DEBUG
- --log-level `<level>` 设置多个logger级别的简便方式，一次将critical, error, warn, debug设置到odoo和werkzeug logger上

> odoo还为多种logger提供debug模式：
>  1.debug_sql 将sql 记录级别设置为debug，相当于--log-sql
>  2.debug_rpc 将odoo和http请求设置为debug级别，相当于--log-level debug --log-request
>  3.debug_rpc_answer 将odoo和http请求及输出记录级别设置为debug，相当于--log-level debug --log-request --log-response

# 脚手架

脚手架用于创建模块的基本结构，通过odoo-bin scaffold 命令加下面的参数来执行：

- -t `<template>` 模板文件夹，经过jinja2解释并复制到目标文件夹
- name 需要创建的模块名，会根据这个名字去自动生成模块文件夹名之类的
- destination 模块创建后所存放的文件夹，默认当前执行环境的文件夹

# 配置文件

大部分命令行选项可以通过一个配置文件来指定，大部分只需将选项名前的-移除，并将其他的-替换为_，如（--db-template=>db_template)
 某些转换比较特别：

- --db-filter => dbfilter
- --no-xmlrpc => xmlrpc(boolean)
- 预先通过--log-xxx选项设置的记录直接被添加到log_handler，可直接在配置文件中设置
- --smtp => smtp_server
- --database => db_name
- --debug => debug_mode (boolean)
- --i18n-import,--i18n-export 在配置文件中无效

默认的配置文件是$HOME/.odoorc，可用--config指定，并可用--save 来保存当前配置