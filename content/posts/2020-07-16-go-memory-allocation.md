---
title: Golang 内存分配 - stack and heap
tags: ["go", "stack", "heap"]
date: 2020-04-29
lastmod: 2020-06-15
---


##

https://golang.org/doc/faq#stack_or_heap



## stack and pointers
我不能解释地像 [William Kennedy](https://www.ardanlabs.com/blog/2017/05/language-mechanics-on-stacks-and-pointers.html)一样优秀，那就让我来做一个汇总.

一个变量可以被分配在 `heap`或是`stack`:
1. The stack contains the __ongoing__ variables for a given goroutine. Once a function returned, the variables are popped from the stack.
2. The heap contains the shared variables (global variables, etc.), and anything on the heap is maaged by the Garbage Collector.

简而言之，当你需要`Sharing`变量，可以使用指针；
  * 如果你需要（`passing pointers` or `passing referencs`） (__sharing down__)，变量定义通常在 `stack` 内
  * 如果你需要（`returning pointers` or `returning references`）(__sharing up__)，变量定义通常在 `heap` 内

_注_: 指针变量像其他的变量一样并不特殊，它们有一个内存分配而且它们保存一个值, 不论值的类型它们均可以指向，通常也占用相同的内存大小。

_注_: The stack is important because it provides the physical memory space for the frame boundaries that are given to each individual function.

_注_: All stack memory __below__ the active frame is invalid but memory from the active frame and above is valid.

_注_: It’s during each function call, when the frame is taken, that the stack memory for that frame is wiped clean.

_注_: Making a function call means the goroutine needs to frame __a new section of memory__ on the stack.


## From the FAQ 
https://golang.org/doc/faq#stack_or_heap

When possible, the Go compilers will allocate variables that are local to a function in that function's stack frame.

However, if the compiler __cannot prove__ that the variable is __not referenced after the function returns__, then the compiler must allocate the variable on the garbage-collected heap to avoid dangling pointer errors.


## 结论
1. Pointers serve __one purpose__, to __share a value with a function__ so the function can read and write to that value even though the value does not exist directly inside its own frame.

2. Each semantic comes with a benefit and cost. __Value semantics__ keep values on the __stack__ which reduces pressure on the __GC__. However, there are different copies of any given value that must be stored, tracked and maintained. __Pointer semantics__ place values on the __heap__ which can put pressure on the __GC__. However, they are efficient because there is only one value that needs to be stored, tracked and maintained. The key is using each semantic correctly, consistently and in balance.



## Ask the compiler

### Compiler flags

* go help build
* go tool compile -h
* go build -gcflags "-m"


## 参考
https://golang.org/doc/faq#stack_or_heap