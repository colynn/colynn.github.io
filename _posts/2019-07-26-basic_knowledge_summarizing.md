---
layout: post
title: python 语法汇总
tags: [python]
comments: true
---

### 列表与元组的区别；
1. __语法差异：声名方式不同__ 第一个不同点是元组的声明使用小括号，而列表使用方括号，当声明只有一个元素的元组时，需要在这个元素的后面添加英文逗号；

2. __不可修改类型__ 第二个不同点是元组声明和赋值后，不能像列表一样添加、删除和修改元素，也就是说元组在程序运行过程中不能被修改,无法复制。

    ```
    >>> copy_t = tuple(t)
    >>> print(t is copy_t)
    True
    >>> copy_l = list(l)
    >>> print(l is copy_l)
    False
    ```
3. __内存大小差异__ Python将低开销的较大的块分配给元组，因为它们是不可变的。 对于列表则分配小内存块。 与列表相比，元组的内存更小。 当你拥有大量元素时，元组比列表快。列表的长度是可变的。
    ```
    >>> l = ['https://china-testing.github.io/', 'https://www.soco.com']
    >>> t = ('https://china-testing.github.io/', 'https://www.soco.com')
    >>> print(l.__sizeof__())
    56
    >>> print(t._sizeof__())
    40
    ```



_注_: 用于列表的排序、替换、添加等方法也不适用于元组，适用于元组的主要运算有元组的合并、遍历、长度、求元组的最大值和最小值等操作方法。

元组的不可修改特性可能会让元组变得非常不灵活，因为元组作为容器对象，很多时候需要对容器的元素进行修改，这在元组中是不允许的。元组可以说是列表数据的一种补充，数据的不可修改性在程序设计中也是非常重要的。

__例如__，当需要将数据作为参数传递给API，但不希望API修改参数时，就可以传递一个元组类型；再如，当需要定义一组Key时，也可以采用元组类型。因此可以说元组和列表是互为补充的数据类型。


### 元组是否可以作为字典的key， 分场景；
首先一个对象能不能作为字典的key, 就取决于其有没有__hash__方法。 所以除了容器对象（list/dist/set）和内部包含容器对象的tuple 是不可作为字典的key, 其他的对象都可以。



### 容器类型的对象，如字典的内存结构


字典的keys() 方法会在遍历前生成一个临时的列表，导致上面的代码消耗大量内存并且运行缓慢。正确的方式，是使用默认的 iterator。默认的 iterator 不会分配新内存，也就不会造成上面的性能问题


### 迭代器、生成器、可迭代对象
1. 基于一个可迭代对象（即容器对象 list/tuple/生成器/迭代器）可以产生的 生成器或迭代器；
2. 迭代器、生成器 均有next()方法；
3. 迭代器是一次性载入内存; 生成器使用了『惰性计算』/『延迟求值』的机制，在每次计算出一个条目后，把这个条目“产生”(yield)出来.


### 进程、线程、协程 应用场景

### Garbage Collections/cyclical references

Generational is a type of tracing grabage collection.

Generational Garbage Collection is based on the gheory that __most objects die young__.


### 数据库索引


### tcp 三次握手、四次挥手

### 赋值，深拷贝，浅拷贝

### with

### 闭包

### 排序算法及复杂度



