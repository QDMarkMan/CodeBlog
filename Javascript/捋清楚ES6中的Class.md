# 捋清楚ES6中的class

在近期工作中开始使用```class```这个关键字。便顺手总结了一些

## 关于类

> ECMAScript 2015 中引入的 JavaScript 类实质上是 JavaScript 现有的基于原型的继承的语法糖。类语法不会为JavaScript引入新的面向对象的继承模型。 --来自MDN

> class 声明创建一个基于原型继承的具有给定名称的新类。 --来自MDN

从上面这两段我们可以明确得看出，ES6中出现得类只是一个 **特殊得函数**。也就是说它并不是想Java等强类型中得class那样是个面向对象得继承模型。它实质上仅仅是一个语法糖而已。

定义中可以看的很明显：**基于原型继承**。其实看到这里我们大概能猜到这是个语法糖了。
> 基本上，ES6 的class可以看作只是一个语法糖，它的绝大部分功能，ES5 都可以做到，新的class写法只是让对象原型的写法更加清晰、更像面向对象编程的语法而已。上面的代码用 ES6 的class改写。 -- 来自阮一峰老师

### 基础定义

- 定义一个类

定义一个类非常的简单 直接使用`class`关键字就可以, 也可以使用*类表达式*来定义

类表达式是用来定义类的一种语法。和函数表达式相同的一点是，类表达式可以是命名也可以是匿名的。如果是命名类表达式，这个名字只能在类体内部才能访问到。JavaScript 的类也是基于原型继承的。
```js
// 关键字定义方式
class name [extends] {
  // class body
}
// 类表达式 className其实没什么用  因为这个class是一个匿名类
const MyClass = class [className] [extends] {
  // class body
}
```
根据执行函数， 我们可以通过类表达式写出来自自执行类
```js
// 自执行类
let AutoClass = new class Auto {
  constructor (name) {
    this.name = name
  }
  sayName () {
    console.log(this.name)
  }
}('自执行类')
AutoClass.sayName()
```

- 这个类到底是什么

ES6 的类，其实完全可以看作构造函数的另一种写法。通过下面的代码我们可以得知类的数据类型就是函数，类本身就指向构造函数。
```js
// 类其实就是函数
class Point {}
const ponit = new Point()
console.log(typeof Point)// "function"
Point = Point.prototype.constructor // true
```
注意：类必须使用`new`调用，否则会报错。这是它跟普通构造函数的一个主要区别，后者不用`new`也可以执行。


- `constructor`

`constructor`方法是类的默认方法，通过`new`命令生成对象实例时，自动调用该方法。一个类必须有`constructor`方法，如果没有显式定义，一个空的`constructor`方法会被默认添加。

有些时候我们可以显示去指定返回另外一个对象
```js
// 修改constructor中的this
class Foo {
  constructor () {
    return Object.create(null)
  }
}
console.log(new Foo() instanceof Foo) //false
```

- 类的实例对象

生成类的实例对象的写法，与 ES5 完全一样，也是使用`new`命令。而且与 ES5 一样，实例的属性除非显式定义在其本身（即定义在`this`对象上），否则都是定义在原型上（即定义在`class`上）。怎么理解呢： 定义在`this`上就是我这个实例中的，不然就是原型链中的。 如下面例子
```js
// 实例一个对象
class Instance {
  constructor(x, y) {
    this.x = x;
    this.y = y;
  }

  toString() {
    return '(' + this.x + ', ' + this.y + ')';
  }
}
const single = new Instance(1,2)
single.toString()
console.log(single.hasOwnProperty('x')) // true
console.log(single.hasOwnProperty('y'))// true
console.log(single.hasOwnProperty('toString'))// false
console.log(single.__proto__.hasOwnProperty('toString')) // true
```

- **私有方法和属性**

私有方法是常见需求，但 ES6 不提供，只能通过变通方法模拟实现。

一种做法是在命名上进行区分 二是是用`Symbol`进行区分
```js
// 私有方法和属性
// 1: 简单方式
class Self {
  // 公共方法
  say (name) {
    this._say(name)
  }
  // 私有方法
  _say (name) {
    return this.my = name
  }
}
// 2: 使用symbol实现
const bar = Symbol('bar')
const name = Symbol('name')
class MyClass {
  // 公共方法
  foo(bar) {
    this[bar](bar)
  }
  // 私有方法  bar和snaf都是Symbol值，导致第三方无法获取到它们，因此达到了私有方法和私有属性的效果。
  [bar](bar) {
    return this[name] = bar
  }
}
```
私有属性的提案

目前，有一个提案，为class加了私有属性。方法是在属性名之前，使用#表示。
```js
// 私有属性提案
class NewSelf {
  #name
  constructor(){
    #name = '私有属性提案'
  }
}
```

说到私有属性，不免多说一嘴， 用`ES5`如何实现私有属性和只读属性 ？
1. 使用闭包来实现私有属性
```js
// ES5闭包实现实现私有属性
const SelfEs5Func = function (name) {
  let temp = name
  this._name = function (){
    return temp
  }
}
let es5 = new SelfEs5Func('闭包中的私有属性')
console.log(es5._name())
```
2. 使用`Object.defineProperty()`实现只读属性
```js
// 使用object来实现
let SelfObject = {}
Object.defineProperty(SelfObject, 'freeze', {
  get () {
    return '我是个只读的对象'
  },
  set () {
    console.log('this is readonly')
  }
})
console.log(SelfObject.freeze);
SelfObject.freeze = '测试'
```












