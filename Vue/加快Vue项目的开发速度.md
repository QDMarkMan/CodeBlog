# 加快Vue项目的开发速度

> 现如今的开发，比如是内部使用的管理平台这种项目大都时间比较仓仓促。实际上来说在使用了webpack + vue 这一套来开发的话已经大大了提高了效率。但是对于我们的开发层面。还是有很多地方可以再次提高我们的项目开发效率，让我们更加专注于业务，毕竟时间就是生命。下面我们挨个来探讨。

## 巧用Webpack
`Webpack`是实现我们前端项目工程化的基础，但其实她的用处远不仅仅如此，我们可以通过`Webpack`来帮我们做一些自动化的事情。首先我们要了解`require.context()`这个API

### `require.context()`
> 您可以使用[require.context（）](https://webpack.js.org/guides/dependency-management/#require-context)函数创建自己的上下文。 它允许您传入一个目录进行搜索，一个标志指示是否应该搜索子目录，还有一个正则表达式来匹配文件。

其实是`Webpack`通过解析 `require()` 的调用，提取出来如下这些信息：
```bash
Directory: ./template
Regular expression: /^.*\.ejs$/
```
然后来创建我们自己的上下文，什么意思呢，就是`我们可以通过这个方法筛选出来我们需要的文件并且读取`

下面我们来简单看一看使用:
```js
/**
* @param directory 要搜索的文件夹目录不能是变量，否则在编译阶段无法定位目录
* @param useSubdirectories  是否搜索子目录
* @param regExp 匹配文件的正则表达式
* @return function 返回一个具有 resolve, keys, id 三个属性的方法
          resolve() 它返回请求被解析后得到的模块 id
          keys() 它返回一个数组，由所有符合上下文模块处理的请求组成。 
          id 是上下文模块里面所包含的模块 id. 它可能在你使用 module.hot.accept 的时候被用到
*/
require.context('demo', useSubdirectories = false, regExp = /\.js$/)
// (创建了）一个包含了 demo 文件夹（不包含子目录）下面的、所有文件名以 `js` 结尾的、能被 require 请求到的文件的上下文。
```
不要困惑，接下来我们来探讨在项目中怎么用。
### 组织路由
对于`Vue`中的路由，大家都很熟悉，类似于声明式的配置文件，其实已经很简洁了。现在我们来让他更简洁

1. 分割路由

  首先为了方便我们管理，我们把`router`目录下的文件分割为以下结构
  ```js
  router                           // 路由文件夹
    |__index.js                    // 路由组织器：用来初始化路由等等
    |__common.js                   // 通用路由：声明通用路由
    |__modules                     // 业务逻辑模块：所以的业务逻辑模块
          |__index.js              // 自动化处理文件：自动引入路由的核心文件
          |__home.js               // 业务模块home：业务模块
          |__a.js                  // 业务模块a
    
  ```
2. `modules`文件夹中处理业务模块
  
  `modules`文件夹中存放着我们所有的业务逻辑模块，至于业务逻辑模块怎么分，我相信大家自然有自己的一套标准。我们通过上面提到的`require.context()`接下来编写自动化的核心部分`index.js`。
  ```js
  const files = require.context('.', true, /\.js$/)

  console.log(files.keys()) // ["./home.js"] 返回一个数组
  let configRouters = []
  /**
  * inject routers
  */
  files.keys().forEach(key => {
    if (key === './index.js') return
    configRouters = configRouters.concat(files(key).default) // 读取出文件中的default模块
  })
  export default configRouters // 抛出一个Vue-router期待的结构的数组
  ```
  自动化部分写完了，那业务组件部分怎么写？ 这就更简单了
  ```js
  import Frame from '@/views/frame/Frame'
  import Home from '@/views/index/index'
  export default [
      // 首页
      {
        path: '/index',
        name: '首页',
        redirect: '/index',
        component: Frame, 
        children: [ // 嵌套路由
          {
            path: '',
            component: Home
          }
        ]
      }
  ]
  ```
3. `common`路由处理 
  我们的项目中有一大堆的公共路由需要处理比如`404`阿，`503`阿等等路由我们都在`common.js`中进行处理。
  ```js
  export default [
    // 默认页面
    {
      path: '/',
      redirect: '/index',
      hidden:true
    },
    // 无权限页面
    {
      path: '/nopermission',
      name: 'nopermission',
      component: () => import('@/views/NoPermission')
    },
    // 404
    {
      path: '*',
      name: 'lost',
      component: () => import('@/views/404')
    }
  ]
  ```
4. 路由初始化
  这是我们的最后一步了，用来初始化我们的项目路由
  ```js
  import Vue from 'vue'
  import VueRouter from 'vue-router'
  import RouterConfig from './modules' // 引入业务逻辑模块
  import CommonRouters from './common' // 引入通用模块
  Vue.use(VueRouter)
  export default new VueRouter({
    mode: 'history',// 需要服务端支持
    scrollBehavior: () => ({ y: 0 }),
    routes: RouterConfig.concat(CommonRouters)
  })
  ```
  估计有些朋友代码写到这还不知道到底这样做好处在哪里。我们来描述一个场景，比如按照这种结构来划分模块。正常的情况是我们创建完`home.js`要手动的把这个模块`import`到路由文件声明的地方去使用。但是有了上面的`index.js`，在使用的时候你只需要去创建一个`home.js`并抛出一个符合`VueRouter`规范的数组，剩下的就不用管了。`import RouterConfig from './modules' // 引入业务逻辑模块` 已经帮你处理完了。另外扩展的话你还可以把`hooks`拿出来作为一个单独文件。

### 全局组件统一声明
同样的道理，有了上面的经验，我们照葫芦画瓢来处理一下我们的全局组件。这就没什么可说的了，直接上核心代码
1. 组织结构
  ```js
  components                       // 组件文件夹
    |__xxx.vue                     // 其他组件
    |__global                      // 全局组件文件夹
          |__index.js              // 自动化处理文件
          |__demo.vue              // 全局demo组件
  ```
2. `global`处理
  ```js
  import Vue from 'vue'
  let contexts = require.context('.', false, /\.vue$/)
  contexts.keys().forEach(component => {
    let componentEntity = contexts(component).default
    // 使用内置的组件名称 进行全局组件注册
    Vue.component(componentEntity.name, componentEntity)
  })
  ```
3. 使用和说明
  
  这个使用起来就更简单了，直接在`app.js`引用这个文件就行。

  **注意**：我之前看到有些人做法是使用组件名去区分全局组件和普通组件，然后通过正则去判断需不需要全局注册。我是直接把全局的组件放到`global`文件夹下，然后组件的注册名称直接使用`component.name`。至于使用哪种方式就比较看个人了。

## 充分利用NodeJS
放着`node`这么好得东西不用真是有点浪费，那么我们来看看`node`能为我们增加效率做出什么贡献。

有这么一个场景，我们每次创建模块的时候都要新建一个`vue`文件和对应的`router`配置，而且新页面的大部分东西都还差不多，还得去复制粘贴别得页面。这想想就有点`low`。那既然有了`node`我们可不可以通过`node`来做这写乱七八糟得事情？ 下面来把我们的想法付诸于显示。

我们实现这个功能主要要借助`Node`的[fs](http://nodejs.cn/api/fs.html)和[process](http://nodejs.cn/api/process.html), 感兴趣的话可以深入研究一下。

首先我们要编写我们的`node`脚本，这里是一个比较简单的版本。什么验证文件夹或者文件的都没有，只是来实现我们这个想法：
```js
/*
 * fast add new module script
 */
const path = require('path')
const fs = require('fs')
const chalk = require('chalk')
const reslove = file => path.resolve(__dirname, '../src', file)
// symbol const
const RouterSymbol = Symbol('router'),
      ViewsSymbol = Symbol('views')
// root path
const rootPath = {
  [RouterSymbol]: reslove('router/modules'),
  [ViewsSymbol]: reslove('views')
}
//loggs
const errorLog = error => console.log(chalk.red(`${error}`))
const defaultLog = log => console.log(chalk.green(`${log}`))
// module name
let moduleName = new String()
let fileType = new String()
//const string
const vueFile = module => (`<template>

</template>

<script>
export default {
  name: '${module}',
  data () {
    return {

    }
  },
  methods: {

  },
  created() {
    
  }
}
</script>

<style lang="less">

</style>
`)
// route file
const routerFile = module => (`// write your comment here...
export default [
  {
    path: '/${module}',
    name: '',
    redirect: '/${module}',
    component: () => import('@/views/frame/Frame'),
    children: [
      {
        path: '',
        fullPath: '',
        name: '',
        component: () => import('@/views/${module}/index')
      }
    ]
  }
]
`)
/**
 * generate file
 * @param {*} filePath 
 * @param {*} content 
 * @param {*} dirPath 
 */
const generateFile = async (filePath, content, dirPath = '') =>{
  try {
    // create file if file not exit
    if (dirPath !== '' && ! await fs.existsSync(dirPath)) {
      await fs.mkdirSync(dirPath)
      defaultLog(`created ${dirPath}`)
    }
    if (! await fs.existsSync(filePath)) {
      // create file
      await fs.openSync(filePath, 'w')
      defaultLog(`created ${filePath}`)
    }
    await fs.writeFileSync(filePath, content, 'utf8')
  } catch (error) {
    errorLog(error)
  }
}
// module-method map
const generates = new Map([
  ['view', async (module) => {
    // module file
    const filePath = path.join(rootPath[ViewsSymbol], module)
    const vuePath = path.join(filePath, '/index.vue')
    await generateFile(vuePath, vueFile(module), filePath)
  }],
  // router is not need new folder
  ['router',async (module) => {
    const routerPath = path.join(rootPath[RouterSymbol], `/${module}.js`)
    await generateFile(routerPath, routerFile(module))
  }]
])
defaultLog(`请输入模块名称(英文)：`)
// files
const files = ['view', 'router']
// 和命令行进行交互 获取的创建的模块名称
process.stdin.on('data', (chunk) => {
  try {
    if (!moduleName) {
      moduleName = chunk
    } else {
      chunk = chunk.slice(0,-2) // delete /n
      defaultLog(`new module name is ${chunk}`)
      files.forEach(async (el, index) => {
        // 执行创建语句
        await generates.get(`${el}`).call(null, chunk.toString())
        if (index === files.length-1) {
          process.stdin.emit('end')
        }
      })
    }
  } catch (error) {
    errorLog(error)
  }
})
process.stdin.on('end', () => {
  defaultLog('create module success')
})
```
下面我们看使用的流程
![Vue](./node-vue.jpg 'Vue')
这样我们就分别创建了`vue`和`router`的文件，而且已经注入了内容。按照我们提前声明的组件

**注意：这只是一个简单的思路，通过Node强大的文件处理能力，我们能做的事情远不止这些。**

## 发挥Mixins的威力
`Vue`中的`混入`[mixins](https://cn.vuejs.org/v2/guide/mixins.html)是一种提供分发 `Vue` 组件中可复用功能的非常灵活的方式。听说在3.0版本中可能会用Hooks的形式实现，但这并不妨碍它的强大。基础部分的可以看[这里](https://github.com/QDMarkMan/CodeBlog/blob/master/Vue/Vue%E5%BC%80%E5%8F%91%E4%B8%AD%E9%97%AE%E9%A2%98%E9%9B%86%E9%94%A6.md)。这里主要来讨论`mixins`能在什么情景下帮助我们。


比如我们的大量的表格页面，仔细一扒拉你发现非常多的东西都是可以复用的例如`分页`，`表格高度`，`加载方法`， `laoding声明`等一大堆的东西。下面我们来整理出来一个简单的`list.vue`

### 通用mixins
```js
const list = {
  data () {
    return {
      // 这些东西我们在list中处理，就不需要在每个页面再去手动的做这个了。
      loading: false, // 伴随loading状态
      pageNo: 1, // 页码
      pageSize: 15, // 页长
      totalCount: 0, // 总个数
      pageSizes: [15, 20, 25, 30], //页长数
      pageLayout: 'total, sizes, prev, pager, next, jumper', // 分页布局
      list: []
    }
  },
  methods: {
    // 分页回掉事件
    handleSizeChange(val) {
      this.pageSize = val
      // todo
    },
    handleCurrentChange (val) {
      this.pageNo = val
      // todo
    },
    /**
     * 表格数据请求成功的回调 处理完公共的部分（分页，loading取消）之后把控制权交给页面
     * @param {*} apiResult 
     * @returns {*} promise
     */
    listSuccessCb (apiResult = {}) {
      return new Promise((reslove, reject) => {
        let tempList = [] // 临时list
        try {
          this.loading = false
          // todo
          // 直接抛出
          reslove(tempList)
        } catch (error) {
          reject(error)
        }
      })
    },
    /**
     * 处理异常情况
     * ==> 简单处理  仅仅是对表格处理为空以及取消loading
     */
    listExceptionCb (error) {
      this.loading = false
      console.error(error)
    }
  },
  created() {
    // 这个生命周期是在使用组件的生命周期之前
    this.$nextTick().then(() => {
      // todo
    })
  }
}
export default list
```
下面我们直接在组件中使用这个`mixins`
```js
import mixin from '@/mixins/list' // 引入
import {getList} from '@/api/demo'
export default {
  name: 'mixins-demo',
  mixins: [mixin], // 使用mixins
  data () {
    return {
    }
  },
  methods: {
    // 加载列表
    load () {
      const para = {
      }
      this.loading = true
      getList(para).then((result) => {
        this.listSuccessCb(result).then((list) => {
          this.list = list
        }).catch((err) => {
          console.log(err)
        })
      }).catch((err) => {
        this.listExceptionCb(err)
      })
    }
  },
  created() {
    this.load()
  }
}
</script>
```
使用了`mixins`之后一个简单的有`loadoing`, `分页`,`数据`的表格大概就只需要上面这些代码。

### mixins做公共数据的管理

有些时候我们有一些公共的数据它可能3，4个模块取使用但是又达不到全局的这种规模。这个时候我们就可以用`mixins`去管理他们，比如我们有几个模块要使用用户类型这个列表，我们来看使用`mixins`来实现共享。
```js
// types.js
import {getTypes} from '@/api/demo' // ajax
export default {
  data () {
    return {
      types: [] // ==>  {name: '', value: ''}
    }
  },
  methods: {
    // 获取列表
    getAllTypesList () {
      getApiList().then((result) => {
        // todo
        this.types = result // 假设result就是我们需要使用的数据
      }).catch((err) => {
        console.error(err)
      })
    }
  },
  created() {
    // 在需要使用这个mixins的时候取自动请求数据  这个可要可不要  你想在父组件中执行也是ok的
    this.getAllTypesList()
  }
}
```
在组件中引用
```js
import typeMixin from '@/mixins/types'
export default {
  name: 'template',
  mixins: [typeMixin],
  data () {
    return {
      // types这个数组在使用组件中不用多余的定义，直接拿来用就行
      type: ''
    }
  },
  methods: {
  }
}
```
直接使用
```html
<!--  -->
<el-select v-model="type" clearable placeholder="请选择类型">
    <el-option v-for="item in types" :key="item.id" :label="item.templateName" :value="item.id"></el-option>
  </el-select>
```
我们这样就可以不用`vuex`来去管理那些只有在模块间复用几次的数据，**而且非常方便得去取我们想要得数据，连定义都省了**。但是这有一个缺点。就是每次都会去重新请求这些数据。如果你不在乎这一点点瑕疵的话，我觉得用起来是完全ok得。

**注意：** <font color="red">`mixins`它固然是简单的，但是注释和引用一定要做好，不然的话新成员进入团队大概是一脸的懵逼，而且也不利于后期的维护。也是一把双刃剑。另外：全局`mixins`一定要慎用，如果不是必须要用的话我还是不建议使用。</font>

## 进一步对组件进行封装

大家都知道组件化的最大的好处就是高度的可复用性和灵活性。但是组件怎么封装好，封装到什么程度让我们更方便。这是没有标准的答案的。我们只有根据`高内聚，低耦合`的这个指导思想来对我们的业务通用组件来进行封装，让我们的业务页面结构更加的简洁，加快我们的开发效率。封装多一点的话页面可能会变成这样:
```html
<template>
  <box-content>
    <!-- 头部标题部分 -->
    <page-title>
      <bread slot="title" :crumbs="[{name: 'xx管理', path: '', active: true, icon: ''}, {name: 'xxxx', path: '', active: true, icon: ''}]"></bread>
    </page-title>
    <!-- 表格部分 -->
    <div>
      <base-table v-loading="loading" :columns="headers" :list="list" :page-no ="pageNo" :page-size="pageSize" :total-count="totalCount" @delete="deleteItm"  @change-size="handleSizeChange" @change-page="handleCurrentChange">
      </base-table>
    </div>
  </box-content>
</template>
```
有什么东西一目了然。

### 无状态组件
最容易勾起我们封装欲望的就是无状态`HTML`组件，例如我们除去`header`, `menu`之后的`content`部分。没有什么需要复杂的交互，但是我们每个页面又都得写。你说不拿它开刀拿谁开🔪
```html
<template>
  <div class="container-fluid" :class="[contentClass]">
      <el-row>
          <el-col :span="24">
              <!-- box with #fff bg -->
              <div class="box">
                  <div class="box-body">
                      <slot></slot>
                  </div>
              </div>
          </el-col>
      </el-row>
  </div>
</template>
```
上面这个处理非常的简单，但是你在项目中会非常频繁的使用过到，那么这个封装就很有必要了。

### ElementUI table组件封装

`ElementUI`中得组件其实已经封装得很优秀了，但是表格使用得时候还是有一堆得代码在我看来是不需要在业务中重复写得。封装到靠配置来进行表格得书写得一步我觉得就差不多了，下面是一个小`demo`
```html
<template>
  <el-row>
    <el-col :span="24">
      <el-table :data="list" border size="mini" @selection-change="handleSelectionChange" :max-height="tableHeight" v-bind="$attrs"> <!--   -->
        <template v-for="(column, index) in columns">
          <slot name="front-slot"> </slot>
          <!-- 序号 -->
          <el-table-column :key="index" v-if="column.type === 'selection'" type="selection" width="55"> </el-table-column>
          <!-- 复选框 -->
          <el-table-column :key="index" v-else-if="column.type === 'index'"  type="index" width="50" label="序号"> </el-table-column>
          <!-- 具体内容 -->
          <el-table-column :key="index" v-else align="left" :label="column.title" :width="column.width">
            <template slot-scope="scope">
              <!-- 仅仅显示文字 -->
              <label v-if="!column.hidden"> <!-- 如果hidden为true的时候 那么当前格可以不显示，可以选择显示自定义的slot-->
                <!-- 操作按钮 -->
                <label v-if="column.type === 'operate'">
                  <a href="javascript:void(0)" class="operate-button" v-for="(operate, index) in column.operates" :key="index" @click="handleClick(operate, scope.row)">
                    {{operate.name}}
                    &nbsp;&nbsp;
                  </a>
                </label>
                <span v-else>
                  {{scope.row[column.key]}}
                </span>
              </label>
              <!-- 使用slot的情况下 -->
              <label v-if="column.slot">
                <!-- 具名slot -->
                <slot v-if="column.slot" :name="column.slot" :scope="scope"></slot>
              </label>
            </template>
          </el-table-column>
        </template>
        <!--默认的slot -->
        <slot/>
      </el-table>
    </el-col>
  </el-row>
</template>
```
```js
export default {
  name: 'base-table',
  props: {
    // 核心数据
    list: {
      type: Array,
      default: () => []
    },
    // columns
    columns: {
      type: Array,
      required: true,
      default: () => []
    }
  },
  data () {
    return {
      tableHeight: xxx
    }
  },
  methods: {
    // 处理点击事件
    handleClick(action, data) {
      // emit事件
      this.$emit(`${action.emitKey}`, data)
    }
  }
}
```
使用：
```html
<base-table v-loading="loading" :columns="headers" :list="list"  @view="viewCb">
  <!-- 自定义的slot -->
  <template slot="demoslot" slot-scope="{scope}">
    <span>
      {{scope.row}}
    </span>
  </template>
  <!-- 默认的slot  如果交互很复杂 我们还可以直接使用表格内部的组件 -->
  <el-table-column
    label="操作"
    width="200"
  >
    <template slot-scope="scope">
      <a href="javascript:void(0)" @click="defaultSlot(scope.row)">xxx</a>
    </template>
  </el-table-column>
</base-table>
```
```js
export default {
  name: 'table-demo',
  data () {
    return {
      // 表格头部配置
      headers: [
        { key: 'xxx', title: '测试' },
        { title: 'xxx', hidden: true, slot: 'demoslot'},
        {
          title: '操作', type: 'operate',
          operates: [
            {name: '详情',emitKey: 'view'}
          ]
        }
      ]
    }
  },
  methods: {
    viewCb(){
      // todo
    },
    defaultSlot(){
      // todo
    }
  } 
}
```
这样封装过的表格，应付基本的一些需求问题应该不大。至于特殊的要求可以一步一步的进行完善。

# 总结

这些东西并不是什么语法糖，是真正可以在项目中加快我们的效率。让我们的自己乃至整个团队从繁杂的重复复制粘贴中解脱一点。至于速度和质量的问题。我是觉得使用公共组件质量可控性会更高一些。我建议公共得东西注释一定要写得全面和详细，这样可以极大的降低我们的交流成本。至于组件的封装还是要看你的业务。

没想到这么多人看😂,加班加点整理出来了[示例代码](https://github.com/QDMarkMan/vue-base-template),比较简陋😂😂

[原文地址](https://github.com/QDMarkMan/CodeBlog/blob/master/Vue/%E5%8A%A0%E5%BF%ABVue%E9%A1%B9%E7%9B%AE%E7%9A%84%E5%BC%80%E5%8F%91%E9%80%9F%E5%BA%A6.md)  如果觉得有用得话给个⭐吧