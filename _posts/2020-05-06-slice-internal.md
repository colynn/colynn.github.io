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

### 示例

```
func SubtractOneFromLength(slice []byte) []byte {
	slice = slice[0 : len(slice)-1]
	return slice
}

var buffer [256]byte

func main() {
	slice := buffer[10:20]
	for i := 0; i < len(slice); i++ {
		slice[i] = byte(i)
	}
	fmt.Println("Before: len(slice) =", len(slice))
	newSlice := SubtractOneFromLength(slice)

	fmt.Println("After:  len(slice) =", len(slice))
	fmt.Println("After:  len(newSlice) =", len(newSlice))
}

```


通过上面的函数，我们可以修改 slice 的内容，但是它的头部信息不能被修改。slice的长度不会被修改通过调用函数，因为函数传输的一个原有 slice 的拷贝， 而不是原有的slice。

因为如果我们想修改 slice 的头部信息，我们必须返回 slice本身作为一个结果参数。如上面的函数一样，函数返回一个新的长度，然后赋值给`newSlice`.

或者是处理函数时我们使用指针，如下面的函数：

```
func SubtractOneFromLength(slice *[]byte) *[]byte {
	slicepr := *slice
	*slice = slicepr[0 : len(*slice)-1]
	return slice
}

var buffer [256]byte

func main() {
	slice := buffer[10:20]
	for i := 0; i < len(slice); i++ {
		slice[i] = byte(i)
	}
	fmt.Println("Before: len(slice) =", len(slice))
	newSlice := SubtractOneFromLength(&slice)

	fmt.Println("After:  len(slice) =", len(slice))
	fmt.Println("After:  len(newSlice) =", len(*newSlice))
}
```

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

`append` 函数添加 `x` 元素至 slice 的末尾，如果需要更大的容量，则增加切片。


### Append 时间复杂度

> 概述：`append()`的摊销成本为O(1), 但最坏情况的成本为O(N).

示例代码：

```
// main.go
package main
var size = 1 << 20
func Append() {
   var s []int
   for i := 0; i < size; i++ {
      s = append(s, i)
   }
   _ = s
}
func Make() {
   s := make([]int, size)
   for i := 0; i < size; i++ {
      s[i] = i
   }
   _ = s
}

```

```
// main_test.go
package main
import "testing"
func BenchmarkAppend(b *testing.B) {
   for n := 0; n < b.N; n++ {
      Append()
   }
}
func BenchmarkMake(b *testing.B) {
   for n := 0; n < b.N; n++ {
      Make()
   }
}
```

运行结果 `go test -bench=.`

```
goos: darwin
goarch: amd64
pkg: newops/tmp/slice-append
BenchmarkAppend-8            302           3925864 ns/op
BenchmarkMake-8             1353            886194 ns/op
PASS
ok      newops/tmp/slice-append 2.704s
```

从结果可以看出，如果我们创建带容量的`slice`, 可以节约将近88%的 CPU使用时间。


## 结论
1. `append()`可以是昂贵的，尽管其摊销成本在理论上为O(1)。
2. 当`slice`的大小较小时，分配内存可能是使用`slice`时最耗时的部分；当大小增加时，在内存中移动数据将占用更多的CPU时间。
3. 如果我们知道`slice`的长度，我们应该使用 `make()`创建`slice`而不使用动态的追加`slice`。

## 参考 

1. [Go Slices: usage and internals](https://blog.golang.org/slices-intro)
2. [Array, slices(and strings): The mechanics of 'append'](https://blog.golang.org/slices)
3. [Golang: The time complexity of append()](https://medium.com/vendasta/golang-the-time-complexity-of-append-2177dcfb6bad)