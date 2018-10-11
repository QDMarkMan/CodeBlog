# Webpack 中tree-shaking的威力
> tree shaking 是一个术语，通常用于描述移除 JavaScript 上下文中的未引用代码(dead-code)。它依赖于 ES2015 模块系统中的静态结构特性，例如 import 和 export。这个术语和概念实际上是兴起于 ES2015 模块打包工具 rollup。

**tree-shaking**语义上来解释就是摇树，理解起来就像是摇树，把无用的树叶（代码）全部都清除掉。以达到减少体积的目的。

## 使用

webpack实际上在我们使用的时候已经开启了，但是我们没有特意的去配置。比如我们在开启**production**模式的时候webpack会贴心的为你开启**tree-shaking**模式。

## 按照官方的示例来使用一下

> 新的 webpack 4 正式版本，扩展了这个检测能力，通过 package.json 的 "sideEffects" 属性作为标记，向 compiler 提供提示，表明项目中的哪些文件是 "pure(纯的 ES2015 模块)"，由此可以安全地删除文件中未使用的部分。

- 添加通用模块Math.js
```js

export function square(x) {
  return Math.pow(x,2)
}

export function cube(x) {
  return Math.pow(x, 3)
}
```
- 在index中引用
```js
// webpack 官方tree-shaking demo
import {cube} from './math'

function component () {
  var element = document.createElement('pre')
  element.innerHTML = [
       'Hello webpack!',
       '5 cubed is equal to ' + cube(5)
     ].join('\n\n')
  return element
}

document.body.appendChild(component())

```
<font color="red">注意：</font> **我们导入了cube 但是我们从来没有导入square方法**,这就是所谓的未引用代码(dead code)。

- 清除无用代码

这里需要将文件标记为无副作用(side-effect-free)
>在一个纯粹的 ESM 模块世界中，识别出哪些文件有副作用很简单。然而，我们的项目无法达到这种纯度，所以，此时有必要向 webpack 的 compiler 提供提示哪些代码是“纯粹部分”。

这种方式是通过 package.json 的 "sideEffects" 属性来实现的。
```bash
{
  "name": "your-project",
  "sideEffects": false
}
如果你的代码确实有一些副作用，那么可以改为提供一个数组：
{
  "name": "your-project",
  "sideEffects": [
    "./src/some-side-effectful-file.js"
  ]
}
```
数组方式支持相关文件得绝对路径，相对路径和glob模式

<font color="red">注意：</font> 任何导入的文件都会受到 tree shaking 的影响。这意味着，如果在项目中使用类似 css-loader 并导入 CSS 文件，则需要将其添加到 side effect 列表中，以免在生产模式中无意中将它删除。

```bash
{
  "name": "your-project",
  "sideEffects": [
    "./src/some-side-effectful-file.js",
    "*.css"
  ]
}
```
## 目前得缺点
**tree-shaking** 其实已经做得很好了， 但是还是有着一定得缺点，例如下面这种情况

下面是我们自己的一个模块
```js
import * as _ from 'lodash-es'

const fun1 = function (value) {
  return _.isArray(value)
}

const fun2 = function (value) {
  return value
}

export {
  fun1,
  fun2
}
```
下面来引用这个模块
```js
// tree-shaking无法起作用的地方
import {fun2} from './func'
/**
 * 在func.js文件中fun1 和 fun2 我们用到了fun2 但是fun2并没有用到lodash这个个库 所以这个时候等于引入了无用代码，
 * 这个时候webpack就没有能力把这些代码去除掉
 */
fun2(222)
```

我们可以明确的看出来lodash是我们没有用的模块，但是依然会被打包进去，这个时候webpack是shaking不掉这种代码的，但是```webpack-deep-scope-analysis-plugin```这个库可以做到，具体可以看[这个文章](https://juejin.im/post/5b8ce49df265da438151b468)


# 总结
业务层面得优化没有银弹，业务需要到哪一步，我们必须优化到哪一步。没有一个确定得死规则。
