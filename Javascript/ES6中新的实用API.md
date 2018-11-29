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
- `Set`基础

>ES6 提供了新的数据结构 Set。ES6 提供了新的数据结构 `Set`。**它类似于数组**，但是成员的值都是**唯一**的，没有重复的值。

`Set`是一个构造函数。它用来生成`Set`结构。
```js
// set
const set1 = new Set([1,2,2,3,3,3,3])
console.log(set1) // Set(3)
```
`Set` 函数可以接受一个数组（或者具有 `iterable` 接口的其他数据结构）作为参数，用来初始化。类数组对象也可以用来作为参数。也可以先创建一个`Set`对象，然后通过`add`方法向对象中新增值。
```js
// 数组对象
const arrayLikes = document.getElementsByTagName('div')
console.log(arrayLikes) // HTMLCollection
const set2 = new Set(arrayLikes)
console.log(set2.size) // 27
// 先创建set
const setFirst = new Set()
const setArr = [1,2,3,1,12,3,4,3,4,3,4]
setArr.forEach(element => {
  setFirst.add(element)
})
console.log(setFirst) //  Set(5) {1, 2, 3, 12, 4}
```

- `Set`内置属性和方法

  `Set`属性：
  - `Set.prototype.constructor`: 构造函数，默认就是`Set`函数
  - `Set.prototype.size`: 返回`Set`对象得成员总数

- `Set`方法
  
**操作方法（操作数据）**
  - `add(value)`: 新增数据,返回`Set`
  - `delete(value)`: 删除某个值，返回一个布尔值，表示删除是否成功。
  - `has(value)`: 这个值是否是`Set`的成员
  - `clear`: 清除所有的成员
  ```js
  let setMethod = new Set()
  setMethod.add(1).add(2).add(2) // Set(2) {1, 2}
  console.log(setMethod) // 
  setMethod.has(1) // true
  setMethod.delete(1) //
  console.log(setMethod) // Set(1) {2}
  setMethod.clear() 
  console.log(setMethod) // Set(0) {}
  ```
**遍历方法**
  1. `keys()` => 返回键名的遍历器。
  2. `values()` => 返回键值的遍历器 因为`Set`结构没有键名，只有键值（或者说键名和键值是同一个值），所以keys方法和values方法的行为完全一致。
  3. `entries()` => 返回所有成员的遍历器
  4. `forEach` => 遍历 `Map` 的所有成员
  ```js
  // 遍历方法
  let setFor = new Set(['red', 'green', 'blue'])
  console.log(setFor.keys()) // SetIterator {"red", "green", "blue"}
  for (const iterator of setFor.keys()) {
    console.log(iterator) // red green blue
  }
  console.log(setFor.values()) // SetIterator {"red", "green", "blue"}
  for (const iterator of setFor.values()) {
    console.log(iterator) // red green blue
  }
  console.log(setFor.entries())
  for (const iterator of setFor.entries()) {
    // entries方法返回的遍历器，同时包括键名和键值，所以每次输出一个数组，它的两个成员完全相等。
    console.log(iterator) //  ["red", "red"] ["green", "green"] ["blue", "blue"]
  }
  // forEach 方法
  setFor.forEach((value, key) => console.log(key + ' : ' + value))  
  ```
- `Set`实际使用
  1. 数组转化：
    
    `Array.from`方法可以将 `Set` 结构转为数组。
    ```js
    const setArrays = new Set([1, 2, 3, 4, 5])
    let arraySets = Array.from(setArrays)
    console.log(arraySets) // [1, 2, 3, 4, 5]
    ```
  2. 数组快速去重：
  
    配合`...`展开运算符快速去重。
    ```js
    // 配合`...`展开运算符快速去重。
    let numbers = [1,2,1,1,23,4,12,1,1]
    console.log([...new Set(numbers)])// [1, 2, 23, 4, 12]
    ```
  3. 并集（Union）、交集（Intersect）和差集（Difference）实现：
    ```js
    //求两个数组的集合
    const arrayA = [1,2,3], arrayB= [2,3,4]
    let setA = new Set(arrayA), setB = new Set(arrayB)
    // 并集
    const union = new Set(arrayA.concat(arrayB))
    console.log(union) //  Set(4) {1, 2, 3, 4}
    // 交集
    const intersect = new Set(arrayA.filter(x => setB.has(x)))
    console.log(intersect) // Set(2) {2, 3}
    // 差集
    const differenceA = arrayA.filter(x => !setB.has(x))
    const diefferenceB = arrayB.filter(x => !setA.has(x))
    const difference = new Set([...differenceA, ...diefferenceB])
    console.log(difference) // set(2) {1, 4}
    ```

- `WeakSet`
> WeakSet 结构与 Set 类似，也是不重复的值的集合。但是，它与 Set 有两个区别。首先，WeakSet 的成员只能是对象，而不能是其他类型的值。 

区别的和特点都和下面的`WeakMap`是一样的。

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

  `Map`相对于`Set`来讲，仅仅是多了个`get`方法

- 遍历
  `Map`和`Set`的遍历方法都是一致的

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
- `weakMap`
这个稍微的说一点点吧。`WeakMap`的专用场合就是，它的键所对应的对象，可能会在将来消失。`WeakMap`结构有助于防止内存泄漏。

`WeakMap`和`Map`的数据结构是一样的，但是区别有如下:
  1. `WeakMap`只接受对象作为键名（`null`除外），不接受其他类型的值作为键名。基础类型数据作为键的时候会报错。
  2. `WeakMap`的键名所指向的对象，不计入垃圾回收机制。意思就是它的键都是**弱引用**。只要所引用的对象的其他引用都被清除，垃圾回收机制就会释放该对象所占用的内存。也就是说，一旦不再需要，WeakMap 里面的键名对象和所对应的键值对会自动消失，不用手动删除引用。
  ```js
  // weakMap
  let weakObj = {
    a: 1
  }
  const weakMap = new WeakMap()
  weakMap.set(weakObj, 1)
  console.log(weakMap.get(weakObj)) //  1
  weakObj = null
  console.log(weakMap.get(weakObj)) //  undefined 这个key会跟着weakObj的销毁而销毁
  //但是值却不被影响
  let valueObj = {
    a: '我是 value object key'
  }
  let value = {
    a: 'value '
  }
  weakMap.set(valueObj, value)
  console.log(weakMap.get(valueObj)) // {a: "value "}
  value = null
  console.log(weakMap.get(valueObj)) // {a: "value "} 
  ```
  注意： **`WeakMap`中只有键是弱引用，值不受影响**
  
  3. `WeakMap`没有遍历操作也没有size属性。只有4个方法可用`get()、set()、has()、delete()`。

## Proxy
> Proxy 用于修改某些操作的默认行为，等同于在语言层面做出修改，所以属于一种“元编程”（meta programming），即对编程语言进行编程。

听说`VUE3.0`会抛弃`Object.defineProperty()`转向`Proxy`的怀抱。这就能看出来`Proxy`的强大了。那我们一起来揭开`Proxy`的神秘面纱。但是要注意，**`Proxy`现在的兼容性及其的差，而且没有垫片支持。**

### 基本概念

`Proxy: 代理` 通过中文意思我们不难这个东西主要是代理某种操作，所以可以成为`代理器`。他的目的是**在目标对象架设一层代理对操作进行“拦截”**。意思大概就是：想动我对象大哥可以，你得先过了我`Proxy`这一关。下面我们来实现一个代理：

通过`Proxy` 构造函数来生成一个`Proxy`对象：`Proxy(target: {}, handler: ProxyHandler<{}>): {}`

`target`: 表示要拦截的对象
`handler`: handler参数也是一个`ProxyHandler`对象，用来定制拦截行为。 包括了`get`，`set`方法。
```js
var proxy = new Proxy(target, handler);
```
```js
// 下面代码对一个空对象架设了一层拦截，重定义了属性的读取（get）和设置（set）行为
const proxyObj = new Proxy({}, {
  /**
   * 读取行为
   * @param {*目标对象} target
   * @param {*目标key} key 
   * @param {*接收对象(Symbol值)} receiver 
   */
  get (target, key, receiver) {
    console.log(target) // {count: 1}
    console.log(`getting ${key}!`); // getting count!
    return Reflect.get(target, key, receiver);
  },
  /**
   * 拦截设置值
   * @param {*目标对象} target 
   * @param {*属性名} key 
   * @param {*设置value} value 
   * @param {*proxy实例本身(Symbol)} receiver 
   */
  set (target, key, value, receiver) {
    console.log(target) // {}
    console.log(receiver) // Proxy {}
    console.log(`getting ${key} ${value}!`);
    return Reflect.set(target, key, value, receiver);
  }
})
proxyObj.count = 1 // getting count 1!
console.log(proxyObj.count) // 1
```
### 基本使用
- 下面我们来定义更实际的拦截。
```js
// 更实际的拦截
const objForPro1 = {
  a: 1
}
const proxyObj1 = new Proxy(objForPro1, {
  set (target, key) {
    return 1
  }
})
// 拦截针对的是Proxy对象！
proxyObj1.a = 4
proxyObj1.a = 5
console.log(proxyObj1.a)
```
看上面的情况就可以看出来，无论怎么改都无法修改属性的值。简简单单的实现了一个只读属性。多强大阿朋友们。

注意：**其中的拦截是针对`Proxy`对象，不是target对象！**

- 如果handler没有设置任何拦截，那就等同于直接通向原对象。
```js
const proxyObj2 = new Proxy(objForPro1, {})
proxyObj2.a = 2
console.log(proxyObj2.a) // 2
```

- `Proxy`实例也可以作为其他对象的原型对象
```js
// Proxy 实例也可以作为其他对象的原型对象。
const objByProxy = Object.create(proxyObj2) // objByProxy 本身没有a这个属性 但是proxyObj2有
console.log(objByProxy.a) // 2
```
- 同一个拦截器函数，可以设置**拦截多个操作**。
```js
// 同一个拦截器函数，设置拦截多个操作。
const mutiHandle = {
  get (target, name) {
    if (name === 'prototype') {
      return Object.prototype
    }
    console.log(`Hello ${name}`);
    return `Hello ${name}`
  },
  /**
   * 自定义apply 函数
   * @param {*} target 
   * @param {*} thisBinding 
   * @param {*} args 
   */
  apply (target,thisBinding, args) {
    console.log(`rewrite call && args  = ${[...args]}`);
    return args[0]
  },
  /**
   * 自定义构造器函数
   * @param {*} target 
   * @param {*} args 
   */
  construct (target, args) {
    console.log(`rewrite construct && args  = ${[...args]}`);
    return {value: args[1]}
  }
}
let proxyObj3 = new Proxy(function(x, y) {
  return x + y
}, mutiHandle)
console.log(proxyObj3(1,2))// call： 调用函数被拦截 rewrite call && args  = 1,2
console.log(new proxyObj3(1, 2)) // {value: 2}  rewrite construct && args  = 1,2
console.log(proxyObj3.prototype === Object.prototype) // true 因为我们拦截了get 
console.log(proxyObj3.a === "Hello a") // true 因为我们同时也拦截了value
```

### `Proxy支持拦截得属性和实例内置方法`
**Proxy 支持的拦截操作一览，一共 13 种。**
- `get(target, propKey, receiver)`：拦截对象属性的读取，比如proxy.foo和proxy['foo']。
- `set(target, propKey, value, receiver)`：拦截对象属性的设置，比如proxy.foo = v或proxy['foo'] = v，返回一个布尔值。
- `has(target, propKey)`：拦截propKey in proxy的操作，返回一个布尔值。
- `deleteProperty(target, propKey)`：拦截delete proxy[propKey]的操作，返回一个布尔值。
- `ownKeys(target)`：拦截`Object.getOwnPropertyNames(proxy)、Object.getOwnPropertySymbols(proxy)、Object.keys(proxy)、for...in`循环，返回一个数组。该方法返回目标对象所有自身的属性的属性名，而Object.keys()的返回结果仅包括目标对象自身的可遍历属- 性。
- `getOwnPropertyDescriptor(target, propKey)`：拦截Object.getOwnPropertyDescriptor(proxy, propKey)，返回属性的描述对象。
- `defineProperty(target, propKey, propDesc)`：拦截Object.defineProperty(proxy, propKey, propDesc）、- Object.defineProperties(proxy, propDescs)，返回一个布尔值。
- `preventExtensions(target)`：拦截Object.preventExtensions(proxy)，返回一个布尔值。
- `getPrototypeOf(target)`：拦截Object.getPrototypeOf(proxy)，返回一个对象。
- `isExtensible(target)`：拦截Object.isExtensible(proxy)，返回一个布尔值。
- `setPrototypeOf(target, proto)`：拦截Object.setPrototypeOf(proxy, proto)，返回一个布尔值。如果目标对象是函数，那么还有两种额- 外操作可以拦截。
- `apply(target, object, args)`：拦截 Proxy 实例作为函数调用的操作，比如proxy(...args)、proxy.call(object, ...args)、- proxy.apply(...)。
- `construct(target, args)`：拦截 Proxy 实例作为构造函数调用的操作，比如`new proxy(...args)`。


**Proxy内置方法，一共 14 个。**
- `proxy.get(target, key, receiver)`: 拦截某个属性的**读取操作**
```js
/**
 * 读取行为
 * @param {*目标对象} target
 * @param {*目标key} key 
 * @param {*接收对象(Symbol值)} receiver 
 */
get (target, key, receiver) {

}
```
`get`方法是可以继承的。
```js
// get方法的继承
let proto = new Proxy({}, {
  get (target, key) {
    console.log('GET ' + key)
    return target[key]
  }
})
let proxyObj4 = Object.create(proto) // proxyObj4直接继承了proto中的get 方法
proxyObj4.foo // foo 
```
`get()`实际用在对数组复索引的取值
```js
// get 根据复索引取值
function createArr (...args) {
  let handler = {
    get (target, key, receiver) {
      const index = Number(key)
      // 构建真实的key
      if (key < 0) {
        key = String(target.length + index);
      }
      // Reflect以后再说
      return Reflect.get(target, key, receiver)
    }
  }
  let target = []
  target.push(...args)
  // 返回一个被proxy拦截的数组对象
  return new Proxy(target, handler);
}
const proxyArr = createArr('a','b', 'c', 'd')
console.log(proxyArr[-1]) // d 
```
- `proxy.set(target, key, value, receiver)`: 拦截某个属性的**赋值操作**
这个方法用得最多，接受4个参数
```js
/**
 * 设置行为
 * @param {*目标对象} target 
 * @param {*属性名} key 
 * @param {*设置value} value 
 * @param {*proxy实例本身(Symbol)} receiver 
 */
set (target, key, value, receiver) {
  
}
```
注意：**如果目标对象自身的某个属性，不可写且不可配置，那么`set()`将不起作用。**
```js
// 如果目标对象自身的某个属性，不可写且不可配置，那么set方法将不起作用。
const readObj = {}
// 又是一种设置只读属性的方法
Object.defineProperty(readObj, 'name', {
  value: 'readOnly',
  writable: false,
})
const uselessHandle = {
  set (target, key, value, receiver) {
    target[key] = 'set by proxy'
  }
}
const uselessProxy = new Proxy(readObj, uselessHandle)
// uselessProxy.name = 'proxy' // Cannot assign to read only property 'name' of object '#<Object>
console.log(uselessProxy.name)// readOnly
```
- `proxy.apply(target, targetThis, args)`: 拦截函数的**调用、call和apply操作**\
`apply`方法可以接受三个参数，分别是目标对象、目标对象的上下文对象（this）和目标对象的参数数组。说简单一点就是你这个函数执行之前呀先执行被我拦截的东西。因为函数也是对象，所有函数也是可以被当作是`Proxy`对象。
```js
/**
 * 自定义apply 函数
 * @param {*目标对象} target 
 * @param {*目标this} thisBinding 
 * @param {*参数数组} args 
 */
apply (target,thisBinding, args) {
  console.log(`rewrite call && args  = ${[...args]}`);
  return args[0]
}
```
下main来看一个简单的例子。**被拦截了之后函数原本的内容就不会存在了**，`Proxy`就是这么的为所欲为。
```js
// 简单的apply()
let applyFunc = function() {console.log(`I am target function`)}
const applyProxy = new Proxy(applyFunc, {
  /**
   * @param {*} target 
   * @param {*} targetThis 
   * @param {*} args 
   */
  apply (target, targetThis, args) {
    console.log(`i am proxy`)  
  }
})
applyProxy() // i am proxy
```
- `proxy.has(target, key)`：拦截**`HasProperty`**操作
判断对象是否具有某个属性时，这个方法会生效。典型的操作就是`in`运算符。这也是一个魔术一样的方法。下面我们开始变魔术，隐藏部分属性
```js
// 使用has() 隐藏部分属性
const hasHandle = {
  /**
   * 隐藏“_”开头的属性
   * @param {*} target 
   * @param {*} key 
   */
  has (target, key) {
    // 变魔术开始了
    if (key[0] === '_') {
      return false
    }
    return key in target
  }
}
let hasObj = {name: 'has', _age: 'private'}
let hasProxy = new Proxy(hasObj, hasHandle)
console.log('_age' in hasProxy) // false
```
如果原对象不可配置或者禁止扩展，这时`has`拦截会报错。
```js
// 不可扩展的时候使用has会报错
let noHasObj = {a: 1}
Object.preventExtensions(noHasObj) // 禁止扩展
const noHasProxy = new Proxy(noHasObj, {
  has: function(target, prop) {
    return false
  }
})
// console.log('a' in noHasProxy) // Uncaught TypeError: 'has' on proxy: trap returned falsish for property 'a' but the proxy target is not extensible
```
值得注意的是，**has方法拦截的是HasProperty操作，而不是HasOwnProperty操作，即has方法不判断一个属性是对象自身的属性，还是继承的属性。虽然for...in循环也用到了in运算符，`但是has拦截对for...in循环不生效。`**
```js
// has对for in 循环是不起作用的
for (const key in hasProxy) {
  console.log(key)// name _age
}
```
- `proxy.construct(target, args, newTarget)`：拦截构造器方法，但是它拦截的主要是`new`操作符
```js
// construct 拦截
let newProxy = new Proxy(function () {
  console.log(`old construct`)
}, {
  /**
   * @param {*目标对象} target 
   * @param {*参数列表} args 
   * @param {*new命令作用的构造函数（例子中的newProxy）} newTarget 
   */
  construct: function(target, args, newTarget) {
    console.log(target)
    console.log('proxy construct called: ' + args.join(', '));
    console.log(newTarget)
    return { value: args[0] * 10 } // 这个地方一定要但会对象，否则会报错。
  }
})
let proxyValue = new newProxy(1)
console.log(proxyValue.value) // 10
```
- `proxy.deleteProperty(target, key) `：用于拦截`delete`操作符
一旦检测到非法操作，需要抛出异常来终止当前执行。
```js
// delete操作符拦截
let deleteObj = {
  ok: 'ok',
  _no: 'no'
}
const deleteProxy = new Proxy(deleteObj, {
  /**
   * @param {*} target 
   * @param {*} key 
   */
  deleteProperty (target, key) {
    if (key[0] === '_') {
      throw new Error(`Invalid attempt to private "${key}" property`);
    }
  }
})
delete deleteProxy._no //报错
```
注意：**目标对象自身的不可配置（configurable）的属性，不能被`deleteProperty`方法删除，否则报错。**

- `proxy.defineProperty()`: 拦截了`Object.defineProperty`操作
```js
// 拦截Object.defineProperty
const defineHandle = {
  /**
   * @param {*} target 
   * @param {*} key 
   * @param {*修饰符} descriptor 
   */
  defineProperty  (target, key, descriptor) {
    console.log(descriptor) // {value: "demo", writable: true, enumerable: true, configurable: true}
    return false
  }
}
const defineProxy = new Proxy({}, defineHandle)
defineProxy.demo = 'demo' // 不会生效
```
注意：**如果目标对象不可扩展（non-extensible），则defineProperty不能增加目标对象上不存在的属性，否则会报错。另外，如果目标对象的某个属性不可写（writable）或不可配置（configurable），则defineProperty方法不得改变这两个设置。**
- `proxy.getOwnPropertyDescriptor()`：拦截`getOwnPropertyDescriptor`操作
```js
// 拦截getOwnPropertyDescriptor
const getOwn = {
  getOwnPropertyDescriptor (target, key) {
    if (key[0] === '_') {
      return;
    }
    return Object.getOwnPropertyDescriptor(target, key)
  }
}
let ownTaget = {
  _foo: 'foo',
  bar: 'bar'
}
const ownProxy = new Proxy(ownTaget, getOwn)
console.log(Object.getOwnPropertyDescriptor(ownProxy, '_foo'))// undefined
console.log(Object.getOwnPropertyDescriptor(ownProxy, 'bar'))// {value: "bar", writable: true, enumerable: true, configurable: true}
```
  
- `proxy.getPrototypeOf()`: 拦截一系列获取`prototype`得操作

具体主要是拦截以下得操作
- `Object.prototype.__proto__`
- `Object.prototype.isPrototypeOf()`
- `Object.getPrototypeOf()`
- `Reflect.getPrototypeOf()`
- `instanceof`
```js
// getPrototypeOf()拦截
let protoObj = {
  name: 'protoObj'
}
const protoProxy = new Proxy({}, {
  getPrototypeOf (target) {
    console.log('protoObj be proxyed')
    return protoObj
  }
})
console.log(Object.getPrototypeOf(protoProxy) === protoObj)// true
```
- `proxy.isExtensible()`：拦截`Object.isExtensible（检测对象是否可扩展）`操作。
```js
// 拦截Object.isExtensible（检测对象是否可扩展）
let extendObj = {}
const extendProxy = new Proxy(extendObj, {
  isExtensible (target) {
    console.log("call isExtensible proxy")
    return true
  }
})
Object.isExtensible(extendProxy)
```
注意：**该方法只能返回布尔值，否则返回值会被自动转为布尔值。**

- `proxy.preventExtensions()`：拦截`Object.preventExtensions(取消对象得可扩展性)`操作
```js
// 拦截preventExtensions操作
const preventExtHandle = {
  preventExtensions(target) {
    console.log('you will preventExtensions target')
    Object.preventExtensions(target)
    return true
  }
}
let preventExtPro = new Proxy({}, preventExtHandle)
Object.preventExtensions(preventExtPro)
```
注意：**该方法只能返回布尔值，否则返回值会被自动转为布尔值。**


- **`proxy.ownKeys(target)`**： 用来拦截对象自身属性的`读取操作`。

从这个ownKeys我们其实就能知道这个方法是干什么的，具体来说，拦截以下操作。
- `Object.getOwnPropertySymbols()`
- `Object.getOwnPropertyNames()`
- `Object.keys()`
- `for...in循环`
```js
// 拦截ownKeys
let keysObj = {
  a: 'a',
  b: 'b',
  c: 'c',
  _d: 'd'
}
let keysProxy = new Proxy(keysObj, {
  /**
   * 拦截只返回a
   * @param {*} target 
   */
  ownKeys(target) {
    return ['a']
  }
})
for (const key in keysProxy) {
  console.log(key) // a
}
// ownKeys拦截getOwnPropertyNames 方法返回一个由指定对象的所有自身属性的属性名（包括不可枚举属性但不包括Symbol值作为名称的属性）组成的数组。
console.log(Object.getOwnPropertyNames(keysFilteProxy)) // ["name", "enumerabled"]
```
注意：**使用`Object.keys`方法时，下面这三类属性时会被ownKeys方法自动过滤的**
1. 目标对象上不存在的属性
2. 属性名为 `Symbol` 值
3. 不可遍历（`enumerable`）的属性  
```js
// Object.keys的特殊情况
let keySymbol = Symbol('symbol')
let keysFilterObj = {
  name: 'name',
  [keySymbol]: 'symbol'
}
// 不可枚举属性
Object.defineProperty(keysFilterObj, 'enumerabled', {
  enumerable: false,
  configurable: true,
  writable: true,
  value: 'enumerabled'
})
let keysFilteProxy = new Proxy(keysFilterObj, {
  ownKeys (target) {
    return ['name', keySymbol, 'enumerabled']
  }
})
console.log(Object.keys(keysFilteProxy)) // ["name"]
```
- `proxy.setPrototypeOf(target)`：用来拦截`Object.setPrototypeOf`方法。
```js
// 拦截setPrototypeOf 
let setProtObj = {
  foo: 'foo'
}
const setProtProxy = new Proxy(setProtObj, {
  setPrototypeOf (target) {
    console.log(`called setPrototypeOf`)
    return true // 只能返回boolean值
  }
})
Object.setPrototypeOf(setProtProxy, Array)
```
通过这个方法我们可以不允许修改`proxy`实例的`prototype`。
  
- `Proxy.revocable(target, handler)`: 这个就比较特殊了，他不是用来拦截的。他返回一个`可取消的 Proxy 实例`。
```js
// 返回一个可取消的 Proxy 实例

let {proxy, revoke} = Proxy.revocable({bar: 'bar'}, {})
console.log(proxy) // Proxy {} proxy实例
console.log(revoke) // 取消proxy的函数
proxy.foo = 'foo'
console.log(proxy.bar)
console.log(proxy.foo)
revoke()
// console.log(proxy.bar) // Cannot perform 'get' on a proxy that has been revoked 此时代理被取消，所以proxy实例对象中已经没有不存在了
```
`Proxy.revocable`的一个使用场景是，目标对象不允许直接访问，必须通过代理访问，一旦访问结束，就收回代理权，不允许再次访问。

### 注意点
- `this`指向问题，`Proxy`它不是目标对象的透明代理，所以`this`的指向其实是指向`proxy`

## Relfact
> Reflect对象与Proxy对象一样，也是 ES6 为了操作对象而提供的新 API。Reflect对象的设计目的有这样几个。这也算是对object开始改善的第一步，以及对Object的补充。

### 基础知识
官方解释`Relfact`有以下的作用
1. 将一些**明显属于语言内部的方法**(例如`Object.defineProperty`)部署到`Relfact`对象上，我觉得是为了解耦以及更加纯粹得对象和方法管理。现在`Object`和`Relfact`上都有某些方法，以后得新方法将只部署到`Relfact`上。也就是说，从`Reflect`对象上可以拿到语言内部的方法。
2. 修改某些`Object`方法的返回结果，让其变得更合理
```js
// Relfact
//Object.defineProperty(obj, name, desc)在无法定义属性时，会抛出一个错误，而Reflect.defineProperty(obj, name, desc)则会返回false。
// 没有 Relfact
try {
  Object.defineProperty(target, property, attributes);
} catch (error) {
  console.log(error)
} 
// 使用Relfact Reflect.defineProperty() 返回true或者fasle
if ( Reflect.defineProperty(target, property, attributes)) { 
  console.log(true)
} else {
  console.log(false)
}
```
3.  让`Object`操作都变成函数行为。
```js
// 让Object操作都变成函数行为。
// 某些Object操作是命令式，比如name in obj和delete obj[name]，而Reflect.has(obj, name)和Reflect.deleteProperty(obj, name)让它们变成了函数行为。
let relfactObj = {
  name: 'Reflect'
}
// old 
console.log('name' in relfactObj)
// Reflect
console.log(Reflect.has(relfactObj, 'name'));
```
4. 与`Proxy`天衣无缝的融合。`Reflect`对象的方法与`Proxy`对象的方法一一对应，只要是`Proxy`对象的方法，就能在`Reflect`对象上找到对应的方法。下面是一个例子
```js
// 与proxy配合
let logObj = new Proxy(relfactObj, {
  get (target, name) {
    console.log('get ', target, name)
    return Reflect.get(target, name)
  },
  deleteProperty (target, name) {
    console.log('delete ', target, name)
    return Reflect.deleteProperty(target, name)
  },
  has(target, name) {
    console.log('has ' + name)
    return Reflect.has(target, name)
  }
})

console.log('name' in logObj)// true  has name
console.log(logObj.name)// get  {name: "Reflect"} name  Reflect
```
这样的操作是为了保证原生行为能够正常执行。

### 静态方法
下面的这些静态方法， 大都是和`Proxy`中的是一一对应的而且用处也都基本相同。这些基本是为了统一`API`，因为在`Object`中的这些操作都是稍微有点混乱的。基本上都是`Object`的替代方法。那下面我们就来进行新旧对比

- `Reflect.apply(target, thisArg, args)`： 等同于`Function.prototype.apply.call(func, thisArg, args)`
- `Reflect.construct(target, args)`：替代`new`来生成一个实例，`args`是一个数组
```js
let ReflectCons = function (name) {
  this.name = name
}
// old
// const consReflect = new ReflectCons('ReflectCons')
// new
const consReflect = Reflect.construct(ReflectCons, ['ReflectCons'])
console.log(consReflect.name)
```
- `Reflect.get(target, name, receiver)`: 查找并返回`target`对象的`name`属性，如果没有该属性，则返回`undefined`。
- `Reflect.set(target, name, value, receiver)`： 设置`target`对象的`name`属性等于`value`。可以取代默认的赋值行为
注意，**如果 Proxy对象和 Reflect对象联合使用，前者拦截赋值操作，后者完成赋值的默认行为，而且传入了receiver，那么Reflect.set会触发Proxy.defineProperty拦截。**
```js
// Reflect.set
// 如果 Proxy对象和 Reflect对象联合使用，前者拦截赋值操作，后者完成赋值的默认行为，而且传入了receiver，那么Reflect.set会触发Proxy.defineProperty拦截。
let setReObj = {
  name: 'setReObj'
}
const setReProxy = new Proxy (setReObj, {
  set (target, name, value, receiver) {
    console.log('set')
    Reflect.set(target, name, value, receiver)
    return true
  },
  defineProperty (target, key , attribute) {
    console.log('defineProperty');
    Reflect.defineProperty(target, key, attribute);
  }
})
setReProxy.name = 'aaa'
```
- `Reflect.defineProperty(target, name, desc)`: 基本等同`Object.defineProperty`
- `Reflect.deleteProperty(target, name)`:基本等同`Object.deleteProperty`
- `Reflect.has(target, name)`: 等同于`in`
- `Reflect.ownKeys(target)`: 用于返回对象的所有属性，基本等同于`Object.getOwnPropertyNames`与`Object.getOwnPropertySymbols`之和。
```js
// Reflect.ownKeys(target)
const keysReflect = {
  [Symbol('key')] : 'key',
  key: 'keykey'
}
console.log(Reflect.ownKeys(keysReflect)) // ["key", Symbol(key)]
```
- `Reflect.isExtensible(target)`: 对应`Object.isExtensible` 返回布尔值
- `Reflect.preventExtensions(target)`: 对应`Object.preventExtensions`方法,让一个对象不可扩展
- `Reflect.getOwnPropertyDescriptor(target, name)`：基本等同于`Object.getOwnPropertyDescriptor`
- `Reflect.getPrototypeOf(target)`: 基本等同于`Object.getPrototypeOf`
- `Reflect.setPrototypeOf(target, prototype)`: 基本等同于`Object.setPrototypeOf`,它返回一个布尔值

## Promise

## Iterator


## Decorator