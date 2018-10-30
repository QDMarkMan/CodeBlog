# JavaScript中的模块
> 很多人搞不懂AMD和CMD的区别，其实我自己分的也不是特别的清楚。这次我们一起来区分这两种模块化的标准,争取这一次嚼碎她。

## AMD和CMD

### AMD(异步模块定义)
> AMD是"Asynchronous Module Definition"的缩写，意思就是"异步模块定义"。它采用异步方式加载模块，模块的加载不影响它后面语句的运行。所有依赖这个模块的语句，都定义在一个回调函数中，等到加载完成之后，这个回调函数才会运行。

写起来是这样的： 
```js
require([module], callback);
// 引入进来就可以使用
require(['clock'],function(clock){
  // 模块已经存在
  clock.start()
})
```

### CMD(通用模块定义)
> CMD (Common Module Definition), 是seajs推崇的规范，CMD则是依赖就近，用的时候再require。

写起来是这样的： 
```js
define(function(require, exports, module) {
  var clock = require('clock'); // 此时使用我们需要此时去引入
  clock.start();
});
```

## 区别

以下区别总结部分来自[知乎](https://www.zhihu.com/question/20351507?sort=created)

1. `AMD` 是 `RequireJS` 在推广过程中对模块定义的规范化产出。 `CMD` 是 `SeaJS` 在推广过程中对模块定义的规范化产出。
2. 执行顺序的不同。`AMD`是**提前执行**， `CMD`是**延迟执行**。`RequireJS` 从 2.0 开始，也改成可以延迟执行（根据写法不同，处理方式不同）。`CMD` 推崇 **as lazy as possible**.
3.  `CMD` 推崇**依赖就近**，`AMD` 推崇**依赖前置**。这是什么意思呢？ 在我们使用模块的时候，`CMD`允许我们在代码体中真正需要的时候去引入，但是`AMD`默认推崇的是在代码文件引入模块的一开始我们就要把这个模块引入进来。就像下面的代码：
    ```js
    // CMD
    define(function(require, exports, module) {
      // 不同的模块在使用的时候去引用
      var start = require('start')
      start.start()
      // ... 代码
      var clock = require('clock'); // 此时使用我们需要此时去引入
      clock.start()
    })
    // AMD
    require(['start','clock'],function(start, clock){ // 必须在一开始就引用依赖
      start.start()
      clock.start()
    })
    ```
    其实`AMD` 也支持 `CMD`的写法，同时还支持将 require 作为依赖项传递，但 `RequireJS` 的作者默认是最喜欢上面的写法，也是官方文档里默认的模块定义写法。

4. `AMD`的API默认是**一个当多个**, `CMD`的API严格区分。而且`CMD`推崇的是**职责单一**。比如 `AMD` 里，require 分**全局 require 和局部 require**，都叫 require。`CMD` 里，没有全局 require，**而是根据模块系统的完备性，提供 seajs.use 来实现模块系统的加载启动**。CMD 里，每个 API 都**简单纯粹**。

5. **依赖模块执行时机不同**,`AMD`在加载模块完成后就会执行改模块，所有模块都加载执行完后会进入require的回调函数，执行主逻辑，这样的效果就是依赖模块的执行顺序和书写顺序不一定一致，看网络速度，哪个先下载下来，哪个先执行，但是主逻辑一定在所有依赖加载完成后才执行， `CMD`加载完某个依赖模块后并不执行，只是下载而已，在所有依赖模块加载完成后进入主逻辑，遇到require语句的时候才执行对应的模块，这样模块的执行顺序和书写顺序是完全一致的。

6. `AMD`在使用的时候提前加载，体验可能会好点，用户在使用的时候没有延迟。但是`CMD`在需要的时候才加载，性能会更好。

至于`Require.js`和`Sea.js`的不同，可以看[这里](https://github.com/seajs/seajs/issues/277)

## 现代化的模块(CommonJS 和 ES6 Module)

`AMD`和`CMD`主要是用在浏览器端，**这些标准渐渐的已经淡出开发者们的视线，但是他们的思想还还是值得我们了解一下啊**。至于现在，我们现在使用的主角是`Commonjs`和`ES6 Module`。 在现在前端借助`webpack`来实现大前端的情况下我们在实际开发中还是这些用的比较多，`CommonJS`主要是应用在服务端。`ES Module`旨在统一`JavaScript`中的所有模块。

## CommonJS

### 定义
> Nodejs的模块系统就采用CommonJS模式。CommonJS标准规定，一个单独的文件就是一个模块，模块内将需要对外暴露的变量放到exports对象里，可以是任意对象，函数，数组等，未放到exports对象里的都是私有的。用require方法加载模块，即读取模块文件获得exports对象。

`CommonJS`肯定是很重要的，毕竟我们前端工程化赖以生存的`Node.js`中是用的就是`CommonJS`标准。

`CommonJS`规范中：
- 一个文件就是一个模块，拥有单独的作用域

- 普通方式定义的变量、函数、对象都属于该模块内

- 通过`require`来加载模块:`require`命令的基本功能是，读入并执行一个JavaScript文件，然后返回该模块的exports对象。如果没有发现指定模块，会报错。

- 通过`exports`和`modul.exports`来暴露模块中的内容

有一点要注意：`CommonJS`规范加载模块是同步的，也就是说，只有加载完成，才能执行后面的操作

### AMD规范与CommonJS规范的兼容性
`CommonJS`规范加载模块是同步的,`AMD`规范则是非同步加载模块，允许指定回调函数。由于`Node.js`主要用于服务器编程，模块文件一般都已经存在于本地硬盘，所以加载起来比较快，不用考虑非同步加载的方式，所以`CommonJS`规范比较适用。但是，如果是浏览器环境，要从服务器端加载模块，这时就必须采用非同步模式，因此浏览器端一般采用AMD规范。

`AMD`规范允许输出的模块兼容`CommonJS`规范，这时`define`方法需要写成下面这样：
```js
define(function (require, exports, module){
  var someModule = require("someModule");
  var anotherModule = require("anotherModule");

  someModule.doTehAwesome();
  anotherModule.doMoarAwesome();

  exports.asplode = function (){
    someModule.doTehAwesome();
    anotherModule.doMoarAwesome();
  }
})
```
## ES6 Module

> 在 ES6 之前，社区制定了一些模块加载方案，最主要的有 CommonJS 和 AMD 两种。前者用于服务器，后者用于浏览器。ES6 在语言标准的层面上，实现了模块功能，而且实现得相当简单，完全可以取代 CommonJS 和 AMD 规范，成为浏览器和服务器通用的模块解决方案。

ES6 模块的设计思想是尽量的静态化，使得编译时就能确定模块的依赖关系，以及输入和输出的变量。`CommonJS` 和 `AMD` 模块，都只能在运行时确定这些东西。比如，`CommonJS` 模块就是对象，输入时必须查找对象属性。

ES6 模块不是对象，而是通过`export`命令显式指定输出的代码，再通过`import`命令输入。
```js
/*
  下面这些代码 实际上是从从 fs模块加载 3 个方法，其他方法不加载。这种加载称为“编译时加载”或者静态加载，
  即 ES6 可以在编译时就完成模块加载，效率要比 CommonJS 模块的加载方式高
*/
import { stat, exists, readFile } from 'fs' 
```

除了静态加载带来的各种好处，ES6 模块还有以下好处。

- 不再需要`UMD`模块格式了，将来服务器和浏览器都会支持 ES6 模块格式。目前，通过各种工具库，其实已经做到了这一点。
- 将来浏览器的新 `API` 就能用模块格式提供，不再必须做成全局变量或者`navigator`对象的属性。
- 不再需要对象作为命名空间（比如`Math`对象），未来这些功能可以通过模块提供。

### require 和 import的区别

这个也有很多人搞混，既然讲到这儿那我们就好好谈一谈。

本质上的差别：
- `CommonJS` 还是 `ES6 Module` 输出都可以看成是一个具备多个属性或者方法的对象
- `default` 是 `ES6 Module` 所独有的关键字，`export default fs` 输出默认的接口对象，`import fs from 'fs'` 可直接导入这个对象；
- `ES6 Module` 中导入模块的属性或者方法是**强绑定的，包括基础类型**；而 `CommonJS` 则是普通的值传递或者引用传递。

    这个可能不是很好理解，看下面的例子
    ```js
    // counter.js
    exports.count = 0
    setTimeout(function () {
      console.log('increase count to', ++exports.count, 'in counter.js after 500ms')
    }, 500)

    // commonjs.js
    const {count} = require('./counter')
    setTimeout(function () {
      console.log('read count after 1000ms in commonjs is', count)
    }, 1000)

    //es6.js
    import {count} from './counter'
    setTimeout(function () {
      console.log('read count after 1000ms in es6 is', count)
    }, 1000)
    // 分别运行
    ➜  test node commonjs.js
    increase count to 1 in counter.js after 500ms
    read count after 1000ms in commonjs is 0 // 仅仅传递出了count的值
    ➜  test babel-node es6.js
    increase count to 1 in counter.js after 500ms
    read count after 1000ms in es6 is 1 // 对count进行了强绑定
    ```
- `import`是`read-only`的

## UMD

`CommonJS module`以服务器端为第一的原则发展，选择同步加载模块。它的模块是无需包装的,但它仅支持对象类型（objects）模块。`AMD`以**浏览器为第一**（browser-first）的原则发展，选择异步加载模块。它的模块支持对象、函数、构造器、字符串、JSON等各种类型的模块,因此在浏览器中它非常灵活。这迫使人们想出另一种更通用格式 `UMD(Universal Module Definition)`，希望提供一个前后端跨平台的解决方案。

UMD的目标很简单，先判断是否支持`AMD`（`define`是否存在），存在则使用`AMD`方式加载模块。再判断是否支持`Node.js`模块格式（`exports`是否存在），存在则使用`Node.js`模块格式。前两个都不存在，则将模块公开到全局（`window或global`）。简而言之是为了进行服务端和客户端的兼容。比如一个库要同时兼容客户端和服务端那么最好是使用`UMD`格式规范。

# 总结
`AMD`和`CMD`迟早会是过去，`CommonJS`注重服务端，是同步的。`ES6 Module`是官方标准，推荐使用。`UMD`是为了兼容客户端和服务端提出的一种折中方案。

[原文地址](https://github.com/QDMarkMan/CodeBlog/blob/master/Javascript/JavaScript中的模块.md)  如果觉得有用得话给个⭐吧

