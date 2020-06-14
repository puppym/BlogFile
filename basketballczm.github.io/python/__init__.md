## __init__.py  
__init__.py 文件的作用是将文件夹变为一个Python package,Python 中的每个package的包中，都有__init__.py 文件。该文件夹下的每一个.py文件都是一个python的module。

当前遇到的一个问题是，一个kubeshell文件夹下面有__init__.py。然后该文件下运行main.py，别的.py文件进行相互kubeshell.style 导入导出style模块就报错，但是将main.py文件提出到kubeshell文件夹之外就不会报错。

```
    |-- kubeshell
        |---__init__.py
        |---1.py 
        |---2.py 
        |---main.py 
```
在1.py中导入from kubeshell.2 import style，这样运行main.py 的时候会报错。 

正确的写法 from .2 import style  这样其实也就是把2中的所有内容导入到当前文件当中
from . import 2     这样不能导入名字的符号表，只能导入执行的代码， 在__init__.py中可以填入from . import 2 导入的代码是模块初始化化的代码。

```
    |-- kubeshell
        |---__init__.py
        |---1.py 
        |---2.py 
    |---main.py 
```
这样改目录结构之后，执行main.py 就不会报错模块找不到。 
在这种情况下，main.py在导入包kubeshell时，首先会导入__init__.py中的内容，例如在日志管理的流程中，需要先在__init__.py中导入from . import logger。相当于日志模块的初始化在main.py的开头就被执行(记一次报错)，其实也可以在main.py中from kueshell import logger

## python导入模块的顺序 
1. 当前目录模块 
2. 系统目录模块 

## 参考文档
* https://python3-cookbook.readthedocs.io/zh_CN/latest/c10/p03_import_submodules_by_relative_names.html
