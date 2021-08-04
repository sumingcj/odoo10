## 命令行方式安装新模块

```jsx
~/odoo-dev/odoo/odoo-bin -d 数据库名 -i 模块名
```

## 命令行方式更新模块

```jsx
~/odoo-dev/odoo/odoo-bin -d 数据库名 -u 模块名
```



Odoo14模块视图不显示问题总结

    问题一： 安装界面不出现模块
    问题二：模块安装成功，但是在主页上不显示
    问题三：模块中的二级、三级导航菜单不显示

问题一： 安装界面不出现模块

http://127.0.0.1:8069/web?debug=1进入debug模式，点击上面的【更新】和【更新应用列表】

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210216215130651.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2RvcmxvbG8=,size_16,color_FFFFFF,t_70)

确保配置文件中包含了模块所处的目录，注意多个路径之间以逗号分隔。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210216214054358.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2RvcmxvbG8=,size_16,color_FFFFFF,t_70)

检查启动参数，检查配置文件名称是否正确。再重启Odoo服务。（掉进这个坑的可不止一个人）

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210216215342481.png)


最后，如果还是不行，尝试修改配置文件里的http_port端口，重启Odoo服务后，使用新的端口进行访问再试（原因可能访问的是另一个Odoo服务，这个坑我跳了两次，所以最好是新建Odoo项目的时候就把端口给改了）。

问题二：模块安装成功，但是在主页上不显示

注意：如果您按照下面任意一个步骤做了修改，记得重启服务和升级模块再检查是否恢复。

在模块的__manifest__.py文件中，
检查是否有'application:True'这段参数，它保证了这个模块是否能够以app形式在页面显示。
另外，data参数中xml和csv文件的路径和名称确保正确。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210216222049240.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2RvcmxvbG8=,size_16,color_FFFFFF,t_70)

检查views目录中的xml文件。是否包含顶层的menuitem元素。它保证了在主页上能够显示模块的名称和图标。此外，其中的sequence值尝试改大一些（若与同级menuitem的sequence属性值一样，可能会产生冲突导致不显示）。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210216221009785.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2RvcmxvbG8=,size_16,color_FFFFFF,t_70)

如果模块的models目录中，已经创建了py文件。那么需要再创建secruity目录，里面再创建子文件ir.model.access.csv，用于定义权限规则。不设置会导致图标不可见（如果models目录为空则不影响——严格来说是里面没有定义任何对象就不受影响）。

最主要的是这几个字段

- 最最重要的C3列。如果对象的属性_name="dtcloud.carweight"，那么应当按照下图最后一行的方式命令；
- C4到C6列在测试阶段按照下图内容照抄就行；
- ps：C1、C2随意起名。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210219021429853.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2RvcmxvbG8=,size_16,color_FFFFFF,t_70)

给顶层的menuitem元素添加上action属性。（我好几次都是这样解决的，可以正常来讲，顶层的menuitem不需要加action属性，而且其它模块不加也正常。）

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210217025214772.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2RvcmxvbG8=,size_16,color_FFFFFF,t_70)

卸载并！删除 ！模块，重启服务后重新安装。（目前解决过两次）

删除数据库重新建一个同名的，然后重启服务再次安装。或者直接另建一个模块从头开始写。（如果新建数据库名字改了，记得改配置文件和启动参数！！！）

备注：删数据库的主要目的不是为了解决问题，而是为了验证odoo这个框架稳定性（因为代码基本查不出哪里有问题，只能怀疑odoo本身了）。在重新创建数据库后，odoo会重新进行初始化，如果还有问题，安心重写代码反而效率更高。

# 问题三：模块中的二级、三级导航菜单不显示

1. 同问题二第4步，确保同级菜单`sequence`值没有重复。如果怕出问题，所有`sequence`的值都设置成不一样的就行。
2. 二、三级菜单额外增加了parent和action属性。确保`parent`值指向正确的父级菜单id；

![parent和action属性](https://img-blog.csdnimg.cn/20210219015813739.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2RvcmxvbG8=,size_16,color_FFFFFF,t_70)



