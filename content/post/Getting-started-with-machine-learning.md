---
title: "机器学习入坑记录"
date: 2018-03-02T12:32:53+08:00
toc: true
author: "zvector"
author_homepage: "https://zvector.tk/"
tags: ["notes", "machine learning"]
categories: ["技术备忘录"]
CreativeCommons: true
draft: false
---

## 跳坑前的废话

今天偶然看到了谷歌的机器学习速成课程，地址[在这](https://developers.google.com/machine-learning/crash-course/prereqs-and-prework)。兴趣浓厚，虽说是“速成”，但所要学习的东西也不少。很长一段时间以来，我对机器学习就很感兴趣，无奈机器学习是有一定的入门门槛的，网上资料虽多，选择繁复，但要挑选出适合自己的一套学习资料还是颇费一番功夫。正好此次谷歌官方推出了针对不同学习基础学习者的机器学习速成课程，趁次机会，决定跳坑，亲自尝试这个课程，也可实验实验自己是不是这块料：）（莫名想起自己就是这样开了无数个未填上的坑，汗）

## ML学习前的准备

根据官网的说明，我对应于对机器学习知之甚少的一栏，做为学习机器学习的准备，先从Pandas库的学习入门，此篇可看做学习Pandas库的学习笔记吧。

## 正式学习-Pandas篇

Pandas库是什么呢？根据维基百科的说法：

> pandas is a software library written for the Python programming language for data manipulation and analysis. In particular, it offers data structures and operations for manipulating numerical tables and time series. 

Pandas是一个用于数据处理和分析的强大Python扩展库，Pandas为数据列表和时间序列提供封装好的数据结构和操作。

此篇是为ML学习铺路，故不会涉及Pandas库的高阶用法，仅记录初级使用指南。

根据教程，学习目标如下：

- 大致了解 *pandas* 库的 `DataFrame` 和 `Series` 数据结构
- 存取和处理 `DataFrame` 和 `Series` 中的数据
- 将 CSV 数据导入 pandas 库的 `DataFrame`


下面，以官网的[笔记](https://colab.research.google.com/notebooks/mlcc/intro_to_pandas.ipynb)为例，简要说明Pandas库的几种常用方法。

先从Pandas库的导入开始吧。

```python
import pandas as pd
pd.__version__ # print pandas version
```

Pandas中最主要的两类数据结构为DataFrame和Series。可分别类比为一种包含多行和多列的关系型数据表，以及单独一列的数据记录。Pandas通过封装好的上述数据结构，简化并提供了一系列的对数据的操作。那么，如何创建基本的数据结构呢，有几种方法，如下：

我们可以创建单独的Series：

```python
pd.Series(['beijing', 'shanghai', 'guangzhou'])

# output
0      beijing
1     shanghai
2    guangzhou
dtype: object
```

也可以使用类似Python的dict操作，一次性创建一个DataFrame对象：

```python
city_names = pd.Series(['beijing', 'shanghai', 'guangzhou'])
population = pd.Series([1961240, 2301910, 1270080])
pd.DataFrame({ 'City name': city_names, 'Population': population })

# output
    City name	Population
0	beijing	    1961240
1	shanghai	2301910
2	guangzhou	1270080
```

当然还可以从CSV文件中读取导入：

```python
california_housing_dataframe = pd.read_csv("https://storage.googleapis.com/ml_universities/california_housing_train.csv", sep=",")
california_housing_dataframe.describe() # describe()显示整个DataFrame
california_housing_dataframe.head() # 也可以使用head()显示前几个记录
california_housing_dataframe.hist('housing_median_age') # 使用hist(argv)以图标的形式显示选定的argv的分布
```

上述是Pandas如何创建数据表和几种浏览数据表的方式，那么，具体的访问DataFrame中的数据应该怎么做呢？类似Python 的dict/list方法，Pandas中使用同样的方式访问数据。

```python
cities = pd.DataFrame({ 'City name': city_names, 'Population': population})
print type(cities['City name'])
cities['City name']

# output
<class 'pandas.core.series.Series'>
0      beijing
1     shanghai
2    guangzhou
Name: City name, dtype: object
```

同样地，访问具体的一个或多个城市：

```python
cities['City name'][1] # 'shanghai'
print type(cities[0:2])
cities[0:2]

# output

<class 'pandas.core.frame.DataFrame'>
    City name	Population
0	beijing	    1961240
1	shanghai	2301910
```

Pandas对Series提供了一种单列转换方法，Series.apply以lambda函数做为参数，返回lambda函数应用后的值，例如，筛选人口大于100万的城市

```python
population.apply(lambda val: val > 1000000)

# output
0    True
1    True
2    True
dtype: bool
```

数据的访问如上所述，那么数据的添加呢？类似Python的dict，直接向DataFrame中添加两个Series：

```python
cities['Area square miles'] = pd.Series([32.52, 123.45, 67.89])
cities['Population density'] = cities['Population'] / cities['Area square miles']
cities

# output
   City name	Population	Area square miles	Population density
0	beijing	   196124000	46.87	            4.184425e+06
1	shanghai	230191000	176.53	            1.303977e+06
2	guangzhou	127008000	97.92	            1.297059e+06
```

上面就是我根据谷歌官方的[笔记教程](https://colab.research.google.com/notebooks/mlcc/intro_to_pandas.ipynb?hl=zh-cn)总结的非常简略的Pandas使用笔记。粗略地过了一遍Pandas，根据[前提条件和准备](https://developers.google.com/machine-learning/crash-course/prereqs-and-prework)，接下来就是TensorFlow的低阶基础知识。欲知后事如何，且看下篇博文。