# Dart入门

**未完成**

## 前言 

最近在学习`flutter`，用到了`dart`那么我们来仔细的搞一搞`dart`语言

## `Dart`是什么 ？

> Dart 是一种 易于学习、 易于扩展、并且可以部署到 任何地方 的 应用 编程 语言。

具体的描述可以看`Dart`[官网](http://dart.goodev.org/)

## 学习环境的搭建

我们要搭建起码的`sdk`为学习编译做准备， 搭建简单的开发环境可以参考[这里](https://blog.csdn.net/u012331525/article/details/81710841), 过程也很简单。

## 开始学习

下面开始正式学习`Dart`语言

### 基础部分

## 回过头来看问题

带着一些有价值的问题，有目的的去学习`Dart`

- `Dart`是值类型还是引用类型 ？ 
  
  答：`Dart`是**引用类型**。我们每次调用函数，传递过去的都是对象的内存地址，而不是这个对象的复制

- `Dart`数值型介绍一下 ？
  `dart:core` 库定义了 `num`, `int`, 和 `double` 类，这些类 定义一些操作数字的基础功能。

  其中`num`是`int`和`double`的父类，在使用的的时候三种类型都可以使用

  使用`parse()`方法可以把字符串 转换为 整数或者浮点数。
  使用`toString/toStringAsFixed()`方法可以把数字转化为字符串/限定位数的字符串。
  

- `Dart`中`..`操作符的作用？
  

- `Dart`中`Object`和`dynamic `有什么区别 ？
  
  在`Dart`中有两种类型: `Object` 和 `dynamic`。
  **`Object`**  适合表示可以是任何**对象**的场景。
  **`dynamic`**  适用于更复杂的场景，可以表示更多的类型， 如意味着 `Dart` 的类型系统已经不足以表示的一系列允许的类型，或者值来自 `interop` 或 其他超过静态类型系统范围的类型，或者你想明确地声明运行时动态处理的变量, 也可以理解为是一种任意的类型。

  ```dart
  void log(Object object) {
    print(object.toString());
  }
  /// Returns a Boolean representation for [arg], which must
  /// be a String or bool.
  bool convertToBool(dynamic arg) {
    if (arg is bool) return arg;
    if (arg is String) return arg == 'true';
    throw new ArgumentError('Cannot convert $arg to a bool.');
  }
  ```

- `Dart`中的闭包是什么样的 ？


- `Dart`中的模块化？
  `dart`中的模块是以`package`的形式存在，比如在模块1中定义的`class`在模块2中引用之后可以直接使用。可以通过绝对路劲个或者相对路劲进行引用。还是很好理解的
  
  `module1.dart`
  ```dart
  class Todo {
    final String who;
    final String what;
    const Todo(this.who, this.what);
  }
  ```
  `module2.dart`
  ```dart
  import 'package:项目名/module1.dart'; // 绝对路径引入
  // import './module1.dart'; // 相对路径引入
  // 可以直接使用Todo类
  ```


