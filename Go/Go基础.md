# Go基础

[toc]

前一段时间开始学习`Go`语言，在这里记录一下基础。

## 数据类型

任何一门语言的语言类型都是基础中的重点，都必须要弄清楚。

- `Boolean`类型

  值： `true/falgse`

  示例：

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

    

  示例：

  ```go
  var numType int
  ```

- `string`：字符串类型

  值：utf-8类型字符串。

  示例： 

  ```go
  var strType string
  strType = "string"
  strType = `
  	多行字符串
  `
  ```

- `nil`： （空值/零值）未初始化引用

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

  核心概念：

  - 类型指针，允许对这个指针类型的数据进行修改，传递数据可以直接使用指针，而无须拷贝数据，类型指针不能进行偏移和运算。
  - 切片，由指向起始元素的原始指针、元素数量和容量组成。

  ![image-20200316221709742](/Users/etongfu/Documents/Code/CodeBlog/Go/images/pointer.png)

  示例：

  ```go
  p := 42
  pv := &p
  *p++ // p= 43
  ```

  用途：

  - 类型指针：对指针指向的数据进行修改。
  - 函数穿参使用指针：避免值传递拷贝额外数据。

- `Array`：数组（引用类型）。

  定义：数组是一个由固定长度的特定类型元素组成的序列，一个数组可以由零个或多个元素组成。因为数组的长度是固定的，所以在Go语言中很少直接使用数组。

  值： 大小固定的容器。

  示例：

  ```go
  // 默认初始化
  var arr [3]int
  // 指定长度和初始值
  var arrLen [4]int{1, 2, 3, 4}
  // 指定初始值
  var autoLenArr [...]int{1,2,3,4,5,6}
  ```

- `Slice`：切片（切片类型）。

  定义：对数组的一个连续片段的引用，所以切片是一个引用类型（因此更类似于 C/[C++](http://c.biancheng.net/cplus/) 中的数组类型，或者 [Python](http://c.biancheng.net/python/) 中的 list 类型），这个片段可以是整个数组，也可以是由起始和终止索引标识的一些项的子集，需要注意的是，终止索引标识的项不包括在切片内。

  值：大小不固定的容器。

  示例：

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
  // ------------------------------ 切片操作 ------------------------------ //
  var sliceA []int
  // 切片新增元素
  sliceA = append(sliceA, 1, 2, 3)
  
  // 切片复制
  sliceB := make([]int, 3)
  copy(sliceB, sliceA) // sliceB = [1 2 3] sliceA = [1 2 3]
  
  // 切片删除元素
  // sliceA = sliceA[1:]                        //删除开头1个元素
  sliceA = append(sliceA[:0], sliceA[1:]...) // 不移动数据指针，但是将后面的数据向开头移动，可以用 append 原地完成
  fmt.Println(sliceA)
  sliceB = append(sliceB[:2], sliceB[3:]...) //  a = append(a[:i], a[i+N:]...) // 删除中间N个元素
  sliceA = sliceA[:len(sliceA)-1]            // a = a[:len(a)-N] // 删除尾部N个元素
  
  // 遍历切片
  for index, value := range sliceB {
    fmt.Println(index, value)
  }
  
  // ------------------------------ 二维切片 ------------------------------ //
  
  // 声明二维切片
  var slice [][]int
  // 赋值
  slice = [][]int{{1, 2}, {3, 4}} // [[1 2] [3 4]]
  
  ```

- `Map`：映射（引用类型）

  定义： 一种元素对（pair）的无序集合，pair 对应一个 key（索引）和一个 value（值），所以这个结构也称为关联数组或字典，这是一种能够快速寻找值的理想结构，给定 key，就可以迅速找到对应的 value

  值：key-value 形式的容器。