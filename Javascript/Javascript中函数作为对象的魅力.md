# Javascript中函数作为对象的魅力

`Javascript`赋予了函数非常多的特性，其中最重要的特性之一就是将函数作为`第一型的对象`。那就意味着在`javascript`中**函数可以有属性，可以有方法， 可以享有所有对象所拥有的特性。并且最重要的，她还可以直接被调用**

我们简单的试验一下就可以发现
```js
// 简单实验 函数作为对象的存在
let fn = function () {}
fn.prop = 'fnProp'
console.log(fn.prop) // fnProp
```
为函数添加属性的这个特性我觉的大家在平时的开发中基本没什么尝试或者是使用过，但是在一些JS库或者是事件回掉管理中都能发挥出很大的用处。下面一起来看几个例子。

## 函数缓存

在某有一些的情况下我们可以要存储一组相关但是相互又独立的函数。这个需求看起来很`easy`，实现起来也不复杂。最显而易见的做法是使用一个数组来保存所有的函数，
这样不是不可以，但是显然这种做法不是最好的。下面通过为函数属性我们呢来实现这个我们的目的

```js
// 1：函数缓存示例
let store = {
  nextId: 1, // id
  cache: {}, // 缓存
  add (fn) {
    // 如果函数中没有id属性那么就缓存
    if (!fn.id) {
      console.log(`begin add func ${fn.name}`)
      fn.id = store.nextId ++
      // 设置完缓存之后返回true
      return !!(store.cache[fn.id] = fn)
    } else {
      console.log(`${fn.name} is already in cache`)
    }
  }
}
function storeCache() {}
store.add(storeCache) // begin add func storeCache
store.add(storeCache) // storeCache is already in cache
```

上面的这一段代码逻辑清晰，`store`对象用来管理我们的缓存，`cache`属性用来存储函数，`nextId`属性用来保存当前的缓存Id，`add()`方法用来设置存储，先来判断当前函数是否已经在缓存中然后再去设置缓存，这样就能限制函数的重复添加，最后返回`true`。

> `!!`构造是一种可以将任意Javascript表达式转化为其等效布尔值的简单方式。


## 缓存记忆函数

这种函数可以记住之前已经计算过的结果，避免了不必要的计算，这显然是能够提升代码性能的。

在举例之前我们先来看看这种方式的优缺点
优点
  - 缓存了之前的结果，最终用户享有性能优势
  - 实际上是发生在幕后，操作无感

缺点
  - 内存的牺牲这是肯定的
  - 打破了存粹性（一个函数或者方法应该只做好一件事）
  - 如果方法中有算法，那么很难测量这个算法的性能

了解了优缺点我们来看一个简单的计算素数的例子（不是很严谨）

```js

// 2: 缓存记忆函数
function isPrime (value) {
  if (!isPrime.anwers) isPrime.anwers = {}
  // 先从缓存里面取
  if (isPrime.anwers[value] != null ) {
    return isPrime.anwers[value]
  }
  // 开始进行判断和计算
  let prime = value != 1
  for (let index = 2; index < value; index++) {
    if (value % index == 0) {
      prime = false
      break;
    } 
  }
  // 保存计算出来的值
  return isPrime.anwers[value] = prime
}

console.log(isPrime(5))
console.log(`从函数记忆中直接读取${isPrime.anwers[5]}`)
```

这里呢 好处是特别明显的我们再次的取用`isPrime.anwers[5]`的时候不需要经过任何的计算，**但是大型的计算要主要内存的使用**

## 缓存记忆DOM元素

通过元素的标签查询`DOM`的操作的的代价是昂贵的，各位前端大佬肯定都很清楚。我们下面使用缓存记忆的方式来进行这个操作

```js
// 3:缓存记忆DOM元素
function getElements (name) {
  if (!getElements.cache) getElements.cache = {}
  return getElements.cache[name] = getElements.cache[name] || document.getElementsByTagName(name);
}
console.log(getElements('div')) // HTMLCollection
console.log(getElements.cache['div']) // HTMLCollection
```

这个函数和上面的缓存使用的同一个手法，而且这简单的4句代码能为我们的性能带来大幅度的提升。这也算是一种超能力吧。函数的很多特性都和其上下文有关，接下来我们研究一个和上下文又换的例子。

## 伪造数组方法(上下文相关)

在一些情况下我们想创建一个包含一组数据的对象，但是这个数据包含很多的状态，比如和集合项有关的元数据那么我们用数组来存就不太合适了。那么这里我们就用对象的方式来假扮数组。通过改变上下文来完成一些`“不法的行为”`

```js
// 4：伪造数组方法
// <input type="button" id="add" >
// <input type="button" id="remove" >
let elems = {
  length: 0, //为了保存个数
  add (elem) {
    Array.prototype.push.call(this, elem)
  },
  gather (id) {
    this.add(document.getElementById(id))
  }
}
elems.gather('add')
elems.gather('remove')
console.log(elems[0]); // <input type="button" id="add" >
console.log(elems[1]); // <input type="button" id="remove" >
console.log(elems.length); // 2
console.log(elems);
/**
  0: input#add
  1: input#remove
  add: ƒ add(elem)
  gather: ƒ gather(id)
  length: 2
 */
```

在我还对`JS`懵懵懂懂的时候看到这样的操作被秀了一脸，简直是刺激了我幼小的心灵。

我们在`add`函数中实现了把元素添加到了集合中，而且`Array`又正好提供`push`方法， 不用白不用。这种操作也是直白的展示了函数上下文的超强特性。


# 总结

`Javascript`强大的灵活性， 也带来更多的可能性。 路漫漫其修远兮，吾将上下而求索。

参考资料： 《JavaScript忍者秘籍》

 [代码地址](https://github.com/QDMarkMan/usually-accumulated/blob/master/src/func_width_obj.js)

 [原文地址](https://github.com/QDMarkMan/CodeBlog/blob/master/Javascript/Javascript中函数作为对象的魅力.md) 如果觉得有帮助的得话给个⭐吧😁

