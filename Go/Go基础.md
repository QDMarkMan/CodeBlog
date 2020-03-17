# Go基础

[toc]

前一段时间开始学习`Go`语言，在这里记录一下基础。

## 数据类型

任何一门语言的语言类型都是基础中的重点，都必须要弄清楚。

- `Boolean`类型

  值： `true/falgse`

  定义：

  ```go
  var boolType bool
  boolType = false
  ```

- `Number`类型

  值：

  - `uint8`：无符号8位整数。

  - `uint16`：无符号16位整数。

  - `uint64`：无符号64位整数。

  - `int8`：有符号8位整数。

  - `int16`：有符号16位整数。

  - `int32`：有符号32位整数。

  - `int64`：有符号64位整数。

  - `float32`：32位浮点数。

  - `float64`：64位浮点数。

  - `complex64`：32位实数和虚数。

  - `complex128`：64位实数和虚数。

  - `byte`：类似`uint8`。

  - `rune`：类似`int32`。

  - `uint`：32或64位数。

  - `int`：与`uint`一样大小。

  - `uintptr`：无符号整型，用于存放指针。

    

  定义：

  ```go
  var numType int
  ```

- `string`：字符串类型

  值：utf-8类型字符串。

  定义： 

  ```go
  var strType string
  strType = "string"
  strType = `
  	多行字符串
  `
  ```

- `nil`： 未初始化引用

  以下类型默认初始值： 

  ```go
  pointers -> nil
  slices -> nil
  maps -> nil
  channels -> nil
  functions -> nil
  interfaces -> nil
  ```

  

- `pointer`：指针类型

  值：使用&，* 完成操作。

  ![image-20200316221709742](/Users/etongfu/Documents/Code/CodeBlog/Go/images/pointer.png)

  定义：

  ```go
  p := 42
  pv := &p
  *p++ // p= 43
  ```

  用途：

  - 类型指针：对指针指向的数据进行修改。
  - 函数穿参使用指针：避免值传递拷贝额外数据。

- `Array`：数组类型。

  值： 大小固定的容器。

  定义：

  ```go
  // 默认初始化
  var arr [3]int
  // 指定长度和初始值
  var arrLen [4]int{1, 2, 3, 4}
  // 指定初始值
  var autoLenArr [...]int{1,2,3,4,5,6}
  ```

- `Slice`：切片类型。

  值：大小不固定的容器。

  定义：

  ```go
  // 1: 直接声明切片
  var varSlice []type
  // 2: 声明空切片
  var emptySlice = []type{} 
  // 3: 使用make
  // make([]type, length, capacity)
  // len是长度，cap是预分配的空间
  makeSlice := make([]int, 2, 10)
  // 4: 从数组中截取切片
  var arr = [4]int{1,2,3,4}
  var sliceArr = arr[1:2] // [1]
  ```

  操作：

  ```go
  // 切片新增元素
  ```

  