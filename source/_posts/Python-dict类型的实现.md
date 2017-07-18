---
title: Python dict类型的实现
date: 2017-07-18 16:38:09
tags:
        - python
        - dict
        - hash
---

程序员们的经验里面，通常都会认为字典和集合的速度是非常快的，字典的搜索的时间复杂读为O(1)，

为什么能有这么快呢？在于字典和集合的后台实现。

## 散列表 Hash table

散列表是一个稀疏数组，散列表里面的单元叫做表元 `bucket`， 在dict的散列表中，每个键值对都占用一个表元，每个表元有两个部分：一个对键值的引用，一个对值的引用。因为所有表元大小一致，可以通过偏移量来读取某个表元。由于是稀疏数组，python会设法保证还有大约三分之一的表元是空的，快要到达这个阈值的时候，会把原有的散列表复制到一个更大的空间里面。

如果要把一个对象放入散列表，那么需要先计算这个元素的散列值

``` python
map(hash, (0, 1, 2, 3))
# [0, 1, 2, 3]

map(hash, ("namea", "nameb", "namec", "named"))
# [6674681622036885098, -1135453951840843879, 3071659021342785694, 5386947181042036450]
```

### 常用构造hash函数的方法

构造散列函数有多种方式，比如直接寻址法、数字分析法、平方取中法、折叠法、随机数法、除留余数法等。

#### 折叠法

#### 取中法

### 散列表算法

为了获取my_dict[search_key] 的值， Python会调用hash(search_key) 来计算散列值，把这个值的最低几位数字当做偏移量，在散列表里面查找表元，如果摘到的表元是空的，则抛出KeyError异常，若不是空的， 表元里会有一对found_key: found_value，然后Python会检查search_key是否等于found_key，如果相等，就返回found_value。

![从字典取值流程图](http://7xkfga.com1.z0.glb.clouddn.com/hash_search.png)

如果search_key和found_key不匹配的话，就叫做散列冲突。

### 散列冲突解决方法


#### 开放寻址法 Open addressing

Python是使用开放寻址法中的二次探查来解决冲突的。如果使用的容量超过数组大小的2/3，就申请更大的容量。数组大小较小的时候resize为*4，较大的时候resize*2。实际上是用左移的形式。

## 参考

[Python dictionary implementation](http://www.laurentluce.com/posts/python-dictionary-implementation/)

[Fluent Python](https://www.amazon.com/Fluent-Python-Concise-Effective-Programming/dp/1491946008/ref=sr_1_1?ie=UTF8&qid=1500368395&sr=8-1&keywords=fluent+python)
