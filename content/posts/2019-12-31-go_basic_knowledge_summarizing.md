---
title: Go 语法汇总 - Block
categories: ["基础手册"]
tags: ["go"]
date: 2019-03-15
lastmod: 2020-03-15
---

## 变量声明
1. 指定变量类型，如果没有初始化，则变量默认为零值;
    ```go
    package main
    import "fmt"
    func main() {

        // 声明一个变量并初始化
        var a = "RUNOOB"
        fmt.Println(a)

        // 没有初始化就为零值
        var b int
        fmt.Println(b)

        // bool 零值为 false
        var c bool
        fmt.Println(c)
    }
    ```

2. 根据值自行判定变量类型
    ```go
    var v_name = value
    ```
3. 省略 var, 注意 := 左侧如果没有声明新的变量，就产生编译错误
    ```go
    var intVal int 

    intVal :=1 // 这时候会产生编译错误

    intVal,intVal1 := 1,2 // 此时不会产生编译错误，因为有声明新的变量，因为 := 是一个声明语句
    ```

## 多变量声明

```go
package main

var x, y int
var (  // 这种因式分解关键字的写法一般用于声明全局变量
    a int
    b bool
)

var c, d int = 1, 2
var e, f = 123, "hello"

// 短声明方式
// 这种不带声明格式的只能在函数体中出现
// g, h := 123, "hello"

func main(){
    g, h := 123, "hello"
    println(x, y, a, b, c, d, e, f, g, h)
}
```

_注_: 
1. 类型在变量名 之后。
2. 当连续两个或多个函数的已命名形参类型相同时，除最后一个类型以外，其它都可以省略。
3. var 语句可以出现在包或函数级别。
4. 变量声明可以包含初始值，每个变量对应一个; 初始化值已存在，则可以省略类型；变量会从初始值中获得类型。
5. 常量可以是字符、字符串、布尔值或数值, 常量不能用 := 语法声明。

## 零值
没有明确初始值的变量声明会被赋予它们的 零值。

零值是：
* 数值类型为 0，
* 布尔类型为 false，
* 字符串为 ""（空字符串）


## Go 的基本类型有

```go
bool

string

int  int8  int16  int32  int64
uint uint8 uint16 uint32 uint64 uintptr

byte // uint8 的别名

rune // int32 的别名
    // 表示一个 Unicode 码点

float32 float64

complex64 complex128
```


本例展示了几种类型的变量。 同导入语句一样，变量声明也可以“分组”成一个语法块。

```go
package main

import (
	"fmt"
	"math/cmplx"
)

var (
	ToBe   bool       = false
	MaxInt uint64     = 1<<64 - 1
	z      complex128 = cmplx.Sqrt(-5 + 12i)
)

func main() {
	fmt.Printf("Type: %T Value: %v\n", ToBe, ToBe)
	fmt.Printf("Type: %T Value: %v\n", MaxInt, MaxInt)
	fmt.Printf("Type: %T Value: %v\n", z, z)
}
```

_注_: 
* int, uint 和 uintptr 在 32 位系统上通常为 32 位宽，在 64 位系统上则为 64 位宽。 当你需要一个整数值时应使用 int 类型，除非你有特殊的理由使用固定大小或无符号的整数类型。
*  reflect.TypeOf(x) 打印x的类型
* 常用的格式化字符串：

```text
%v the value in a default format, when printing structs, the plus flag (%+v) adds field names 

%#v a Go-syntax representation of the value 
%T a Go-syntax representation of the type of the value
%%	a literal percent sign; consumes no value

```

* Boolean:

```text
%t	the word true or false
```

* Integer:

```text
%b	base 2
%c	the character represented by the corresponding Unicode code point
%d	base 10
%o	base 8
%O	base 8 with 0o prefix
%q	a single-quoted character literal safely escaped with Go syntax.
%x	base 16, with lower-case letters for a-f
%X	base 16, with upper-case letters for A-F
%U	Unicode format: U+1234; same as "U+%04X"
```

## >> << 操作符

In decimal, 8 >> 1 is 4, and 8 << 1 is 16. Shifting left by one is the same as multiplication by 2, and shifting right by one is the same as dividing by two, discarding any remainder.

It makes a lot of sense when I think that I was seeing this in code dealing with powers of 2 (1 << power = 2^power) 

等价于这样： (x << n == x*2^n ) (x >> n == x*2^(-n)) 

 
## For循环
Go 只有一种循环结构：for 循环。

* 基本的 for 循环由三部分组成，它们用分号隔开：

初始化语句：在第一次迭代前执行
条件表达式：在每次迭代前求值
后置语句：在每次迭代的结尾执行
初始化语句通常为一句短变量声明，该变量声明仅在 for 语句的作用域中可见。

一旦条件表达式的布尔值为 false，循环迭代就会终止。

注意：和 C、Java、JavaScript 之类的语言不同，Go 的 for 语句后面的三个构成部分外没有小括号， 大括号 { } 则是必须的。

```go
package main

import "fmt"

func main() {
	sum := 0
	for i := 0; i < 10; i++ {
		sum += i
	}
	fmt.Println(sum)
}

```

* 初始化语句和后置语句是可选的。

```go 
package main

import "fmt"

func main() {
	sum := 1
	for ; sum < 1000; {
		sum += sum
	}
	fmt.Println(sum)
}
```
* 此时你可以去掉分号，因为 C 的 while 在 Go 中叫做 for。

```go
package main

import "fmt"

func main() {
	sum := 1
	for sum < 1000 {
		sum += sum
	}
	fmt.Println(sum)
}
```


* for 循环的 range 形式可遍历切片或映射。

可以将下标或值赋予 _ 来忽略它。

```go
for i, _ := range pow
for _, value := range pow
```

若你只需要索引，忽略第二个变量即可。

```go
for i := range pow
```

代码示例

```go
package main

import "fmt"

func main() {
	pow := make([]int, 10)
	for i := range pow {
		pow[i] = 1 << uint(i) // == 2**i
	}
	for i, value := range pow {
		fmt.Printf("%v\t%d\n", i, value)
	}
}
```

## defer
defer 语句会将函数推迟到外层函数返回之后执行。

* 推迟调用的函数其参数会立即求值，但直到外层函数返回前该函数都不会被调用。

```go
package main

import "fmt"

func main() {
	defer fmt.Println("world")

	fmt.Println("hello")
}

```

* 推迟的函数调用会被压入一个栈中。当外层函数返回时，被推迟的函数会按照后进先出的顺序调用。

```go
package main

import "fmt"

func main() {
	fmt.Println("counting")

	for i := 0; i < 10; i++ {
		defer fmt.Println(i)
	}

	fmt.Println("done")
}

```

## 指针
Go 拥有指针。指针保存了值的内存地址。

类型 *T 是指向 T 类型值的指针。其零值为 nil。

```go
var p *int
```

& 操作符会生成一个指向其操作数的指针。

```go
i := 42
p = &i
```

* 操作符表示指针指向的底层值。

```go
fmt.Println(*p) // 通过指针 p 读取 i
*p = 21         // 通过指针 p 设置 i
```

这也就是通常所说的“间接引用”或“重定向”。

与 C 不同，Go 没有指针运算。

## 结构体指针
结构体字段可以通过结构体指针来访问。

如果我们有一个指向结构体的指针 p，那么可以通过 (*p).X 来访问其字段 X。不过这么写太啰嗦了，所以语言也允许我们使用隐式间接引用，直接写 p.X 就可以。

```go
package main

import "fmt"

type Vertex struct {
	X int
	Y int
}

func main() {
	v := Vertex{1, 2}
	p := &v
	(*p).Y = 19
	p.X = 1e9
	fmt.Println(v)
}
```

## 结构体文法 
结构体文法通过直接列出字段的值来新分配一个结构体。

使用 Name: 语法可以仅列出部分字段。（字段名的顺序无关。）

特殊的前缀 & 返回一个指向结构体的指针。

```go
package main

import "fmt"

type Vertex struct {
	X, Y int
}

var (
	v1 = Vertex{1, 2}  // 创建一个 Vertex 类型的结构体

    // ?? 这种就是结构体文法吗？
	v2 = Vertex{X: 1}  // Y:0 被隐式地赋予
	v3 = Vertex{}      // X:0 Y:0
	p  = &Vertex{1, 2} // 创建一个 *Vertex 类型的结构体（指针）
)

func main() {
	fmt.Println(v1, p.X, v2, v3)
}
```

## 数组
类型 [n]T 表示拥有 n 个 T 类型的值的数组。

表达式

```go
var a [10]int
```
会将变量 a 声明为拥有 10 个整数的数组。

数组的长度是其类型的一部分，因此数组不能改变大小。这看起来是个限制，不过没关系，Go 提供了更加便利的方式来使用数组。

```go
package main

import "fmt"

func main() {
	var a [2]string
	a[0] = "Hello"
	a[1] = "World"
	fmt.Println(a[0], a[1])
	fmt.Println(a)

    // ?? 不理解为何种用法
	primes := [6]int{2, 3, 5, 7, 11, 13}  
	fmt.Println(primes)
}
```

## 切片
每个数组的大小都是固定的。而切片则为数组元素提供动态大小的、灵活的视角。在实践中，切片比数组更常用。

类型 []T 表示一个元素类型为 T 的切片。

切片通过两个下标来界定，即一个上界和一个下界，二者以冒号分隔：

```go
a[low : high]
```
它会选择一个 __半开区间，包括第一个元素，但排除最后一个元素。__

以下表达式创建了一个切片，它包含 a 中下标从 1 到 3 的元素：

```go
a[1:4]
```


* 切片的默认行为 

对于数组
```go
var a [10]int
```

来说，以下切片是等价的：
```go
a[0:10]
a[:10]
a[0:]
a[:]
```

_注_: 
* 切片就像数组的引用
* 切片并不存储任何数据，它只是描述了底层数组中的一段。
* 更改切片的元素会修改其底层数组中对应的元素。
* 与它共享底层数组的切片都会观测到这些修改。
* 切片s的长度和容量可通过表达式 len(s) 和 cap(s) 来获取。
* 切片的零值是 nil.

* 切片练习??：

```sh
实现 Pic。它应当返回一个长度为 dy 的切片，其中每个元素是一个长度为 dx，元素类型为 uint8 的切片。当你运行此程序时，它会将每个整数解释为灰度值（好吧，其实是蓝度值）并显示它所对应的图像。

图像的选择由你来定。几个有趣的函数包括 (x+y)/2, x*y, x^y, x*log(y) 和 x%(y+1)。

（提示：需要使用循环来分配 [][]uint8 中的每个 []uint8；请使用 uint8(intValue) 在类型之间转换；你可能会用到 math 包中的函数。）
```

## 函数值
函数也是值。它们可以像其它值一样传递。

函数值可以用作函数的参数或返回值。

```go
package main

import (
	"fmt"
	"math"
	"reflect"
)

func compute(fn func(float64, float64) float64) float64 {
	return fn(3, 4)
}

func main() {
	hypot := func(x, y float64) float64 {
		return math.Sqrt(x*x + y*y)
	}
	fmt.Println(hypot(5, 12))
	
	fmt.Println(reflect.TypeOf(hypot))
	
	fmt.Println(compute(hypot))
	fmt.Println(compute(math.Pow))
}
```

输出：

```sh
13
func(float64, float64) float64
5
81
```


## 接口
接口类型 是由一组方法签名定义的集合。

接口类型的变量可以保存任何实现了这些方法的值。

### Compile time checks

```go
// Notify ..
type Notify interface {
	Send() string
}

// Email ..
type email struct{}

var _ Notify = (*email)(nil)

// Send ..
func (e *email) Send() string {
	return ""
}
```

注解：
*  `var _ Notify = (*email)(nil)` It provides a static (compile time) check that `email` satisfies the `Notify` interface. 

* The `_` used as a name of the variable tells the compiler to effectively discard the RHS value, but to type-check it and evaluate it if it has any side effects, but the anonymous variable per se doesn't take any process space.

* If you forget a method or if the interface changes, you’ll get a compile-time error drawing your attention to it immediately:
cannot use (*email)(nil) (type *email) as type Notify in assignment:
*email does not implement Notify (missing SomeMethod method)

* `(*email)(nil)` Essentially you are casting `nil` to a type of pointer to `MyType`. It’s a zero-memory way to represent a pointer to your struct in Go.

## 空接口

指定了零个方法的接口值被称为 *空接口：*

```go
interface{}
```

空接口可保存任何类型的值。（因为每个类型都至少实现了零个方法。）

空接口被用来处理未知类型的值。例如，```fmt.Print``` 可接受类型为 ```interface{}``` 的任意数量的参数。


## 类型断言

类型断言 提供了访问接口值底层具体值的方式。

```go
t := i.(T)
```

该语句断言接口值 i 保存了具体类型 T，并将其底层类型为 T 的值赋予变量 t。

若 i 并未保存 T 类型的值，该语句就会触发一个恐慌。

为了 判断 一个接口值是否保存了一个特定的类型，类型断言可返回两个值：其底层值以及一个报告断言是否成功的布尔值。

```go
t, ok := i.(T)
```

* i 代码要判断的变量， 必须为interface类型才可以进行类型断言；
* T 代表要被判断的类型，eg: string, int64, struct 
* t 代表返回的值，
* ok 代码是否为该类型

若 i 保存了一个 T，那么 t 将会是其底层值，而 ok 为 true。

否则，ok 将为 false 而 t 将为 T 类型的零值，程序并不会产生恐慌。

请注意这种语法和读取一个映射时的相同之处。

示例代码

```go
s := "baidu"
if v, ok := s.(string); ok {
	fmt.Println(v)
}
```

```sh
invalid type assertion: s.(string) (non-interface type string on left)
```
在这里只要是在声明时或函数传进来的参数不是interface类型那么做类型断言都是回报 non-interface的错误的

所以我们只能通过将s作为一个interface{}的方法来进行类型断言 如下代码所示

```go
s := "BrainWu"
if v, ok := interface{}(s).(string); ok {
	fmt.Println(v)
}
```

将s显示的转换为interface{}接口类型则可以进行类型断言了

2 当函数作为参数并且被调用函数将参数类型指定为interface{}的时候是没有办法直接调用该方法的
比如如下代码是错误的在编译期间就会报错
`cannot convert in (type interface {}) to type Handler: need type assertion `

```go
func ServeHTTP(s string) {
	fmt.Println(s)
}

type Handler func(string)

func panduan(in interface{}) {
	Handler(in)("wujunbin")
}

func main() {
	panduan(Handler(ServeHTTP))
}
```

根据错误提示是说要我们先进行类型断言才可以继续使用该类型的函数

```go
if v, ok := in.(Handler); ok {
	//跟什么类型判断就只能调用什么类型的方法
	v("BrainWu")
}
```

只有让传进来的in参数先与Handler进行类型判断 如果返回值是OK则代表类型相同才能进行对应的方法调用

另外进行类型断言之后如果断言成功 就只能使用该类型的方法比如对一个结构体S进行与A接口断言

S实际上实现了A B两个接口 // 待进一步理解？？

A interface 具有 a()方法  B interface 具有 b()方法 如果结构体S作为参数被传入一个函数中并且在该函数中是interface{}类型

那么进行与A的类型断言之后就只能调用a()而不能调用b()因为编译器只知道你目前是A类型却不知道你目前也是B类型。


3 另外讲解 switch与类型断言的结合使用还是比较方便的, 比如下面这个例子

```go
package main

import (
	"fmt"
)

type Element interface {}

func main() {
	var e Element = 100
	switch value := e.(type) {
	case int:
		fmt.Println("int", value)
	case string:
		fmt.Println("string", value)
	default:
		fmt.Println("unknown", value)
	}
}
```

打印结果：

```sh
int 100
```


* 参考链接： https://blog.csdn.net/cbmljs/article/details/82966907


__注__: 
* 类型选择中的声明与类型断言 i.(T) 的语法相同，只是具体类型 T 被替换成了关键字 type。

## Stringer


## Reader

## Goroutines和Channels
1. 如果我们使用了无缓存的channel, 那么两个慢的goroutines将会因为没有人接收而永远卡住。这种情况，称为goroutines泄漏，这将是一个BUG.和垃圾变量不同，泄漏的goroutines并不会被自动回收，因此确保每个不再需要的goroutine能正常退出是重要的。
2. 当 for range 遍历 channel 时，如果发送者没有关闭 channel 或在 range 之后关闭，都会导致 deadlock(死锁)。
3. 同步的 channel 千万不要在同一个 goroutine 协程里发送和接收数据。可能导致deadlock死锁。
4. 对于没有缓冲区的 channel (unbuffered channel) 容易产生死锁的几个代码片段分析，总结下：
   	- channel 要用 make 进行初始化操作
   	- 读取和写入要配对出现，并且不能在同一个 goroutine 里
   	- 一定先用 go 起一个协程执行读取或写入操作
   	- 多次写入数据，for 读取数据时，写入者注意关闭 channel(代码片段5)
5. 有缓冲区的channel总结：
 	- 如果 channel 满了，发送者会阻塞
 	- 如果 channle 空了，接收者会阻塞
 	- 如果在同一个 goroutine 里，写数据操作一定在读数据操作前

## Labels in Go

### `goto`

```go
func main() {
    fmt.Println(1)
    goto End
    fmt.Println(2)
End:
    fmt.Println(3)
}
```

```sh
> ./bin/sandbox
1
3
```
 
Labels are not block scoped (more about blocks and scoping in “Blocks in Go” and “Scopes in Go”) so it’s impossible to redeclare label inside nested block:

```go
    goto X
X:
    {
    X:
    }
```

as Go compiler will complain about already declared label.

__注__: goto cannot move into other block

Identifiers of the labels live in a separate space so they don’t conflict with i.e. variables identifiers. The code below works just fine even that x is used for two things:

```go
    x := 1
    goto x
x:
    fmt.Println(x)
```

### `break`

`break` statement is traditionally used to terminate innermost `for` or `switch` statement. In Go it’s also possible to end this way novel `select` statement.

People having experience with languages like `C` and derivates know that each case clause needs to have break as a last statement. Otherwise execution in transferred to the next case clause which is not what programmer wants most of the time. 

Go however works conversely — there is a `fallthrough` statement to explicitly move to the next clause:

```go
switch 1 {
case 1:
    fmt.Println(1)
case 2:
    fmt.Println(2)
}
```

```sh
> ./bin/sandbox
1
```

```go
switch 1 {
case 1:
    fmt.Println(1)
    fallthrough
case 2:
    fmt.Println(2)
}
```

```sh
> ./bin/sandbox
1
2
```

没带标签的`break`语句仅能终止最内层的`for`，`switch`或是 `select`， 添加了标签,`break`则提供了更多地可能性。

Label for `break` statement must be the one associated with enclosing `for`, `switch` or `select` statement.

```go
SwitchStatement:
    switch 1 {
    case 1:
        fmt.Println(1)
        for i := 0; i < 10; i++ {
            break SwitchStatement
        }
        fmt.Println(2)
    }
	fmt.Println(3)
```

```sh
> ./bin/sandbox
1
3
```
__注__: `break` statement cannot span over function boundaries

__注__: It’s possible to terminate `for`, `switch` or `select` no matter where break statement is located.

### `continue` statement

It works in a similar way to `break` statement but begins next iteration instead of stopping and can be used only for loops:

```go
OuterLoop:
    for i := 0; i < 3; i++ {
        for j := 0; j < 3; j++ {
            fmt.Printf(“i=%v, j=%v\n”, i, j)
            continue OuterLoop
        }
    }
> ./bin/sandbox
i=0, j=0
i=1, j=0
i=2, j=0
```

## 疑问

### 1. 结构体 函数定义 如下两种方式的不同点

#### 方式1

```go
func (t *T) M() {
	fmt.Println(t.S)
}

func main() {
	var i I = T{"hello"}
	i.M()
}
```

#### 方式2

```go
func (t T) M() {
	fmt.Println(t.S)
}

func main() {
	var I = T{"hello"}
	I.M()
}
```

__注解__: 
	
* 这两种方式均可以为结构体定义函数，方式1是为结构体指针对象 
	新增函数，方式2是为结构体对象新增函数。

* 场景不同，定义函数的方式也就不同，但是更推荐使用方式1，因为我们在声明、使用结构体变量多数也是作为指针变量使用的。

### 2. fmt Print/Println/Printf/Sprintf的区别
* `fmt.Print` 不会自动换行，也不能处理占位符。
* `fmt.Println` 会自动换行，但不能处理占位符。
* `fmt.Printf` 它通过 `os.Stdout` 输出格式化的字符串，可以处理占位符。
* `fmt.Sprintf` 则格式化并返回一个字符串而不带任何输出。

示例代码：

```go
package main

import ("fmt")
func main() {
	var test1 int = 1

	var test2 int = 2

	fmt.Println("Println输出，不能处理占位符：%d", test1)

	fmt.Print("Print输出, 并没有换行", test1)
	fmt.Println("Println输出", test1)


	fmt.Printf("Printf输出，可以解析占位符：%d, 不会自动换行", test1)

	fmt.Printf("Printf输出,test1: %d, test2: %d", test1, test2)
}

```

运行输出

```sh
Println输出，不能处理占位符：%d 1
Print输出, 并没有换行1Println输出 1
Printf输出，可以解析占位符：1, 不会自动换行Printf输出,test1: 1, test2: 2123
```

### 3. 字符串与数字转换
#### 字符串转为int64

```go
var string1 = "123"
stringInt64, err := strconv.ParseInt(string1, 10, 64)
```

```go
var a = "123"
if b, error := strconv.Atoi(a); error == nil {
	fmt.Printf("b: %T, %d", b, b)
}
```

* 数字拼接为字符串

```go
d := strconv.Itoa(1234)
fmt.Printf("string: %s", d)
```

* int 转换为二进制、十六进制
```go
binary := strconv.FormatInt(int64(i), 2)
hexadecimal := strconv.FormatInt(int64(i), 16)
```

* 字符串拼接

```go
newString := strings.Join([]string{"string01", "string02"}, "-")
fmt.Printf("newString: %s ", newString)
```

* 字符串前、后缀

```go
// 前缀
strings.HasPrefix(s, prefix string) bool

// 后缀
strings.HasSuffix(s, suffix string) bool
```

* 字符串包含

```go
strings.Contains(s, substr string) bool
```

### 4. 数字操作

#### 官方的math 包中提供了取整的方法，向上取整math.Ceil() ，向下取整math.Floor()

```go
package main
import (
	"fmt"
	"math"
)
func main(){
	x := 1.1
	fmt.Println(math.Ceil(x))  // 2
	fmt.Println(math.Floor(x))  // 1
}
```

### 5. 切片文法、数组文法 使用场景，时间复杂度？
	
#### 切片合并

```go
	a := []int{1, 2, 3}
	b := []int{2, 3, 4, 5, 6}
	a = append(a, b...)
```
	

### 6. omitempty 在go中的使用

示例：
```go
type Person struct {
    Name string `json:"name"`
    Age  int    `json:"age"`
    Addr string `json:"addr,omitempty"`
}
```

* 有了omitempty后，如果addr为空， 则生成的json中没有addr字段。
* 如果没有定义 omitempty，则始终会返回addr这个字段。

### Refer to
1. [Go << and >> operators](https://stackoverflow.com/questions/5801008/go-and-operators)
2. [Labels in Go](https://medium.com/golangspec/labels-in-go-4ffd81932339)
