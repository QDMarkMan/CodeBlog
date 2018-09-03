# JavaScript种高阶函数的魅力
> 高阶函数是指至少满足下列条件之一的函数 1：函数可以作为参数被传递 2：函数可以作为返回值输出

    JavaScript语言中的函数显然的是满足了高阶函数的条件，下面我们一起来探寻JavaScript种高阶函数的魅力。
## 高阶函数实现AOP
  **AOP**(面向切面编程)的主要作用就是把一些和核心业务逻辑模块无关的功能抽取出来，然后再通过“动态织入”的方式掺到业务模块种。这些功能一般包括日志统计,安全控制,异常处理等。AOP是Java Spring架构的核心。下面我们就来探索一下再Javascript种如何实现**AOP**

  在JavaScript种实现AOP，都是指把一个函数“动态织入”到另外一个函数中，具体实现的技术有很多，我们使用```Function.prototype```来做到这一点。代码如下
  ```js
  /**
  * 织入执行前函数
  * @param {*} fn 
  */
  Function.prototype.aopBefore = function(fn){
    console.log(this)
    // 第一步：保存原函数的引用
    const _this = this
    // 第四步：返回包括原函数和新函数的“代理”函数
    return function() {
      // 第二步：执行新函数，修正this
      fn.apply(this, arguments)
      // 第三步 执行原函数
      return _this.apply(this, arguments)
    }
  }
  /**
  * 织入执行后函数
  * @param {*} fn 
  */
  Function.prototype.aopAfter = function (fn) {
    const _this = this
    return function () {
      let current = _this.apply(this,arguments)// 先保存原函数
      fn.apply(this, arguments) // 先执行新函数
      return current
    }
  }
  /**
  * 使用函数
  */
  let aopFunc = function() {
    console.log('aop')
  }
  // 注册切面
  aopFunc = aopFunc.aopBefore(() => {
    console.log('aop before')
  }).aopAfter(() => {
    console.log('aop after')
  })
  // 真正调用
  aopFunc()
  ```

## currying（柯里化）
 关于**curring**我们首先要聊的是**函数柯里化**。
 > **curring**又称部分求值。一个**curring**的函数首先会接受一些参数，接受了这些参数之后，该函数并不会立即求值，二十继续返回另外一个函数，刚才传入的参数在函数形成的闭包中被保存起来。待到函数中被真正的需要求值的时候，之前传入的所有参数被一次性用于求值。

 生硬的看概念不太好理解，我们来看接下来的例子
 我们需要一个函数来计算一年12个月的消费，在每个月月末的时候我们都要计算消费了多少钱。正常代码如下
 ```js
 // 未柯里化的函数计算开销
let totalCost = 0
const cost = function(amount, mounth = '') {
  console.log(`第${mounth}月的花销是${amount}`)
  totalCost += amount
  console.log(`当前总共消费：${totalCost}`)
}
cost(1000, 1) // 第1个月的花销
cost(2000, 2) // 第2个月的花销
// ...
cost(3000, 12) // 第12个月的花销
 ```
 总结一下不难发现，如果我们要计算一年的总消费，没必要执行12次。只需要在年底执行一次计算就行，接下来我们对这个函数进行部分柯里化的函数帮助我们理解
 ```js
// 部分柯里化完的函数
const curringPartCost = (function() {
  // 参数列表
  let args = []
  return function (){
    /**
     * 区分计算求值的情况
     * 有参数的情况下进行暂存
     * 无参数的情况下进行计算
     */
    if (arguments.length === 0) {
      let totalCost = 0
      args.forEach(item => {
        totalCost += item[0]
      })
      console.log(`共消费：${totalCost}`)
      return totalCost
    } else {
      // argumens并不是数组，是一个类数组对象
      let currentArgs = Array.from(arguments)
      args.push(currentArgs)
      console.log(`暂存${arguments[1] ? arguments[1] : '' }月，金额${arguments[0]}`)
    }
  }
})()
curringPartCost(1000,1)
curringPartCost(100,2)
curringPartCost()
 ```
 接下来我们编写一个通用的curring， 节后一个即将被curring的函数。代码如下
 ```js
 // 通用curring函数
const curring = function(fn) {
  let args = []
  return function () {
    if (arguments.length === 0) {
      console.log('curring完毕进行计算总值')
      return fn.apply(this, args)
    } else {
      let currentArgs = Array.from(arguments)[0]
      console.log(`暂存${arguments[1] ? arguments[1] : '' }月，金额${arguments[0]}`)
      args.push(currentArgs)
      // 返回正被执行的 Function 对象，也就是所指定的 Function 对象的正文，这有利于匿名函数的递归或者保证函数的封装性
      return arguments.callee
    }
  }
}
// 求值函数
let costCurring = (function() {
  let totalCost = 0
  return function () {
    for (let i = 0; i < arguments.length; i++) {
      totalCost += arguments[i]
    }
    console.log(`共消费：${totalCost}`)
    return totalCost
  }
})()
// 执行curring化
costCurring = curring(costCurring)
costCurring(2000, 1)
costCurring(2000, 2)
costCurring(9000, 12)
costCurring()
 ```

 ## 函数节流
 JavaScript中的大多数函数都是用户主动触发的，一般情况下是没有性能问题，但是在一些特殊的情况下不是由用户直接控制。容易大量的调用引起性能问题。毕竟DOM操作的代价是非常昂贵的。下面将列举一些这样的场景：
 - ```window.resise```事件。
 - ```mouse, input```等事件。
 - 上传进度
 - ...
 
 下面通过高阶函数的方式我们来实现函数节流
 ```js
 /**
 * 节流函数
 * @param {*} fn 
 * @param {*} interval 
 */
const throttle = function (fn, interval = 500) {
  let timer = null, // 计时器 
      isFirst = true // 是否是第一次调用
  return function () {
    let args = arguments, _me = this
    // 首次调用直接放行
    if (isFirst) {
      fn.apply(_me, args)
      return isFirst = false
    }
    // 存在计时器就拦截
    if (timer) {
      return false
    }
    // 设置timer
    timer = setTimeout(function (){
     console.log(timer)
     window.clearTimeout(timer)
     timer = null
     fn.apply(_me, args)
    }, interval)
  }
}
// 使用节流
window.onresize = throttle(function() {
  console.log('throttle')
},600)
 ```

## 分时函数
 >  节流函数为我们提供了一种限制函数被频繁调用的解决方案。下面我们将遇到另外一个问题，某些函数是用户主动调用的，但是由于一些客观的原因，这些操作会严重的影响页面性能，此时我们需要采用另外的方式去解决。
 
 如果我们需要在短时间内才页面中插入大量的DOM节点，那显然会让浏览器吃不消。可能会引起浏览器的假死，所以我们需要进行分时函数，分批插入。
 ```js
 /**
 * 分时函数
 * @param {*创建节点需要的数据} list 
 * @param {*创建节点逻辑函数} fn 
 * @param {*每一批节点的数量} count 
 */
const timeChunk = function(list, fn, count = 1){
  let insertList = [], // 需要临时插入的数据
      timer = null // 计时器
  const start = function(){
    // 对执行函数逐个进行调用
    for (let i = 0; i < Math.min(count, list.length); i++) {
      insertList = list.shift()
      fn(insertList)
    }
  }
  return function(){
    timer = setInterval(() => {
      if (list.length === 0) {
        return window.clearInterval(timer)
      }
      start()
    },200)
  }
}
// 分时函数测试
const arr = []
for (let i = 0; i < 94; i++) {
  arr.push(i)
}
const renderList = timeChunk(arr, function(data){
  let div =document.createElement('div')
  div.innerHTML = data + 1
  document.body.appendChild(div)
}, 20)
renderList()
 ```
## 惰性加载函数
 > 在Web开发中，因为一些浏览器中的差异，一些嗅探工作总是不可避免的。
 
 因为浏览器的差异性，我们要常常做各种各样的兼容，举一个非常简单常用的例子：在各个浏览器中都能够通用的事件绑定函数。
 
 常见的写法是这样的：
 ```js
 // 常用的事件兼容
const addEvent = function(el, type, handler) {
  if (window.addEventListener) {
    return el.addEventListener(type, handler, false)
  }
  // for IE
  if (window.attachEvent) {
    return el.attachEvent(`on${type}`, handler)
  }
}
 ```
 这个函数存在一个缺点，它每次执行的时候都会去执行if条件分支。虽然开销不大，但是这明显是多余的，下面我们优化一下， 提前一下嗅探的过程：
 ```js
 const addEventOptimization = (function() {
  if (window.addEventListener) {
    return (el, type, handler) => {
      el.addEventListener(type, handler, false)
    }
  }
  // for IE
  if (window.attachEvent) {
    return (el, type, handler) => {
      el.attachEvent(`on${type}`, handler)
    }
  }
})()
 ```
 这样我们就可以在代码加载之前进行一次嗅探，然后返回一个函数。但是如果我们把它放在公共库中不去使用，这就有点多余了。下面我们使用惰性函数去解决这个问题：
 ```js
// 惰性加载函数
let addEventLazy = function(el, type, handler) {
  if (window.addEventListener) {
    // 一旦进入分支，便在函数内部修改函数的实现
    addEventLazy = function(el, type, handler) {
      el.addEventListener(type, handler, false)
    }
  } else if (window.attachEvent) {
    addEventLazy = function(el, type, handler) {
      el.attachEvent(`on${type}`, handler)
    }
  }
  addEventLazy(el, type, handler)
}
addEventLazy(document.getElementById('eventLazy'), 'click', function() {
  console.log('lazy ')
})
 ```
 一旦进入分支，便在函数内部修改函数的实现，重写之后函数就是我们期望的函数，在下一次进入函数的时候就不再存在条件分支语句。

# 总结
 
 JavaScript高阶函数主要是借助了闭包来实现这些扫操作。
 文中出现的[代码地址](https://github.com/QDMarkMan/usually-accumulated/blob/master/src/highOrderFunc.js)
