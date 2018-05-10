---
title: Python优化经验
date: 2017-04-26 16:58:03
tags:

---

Python真的是一不注意就可能导致速度慢了好多倍，这是很夸张的，所以写下一些自己的经验总结一下。

+ 判断key是否在字典dict中直接用`for key in dict`，不要使用`for key in dict.keys()`
+ 需要动态生成array的时候不要用numpy.row_stack()动态生成，先用列表append添加，最后再用列表初始化array即可
+ 尽量使用python的c版本的函数，比如cPickle等
+ 能用列表解析的不要用循环