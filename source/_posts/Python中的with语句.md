---
title: Python中的with语句
date: 2016-07-11 15:45:19
tags: Python
categories: Python

---
# with语句的基本用法
高级编程语言一般都会有异常处理机制来管理资源。像C++,Java,Python等语言中都会有try/finally的机制来管理资源。我们需要保证当try语句块的内容出现异常时，一些资源能够得以释放，例如文件使用后的关闭，线程中锁的自动获取和释放等。例如常见的文件操作，
```python
#打开文件
try:
    f = open(filename)
except:
    do something
finally:
    do something
#使用文件
try:
    do something
except:
    do something
finally:
    f.close()

```
这样我们在处理一些遇到异常时，需要最后释放文件资源。我们会看到这种方式使得代码块变得臃肿，失去了一定的观赏性。另外这需要我们自己来处理每个异常。而with就是一种可以代替try/finally的优雅的解决方案。下面我们还是用打开文件的例子来演示with的基本用法。
```python
with open(filename) as f:
    do something
```
这样是不是比try/finally结构打开文件更加简洁了呢？程序的执行过程就是with后的表达式open返回了一个对象(在这里也叫作上下文管理器)，然后把对象的值赋值给f。如果打开文件有异常，那么就会退出with语句，由`文件`对象默认的异常处理机制来处理该异常。那么如果我需要同时打开多个文件呢？最开始的想法就是嵌套,
```python
with open(filename1) as f1:
    with open(filename2) as f2:
        ...

```
这显然不是一种比较好的解决方式，with提供了一种优雅的解决方案，
```python
with open(filename1) as f1,open(filename2) as f2:
    do something

```
当然利用contextlib中的nested会更加优雅，我们在下面会讲到。    

# 上下文管理器(Context Manager)
## 上下文管理器概述
上下文管理机制就是控制了一个对象的生存周期，并且对该对象进行一些异常处理。对于一个语句块而言，“上文”就是执行之前的一些语句，一般为申请资源（文件，线程等），而“下文”就是执行完该语句块后释放资源（关闭文件，线程锁的释放等），相当于finally下的语句块。下面我们看下上下文管理器的一些基本概念：
+ **上下文管理协议 (Context Management Protocol):** 包含方法`__enter__()`和`__exit__()`，支持该协议的**对象**要实现这两种方法。
+ **上下文管理器 (Context Manager):** 支持上下文管理协议的对象。需要实现`__enter__()`和`__exit__()`方法来建立运行时的上下文。通常用with来调用上下文管理器，也可以通过其他方法来使用。
+ **运行时上下文(runtime context):** 由上下文管理器创建，通过上下文管理器的`__enter__()`和`__exit__()`方法实现，`__enter__()`方法在语句体执行之前进入运行时上下文，`__exit__()`在语句体执行完后从运行时上下文退出。with句支持运行时上下文这一概念。
+ **上下文表达式(Context Expression):** with 语句中跟在关键字 with 之后的表达式，该表达式要返回一个上下文管理器对象。
+ **语句体(with-body):** with 语句包裹起来的代码块，在执行语句体之前会调用上下文管理器的`__enter__()`方法，执行完语句体之后会执行`__exit__()`方法。

除了文件之外，已经加入上下文管理协议支持的模块还有**threading，decimal**等。
## with语句的原理
with的一般格式为：
```python
with context_expression [as target(s)]:
    with-body

```
这里的as赋值是可选的。
在执行了with语句之后的具体工作原理如下：
```python
context_manager = context_expression
exit = type(context_manager).__exit__  
value = type(context_manager).__enter__(context_manager)
exc = True   # True 表示正常执行，即便有异常也忽略；False 表示重新抛出异常，需要对异常进行处理
try:
    try:
        target = value  # 如果使用了 as 子句
        with-body     # 执行 with-body
    except:
        # 执行过程中有异常发生
        exc = False
        # 如果 __exit__ 返回 True，则异常被忽略；如果返回 False，则重新抛出异常
        # 由外层代码对异常进行处理
        if not exit(context_manager, *sys.exc_info()):
            raise
finally:
    # 正常退出，或者通过 statement-body 中的 break/continue/return 语句退出
    # 或者忽略异常退出
    if exc:
        exit(context_manager, None, None, None) 
    # 缺省返回 None，None 在布尔上下文中看做是 False

```

## 自定义上下文管理器对象
下面我们定义自己的上下文管理器对象。
```python
class Demo:
    def __init__(self,tag):
        self.tag = tag
    def __enter__(self):
        print 'Allocate Resource'
    def __exit__(self,exc_type,exc_value,exc_tb):
        print '[Exit %s]: Free resource.' % self.tag
        if exc_tb is None:
            print '[Exit %s]: Exited without exception.' % self.tag
        else:
            print '[Exit %s]: Exited with exception raised.' % self.tag
            return False   # 可以省略，缺省的None也是被看做是False

```
下面是正常和异常测试
```python
with Demo('Normal'):
    print '[with-body] Run without exceptions.'

with Demo('With-Exception'):
    print '[with-body] Run with exception.'
    raise Exception
    print '[with-body] Run with exception. Failed to finish statement-body!'

```
# contextlib模块
contextlib 模块提供了3个对象：装饰器 contextmanager、函数 nested 和上下文管理器 closing。使用这些对象，可以对已有的生成器函数或者对象进行包装，加入对上下文管理协议的支持，避免了专门编写上下文管理器来支持 with 语句。

## 装饰器contextmanager
contextmanager 用于对生成器函数进行装饰，生成器函数被装饰以后，返回的是一个上下文管理器，其 __enter__() 和 __exit__() 方法由 contextmanager负责提供，而不再是之前的迭代子。被装饰的生成器函数只能产生一个值，否则会导致异常RuntimeError；产生的值会赋值给 as 子句中的 target，如果使用了 as 子句的话。下面看一个简单的例子。
```python
from contextlib import contextmanager

@contextmanager
def demo():
    print '[Allocate resources]'
    print 'Code before yield-statement executes in __enter__'
    yield '*** contextmanager demo ***'
    print 'Code after yield-statement executes in __exit__'
    print '[Free resources]'

with demo() as value:
    print 'Assigned Value: %s' % value

```

执行的结果为:
```bash
[Allocate resources]
Code before yield-statement executes in __enter__
Assigned Value: *** contextmanager demo ***
Code after yield-statement executes in __exit__
[Free resources]

```
可以看到，生成器函数中 yield 之前的语句在`__enter__()`方法中执行，yield 之后的语句在`__exit__()`中执行，而yield产生的值赋给了as子句中的value变量。
需要注意的是，contextmanager 只是省略了 `__enter__()`/ `__exit__()`的编写，但并不负责实现资源的“获取”和“清理”工作；“获取”操作需要定义在yield语句之前，“清理”操作需要定义yield语句之后，这样 with 语句在执行`__enter__()`/`__exit__()`方法时会执行这些语句以获取/释放资源，即生成器函数中需要实现必要的逻辑控制，包括资源访问出现错误时抛出适当的异常。

## nested
nested 可以将多个上下文管理器组织在一起，避免使用嵌套 with 语句。我们用nested实现上述提到过的打开多个文件：
```python
from contextlib import nested

with nested(open(filename1),open(filename2)) as (f1,f2):
    do something

```

而nested的底层实现就是嵌套的with语句，发生异常后，如果某个上下文管理器的`__exit__()`方法对异常处理返回False，则更外层的上下文管理器不会监测到异常。

## 上下文管理器 closing
closing 适用于提供了 close() 实现的对象，比如网络连接、数据库连接等，也可以在自定义类时通过接口close()来执行所需要的资源“清理”工作。其实现如下：

```python
class closing(object):
    # help doc here
    def __init__(self, thing):
        self.thing = thing
    def __enter__(self):
        return self.thing
    def __exit__(self, *exc_info):
        self.thing.close()

```
上下文管理器会将包装的对象赋值给as子句的target变量，同时保证打开的对象在with-body执行完后会关闭掉。closing上下文管理器包装起来的对象必须提供 close() 方法的定义，否则执行时会报 AttributeError 错误。
当然我们也能自定义支持closing的对象：

```python
class ClosingDemo(object):
    def __init__(self):
        self.acquire()
    def acquire(self):
        print 'Acquire resources.'
    def free(self):
        print 'Clean up any resources acquired.'
    def close(self):
        self.free()

with closing(ClosingDemo()):
    print 'Using resources'

```
# 参考文献

+ [浅谈Python的with语句](https://www.ibm.com/developerworks/cn/opensource/os-cn-pythonwith/#ibm-pcon)
+ [Python中使用With打开多个文件](http://www.hustyx.com/python/119/)
+ [理解Python中的with…as…语法](http://zhoutall.com/archives/325)