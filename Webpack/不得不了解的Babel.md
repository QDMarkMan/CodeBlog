# 不得不了解的babel
`Babel`

## 概览

- `@babel/runtime`: 一个包含`Babel`模块化运行时助手的库, 主要作用是**将转译的辅助代码从在文件中硬编码方式变为运行时的模块注入方式从而达到在某些条件下（重复代码过多时）缩小编译后的代码体积**。
- `@babel/transform-runtime`: **类库的打包如果需要注入 polyfill 的话，最好使用 @babel/transform-runtime，因为它提供了一种不污染全局作用域的方式**。
  
## Preset
> 预设，相当于编译到哪一个版本。也可以说是某一类 plugin 的集合，包含了某一类插件的所有功能。每年每个 preset 只编译当年批准的内容。 而 babel-preset-env 相当于 es2015 ，es2016 ，es2017 及最新版本。

使用的时候需要安装对应的插件，对应`babel-preset-xxx`，例如下面的配置，需要`npm install babel-preset-env`。
```bash
{
  "presets": ["es2015"]
}
```

如果你搞不懂或者是没有明确的目标使用什么preset，推荐使用`babel-preset-env`，至于为什么，就像上面说的那样。

### env配置

```bash
"presets": [
  ["env", {
    "modules": false,
    "targets": { 
      "browsers": ["> 1%", "last 2 versions", "not ie <= 8"]
    }
  }],
  "stage-2"
]
```
- **targets**： 需要支持的环境，可选例如：chrome, edge, firefox, safari, ie, ios, node，甚至可以制定版本，如`node: 6.5`。也使用`node: current代`表使用当前的版本。

- **browsers**： 浏览器列表，具体是哪些浏览器可以看[browserslist](https://browsersl.ist/)。像这些`" > 1%", "last 2 versions"` 都是查询参数

- **loose**： 是否使用宽松模式，如果设置为`true`，plugins里的插件如果允许，都会采用宽松模式。

- **debug**： 编译是否会去掉`console.log`。这个配置不是必要的。

- **whitelist**： 设置一直引入的plugins列表。

- **es2015/es2016/es2017/latest** 
    - `es2015` 使用es2015的，也就是我们常说的es6的相关方法
    - `es2016` 使用es2016的相关插件，也就是es7.
    - `es2017` 使用es2017的相关插件，也就是es8
    - `latest` latest是一个特殊的presets，包括了es2015，es2016，es2017的插件（目前为止，以后有es2018也会包括进去）。
    - `react` react是一个比较特别的官方推荐的presets，大概是因为比较火吧。加入了`flow`，`jsx`等。


## Stage(stage-0/1/2/3/4)

`stage-x`和上面的`es2015`等有些类似，但是它是按照`JavaScript`的提案阶段区分的，一共有5个阶段。而数字越小，阶段越靠后，存在依赖关系。也就是说`stage-0`是包括`stage-1`的，以此类推。

`stage-4`应该是包括es2015，es2016，es2017。经过测试，babel-preset-stage-4这个npm包是不存在的，如果你单纯的需要stage-4的相关方法，需要引入es2015~es2017的presets。


## Plugin

插件配置:  将某一种需要转化的代码，转为浏览器可以执行代码。插件需要搭配`babel`相应的插件进行配置，可以选择配置插件来满足单个需求。但是这些插件从维护到书写极为麻烦，后来官方统一推荐使用`env`，全部替代了这些单一的插件功能，可以简化配置如下，也就是前面提到了`babel-preset-env`：

这里主要介绍两款常用插件，分别是`babel-plugin-transform-runtime`，`babel-plugin-syntax-dynamic-import`。

- **transform-runtime**

为了解决这种全局对象或者全局对象方法编译不足的情况，才出现了transform-runtime这个插件，但是它只会对es6的语法进行转换，而不会对新api进行转换

- 解决编译中产生的重复的工具函数，减小代码体积
- 非实例方法的`poly-fill，如Object.assign`，但是实例方法不支持，如`"foobar".includes("foo")，`这时候需要单独引入`babel-polyfill`

```bash
{
  "plugins": ["transform-runtime", options]
}
```

`options`有以下的选项
- `helpers` 使用babel的helper函数。 
- `polyfill` 使用babel的polyfill，但是不能完全取代babel-polyfill。
- `regenerator` 使用babel的regenerator。
- `moduleName` 使用对应module处理

- **syntax-dynamic-import**

这个插件主要解决动态引入模块的问题，它会所有的`import` 转化为`require`引入。


## 执行顺序

1. 执行 `plugins` 中所有的插件
2. `plugins` 的插件，按照顺序依赖编译
3. 所有 `plugins` 的插件执行完成，在执行 `presets` 预设。
4. `presets` 预设，按照倒序的顺序执行。(从最后一个执行)
5. 完成编译。

## babel-runtime

`Babel` 转译后的代码要实现源代码同样的功能需要借助一些帮助函数，例如，`{ [name]: 'JavaScript' }` 转译后的代码如下所示：

```js
'use strict';
function _defineProperty(obj, key, value) {
  if (key in obj) {
    Object.defineProperty(obj, key, {
      value: value,
      enumerable: true,
      configurable: true,
      writable: true
    });
  } else {
    obj[key] = value;
  }
  return obj;
}
var obj = _defineProperty({}, 'name', 'JavaScript');
```
这显示是不合适的，导致编译后的代码体积变大。`Babel` 为了解决这个问题，提供了单独的包` babel-runtime `**供编译模块复用工具函数**。

启用插件 `babel-plugin-transform-runtime` 后，`Babel` 就会使用 `babel-runtime` 下的工具函数，转译代码如下：

```js
'use strict';
// 之前的 _defineProperty 函数已经作为公共模块 `babel-runtime/helpers/defineProperty` 使用
var _defineProperty2 = require('babel-runtime/helpers/defineProperty');
var _defineProperty3 = _interopRequireDefault(_defineProperty2);
function _interopRequireDefault(obj) { return obj && obj.__esModule ? obj : { default: obj }; }
var obj = (0, _defineProperty3.default)({}, 'name', 'JavaScript');
```

但是`babel-runtim`还是更适合`JavaScript`库和工具包的实现, 原因如下

1. 避免 `babel` 编译的工具函数在每个模块里重复出现，减小库和工具包的体积。
2. 在没有使用 `babel-runtime` 之前，库和工具包一般不会直接引入`polyfill`。否则像 `Promise` 这样的全局对象会污染全局命名空间，这就要求库的使用者自己提供 `polyfill`。这些 `polyfill` 一般在库和工具的使用说明中会提到，比如很多库都会有要求提供 `es5` 的 `polyfill`。在使用` babel-runtime `后，库和工具只要在 `package.json` 中增加依赖 `babel-runtime`，交给 `babel-runtime` 去引入 polyfill 就行了。


## babel-ployfill

因为`babel`默认只是转换新的`JavaScript`句法，而不转换新的`API `比如`Iterator Generator Set Maps Proxy Relfact Symbol Promise `等全局对象，所以我们需要直接对代码进行垫片操作，IE的兼容是非常的必须的。使用方式：

配合`webpack`使用的话在客户端入口文件中引用一下再在配置文件中加入入口。
```js
// entry
require("babel-polyfill");
//webpak config
module.exports = {
　　entry: ["babel-polyfill", "./app/js"]
}
```
和`babel-runtime`一对比的话：

`babel-runtime`是供编译模块复用工具函数。是锦上添花
`babel-polyfil`是雪中送炭，是转译没有的`api`.

## `browserslist` 

我感觉很多人不知道这个怎么配置

根据提供的目标浏览器的环境来，智能添加`css`前缀，`js`的`polyfill`垫片,来兼容旧版本浏览器，而不是一股脑的添加。避免不必要的兼容代码，以提高代码的编译质量。

我们接下来看看下面的参数代表什么意思，下面的参数看会了，基本就会配置了，这里是[官方文档](https://github.com/browserslist/browserslist#queries)

| 配置 | 说明 |
| :------ | :------ |
| `> 1%` | 全球超过1%人使用的浏览器 |
| `last 2 versions` | 所有浏览器兼容到最后两个版本根据CanIUse.com追踪的版本 |
| `> 5% in US	` | 指定国家使用率覆盖 |
| `Firefox ESR` | 火狐最新版本 |
| `Firefox > 20` | 指定浏览器的版本范围 |
| `not ie <=8` | 方向排除部分版本 |
| `Firefox 12.1	` | 指定浏览器的兼容到指定版本 |
| `unreleased versions	` | 所有浏览器的beta测试版本 |
| `unreleased Chrome versions	` | 指定浏览器的测试版本 |
| `since 2013	` | 2013年之后发布的所有版本 |


# 总结

`Babel`很强大，但是我希望有一天能够不再需要了😁