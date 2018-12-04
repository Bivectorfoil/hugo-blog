---
title: "Python List 方法中 append 与 extend 的区别"
date: 2018-12-04T09:50:02+08:00
toc: true
author: "zvector"
author_homepage: "https://bivectorfoil.github.io/"
tags: ["Python"]
categories: ["技术备忘录"]
CreativeCommons: true
draft: false
---



最近在处理列表中的字符串添加的问题时，发现 Python 中 append 和 extend 两种方法的表现有点出乎我的意料，一番搜索后，找到了一篇十分精彩的回答，如下，我尝试将其翻译一下，以兹学习。

> 原答案地址为：[What is the difference between the list methods append and extend?](https://stackoverflow.com/a/28119966) Plain address: https://stackoverflow.com/a/28119966

> 以下为原回答：

### **List 方法 append 和 extend 有哪些不同点？**

- `append` 将其参数视为单一元素地添加到列表之后。列表的长度自增一。
- `extend` **逐一地**将其参数添加到列表之后，扩展了列表。列表的长度变化取决于可迭代的参数中有多少元素。

#### **`append`**

`list.append` 方法将对象附加到列表的末尾。

```python 
my_list.append(object)
```

无论对象是数字，字符串，另一个列表，或者其他的对象，它都将作为一个单一元素添加到列表末尾。

```bash
>>> my_list
['foo', 'bar']
>>> my_list.append('baz')
>>> my_list
['foo', 'bar', 'baz']
```

所以请牢记在心，一个列表就是一个对象。如果你将一个列表添加到另一个列表中，前者将作为一个单一元素，添加到被添加列表的末尾（也许和你预期的不一样）：

```bash
>>> another_list = [1, 2, 3]
>>> my_list.append(another_list)
>>> my_list
['foo', 'bar', 'baz', [1, 2, 3]]
                     #^^^^^^^^^--- 列表末尾的单一元素
```

#### **`extend`**

`list.extend` 方法通过从可迭代对象中添加元素到列表中来扩展列表：

```python
my_list.extend(iterable)
```

所以使用 extend 方法时，可迭代对象中的每一个元素都被添加到列表中。举个栗子：

```bash
>>> my_list
['foo', 'bar']
>>> another_list = [1, 2, 3]
>>> my_list.extend(another_list)
>>> my_list
['foo', 'bar', 1, 2, 3]
```

请牢记在心，字符串同样也是可迭代对象，所以如果你用字符串来扩展列表，你将把字符串中的每一个字符都（单独地）添加到列表中了（也许和你预期的不一样）：

```bash
>>> my_list.extend('baz')
>>> my_list
['foo', 'bar', 1, 2, 3, 'b', 'a', 'z']
```

### **操作符重载，`__add__`(+) 和 `__iadd__`(+=)**

`+` 和 `+=` 操作符均定义在了 `列表` 中。它们在语义上类似于扩展。

`my_list + another_list` 在内存中创建了第三个列表对象，所以你可以将其结果返回，但它要求第二个迭代对象为列表。

`my_list += another_list` 原地修改了列表（它 *是*  原地操作符，并且列表是可变对象，正如我们所看到的那样）所以它不会创建一个新列表对象。它运行得如 extend 一样，所以第二个可迭代对象可以是任意可迭代对象。

不要混淆了 `my_list = my_list + another_list` 和 `+=` 操作 -- 前者将一个新列表赋值到 `my_list` 。

### **时间复杂度**

`Append` 为常数时间复杂度，O(1)。

`Extend` 为线性时间复杂度，O(k)。

多次重复地调用 `append` 会增加其复杂度，使得 `append` 的时间复杂度与 `extend` 相同，并且，由于 `extend` 是 C 实现的，当你需要重复地将可迭代对象添加到列表中时，使用 `extend` 方法总是更快的。

### **性能对比**

既然 `append` 可以实现和 `extend` 的同等效果，你可能想知道谁的性能更好。下列函数的效果相同：

```python
def append(alist, iterable):
    for item in iterable:
        alist.append(item)

def extend(alist, iterable):
    alist.extend(iterable)
```

测试一下：

```bash
import timeit

>>> min(timeit.repeat(lambda: append([], "abcdefghijklmnopqrstuvwxyz")))
2.867846965789795
>>> min(timeit.repeat(lambda: extend([], "abcdefghijklmnopqrstuvwxyz")))
0.8060121536254883
```

#### **就时间测量问题多说一句**

有评论说到：

> ==完美的答案，我还想知道仅仅添加一个元素时的性能对比==

充分遵循语义行事。如果你想将一个可迭代对象的所有元素都添加到列表中，使用 `extend` 方法。如果你只是想添加一个元素，使用 `append` 方法。

让我们设计一个实验看看：

```python
def append_one(a_list, element):
    a_list.append(element)

def extend_one(a_list, element):
    """creating a new list is semantically the most direct
    way to create an iterable to give to extend"""
    a_list.extend([element])

import timeit
```

我们会看到，不遗余力地使用 `extend` 去创建一个可迭代对象是在浪费时间：

```bash
>>> min(timeit.repeat(lambda: append_one([], 0)))
0.2082819009956438
>>> min(timeit.repeat(lambda: extend_one([], 0)))
0.2397019260097295
```

我们学到的是，仅当我们需要添加一个元素时，使用 `extend` 方法并无更多的好处。

同样地，上述的具体时间测量值并不是关键，关键是，在 Python 中，正确地遵循语义就是在 *做正确的事*。

可以想见，你可能会测试两个类似操作的所需时长并获得模糊或相反的结果。但我们只要关注于做语义正确的事情就好了。

### **结论**

 我们可以看到 `extend` 方法语义上更为简洁，并且它比 `append` 方法运行得更快，<u>当你打算将一个可迭代对象的每一个元素都添加到列表中时。</u>

如果你只有一个单一的元素（而不是一个可迭代对象）需要添加到列表时，使用 `append` 方法。