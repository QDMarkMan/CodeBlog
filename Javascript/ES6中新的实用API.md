# ES6中新的实用API
> ES6中更新了非常多的API，一大部分还是非常的强大。这次来吃透这些新的API。

## Symbol
> ES6 引入了一种新的原始数据类型Symbol，表示独一无二的值。

`Symbol`作为一种原始类型出现在新的`API`中。它的作用在我看来是**在根本上去防止属性名的冲突**。

### 创建一个Symbol值

创建一个Symbol很简单，`Symbol` 值通过`Symbol`函数生成。
```js
// 创建Symbol
let s = Symbol()
console.log(typeof s) // symbol 所以标识Symbol是一个原始类型
```
注意：**因为生成的 Symbol 是一个原始类型的值，不是对象。我们不能使用`new`命令**
Symbol函数可以接受一个字符串作为参数，表示`对 Symbol 实例的描述`，主要是为了在控制台显示，或者转为字符串时，比较容易区分。如果不加参数，它们在控制台的输出都是`Symbol()`。
```js
let s1 = Symbol('s1')
let s2 = Symbol('s2')
console.log(s1, s1.toString())// Symbol(s1) "Symbol(s1)"
console.log(s2, s2.toString())// Symbol(s2) "Symbol(s2)"
```
如果参数是一个对象，那么就会调用该对象的`toString`方法，将其转为字符串，然后才生成一个 `Symbol` 值。
```js
const obj = {
  a: 1
}
let s3 = Symbol(obj)
console.log(s3) // Symbol([object Object])
```
注意：**`Symbol`函数的参数只是表示对当前 `Symbol` 值的描述，因此即使参数相同，`Symbol`函数的返回值夜市是不相等的。**

### 实际用途
- 作为对象的属性名

由于每一个 `Symbol` 值都是不相等的，这意味着 `Symbol` 值可以作为标识符，用于对象的属性名，就能保证不会出现同名的属性。这对于一个对象由多个模块构成的情况非常有用，能防止某些队友不小心改写或覆盖了某一个键。

使用`Symbol`有3种写法
```js
// 当作键名来使用
let objSymbol = Symbol()
let obj1 = {}
obj1[objSymbol] = '第一种写法'
let obj2 = {
  [objSymbol]: '第二种写法'
}
let obj3 = {}
Object.defineProperty(obj3, objSymbol, {
  value: '第三种写法'
})
const obj4 = {}
obj4.objSymbol = '字符串属性!' // 这个是加上了个普通的字符串属性，并不是个Symbol属性
obj4[objSymbol] // undefined
obj4['objSymbol'] // "字符串属性!"
```
`Symbol`作为属性的时候，不能再用点操作符去访问了,因为点运算符后面总是字符串,**在对象的内部，使用 Symbol 值定义属性时，Symbol 值必须放在方括号之中。**

- 作为唯一的常量使用

用于定义一组常量，保证这组常量的值都是不相等的。常量使用 `Symbol` 值最大的好处，就是其他任何值都不可能有相同的值了。
```js
obj4['objSymbol'] // "字符串属性!"
// 作为常量使用
const RED = Symbol('red')
const GREEN = Symbol('green')
function getComputed (color) {
  switch (color) {
    case RED: // 保证条件是唯一的
        return RED
      break;
    case GREEN:
      return GREEN
    break;
    default:
      throw new Error('Color is undefined')
      break;
  }
}
console.log(getComputed(RED)) // Symbol(red)
```

- 消除魔术字符串 
- 在`Singleton`模式中使用

### 使用Symbol时的注意点
- **不能使用`new`命令** <small>原因如上</small>
- **即使参数相同，`Symbol`函数的返回值夜市是不相等的。**<small>原因如上</small>
- **`Symbol` 值不能与其他类型的值进行运算** <small>会报错：`Uncaught TypeError`</small>
- **`Symbol`可以转为字符串，布尔值。但是<font color="red">不能转为数值</font>**
- **`Symbol` 值作为对象属性名时，不能用点运算符。**
- **`Symbol` 在遍历中表现比较特殊**
  `Symbol`在作为属性名的时候该属性不会出现在`for...in、for...of`循环中。也不会被`Object.keys()、Object.getOwnPropertyNames()、JSON.stringify()`返回。但是它也不是那种奇妙的私有属性。如果我们要想遍历到它，常规情况下通过`Object.getOwnPropertySymbols`方法，可以**获取指定对象的所有 `Symbol` 属性名。**
  ```js
  const symObj = {
    [s1]: 's1',
    [s2]: 's2'
  }  
  for (const key in symObj) {
    console.log('symbol key') // 不会输出 甚至检测不到key
    console.log(key) // 没有作用
  }
  const objectSymbols = Object.getOwnPropertySymbols(symObj)
  console.log(objectSymbols) // [Symbol(s1), Symbol(s2)]
  ```
  我们再和`getOwnPropertyNames`对比一下
  ```js
  // 和`getOwnPropertyNames`对比
  const obj4 = {}
  let s4 = Symbol('s4')
  Object.defineProperty(obj4, s4, { 
    value: 'value'
  })
  for (const key in obj4) {
    console.log(key) // 无输出
  } 
  console.log(Object.getOwnPropertyNames(obj4)) // []
  console.log(Object.getOwnPropertySymbols(obj4)) // [Symbol(s4)]
  ```js
  同时还有一个`Reflect.ownKeys`可以获取到所有的key
  ```js
  // 获取全部的keys
  const allKeys = {
    [Symbol('key')] : 'symbol',
    num: 1
  }
  console.log(Reflect.ownKeys(allKeys)) // ["num", Symbol(key)]
  ```




## Set And Maps

## Proxy

## Promise

## Iterator

## Relfact