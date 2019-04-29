---
title: python脚本引用同级文件夹中的方法 
categories: ["python","问题整理"]
tags: ["python"]
date: 2019-04-29 23:01:41 
author: gravel

---

目录结构为：

```
├── dataprocess                                                    
│   ├── config
│   ├── ├──dbconfig.py                // 具体代码.                    
│   ├── test.py
```

需要在`test.py`脚本中引用`dbconfig.py`中的方法，直接引用相对路径，会提示找不到对应的`module`。`python`判断一个目录是否为`Module`主要是通过 `__init__.py`，如果要在让`python`识别到这个目录需要通过系统的环境变量 `sys.path`。

所以解决方法是，在`test.py`引用`dbconfig.py`的方法时，将`config`静态目录加入到文件中。

```
import sys
currentUrl = os.path.dirname(__file__)
sys.path.append(currentUrl)
from config import getdb
getdb()
```



大功告成！