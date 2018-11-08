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

### 相同的Symbol值

`Symbol`虽然是唯一值。但是我们也可以声明除两个完全相等的`Symbol`对象。

- `Symbol.for(string)` 
  `Symbol.for`方法可以做到让我们重新使用一个`Symbol`值它接受一个字符串作为入参 ==> **。如果有，就返回这个 Symbol 值，否则就新建并返回一个以该字符串为名称的 Symbol 值。** `Symbol`会被登记在全局环境中供搜索，`Symbol.for`不会。`Symbol`没调用一次就会生成一种Symbol值。但是`Symbol.for`在参数不变的情况下只会生成一个Symbol值，也称为**登记机制**
  ```js
  //symbol.for
  let sfor1 = Symbol.for('string')
  let sfor2 = Symbol.for('string')
  console.log(sfor1 === sfor2) // true
  ```
- `Symbol.keyFor`
  `Symbol.keyFor`方法返回一个已登记的 Symbol 类型值的key。
  ```js
  console.log(Symbol.keyFor(sfor1)) // string
  ``` 
  注意： **Symbol.for为 Symbol 值登记的名字，是全局环境的，可以在不同的 iframe 或 service worker 中取到同一个值。**

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

魔术字符串指的是，在代码之中多次出现、与代码形成强耦合的某一个具体的字符串或者数值。可以理解为我们通常自己约定的一种状态用字符串来代表,而且在代码种多次出现，与代码强耦合。
```js
function getArea(shape, options) {
  let area = 0;

  switch (shape) {
    case 'Triangle': // 魔术字符串
      area = .5 * options.width * options.height;
      break;
  }
  return area;
}
getArea('Triangle', { width: 100, height: 100 }); // 魔术字符串
```
使用**变量 + `Symbol`**的方式去解决这个问题
```js
const symbolType = {
  triangle: Symbol('triangle')
}
function getArea (shape, options) {
  let a = 0
  switch (shape) {
    case symbolType.triangle:
     a =  options.width * options.height
    break
  }
  return a
}
getArea(symbolType.triangle, {width: 100, height: 100})
```
其实`triangle`的值是什么不重要，只要它不和别的值产生冲突这才最重要。所以这里适合是哟个`Symbol`。

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

### Set


### Map
- `Map`基础
`Map`的出现是为了解决传统`Object`只支持`字符串-值`的`hash`结构。`Map`类似于对象。但是`键`的值不仅限于字符串。
> `Map` 结构提供了`值—值`的对应，是一种更完善的 `Hash` 结构实现。如果你需要“键值对”的数据结构，Map 比 Object 更合适。

但是在使用上`Map`和普通的`Object`就不太一样了。
```js
// Map的基本使用
const map = new Map()
const mapObj = {m: 'map'}
map.set(mapObj, 'this is value')
console.log(map.get(mapObj)) // this is value

console.log(map.has(mapObj)) // true
console.log(map.delete(mapObj)) // true
console.log(map.has(mapObj)) // fasle
```
通过构造函数生成了一个`Map`实例，然后`set`方法赋值，`get`方法取值 `has`方法检测key, `delete`方法删除值。

构造函数还接受**数组**作为入参，该数组的**成员是一个个表示键值对的数组。**。参数看起来有点怪，但是仔细看一看也在情理之中。
```js
// 数组入参
const arrMap = new Map([
  ['name', 'arr'],
  ['title', 'map']
])
console.log(arrMap.has('name'))
console.log(arrMap.get('name')) // arr
console.log(arrMap.get('title'))// map
```
但是数组作为参数，实际上是一个语法糖。是执行了下面的算法
```js
// 语法趟
const arr = [
  ['name', '张三'],
  ['title', 'Author']
]
let map2 = new Map()
arr.forEach(
  ([key, value]) => map2.set(key, value)
)
console.log(map2) // Map(2) {"name" => "张三", "title" => "Author"}
```
事实上，不仅仅是数组，**任何具有 `Iterator` 接口、且每个成员都是一个双元素的数组的数据结构都可以当作Map构造函数的参数。这就是说，Set和Map都可以用来生成新的 Map。**
```js
// set和map作为构造函数
const mapSet = new Set([
  ['foo', 1],
  ['bar', 2]
])
const setMap = new Map(mapSet)
console.log(setMap) // Map(2) {"foo" => 1, "bar" => 2}
const mapMap = new Map([['name', 'mapMap']]) 
const mapMap2 = new Map(mapMap)
console.log(mapMap2) // Map(1) {"name" => "mapMap"}
```
注意点: **只有对同一个对象的引用，Map 结构才将其视为同一个键。这一点要非常小心。**,看起来是同一个值但是实际上并不是，因为`set`进去的数组指向两个不同的地址。
```js
// 键的引用问题
const b = ['b']
const refMap = new Map()
refMap.set(['a'], 555)
refMap.set(b, '指向同一地址')
console.log(refMap);
console.log(refMap.get(['a'])) // undefined
console.log(refMap.get(b)) // 指向同一地址
```
- `Map`内置属性和操作方法

  1. `size`属性
  `size` => 返回`Map`对象中的成员总数
  ```js
  // Map属性
  console.log(refMap.size) // 2
  ```
  2. `set(key, value)` => 设置key值 返回当前`Map`对象，可以使用**链式写法**
  3. `get(key)` => 获取key值，返回key对应的值 没有的话返回`undefined`
  4. `has(key)` => 查询key在不在map中，返回`true/false`
  5. `delete(key)` => 删除某个key，成功返回`true`,失败返回`fasle`
  6. `clear()` => 清除所有成员，没有返回值。

- 遍历
  1. `keys()` => 返回键名的遍历器。
  2. `values()` => 返回键值的遍历器
  3. `entries()` => 返回所有成员的遍历器
  4. `forEach` => 遍历 `Map` 的所有成员
  ```js
  // 遍历
  const forMap = new Map([
    ['for', 'no'],
    ['map',  'yes'],
  ])
  // key
  for (const key of forMap.keys()) {
    console.log(key) // for map
  }
  for (const key of forMap.values()) {
    console.log(key) // no yes
  }
  for (const key of forMap.entries()) {
    console.log(key) //  ["for", "no"] ["map", "yes"]
  }
  // 结构显示
  for (let [key, value] of forMap.entries()) {
    console.log(key, value) // for no, map yes
  }
  // forEach
  forMap.forEach(function(value, key, map) {
    console.log("Key: %s, Value: %s", key, value);
  },this) // Key: for, Value: no  , Key: map, Value: yes
  ```
- 和其他数据结构的转换
  1. `Array` <==> `Map`
    数组转成`Map`的话直接使用构造函数,`Map`转成数组也很方便，使用展开运算符就行了
    ```js
    // 数据结构间的转换
    const mapToArr = [...forMap]
    console.log(mapToArr) // ) [Array(2), Array(2)]
    ```
  2. `Object` <==> `Map`
    Map转为对象之前有一个前提： **如果所有 Map 的键都是字符串，它可以`无损`地转为对象**，如果key值不是字符串，那么会被进行转化。
    ```js
    // 对象和Map
    function mapToObject(map) {
      let obj = {}
      for (const [k, v] of map.entries()) {
        obj[k] = v
      }
      return obj
    }
    console.log(mapToObject(forMap)) // {for: "no", map: "yes"}
    function objToMap(object) {
      const map = new Map()
      for (const key in object) {
        if (object.hasOwnProperty(key)) {
          map.set(key, object[key])
        }
      }
      return map
    }
    console.log(objToMap({ka:'va', kb: 'vb'})) // Map(2) {"ka" => "va", "kb" => "vb"}
    ```

  3. `JSON` <==> `Map`
  这个分两种情况： 
  1. `Map` 的键名都是字符串 这种情况就和上面的对象是一样的了。
  2. `Map` 的键名有非字符串，这时可以选择转为数组 `JSON`，我觉得一般也没什么用处
  ```js
  // 2 数组json
  let jsonMap = new Map().set(true, 7).set({foo: 3}, ['abc'])
  function mapToJson (map) {
    return JSON.stringify([...map])
  }
  console.log(mapToJson(jsonMap)) // [[true,7],[{"foo":3},["abc"]]]
  ```


## Proxy

## Promise

## Iterator

## Relfact