# Vue-cli webpack升级4.0
webpack早已经升级到了4.X版本。 对应的Vue-cli3.0 使用的虽然时webpack4.0， 但是不支持eject，所以暂时不准备去使用vue-cli3.0。 暂时对vue-cli2.9 webpack进行升级
> webpackV4.8.3 + vue2.5.2
## 升级准备
1. 升级核心库<small>对用稳定版本可在npm官网查找</small>
```bash
yarn upgrade webpack@4.8.3 webpack-dev-server@3.1.5 html-webpack-plugin@3.2.0 
```
2. 增加新库

  ```webpack-cli``` 新版的启动webpack 需要额外的command执行
  ```mini-css-extract-plugin``` 官方推荐的新的css抽离插件
  ```bash
yarn add webpack-cli mini-css-extract-plugin -D
  ```
3. 升级loader 建议升级所有的loader,尤其是vue-loader。直接重新安装就行

    - babel-loader 7.1.5
    - css-loader 1.0.0
    - file-loader 1.1.11
    - less-loader 4.1.0
    - url-loader 1.0.1
    - vue-style-loader 4.1.0
    - vue-template-compiler 2.5.16
    - <font color="red">vue-loader</font>
    - 。。。
``` bash
yarn add babel-loader babel-core babel-preset-env url-loader file-loader less-loader vue-style-loader vue-template-compiler css-loader eslint-loader inject-loader postcss-loader vue-loader -D
```
在这里有一个小小的坑 vue-template-compiler 和 vue的版本号一定要对应起来，要不然会一致报错找不到vue-template-compiler模块

4. 还有一个更加快捷的方法 ```npm outdated``` 列出所以可以更新的包。免得再一个个去npm找相对于的可用版本了。如果你够熟练了，推荐你这么干。
## 开始升级
### 主要部分升级
首先对webpack必要的部分进行升级，不升级就跑不动的那一部分就叫必要的部分。
  - mode: "development/production" webpack新增特性，表示当前webpack运行环境,在production环境下，webpack自动开启了代码压缩等等。

    **development模式**
    1. 主要优化了 **增量构建** 速度和开发体验
    2. process.env.NODE_ENV 的值不需要再定义，默认是 development
    3. 开发模式下支持注释和提示，并且支持 eval 下的 source maps

    **production模式**
    1. 生产环境默认开启了很多代码优化（minify，splite等）
    2. 开发时开启注视和验证，并且自动加上了eval devtool
    3. 生产环境不支持watching，开发环境优化了重新打包的速度
    4. 默认开启了Scope hoisting和Tree-shaking（原ModuleConcatenationPlugin）
    5. 自动设置process.env.NODE_ENV到不同环境，也就是不需要DefinePlugin来做这个了
    6. 如果你给mode设置为none，所有默认配置都去掉了
  - vue-loader: vue-loader15 版本需要在插件中声明才可以正常的使用
  ```js
  const { VueLoaderPlugin } = require('vue-loader')
  plugins: [
    new VueLoaderPlugin(),
  ]
  ``` 
  - webpack.optimize.UglifyJsPlugin: 这个整个在V4版本已经废弃，使用```optimization```选项或者是 ``` uglifyjs-webpack-plugin ```来代替。示例代码后续加上
  - optimization.splitChunks: webvpack4中改动最大，影响也最大的就是webpack4使用```optimization.splitChunks```替代了```CommonsChunkPlugin```。以前的```CommonsChunkPlugin```主要用来抽取代码中的共用部分，webpack runtime之类的代码，结合```chunkhash```，实现最好的缓存策略。而```optimization.splitChunks```则实现了相同的功能，并且配置更加灵活。

### 其他部分升级
  - mini-css-extract-plugin: 4.X版本官方推荐用来分离CSS插件。 ```extract-text-webpack-plugin```的最新正式版还没有对webpack4.x进行支持，即使是使用extract-text-webpack-plugin@next版本依然会出现报contenthash错误。使用一样的简单。需要修改两个地方：
    1. loader部分
    
      使用脚手架的话需要更改```utils.js```中的laoder配置
      ```js
      const MiniCssExtractPlugin = require('mini-css-extract-plugin')
      if (options.extract) {
        return [
          {
            loader: MiniCssExtractPlugin.loader,
            options: {
              publicPath: '../../'
            }
          }
          ].concat(loaders)
      } else {
        return ['vue-style-loader'].concat(loaders)
      }
      ```
    2. plugin部分
    
      分离出css文件的话，一般在正式环境用的着<small>测试环境不推荐使用，会影响性能</small>所以直接在```webpack.prod.conf.js```配置即可
      ```js
      const MiniCssExtractPlugin = require('mini-css-extract-plugin')
      plugins: [
        new MiniCssExtractPlugin({
          filename: utils.assetsPath('css/[name].[contenthash].css'),
          chunkFilename: utils.assetsPath('css/[id].[contenthash].css'),
        }),
      ]
      ```
      这个地方多说一嘴关于filename和chunkFilename的区别: <font color="orange">filename 是指在你入口文件entry中引入生成出来的文件名，而chunkname是指那些未被在入口文件entry引入，但又通过按需加载（异步）模块的时候引入的文件。</font>

  - 压缩： css的压缩最好是依靠插件```optimize-css-assets-webpack-plugin``` 这个不仅仅可以对代码进行压缩，还可以对css代码进行优化合并。但是这个地方有一个小坑： <font color="orange">在```cssProcessorOptions```配置中```safe```选项已经修改为```parser: require('postcss-safe-parser') ```，</font> 配置的话
  
    ```js
    const OptimizeCSSPlugin = require('optimize-css-assets-webpack-plugin')
    plucins: [new OptimizeCSSPlugin({
        cssProcessorOptions: config.build.productionSourceMap ? { parser: require('postcss-safe-parser'), map: { inline: false } } : { parser: require('postcss-safe-parser'), discardComments: { removeAll: true } },
        cssProcessor: require('cssnano')
      })
    ]
    ```
    - optimization选项配置： 这个[官网](https://webpack.js.org/configuration/optimization/#src/components/Sidebar/Sidebar.jsx)解释的比较全面
    下面是修改完配置
    ```js
    optimization: {
      providedExports: true,
      usedExports: true,
      //识别package.json中的sideEffects以剔除无用的模块，用来做tree-shake
      //依赖于optimization.providedExports和optimization.usedExports
      sideEffects: true,
      //取代 new webpack.optimize.ModuleConcatenationPlugin()
      concatenateModules: true,
      //取代 new webpack.NoEmitOnErrorsPlugin()，编译错误时不打印输出资源。
      noEmitOnErrors: true,
      splitChunks: {
        chunks: 'all', //'all'|'async'|'initial'(全部|按需加载|初始加载)的chunks
        /* cacheGroups: {
          name: "chunk-iview", // 单独将拆包
          priority: 20, // 权重
          test: /[\\/]node_modules[\\/]iview[\\/]/
        } */
      },
      //提取webpack运行时的代码
      runtimeChunk: {
        name: 'manifest'
      }
    }
    ```
  - html-webpack-plugin： 其实这个可以不需要升级，但是如果升级至15.x版本以上，在使用中需要执行VueLoaderPlugin插件方法，其他用法跟之前保持一致。

  - 还有一些懒加载等等优化的东西没有写，下次再写
## 注意事项
  - html-webpack-include-assets-plugin： 在做项目优化的时候用到了这个插件，但是更新之后死活就是不起作用了。经过再三的尝试之后发现是这个好像是依赖```html-webpack-plugin```,需要先出来页面才能往页面中插入cdn资源，所以我就把```html-webpack-plugin```集中到```webpack.base.conf.js```中之后解决。
  ```js
  const HtmlWebpackIncludeAssetsPlugin = require('html-webpack-include-assets-plugin')
  // 需要注入的cdn 引用的一些外部的样式
  const assets =[
    {path: `https://cdn.bootcss.com/normalize/8.0.0/normalize.min.css`, type: 'css'},
    {path: `https://cdn.bootcss.com/animate.css/3.5.2/animate.min.css`, type: 'css'},
    {path: `https://cdn.bootcss.com/iview/2.14.0/styles/iview.css`, type: 'css'},
  ]
  const minifyConfig = {
    removeComments: true ,
    collapseWhitespace: true ,
    removeAttributeQuotes: true ,
    removeEmptyAttributes: true 
  }
  plugins: [
    // html plugin dev/prod 幻剑融合
    new HtmlWebpackPlugin({
      filename: process.env.NODE_ENV  === 'production' ? config.build.index : 'index.html',
      template: 'index.html',
      inject: true,
      minify: process.env.NODE_ENV === 'production' ? minifyConfig : {},
      chunksSortMode: 'dependency'
    }),
    // 插入文件
    new HtmlWebpackIncludeAssetsPlugin({
      assets,
      append: process.env.NODE_ENV !== 'production',
      publicPath: '',
    }),
  ]
  ```

  # 总结
  vue-cli中的webpack升级总的来说应该还算是比较平滑的，webpack4大大的增加构建的性能。还有一些好的东西会慢慢引入进来, [项目地址](https://github.com/QDMarkMan/vue-base-template)