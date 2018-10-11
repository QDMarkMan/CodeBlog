# Vue 知识集锦

> 关于项目中使用 Vue(2.5.0+)遇到的问题的集锦，写到这上面的都是我认为在实际开发中碰到得问题和需要强化记忆得部分。主要是分 Vue,Vuex,VueRouter 三个大的模块，以及 vue-cli，等等小的模块

## Vue

### **keep-alive**(借鉴大佬博客)

  有的项目大部分组件是没必要多次渲染的，所以 Vue 提供了一个内置组件 keep-alive 来缓存组件内部状态，避免重新渲染，[文档](https://cn.vuejs.org/v2/api/#keep-alive)。

  > 文档：和 `<transition>`相似，`<keep-alive>` 是一个抽象组件：它自身不会渲染一个 DOM 元素，也不会出现在父组件链中。

  **用法**

  1.  缓存动态组件

  ```html
  <!-- <keep-alive>包裹动态组件时，会缓存不活动的组件实例，而不是销毁它们，此种方式并无太大的实用意义。 -->
  <!-- 基本 -->
  <keep-alive>
    <component :is="view"></component>
  </keep-alive>

  <!-- 多个条件判断的子组件 -->
  <keep-alive>
    <comp-a v-if="a > 1"></comp-a>
    <comp-b v-else></comp-b>
  </keep-alive>
  ```

  2.  缓存路由组件


  ```html
  <!-- 配合vue-router 这个用的稍微多点 -->
    <keep-alive>
      <router-view></router-view>
    </keep-alive>
  <!-- 使用keep-alive可以将所有路径匹配到的路由组件都缓存起来，包括路由组件里面的组件，keep-alive大多数使用场景就是这种。 -->
  ```

  **生命周期钩子**

  在被 keep-alive 包含的组件/路由中，会多出两个生命周期的钩子:activated 与 deactivated。

  - **activated 在组件第一次渲染时会被调用，之后在每次缓存组件被激活时调用。**

    activated 调用时机：

    第一次进入缓存路由/组件，**在 mounted 后面**，beforeRouteEnter 守卫传给 next 的回调函数之前调用：

    ```
      beforeMount=> mounted=> activated 进入缓存组件 => 执行 beforeRouteEnter回调
    ```

    因为组件被缓存了，再次进入缓存路由/组件时，不会触发这些钩子：

    - beforeCreate
    - created
    - beforeMount
    - mounted

    不是第一次进入的调用时机：

    ```js
      组件销毁destroyed/或离开缓存deactivated => activated 进入当前缓存组件 => 执行 beforeRouteEnter回调
      // 组件缓存或销毁，嵌套组件的销毁和缓存也在这里触发
    ```

  - **deactivated：组件被停用(离开路由)时调用**

    这个在我理解就是和离开路由的钩子差不多，但是要再那个之后触发。有一点要注意，已经缓存的路由在离开时不用调用一下的钩子：

    - beforeDestroy
    - destroyed

    ```js
    // 以下生命周期都是相对于要离开的组件
    beforeRouteLeave => beforeEach => afterEach => deactivated;
    ```

  3.  缓存特定的路由

      **Props**

      - include - 字符串或正则表达式或数组。只有匹配的组件会被缓存。
      - exclude - 字符串或正则表达式或数组。任何匹配的组件都不会被缓存。就没有activated 与 deactivated钩子了

      
      **匹配规则**  
      1. **匿名组件无法匹配**
      2. 首先匹配组件中name属性
      3. name不可用的话则匹配它的局部注册名称。
      4. 不能匹配嵌套的子组件，只能匹配当前组件
      6. > 文档：<keep-alive>不会在函数式组件中正常工作，因为它们没有缓存实例。
      7. exclude的优先级大于include,也就是说：当include和exclude同时存在时，exclude生效，include不生效。

      **使用方式**

      ```html
        <!-- 逗号分隔字符串 -->
        <keep-alive include="a,b">
          <component :is="view"></component>
        </keep-alive>

        <!-- 正则表达式 (使用 `v-bind`) -->
        <keep-alive :include="/a|b/">
          <component :is="view"></component>
        </keep-alive>

        <!-- 数组 (使用 `v-bind`) -->
        <keep-alive :include="['a', 'b']">
          <component :is="view"></component>
        </keep-alive>

        <!-- 路由中使用 -->
        <keep-alive include='a'>
            <router-view></router-view>
        </keep-alive>
      ```
### **关于组件的生命周期**

一图胜千言

![生命周期](./vue-life.jpg '生命周期')

根据生命周期可以得出进入一个keep-alive组件之后钩子正确的触发顺序
1. ```beforeRouteLeave```: 路由组件的组件离开路由前钩子，可取消路由离开。
2. ```beforeEach```:  路由全局前置守卫，可用于登录验证、全局路由loading等。
3. ```beforeEnter```:  路由独享守卫
4. ```beforeRouteEnter```:  路由组件的组件进入路由前钩子。
5. ```beforeResolve```: 路由全局解析守卫
6. ```afterEach```: 路由全局后置钩子
7. ```beforeCreate```: 组件生命周期，不能访问this。
8. ```created```: 组件生命周期，可以访问this，不能访问dom。
9. ```beforeMount```: 组件生命周期
10. ```deactivated```: 离开缓存组件a，或者触发a的beforeDestroy和destroyed组件销毁钩子。
11. ```mounted```:访问/操作dom。
12. ```activated```:进入缓存组件，进入a的嵌套子组件(如果有的话)。
13. 执行```beforeRouteEnter```回调函数next。

---

### **关于组件通信中的.sync修饰符**

这个修饰符是2.3.0 新增的,主要用于组件间的双向通信
> 在有些情况下，我们可能需要对一个 prop 进行“双向绑定”。不幸的是，真正的双向绑定会带来维护上的问题，因为子组件可以修改父组件，且在父组件和子组件都没有明显的改动来源。

因此Vue推荐使用```update:myPropName``` 的模式触发事件取而代之。在我理解这样就和现有的emit系统保持了抛出数据的一致性。而且同时保证了数据的单向流通。这也符合Vue一贯的设计风格 

**值 ==> (接收数据)** **事件(emit) => (传播数据)** 

示例代码如下：
```js
// 子组件手动emit事件 => 这就是一个传播值得过程
this.$emit('update:title', newTitle)
```
``` html
<!-- 父组件接收 -->
<!-- 1 -->
<text-document v-bind:title="doc.title" v-on:update:title="doc.title = $event"></text-document>
<!-- 2简写 -->
<text-document :title.sync="doc.title"></text-document>
```
注意：<span color="red">
  将 v-bind.sync 用在一个字面量的对象上，例如 v-bind.sync=”{ title: doc.title }”，是无法正常工作的，因为在解析一个像这样的复杂表达式的时候，有很多边缘情况需要考虑。
</span>

这样得设计很大得提高了组件得可维护性和扩展性。使得组件之间得通讯非常得清除，降低了很大得维护成本

### **Vue复用大法Mixin(混入)**
> 混入 (mixins) 是一种分发 Vue 组件中可复用功能的非常灵活的方式。混入对象可以包含任意组件选项。当组件使用混入对象时，所有混入对象的选项将被混入该组件本身的选项。

从定义中不难看出来，混入就是把一个vue对象混入到另一个vue实例中。在被混入的组件和混入中共享this，最终合并为一个Vue组件。这样就大大的提高了模块间的可复用性。通用的data以及methods我们只需要在mixins中声明，在组件中直接就可以调用了。

#### 基础
- 定义一个混入对象就像定义一个Vue对象一样，声明一个对象包括了data，methods，声明周期等等
```js
// 定义一个混入对象
var myMixin = {
  created: function () {
    this.hello()
  },
  methods: {
    hello: function () {
      console.log('hello from mixin!')
    }
  }
}

// 定义一个使用混入对象的组件
var Component = Vue.extend({
  mixins: [myMixin]
})

var component = new Component() // => "hello from mixin!"
```
#### 需要注意的部分
1. **声明顺序**

混入对象的声明周期会并入组件的周期，而且要早于组件的钩子运行。
```js
var mixin = {
  created: function () {
    console.log('混入对象的钩子被调用')
  }
}

new Vue({
  mixins: [mixin],
  created: function () {
    console.log('组件钩子被调用')
  }
})

// => "混入对象的钩子被调用"
// => "组件钩子被调用"
```

2. **选项的合并**

当组件的数据对象和混入对象的数据对象重复的时候，组件的数据对象优先。
值为对象的选项，例如 ```methods, components ```和 ```directives，```将被混合为同一个对象。<font color="yellow">两个对象键名冲突时，取组件对象的键值对。</font>


3. **局部混入和全局混入**

  1. **局部混入（推荐使用）**
  
  大部分的情况下我们是使用局部混入的，在组件的```mixins```数组中声明就是传统的局部混入。
  ```js
  var vm = new Vue({
    mixins: [mixin], // 局部混入
    methods: {
      bar: function () {
        console.log('bar')
      },
      conflicting: function () {
        console.log('from self')
      }
    }
  })
  ```
  2. **全局混入（谨慎使用）**

  Vue也为我们提供了全局混入，但是<font color="red">一旦使用了全局的混入，就会影响到之后创建的所有的Vue实例</font>。下面是创建方式
  ```js
  // 为自定义的选项 'myOption' 注入一个处理器。
  Vue.mixin({
    created: function () {
      var myOption = this.$options.myOption
      if (myOption) {
        console.log(myOption)
      }
    }
  })
  new Vue({
    myOption: 'hello!'
  })
  // => "hello!"
  ```
  

## Vuex

---

## VueRouter
- 监听路由变化
  
  1. **使用watch监听**
  ```js
  // 监听,当路由发生变化的时候执行
  // 1
  watch:{
    $route(to,from){
      console.log(to.path);
    }
  },
  // 2
  watch: {
    $route: {
      handler: function(val, oldVal){
        console.log(val);
      },
      // 深度观察监听
      deep: true
    }
  },

  // 3
  watch: {
    '$route':'getPath'
  },
  methods: {
    getPath(){
      console.log(this.$route.path);
    }
  }
  ```
  2. **用key来阻止“复用”(不太能算是监听变化)**
  > Vue 为你提供了一种方式来声明“这两个元素是完全独立的——不要复用它们”。只需添加一个具有唯一值的 key 属性即可

  ``` html
  <router-view :key="key"></router-view>
  ```
  ``` js
  computed: {
    key() {
      return this.$route.name !== undefined? this.$route.name +new Date(): this.$route +new Date()
    }
  }
  ```
  3. **通过路由独享钩子**
  ``` js
  beforeRouteEnter (to, from, next) {
    // 在渲染该组件的对应路由被 confirm 前调用
    // 不！能！获取组件实例 `this`
    // 因为当钩子执行前，组件实例还没被创建
  },
  beforeRouteUpdate (to, from, next) {
    // 在当前路由改变，但是该组件被复用时调用
    // 举例来说，对于一个带有动态参数的路径 /foo/:id，在 /foo/1 和 /foo/2 之间跳转的时候，
    // 由于会渲染同样的 Foo 组件，因此组件实例会被复用。而这个钩子就会在这个情况下被调用。
    // 可以访问组件实例 `this`
  },
  beforeRouteLeave (to, from, next) {
    // 导航离开该组件的对应路由时调用
    // 可以访问组件实例 `this`
  },
  ```