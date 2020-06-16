---
layout: post
title: JS语法知识汇总
subtitle: 
tags: [js]
comments: true
---

## 数据类型

```
var length = 7;                             // 数字
var lastName = "Gates";                      // 字符串
var cars = ["Porsche", "Volvo", "BMW"];         // 数组
var x = {firstName:"Bill", lastName:"Gates"};    // 对象 
var x = true;     // 布尔值
var y = false;    //布尔值
```

### 数组添加元素
1. push() 结尾添加

| 参数 | 描述 | 
| ------ | ------  |
| newelement1	| 必需。要添加到数组的第一个元素。|
| newelement2	| 可选。要添加到数组的第二个元素。|
| newelementX	| 可选。可添加多个元素。|

2. unshift() 头部添加

| 参数 | 描述 | 
| ------ | ------  |
| newelement1	| 必需。要添加到数组的第一个元素。|
| newelement2	| 可选。要添加到数组的第二个元素。|
| newelementX	| 可选。可添加多个元素。|

3. splice() 方法向/从数组指定位置添加/删除项目，然后返回被删除的项目。

| 参数 | 描述 | 
| ------ | ------  |
| index	| 必需。整数，规定添加/删除项目的位置，使用负数可从数组结尾处规定位置。|
| howmany	| 必需。要删除的项目数量。如果设置为 0，则不会删除项目。|
| item1, ..., itemX	| 可选。向数组添加的新项目。|


### 数组删除元素
1. splice 方法

如：```array.splice(index, nums);```

__注__: 数组长度相应改变,但是原来的数组索引也相应改变，splice参数中第一个index,是删除的起始索引(从index算起); 第二个nums,是删除元素的个数.

此时遍历数组元素可以用普通遍历数组的方式,比如for,因为删除的元素在数组中并不保留。


2. delete 方法

如： ```delete array[index] ```

__注__：这种方式数组长度不变,此时 array[index]对应的index会变为undefined,好处是原来数组的索引也保持不变,此时要遍历数组元素可以才用.这种遍历方式跳过其中undefined的元素，所以非常实用。

获取数组下表的函数

```

/*
* 获取某个元素下标
*
*       arrays  : 传入的数组
*
*       obj     : 需要获取下标的元素
* */
function contains(arrays, obj) {
    var i = arrays.length;
    while (i--) {
        if (arrays[i] === obj) {
            return i;
        }
    }
    return false;
```

### 数组的内置函数

1. ```slice(start, end)``` 方法可从已有的数组中返回选定的元素。
    | 参数 | 描述 |
    | ------ | ------  |
    | start | 必需。规定从何处开始选取。如果是负数，那么它规定从数组尾部开始算起的位置 |
    | end   | 可选。规定从何处结束选取。该参数是数组片断结束处的数组下标。如果没有指定该参数，那么切分的数组包含从 start 到数组结束的所有元素。|

    示例：

    ```
    <script type="text/javascript">

    var arr = new Array(3)
    arr[0] = "George"
    arr[1] = "John"
    arr[2] = "Thomas"

    document.write(arr + "<br />")
    document.write(arr.slice(1) + "<br />")
    document.write(arr)

    </script>
    ```

    输出：

    ```
    George,John,Thomas
    John,Thomas
    George,John,Thomas
    ```

2. 连接数组

    ```
    var arr1=[1,2,3];
    var arr2=[4,5,6];
    var arr3=[7,8,9];
    var arr=arr1.concat(arr2,arr3,[10,11]);
    console.log(arr);//输出 [1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11]
    ```

3. 数组转换为字符串

    ```
    var arr=new Array(1,2,3,4,5);
    var str=arr.join('-');//间隔的字符串，默认为“，”
    console.log(str);//1-2-3-4-5
    ```

4. 查找字符位置

    ```
    indexOf();   // 通过字符找位置，（找相同单词的第一个）
    lastIndexOf(); // 通过字符串找位置，（找相同单词最后一个）
    ```

### 对象添加元素

```
var jsonObj={

      'param1':22,

      'param2' :33

};

 

jsonObj.newParam ='pre';

// 新的属性添加以后，json对象变成：

var jsonObj={

      'param1':22,

      'param2' :33,

      'newParam':'pre'

};
```

### 对象删除元素

```
delete jsonObj.age;
//结果：Object { id: 1, name: "danlis" }
```

__注__： 
1. 对象序列转化为json格式的字符串：```JSON.stringify(jsObj)```
2. 对象反序列化（由json格式的字符串转化为javascript对象）```JSON.parse('{"name": "小明", "age": 14}')```


### 比较运算符

假定 x = 5;

| 运算符 | 描述 | 比较 | 返回 |
| ------ | ------ | ------ |------ |
| == | 等于     | x == 8      | false |
| === | 值相等并且类型相等  | x === 8 | true |
| != | 不相等     | x != 8      | true |
| !== | 值不相等或类型不相等  | x !== 5 | false |

### 条件（三元）运算符

语法

```
variablename = (condition) ? value1:value2
```


实例

```
var voteable = (age < 18) ? "太年轻":"足够成熟";
```


## 常用指令

```
JSON.parse(jsonstr); //可以将json字符串转换成json对象 

JSON.stringify(jsonobj); //可以将json对象转换成json对符串 
```


```
// js 检查字典对象的长度
const dict = {1: 'aa',  2: 'bb', 3: 'cc'}
Object.keys(dict).length
```

```
// concat() 方法用于连接两个或多个数组。
// 该方法不会改变现有的数组，而仅仅会返回被连接数组的一个副本。

arrayObject.concat(arrayX,arrayX,......,arrayX)

```

```
// 字符串转化为 int
const str1 = '1'
parseInt(str1)

// 
```

```
// typeof operator returns a string indicating the type of the unevaluated operand.

if ( typeof {} === 'object') {
    console.log('correct, {} this is object')
}
```


```
/**
    * Performs the specified action for each element in an array.
    * @param callbackfn  A function that accepts up to three arguments. forEach calls the callbackfn function one time for each element in the array.
    * @param thisArg  An object to which the this keyword can refer in the callbackfn function. If thisArg is omitted, undefined is used as the this value.
    */
forEach(callbackfn: (value: T, index: number, array: T[]) => void, thisArg?: any): void;

this.tableData.forEach((_, index) => {
    if (this.tableData[index].Status === 'Running') {
        this.tableData[index].Status = '在线'
    } else if (this.tableData[index].Status === 'Stopped') {
        this.tableData[index].Status = '关闭'
    }
)

```

```
// 时间比较
console.log(new Date('2017-09-28T22:59:02.448804522Z').valueOf())

const currentTime = new Date('2017-09-28T22:59:02.448804522Z').valueOf()
    
const expiryTime = new Date('2017-09-29T22:59:02.448804522Z').valueOf()

if (currentTime < expiryTime) {
    console.log('not expired')
}

// 方式二
new Date('2017-09-28T22:59:02.448804522Z') > new Date()
// true
new Date('2017-09-28T22:59:02.448804522Z') < new Date()

```

```
// 遍历字典
let dict1 = {'user': ['liuyuan','colynn'], 'email': ['colynn@example.com', 'abc.liu@test.com']}

Object.keys(dict1).forEach((item)=>{console.log(item)})

```

## 参考
* [Javascript教程 A re-introduction to JavaScript](https://developer.mozilla.org/en-US/docs/Web/JavaScript/A_re-introduction_to_JavaScript)
