---
title: "Tensorflow中的几个基本概念"
date: 2018-03-03T13:03:34+08:00
author: "zvector"
author_homepage: "https://zvector.tk/"
tags: ["notes", "tensorflow"]
categories: ["技术备忘录"]
CreativeCommons: true
draft: true
---

在前一篇[机器学习入坑记录](https://zvector.tk/post/getting-started-with-machine-learning/)中，我们简要地过了一遍Pandas库的几个常用方法。本篇博文为低阶Tensorflow基础知识，主要学习Tensorflow（下文简称TF）的几个基本概念：张量，指令，图，会话等。

我们在数学和物理学的学习中接触过标量（一个点或一个数字，也可称零维数组），矢量（一条线或一阶数组，也可称一维数组）和矩阵（二维数组）的概念。那么张量是什么呢？对使用TF的我们来说，将张量理解为是一种多维数组，即对上述标量也好，矢量或矩阵也好，张量就是对这些概念的概括和延伸。这样的理解对目前我们的使用已经足够。在TF的学习中，张量的使用是十分普遍和基础的，下面我们来看看如何在TF中创建和使用张量。

张量（sensor）在TF中可做为常量或变量存储中TF的图中，至于什么是TF的图，下文会解释。创建一个张量常量，类似：

`x = tf.constant([5, 2])`

创建一个张量变量，类似：

`y = tf.Variable([5])`

或者先创建变量，再分配一个值：

`y = tf.Variable([0])`

`y = tf.assign([5])`

常量和变量的区别，就是在TF指令（可理解为语句）中，常量是始终返回同一张量值的指令，变量则会返回分配给它的张量指令。无论是常量还是变量，都是存储在TF的图（Graph）中，可以把图理解成封装了张量的抽象图层，张量存储其中，图与图的节点是TF指令，张量流经图。需要在TF的会话（Session）中运行图。根据官网[教程](https://colab.research.google.com/notebooks/mlcc/tensorflow_programming_concepts.ipynb?hl=zh-cn#scrollTo=NzKsjX-ufyVY)，TF编程本质上是一个两步过程：

1. 将常量，变量和指令整合在一个图中。
2. 在一个会话中评估这些常量，变量和指令。

下面是一个简单的TF程序，将两个常量相加：

```python
import tensorflow as tf

# 创建一个图
g = tf.Graph()
# 将此图建立为”默认“图
with g.as_default():
    # 创建两个常量
    x = tf.constant(8, name="x_const")
    y = tf.constant(5, name="y_const")
    sum = tf.add(x, y, name="x_y_sum")
    
    # 创建一个会话，用于运行图
    with tf.Session() as sess:
        print sum.eval()
```

以上是低阶Tensorflow编程的几个基本概念，下篇博文将进一步介绍TF编程中如何创建和使用张量。
