1.odoo 页面加载失败

```shell
Traceback (most recent call last):
  File "/smb/odoo10/odoo/addons/base/ir/ir_attachment.py", line 100, in _file_re                                                                                                                                                             ad
    r = open(full_path,'rb').read().encode('base64')
IOError: [Errno 2] No such file or directory: u'/root/.local/share/Odoo/filestor                                                                                                                                                             e/beginner/21/21cdf7b6948b5e79d8ffb9977dea2dae5d1ae8fa'
```

到 `ir_attachment` 删除对应的`21/21cdf7b6948b5e79d8ffb9977dea2dae5d1ae8fa`

