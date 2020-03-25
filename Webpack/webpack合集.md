# Webpack合集

> Webpack中重要的东西其实不多，大部分都是基础的东西。

## `exclude/include`

`Rule.exclude/include`是 `Rule.resource.exclude/include`的缩写，通过在`loader`中的配置，我们可以转译尽可能少的文件。

**value**: 绝对路径数组

- `exclude`: 指定排除文件。
- `include`: 指定要包含的文件。

demo:

```js
module.exports = {
    //...
    module: {
        rules: [
            {
                test: /\.js?$/,
                use: ['babel-loader'],
                include: [path.resolve(__dirname, 'src')]
            }
        ]
    },
}
```

## `noParse`

可以用来标识一些没有`AMD/CommonJS`规范版本的模块。忽略的文件中不应该含有 `import, require, define` 的调用，或任何其他导入机制。忽略大型的 `library` 可以提高构建性能, 比如`jquery`。

**value**：`[RegExp] | function`

demo:

```js
module.exports = {
    //...
    module: {
        noParse: /jquery|lodash/
    },
    // 3.0 开始
    noParse: function(content) {
      return /jquery|lodash/.test(content);
    }
}
```

## `Cache`

`cache`主要是应对一些开销较大的`loader`。将缓存结果保存在本地。

启用cache有两种方式

- `cache-loader`：安装之后在其他loader之前使用。

  ```js
  module: {
    rules: [
      {
        test: /\.js$/,
        use: ['cache-loader','babel-loader']
      }
    ]
  }
  ```

- `babel-loader`的`cacheDirectory`选项。

  ```js
  module: {
    rules: [
      {
        test: /\.js$/,
        use: {
          loader: 'babel-loader‘,
          options: {
            cacheDirectory: true
          }
        }
      }
    ]
  }
  ```


## `resolve`

配置模块如何解析。例如，当在 ES2015 中调用 `import "lodash"`，`resolve` 选项能够对 webpack 查找 `"lodash"` 的方式去做修改。

如果我们要确定模块都从根目录下查找可以这么配置，但是如果你的依赖中还存在 `node_modules` 目录，那就会出现找不到的情况，一定要慎重使用。

```js
//webpack.config.js
const path = require('path');
module.exports = {
    //...
    resolve: {
        modules: [path.resolve(__dirname, 'node_modules')],
    }
}
```

## `splitChunks`

```js
module.exports = {
  //...
  optimization: {
    splitChunks: {
      chunks: 'async', //这表示将选择哪些块进行优化。  必须三选一： "initial" | "all"(推荐) 提供all可以特别强大，因为这意味着即使在异步和非异步块之间也可以共享 | "async" (默认就是async)
      minSize: 30000, // (默认是30000)：形成一个新代码块最小的体积
      maxSize: 0, // 在分割之前，这个代码块最小应该被引用的次数（译注：保证代码块复用性，默认配置的策略是不需要多次引用也可以被分割）
      minChunks: 1, // 
      maxAsyncRequests: 5, // 按需加载时候最大的并行请求数。
      maxInitialRequests: 3, // 一个入口最大的并行请求数
      automaticNameDelimiter: '~',//默认情况下，webpack将使用块的来源和名称生成名称（例如vendor〜main.js）。 可以指定用于生成名称的定界符。
      name: true, // 字符串或者函数(函数可以根据条件自定义名字)
      cacheGroups: { // 这里开始设置缓存的 chunks
        vendors: {
          chunks: "initial", // 必须三选一： "initial" | "all" | "async"(默认就是async) 
            test: /node_modules/, // 正则规则验证，如果符合就提取 chunk
            name: "vendor", // 要缓存的 分隔出来的 chunk 名称 
            minSize: 30000, // 大小操作30000字节
            minChunks: 1, //最少引用了一次
            enforce: true, // 告诉webpack忽略splitChunks.minSize，splitChunks.minChunks，splitChunks.maxAsyncRequests和splitChunks.maxInitialRequests选项，并始终为此缓存组创建块。
            maxAsyncRequests: 5, // 最大异步请求数， 默认1
            maxInitialRequests : 3, // 最大初始化请求书，默认1
            reuseExistingChunk: true // 可设置是否重用该chunk
        },
        default: {
          minChunks: 2,
          priority: -20,
          reuseExistingChunk: true
        }
      }
    }
  }
};
```

## `node`

> 这些选项可以配置是否 polyfill 或 mock 某些 Node.js 全局变量和模块。这可以使最初为 Node.js 环境编写的代码，在其他环境（如浏览器）中运行。 此功能由 webpack 内部的 NodeStuffPlugin 插件提供。如果 target 是 "web"（默认）或 "webworker"，那么 NodeSourcePlugin 插件也会被激活。

当你需要在某些环境使用`Node`的一些特性，比如`fs, path, __dirname`等等，那么就要配置该选项了。

注意： **只有当`target`是未指定、`web` 或 `webworker` 这三种情况时，此选项才会被激活（通过 `NodeSourcePlugin`）。**

默认值

```js
module.exports = {
  //...
  node: {
    // 以下每个属性都是node中的某个模块或者全局变量
    console: false,
    global: true,
    process: true,
    __filename: 'mock',
    __dirname: 'mock',
    Buffer: true,
    setImmediate: true
    // ...
  }
};
```

属性的取值有以下几种

- `true`: `webpack`提供`polyfill`。
- `mock`: 提供 mock 实现预期接口，但功能很少或没有。
- `empty`: 提供空对象。
- `false`:  什么都不提供。预期获取此对象的代码，可能会因为获取不到此对象，触发 ReferenceError 而崩溃。


## 如何打包一个库

当我们使用`webpack`来开发啊我们自己的库文件的时候有一些东西是必须要理解的，下面就来一起看看。

在打包成模块的时候我们一般是对输出的文件去做配置，我们看`element-ui`这个配置文件

```js
module.exports = {
  //...
  output: {
    path: path.resolve(process.cwd(), './lib'),
    publicPath: '/dist/',
    filename: 'element-ui.common.js',
    chunkFilename: '[id].js',
    libraryExport: 'default', // 指定输出的模块
    library: 'ELEMENT', // 库名称
    libraryTarget: 'commonjs2'// 库的模块类型
  },
};
```
上面这个定义了一个输出模块名称为`someLibName`， 文件名称为`someLibName.js`的`umd`模块

###  `output.library`

`library`一般是作为库的名称去设置，比如`Element-UI`的全局变量是`ELEMENT`那么`library`就是`ELEMENT`, 这个东西问题不大

### `output.libraryExport`

> 配置将通过的公开哪些模块libraryTarget。这是undefined在默认情况下，如果设置相同的行为将被应用libraryTarget到一个空字符串如''将导出整个空间（namespace）对象。

说白了就是你要`export`哪些东西

- `libraryExport: 'default'`文件的的默认导出将分配给库目标
```js
// if your entry has a default export of `MyDefaultModule`
var MyDefaultModule = _entry_return_.default;
```

- `libraryExport: 'MyModule'` 导出指定的模块
```js
var MyModule = _entry_return_.MyModule;
```

- `libraryExport: ['MyModule', 'MySubModule']` 导出多个模块
```js
var MySubModule = _entry_return_.MyModule.MySubModule;
```

通过对`libraryExport`的配置我们实际上是可以生成以下类型的库
```js
MyDefaultModule.doSomething(); // default
MyModule.doSomething(); // MyModule
MySubModule.doSomething(); // MySubModule
```

### `output.libraryTarget`
> 配置库的公开方式。 请注意，此选项与分配给`output.library`的值一起使用。

这个就比较重要了, 这个配置决定了你的库以什么样的模块来到处进而来适用不同的环境`(AMD, commonjs, ES6等等)`

- `output.libraryTarget`: `var`默认值  入口点的返回值将分配给变量：
```js
var MyLibrary = _entry_return_; // _entry_return_ 配置的library
// In a separate script...
MyLibrary.doSomething();
```
- `output.libraryTarget`: `assign`  这将生成隐含的全局，有可能重新分配现有值（不推荐使用）
```js
MyLibrary = _entry_return_;
```
请注意，如果`MyLibrary`之前未定义，则您的库将设置在全局范围内。

- `output.libraryTarget`: `this`  分配给当前模块的`this` 这个也挺玄幻的， 我没用过
```js
this['MyLibrary'] = _entry_return_;

// In a separate script...
this.MyLibrary.doSomething();
MyLibrary.doSomething(); // if this is window
```
- `output.libraryTarget`: `window` 分配给`window`变量, 这是在浏览器使用的时候可以采用这种方式

```js
window['MyLibrary'] = _entry_return_;
window.MyLibrary.doSomething();
```

-★ `output.libraryTarget`: `commonjs`  分配给分配给`exports`对象`output.library`, 在`commonjs`中使用
```js
exports['MyLibrary'] = _entry_return_;
require('MyLibrary').doSomething();
```

- `output.libraryTarget`: `commonjs2`  分配给`module.exports`，至于和上面的有什么区别， 可以看[这篇文章](https://github.com/QDMarkMan/CodeBlog/blob/master/Javascript/JavaScript%E4%B8%AD%E7%9A%84%E6%A8%A1%E5%9D%97.md)
```js
module.exports = _entry_return_;

require('MyLibrary').doSomething();
```
- `output.libraryTarget`: `amd`  作为AMD模块公开，现在也很少用了
```js
// 生成的输出将使用名称“MyLibrary”定义
define('MyLibrary', [], function() {
  return _entry_return_;
});
```
-★ `output.libraryTarget`: `umd`  这会在所有模块定义下公开您的库，允许它与CommonJS，AMD和全局变量一起使用。
