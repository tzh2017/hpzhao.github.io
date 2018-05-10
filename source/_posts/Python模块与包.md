---
title: Python模块与包
date: 2016-07-24 15:52:49
tags: Python
categories: Python

---

# 包和模块概述

## 什么是包和模块？

从文件系统的角度而言，一个包就是一个文件夹，而一个模块就是一个文件。一个包可以包含多个模块，而一个模块里面包含了类，函数，变量。包和模块就像其他语言的库一样，目的是提供一个可复用性，低耦合，具备特定功能的代码。python中的包和模块的导入机制是整个语言中最复杂的部分，我们在本文中只探讨包和模块的管理和使用，具体的机制可以阅读[PEP 302](https://www.python.org/dev/peps/pep-0302/)。我们可以使用发行的包，也可以使用自己定义的包。在自己定义包时，每个目录下要含有一个`__init__.py`文件，可以保持这个文件为空，也可以在里面自定义导入规则。

## 模块的属性
模块的属性是模块的一些信息参数。
### 固有属性
固有属性是一个模块被创建就会自带的默认属性。我们以下面的例子来说明：

```bash
$ touch mod.py
$ python
>>> import mod # 导入模块mod
>>> mod.__dict__ # 模块mod的属性全貌
{'__builtins__': {...}, '__name__': 'mod', '__file__': 'mod.pyc', '__doc__': None, '__package__': None} 
>>> dir(mod) # 只查看属性名
['__builtins__', '__doc__', '__file__', '__name__', '__package__']

```
+ `__builtins__`内建名字空间（参考 名字空间）
+ `__file__`文件名（对于被导入的模块，文件名为绝对路径格式；对于直接执行的模块，文件名为相对路径格式）
+ `__name__`模块名（对于被导入的模块，模块名为去掉“路径前缀”和“.pyc后缀”后的文件名，即`os.path.splitext(os.path.basename(__file__))[0]`；对于直接执行的模块，模块名为`__main__`）
+ `__doc__`文档字符串（即模块中在所有语句之前第一个未赋值的字符串）
+ `__package__`包名（主要用于相对导入）
### 新增属性
新增属性是指我们自己创建的属性，其中包含：
+ `import`导入的模块
+ 变量
+ 函数
+ 类
### 导出属性
共有属性指的是我们在导入模块时会导入的属性。这是有一个变量`__all__`来管理的。如果没有这个变量，那么默认导入所有共有属性(即不以下划线开头的)。我们也可以自己定义要导出哪些属性。
```python
__all__ = [...]

```
# 包和模块的管理
首先我们需要了解python中的包都在什么位置呢？这分为以下几种情况：
+ sudo apt-get install 安装的package存放在`/usr/lib/python2.7/dist-packages`目录中
+ pip 或者 easy_install默认安装的package存放在`/usr/local/lib/python2.7/dist-packages`目录中
+ 手动从源代码或者`pip install --user`安装的package存放在`./local/lib/python2.7/site-packages`目录中

一般来说，自己手动安装的python包的优先级比系统安装的优先级高，这就意味着如果一个包安装了而不同版本，那么优先手动安装的版本。如果从源码安装包，那么安装的位置也要选择`./local/lib/python2.7/site-packages`中便于包的管理。当然你也可以安装到你喜欢的位置，但是要把包的路径添加到python的库搜索路径中，方法会在下面提到。

# 包和模块的导入

## 包和模块的导入风格

python提供了丰富的包的导入方式，但是有一些导入方式并不值得推荐。例如`from math import *`，这种全盘导入的风格我们是不推荐的，因为在math中可能会定义和你文件中同名的变量，函数，类等，而且这种导入会使得加载更多的模块，从而降低了效率。养成良好统一的编码风格非常重要，虽然语言提供了很多奇技淫巧，但是我们要**选择那些好的语言特性，而不是糟糕的特性**，下面是Google推荐的python代码风格：

+ 使用`import x`来导入包和模块
+ 使用`from x import y`来导入模块.其中x是包名，y是不含包名的模块名
+ 使用`from x import y as z`, 如果两个要导入的模块都叫做y或者y太长了
+ 使用`from x.y import z`,导入模块z要用包的全路径，不要嵌套，import之后的要保证只是模块，不含包名

## 通过相对路径导入包和模块

使用相对路径导入模块：

+ 要导入的模块位于同级目录，我们可以直接导入包`import x`
+ 要导入的模块位于同级目录下的子目录，我们可以`from A import x`或`from .A import x`
+ 要导入的模块位于上级目录，我们可以`from ..A import x`
+ 要导入的模块位于其他情况时，我们不推荐使用相对目录导入，而是采用即将介绍的方式导入

## 通过sys.path导入包
当我们想利用的包在工程的其他目录时，我们需要将该路径添加到系统的搜索路径中，有两种方式：
第一种方式是将包的绝对路径添加到系统的搜索路径中，只需调用`sys.path.insert(pos,dir)`或者`sys.path.append(dir)`,这种方式添加是硬链接到你的代码，只在程序运行时有效，但是这种方式在代码部署时需要手工改动代码中的所有路径，不推荐这种方式。

第二种方式就是将该路径添加到python的默认搜索路径中，有两种方式可以实现。

+ 在系统变量`PYTHONPATH`中添加该路径
+ 在`.local/lib/pyhon2.7/site-packages/`下的.pth下添加该目录

我们推荐第二种方式，因为这样在代码部署时我们只需要改动路径的配置文件即可。

## 通过字符串导入包
如果包名在字符串里面，我们可以采用importlib.import_module()函数来导入包，如下例所示：

```python
import importlib
math = importlib.import_module('math')
print math.sin(2)

```
## 模块的重新加载
模块在第一次被加载时被编译，但如果你修改了模块的代码，那么python默认不会重新编译模块，因为这是个代价较大的操作。我们可以在调试模式下用`reload()`函数重新加载模块，但是在工作环境下不要这样做，仅限于调试模式。
```python
import imp
imp.reload(spam)

```
# 包的分发
现在你已经写好自己的包了，你想分享给别人用，那么下面就是如何把包进行封装。加入你有如下的工程目录：

```txt
projectname/
    README.txt
    Doc/
        documentation.txt
    projectname/
        __init__.py
        foo.py
        bar.py
        utils/
            __init__.py
            spam.py
            grok.py
    examples/
        helloworld.py
        ...

```
首先你要编写一个`setup.py`，类似下面这样：
```python
# setup.py
from distutils.core import setup

setup(name='projectname',
    version='1.0',
    author='Your Name',
    author_email='you@youraddress.com',
    url='http://www.you.com/projectname',
    packages=['projectname', 'projectname.utils'],
)

```
下一步，就是创建一个`MANIFEST.in`文件，列出所有在你的包中需要包含进来的非源码文件：
```txt
# MANIFEST.in
include *.txt
recursive-include examples *
recursive-include Doc *
```
确保`setup.py`和`MANIFEST.in`文件放在你的包的最顶级目录中。一旦你已经做了这些，你就可以像下面这样执行命令来创建一个源码分发包了：

```bash
% python setup.py sdist

```
它会创建一个文件比如`projectname-1.0.zip`或`projectname-1.0.tar.gz`,具体依赖于你的系统平台。如果一切正常，这个文件就可以发送给别人使用或者上传至[Python Package Index](http://pypi.python.org/).

# 参考文献

+ [Python Cookbook](http://python3-cookbook.readthedocs.io/zh_CN/latest/chapters/p10_modules_and_packages.html)
+ [Google开源项目风格指南](http://zh-google-styleguide.readthedocs.io/en/latest/google-python-styleguide/contents/)
+ [python的dist-packages目录和site-packages目录的区别](http://blog.sina.com.cn/s/blog_4ddef8f80102v57b.html)
+ [Python基础:模块](http://www.cnblogs.com/russellluo/p/3328683.html#2_2)