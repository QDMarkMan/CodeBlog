# Vue 知识集锦）

> 关于项目中使用 Vue(2.5.0+)遇到的问题的集锦，写到这上面的都是我认为的偏冷门点的知识点。主要是分 Vue,Vuex,VueRouter 三个大的模块，以及 vue-cli，等等小的模块

## Vue

- **keep-alive**(借鉴大佬博客)

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
- **关于组件的生命周期**

一图胜千言

![electron](./vue-life.jpg '生命周期')

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


## Vuex
