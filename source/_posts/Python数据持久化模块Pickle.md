---
title: Python数据持久化模块Pickle
date: 2016-08-11 10:50:30
tags: Python
categories: Python

---

一般在程序的运行中会产生一些我们想要的中间结果，保留这些中间结果能够让我们节省计算或者为其他模块服务。因此，保存程序运行中的一些数据结构是一个常见的需求。当然，我们可以自己定义一些格式来存储这些数据结构，当我们需要恢复数据时候重新构造这些数据结构，然后把数据逐条加进去。但是，显然这不是一种优雅的方式，尤其是对象比较复杂或者比较大时。python给我们提供了一个方便的序列化和反序列化的模块——Pickle。  

pickle是python的内置库，它还有一个用C实现的版本cPickle，C版本速度上会快很多，因此我们一般会选择使用cPickle。cPickle把数据序列化的函数为：  

`cPickle.dump(obj, file[, protocol])`  

这行代码就把我们的对象`obj`保存到文件file中。其中`obj`是对象名，`file`则是文件流，`protocol`是文件的读写方式。如果为了压缩数据protocol可以加上`r`。下面是一个简单的用法示例：

```python
#!/usr/bin/env python
#coding:utf8

import cPickle as pkl

if __name__=='__main__':

    source_data = [str(i) for i in range(1000)]

    pkl.dump(source_data,open('data.dat','wb'))

    recover_data = pkl.load(open('data.dat','rb'))

    print ' '.join(recover_data)

```
