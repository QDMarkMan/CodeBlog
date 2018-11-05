# 不得不了解的babel
`Babel`

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
# 总结
`Babel`很强大，但是我希望有一天能够不再需要了😁