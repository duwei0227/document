---
layout: post
title: Python使用ElementTree操作XML
categories: [Python]
description: Python使用ElementTree操作XML
permalink: python/elementtree.html
tags: ElementTree
---


  本篇使用`xml.etree.ElementTree`模块进行xml文件的解析和创建。`ET`有两个主要的类用于xml的解析和构建-`ElementTree`将整个XML文档作为一棵树，`Element`代表树中的一个节点。



### 一、解析XML

示例 XML 文档：

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



#### 1、从文件中解析XML文档

使用`parse(file_name)`方法解析xml文件，`parse`返回`ElementTree`对象,使用`getroot()`获取根节点对象

```xml
import xml.etree.ElementTree as ET

tree = ET.parse(xml文件全路径)
root = tree.getroot()
print(root.tag)
```

#### 2、从字符串中解析XML文档

`fromstring()`方法从一个字符串中解析xml对象，返回的对象为 `Element`，该对象为xml元素树的根(`root`)节点对象

```xml
import xml.etree.ElementTree as ET

root = ET.fromstring(country_data_as_string)
```



#### 3、解析带有命名空间(Namespace)的XML文档

如果xml文档有声明命名空间时，解析后的文档 标签(`tags`)和属性(`attributes`)都会添加一个前缀，前缀格式为`{uri}sometag`,其中`uri`为完整的`namespace`地址，在进行元素的操作时都需要添加前缀(`prefix`)

在如下示例中包含两个命名空间，一个以前缀`fictional`开头，其他的采用默认命名空间

```xml
<?xml version="1.0"?>
<actors xmlns:fictional="http://characters.example.com"
        xmlns="http://people.example.com">
    <actor>
        <name>John Cleese</name>
        <fictional:character>Lancelot</fictional:character>
        <fictional:character>Archie Leach</fictional:character>
    </actor>
    <actor>
        <name>Eric Idle</name>
        <fictional:character>Sir Robin</fictional:character>
        <fictional:character>Gunther</fictional:character>
        <fictional:character>Commander Clement</fictional:character>
    </actor>
</actors>
```



文档里解析与方法1或者方法2相同，区别在与解析后的文档对象

```xml
root = ET.fromstring(xml_doc)
print(root.tag)

输出：{http://people.example.com}actors
```





### 二、元素查找

#### 1、iter(tag=None)

​	使用当前元素作为根元素构建一颗迭代(`iterator`)树，会迭代当前元素及其所有子元素(包括非直接子元素)。如果`tag`不为`None`或者为`*`时，仅仅匹配元素的`tag`与当前传入的`tag`一致的对象

```xml
# 迭代所有元素
for e in element.iter(t ag='*'):
	print(e.tag)

# 迭代所有 neighbor
for e in element.iter(tag='neighbor'):
    print(e.attrib)


```



#### 2、findall(*match*, *namespaces=None*)

查找所有匹配的直接子元素，查询条件可以是标签名称(`tag name`)或者`xpath`。返回所有匹配元素列表`list`,元素顺序按照文档顺序排序。`namespace`是一个可选的条件，用于匹配包含命名空间的文档

```xml
# 查找文档根节点下所有的country
for e in element.findall('country'):
	print(e.attrib)


```



#### 3、find(*match*, *namespaces=None*)

查找第一个符合条件的直接子元素，查询条件可以是标签名称(`tag name`)或者`xpath`。返回匹配的元素实例或者4`None`,元素顺序按照文档顺序排序。`namespace`是一个可选的条件，用于匹配包含命名空间的文档

```xml
# 查找第一个匹配的 country
first_element = element.find('country')
print(first_element.attrib)

```



#### 4、get(*key*, *default=None*)

根据`key`获取元素属性值，如果属性不存在返回一个默认值

```xml
# 查找第一个匹配的 country
first_element = element.find('country')
print(first_element.attrib)

# 查找 name 属性
first_element.get('name')

# 查找不存在的属性
first_element.get('haha', default='^_^')
```



#### 5、xpath

`ElementTree`支持的`xpath`语法，以下示例从 `root`节点开始解析

```xml
import xml.etree.ElementTree as ET

root = ET.fromstring(countrydata)
```



| 语法                 | 描述                                                         | 示例                                                         |
| -------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| `tag`                | 匹配所有与给定tag匹配的直接子元素。从`Python3.8`开始支持`*`: `{namespace}*`匹配给定命名空间下所有元素；`{*}spam`匹配所有`tag`名为`spam`的元素，前边命名空间可以为任意;`{}*`匹配未限定命名空间的元素 | `element.findall('country')`<br>`element.findall('{}*')`     |
| `*`                  | 查找所有的子元素，包含注释和处理指令。例如`*/egg`匹配所有孙辈的`egg`元素 | 当前元素的所有直接子元素`element.findall('*')`               |
| `.`                  | 查找当前元素节点。通常用于在相对路径中的起始节点             | 当前元素下所有的`year`<br>`element.findall('./country/year')` |
| `//`                 | 选择当前元素下所有级别上的所有子元素(可以跳过元素的层级查找)。例如`//egg`选择整个树中的所有`egg`元素 | `element.findall('.//rank')`                                 |
| `..`                 | 选择父元素。如果路径试图到达起始元素（已调用元素find）的祖先，则返回None。 | 查找`rank`的父元素`element.findall('.//rank/..')`            |
| `[@attrib]`          | 选择具有给定属性的所有元素。                                 | 查找当前元素下所有包含direction属性的元素`element.findall('.//*[@direction]')` |
| `[@attrib='value']`  | 选择给定属性具有给定值的所有元素。该值不能包含引号。         | 查找当前元素下所有包含direction属性且值等于E的元素<br>`element.findall('.//*[@direction="E"]'` |
| `[@attrib!='value']` | 选择给定属性没有给定值的所有元素。该值不能包含引号。<br/>3.10版新增。 | 查找当前元素下所有包含direction属性且值不等于E的元素<br/>`element.findall('.//*[@direction!="E"]')` |
| `[tag]`              | 选择具有名为tag的子元素的所有元素。只支持直系子女。          | 过滤子元素中包含`year`的`country`元素element.findall('./country/[year]')` |
| `[.='text']`         | 选择其完整文本内容（包括子体）等于给定文本的所有元素。<br/><br/>在3.7版中新增。 | 过滤文本值为2011的`year`元素`element.findall('.//year[.="2011"]')` |
| `[.!='text']`        | 选择其完整文本内容（包括子体）不等于给定文本的所有元素。<br/><br/>3.10版新增。 | 过滤文本值不等于2011的`year`元素`element.findall('.//year[.!="2011"]')` |
| `[tag='text']`       | 选择所有元素，这些元素具有一个名为tag的子元素，该子元素的完整文本内容（包括子元素）等于给定的文本。 | 过滤子元素为`year`且`year`的文本值为2011的元素<br>`element.findall('*/[year="2011"]')` |
| `[tag!='text']`      | 选择所有具有名为tag的子元素且其完整文本内容（包括子元素）不等于给定文本的元素。<br/><br/>3.10版新增。 | 过滤子元素为`year`且`year`的文本值不等于2011的元素<br/>`element.findall('*/[year!="2011"]')` |
| `[position]`         | 选择位于给定位置的所有图元。位置可以是整数（1是第一个位置）、表达式last（）（表示最后一个位置）或相对于最后一个位置的位置（例如last（）-1）。 | 查找当前元素下第二个元素`element.findall('*[2]')[0].get('name')` |



### 三、修改XML文档

初始文档：

```xml
<?xml version='1.0' encoding='utf-8'?>
<grandparent>
    <!--我是儿子a-->
    <parent name="a">
        <child>my father is a</child> <!--我是儿子b-->
    </parent>
    <parent name="b">
        <child>my father is b</child>
    </parent>
</grandparent>
```



#### 1、修改元素文本

```python
first_child = root.find('./parent/child')
print('修改前：', first_child.text)
first_child.text = '我的父节点是a'
print('修改后：', first_child.text)
```



#### 2、修改已有元素属性

```python
first_parent = root.find('./parent')
print('修改前name：', first_parent.get('name'))
first_parent.set('name', 'A')
print('修改后name：', first_parent.get('name'))
```



#### 3、新增属性

```python
first_parent = root.find('./parent')
print('新增age前：', first_parent.get('age'))
first_parent.set('age', '18')
print('新增age后：', first_parent.get('age'))
```



#### 4、删除元素

删除元素使用`remove()`方法需要传入一个 `Element`对象

```python
first_parent.remove(first_parent.find('./child'))
```



### 四、构建XML文档

使用`Element`构建根节点元素，不断使用`SubElement`组合构建子元素

所有的元素构建完成以后需要使用`ElementTree`创建一颗文档树，用于`XML`文件的生成

使用`Comment`生成文档注释，并使用`Element.append()`方法追加在想要的位置

```python
root = Element('grandparent')
comment_a = Comment('我是儿子a')
root.append(comment_a)
parent = SubElement(root, 'parent', attrib={"name": "a"})
child = SubElement(parent, 'child')
child.text = 'my father is a'
comment_b = Comment('我是儿子b')
parent.append(comment_b)
parent_brother = SubElement(root, 'parent', attrib={"name": "b"})
child_brother = SubElement(parent_brother, 'child')
child_brother.text = 'my father is b'
# 临时xml结构查看
# ET.dump(root)

tree = ET.ElementTree(root)
# xml_declaration 用于生成xml文档声明
# 将xml文档写入文件
tree.write('test.xml', encoding='utf-8', xml_declaration=True, short_empty_elements=True)

```



写入xml文档有两种方法：

* ```python
  tree = ET.ElementTree(root)
  tree.write('test.xml', encoding='utf-8', xml_declaration=True, short_empty_elements=True)
  ```

* `ET.canonicalize(ET.dump(root))`

格式化输出XML对象到字符串：
默认输出的是一个`byte`类型字符串，可以修改`encoding='unicode'`指定输出格式为字符串，`unicode`对应的字符编码为`cp936`;如果设置`encoding`非`unicode`都按照`byte`处理。
```python
ElementTree.tostring(element, encoding='us-ascii', method='xml', xml_declaration=None, default_namespace=None)
```
  

#### 五、附录

**Python3**文档：[https://docs.python.org/3/library/xml.etree.elementtree.html](https://docs.python.org/3/library/xml.etree.elementtree.html)

