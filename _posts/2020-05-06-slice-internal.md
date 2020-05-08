---
layout: post
title: Go Slice 内部发生了什么
tags: [go, slice]
comments: true
---

## 前言

Go's Slice类型为处理类型化数据序列提供了一种方便且高效的方法。

## Slice internals

slice是一个array片段的描述符，它包含一个指向数组的指针，版本的长度，还有它的容量。

[![slice-segment]({{ site.url }}/img/go/slice/slice-segment.png)]({{ site.url }}/img/go/slice/slice-segment.png)

假如我们声明一个变量 `s = make([]byte, 5)` , 那它的存储形式类似这样：

[![slice-segment]({{ site.url }}/img/go/slice/structured.png)]({{ site.url }}/img/go/slice/structured.png)

_注_: 长度`length`是切片引用的元素数, 容量`capacity`是底层数组中的元素数(从切片指针引用的元素开始).

> The length is the number of elements referred to by the slice. The capacity is the number of elements in the underlying array (beginning at the element referred to by the slice pointer)

正如我们基于slice `s`, 重新赋值`s = s[2:4]`，那么slice的数据结构应该如下(长度为2， 容量为3)：

[![slice-segment]({{ site.url }}/img/go/slice/slice-01.png)]({{ site.url }}/img/go/slice/slice-01.png)

Slice 不会拷贝slice的数据，它会创建一个指向原始数组的新slice. 也正是因为这样, 使slice的如操作arry的索引一样高效。因此， 修改重新切片的元素(而不是切片本身)会修改原始切片的元素。

## Growing slices (the copy and append functions)
为了增加slice的容量，必须创建一个更大的新的slice, 然后将原始的slice 内容拷贝进去。
> 这种技术就是其他语言的动态数组实现如何在幕后工作的。


### Copy
使用built-in的`copy`函数. 拷贝原 slice 至目标 slice,  返回拷贝元素的个数。

```
func copy(dst, src []T) int
```

The copy function supports copying between slices of different lengths (it will copy only up to the smaller number of elements). In addition, copy can handle source and destination slices that share the same underlying array, handling overlapping slices correctly.


### Append

```
func append(s []T, x ...T) []T
```

`append`函数添加`x`元素至 slice 的末尾，如果需要更大的容量，则增加切片。


### the difference


## 参考 

1. [Go Slices: usage and internals](https://blog.golang.org/slices-intro)
2. [Array, slices(and strings): The mechanics of 'append'](https://blog.golang.org/slices)