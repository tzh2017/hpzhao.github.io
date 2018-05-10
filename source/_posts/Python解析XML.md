---
title: Python解析XML
date: 2016-07-11 14:06:57
tags: [Python,XML]
categories: Python

---
# 方法概述
XML是一种常见的结构化的标记语言。解析XML的方法有很多，就python2.7而言就自带了四种解析XML的方法。在Python27/Lib/xml目录下我们可以看到这四种内置的库：*dom,etree,parsers,sax*。dom(*document object model*)解析XML的方法是先把数据在内存中解析成一个树，然后对树进行操作XML；etree(*element tree*)等价于一个轻量级的dom，也是先把XML数据在内存中解析成一棵树，具有方便友好的API，速度快，消耗内存小；parsers提供对C语言编写的expat解析器的一个直接的，底层的API接口，与前两者不同，这种方法并不是一次性把数据加载到内存中，而是基于事件执行解析的，需要用户自己定义回调函数来解析xml，但这个接口并不是标准化的；最后一种sax和parsers一样也是基于事件触发的，需要用户自己定义回调函数。一般第二种etree的解析方式是比较流行的，因为速度较快而且接口非常方便友好，除非在特殊的应用场景下(内存等因素)，一般我们选择比较简单的etree作为我们的工具，本文也是关于etree的常用方法的介绍。
# 解析XML

## 调用标准库
在python2.7中的XML路径下提供了两种etree：一种是ElementTree，一种是cElementTree。加了"c"说明这是基于c语言优化的版本，效率比原始的ElementTree要高(主要是指解析过程)，因此我们优先考虑cElementTree。但由于比较低版本的python中并没有cElementTree，因此我们可以按如下方式调用库
```python
try
    import xml.etree.cElementTree as ET
except ImportError  
    import xml.etree.ElementTree as ET

```
## 解析XML
获取一棵XML树的方法有三种：**解析XML文件**，**解析XML字符串**，**新建解析树**。下面的代码展示了如何解析获得一棵XML树。
### 解析XML文件
```python
tree = ET.parse(filename)
root = tree.getroot()

```
这种方法从文件中读取XML，并在内存中解析为ElementTree类型，然后通过getroot()的方法获取该树的根节点。
### 解析XML字符串
```python
# parse from a string
root = ET.fromstring(str)

```
该方法从字符串里面获得解析树的根节点，注意与从文件中解析不同，**从字符串解析获得的是Element类型，而从文件中解析获取的是ElementTree类型。**
### 新建解析树
```python
root = ET.Element('root')
tree = ET.ElementTree(root)
```
新建一棵树首先确定一个根节点，然后把根节点放到新的解析树。

# 查找XML
在查找XML之前我们先了解一下XML的结构。ElementTree是把XML解析成嵌套的Element元素。因此我们要了解一下一个Element元素都包含哪些内容
+ 标签(tag)。这是一个说明Element存储的数据类型的字符串
+ 属性(attributes)。这是一个节点Element的属性
+ 文本(text)。这是节点Element存储的内容(字符串)
+ 可选的尾字符(tail string)。
+ 子节点(child elements)。  

我们用官方文档的例子来说明：
```xml
<?xml version="1.0"?>
<data>
    <country name="Liechtenstein">
        <rank>1</rank>
        <year>2008</year>
        <gdppc>141100</gdppc>
        <neighbor name="Austria" direction="E"/>
        <neighbor name="Switzerland" direction="W"/>
    </country>
    <country name="Singapore">
        <rank>4</rank>
        <year>2011</year>
        <gdppc>59900</gdppc>
        <neighbor name="Malaysia" direction="N"/>
    </country>
    <country name="Panama">
        <rank>68</rank>
        <year>2011</year>
        <gdppc>13600</gdppc>
        <neighbor name="Costa Rica" direction="W"/>
        <neighbor name="Colombia" direction="E"/>
    </country>
</data>

```
## 查找节点内容
查找节点值就是查找该节点下存储的值，方法很简单。
```python
#直接查找
time = year.text()
#根据节点名查找,返回第一个匹配的节点的内容
time = findtext('year')
#迭代查找,返回所有子节点的text内容
for year in country.itertext():
    print year

```
## 查找属性

第一种需求是查找某个属性名的值，返回的是字符串类型：
```python
name = country.get('name')

```
第二种需求就是获取该节点的全部属性：
```python
#第一种方法返回的是字典
attribs_dict = country.attrib
name = attribs_dict['name']
#第二种方法返回的是列表，元素是key-value组成的tuple
attribs_list = country.items()
#第三种方法返回属性名的列表
keys = country.keys()

```
## 查找节点
### 根据子节点名查找
根据子节点名查找有两种实现方式，第一种是利用find一次性查找，第二种是利用iter迭代查找。
```python
#find查找第一个匹配的子节点
first_child = root.find('country')
ET.dump(first_child)
#findall查找所有匹配的子节点
children = root.findall('country')
for child in children:
    ET.dump(child)
#迭代查找所有匹配的子节点
for country in root.iterfind('country'):
    ET.dump(country)

```
### 根据XPath查找
ElementTree还支持有限的[XPath](https://www.w3.org/TR/xpath)表达式。下面是一些例子。
```python
# Top-level elements
root.findall(".")

# All 'neighbor' grand-children of 'country' children of the top-level elements
root.findall("./country/neighbor")

# Nodes with name='Singapore' that have a 'year' child
root.findall(".//year/..[@name='Singapore']")

# 'year' nodes that are children of nodes with name='Singapore'
root.findall(".//*[@name='Singapore']/year")

# All 'neighbor' nodes that are the second child of their parent
root.findall(".//neighbor[2]")

```
下表是ElementTree支持的XPath语法：

|  Syntax | Meaning |
|---------|---------|
|tag|Selects all child elements with the given tag. For example, spam selects all child elements named spam, and spam/egg selects all grandchildren named egg in all children named spam.|
|*|Selects all child elements. For example, */egg selects all grandchildren named egg.|
|.|Selects the current node. This is mostly useful at the beginning of the path, to indicate that it’s a relative path.|
|//|Selects all subelements, on all levels beneath the current element. For example, .//egg selects all egg elements in the entire tree.|
|..|Selects the parent element.|
|[@attrib]|Selects all elements that have the given attribute.|
|[@attrib='value']|Selects all elements for which the given attribute has the given value. The value cannot contain quotes.|
|[tag]|Selects all elements that have a child named tag. Only immediate children are supported.|
|[tag='text']|  Selects all elements that have a child named tag whose complete text content, including descendants, equals the given text.|
|[position]|Selects all elements that are located at the given position. The position can be either an integer (1 is the first position), the expression last() (for the last position), or a position relative to the last position (e.g. last()-1).|

### 根据下标查找
我们也可以根据下标来直接查询子节点
```python
year = root[0][1]
ET.dump(year)

```
# 修改XML
## 修改属性和文本
```python
#set(key,value)
root.set('name','xml')
ET.dump(root)

root.text = 'hello'
ET.dump(root)

```
## 增加节点
```python
#新建节点
new_element = ET.Element('new')
#顺序插入节点
root.append(new_element)
ET.dump(root)
#按位置插入节点
root.insert(1,new_element)
ET.dump(root)

```
## 删除节点
```python
#删除指定子节点
root.remove(root[0])
ET.dump(root)

#清空所有子节点，并且清除属性和文本
root.clear()
ET.dump(root)

```
# 写入XML
## 写到控制台
```python
#参数可以是ElementTree或Element
ET.dump(root)

```
## 写到文件
```python
tree = ET.parse('demo.xml')
tree.write('a.xml',encoding='utf-8',xml_declaration=True)

```

# 参考文献
本文主要参考[xml.etree.ElementTree](https://docs.python.org/2/library/xml.etree.elementtree.html#id5),更多的API接口可以参考该官方文档。