# Vue源码阅读总结(未完成)
 最近有时间，抽空看了一看`VUE`源码。下面是一些总结

 ## 技巧的总结

- `Vue`是有默认的全局配置的。
  ```js
  Vue.options = {
    // 内置组件
    components: {
      KeepAlive,
      Transition,
      TransitionGroup
    },
    directives: Object.create(null),
    // 语法糖指令
    directives:{
      model,
      show
    },
    filters: Object.create(null),
    _base: Vue
  }
  ```
- `Vue`中实现一个只读对象的方式
- `Vue`中判断一个纯对象的方法

- `Vue`的生命周期`hooks`还可以传递数组。
  
  在`VUE`的合并`hooks`的代码是这样的
  ```js
  // 合并生命周期
  function mergeHook (
    parentVal: ?Array<Function>, // 父组件中的hooks
    childVal: ?Function | ?Array<Function> // 子组件的hooks
  ): ?Array<Function> {
    return childVal
      ? parentVal
        // 这里，合并且生成一个新数组
        ? parentVal.concat(childVal)
        : Array.isArray(childVal) // 这里判断是不是数据
          ? childVal
          : [childVal]
      : parentVal
  }
  ```
  从上面我们可以看出来`mergeHook`一定返回的是个数组。那么说明生命周期我们也可以传递数组进去。那么下面这样的写法就是没问题的。
  ```js
  // 钩子将按顺序执行
  new Vue({
    created: [
      function () {
        console.log('first')
      },
      function () {
        console.log('second')
      },
      function () {
        console.log('third')
      }
    ]
  })
  ```
- `VUE`组件生命周期的另外一种写法
  
  在`Vue`生命周期的源码部分有这么一句话
  ```js
  if (vm._hasHookEvent) {
    vm.$emit('hook:' + hook)
  }
  ```
  从上面这句话我们不难猜到我们可以使用事件监听方式来写生命周期监听
  ```html
  <child
    @hook:beforeCreate="handleChildBeforeCreate"
    @hook:created="handleChildCreated"
    @hook:mounted="handleChildMounted"
    @hook:生命周期钩子
  />
  ```
  不过如果你不是对`Vue`非常的熟悉的话，还是建议不要这样写。


- `VUE`中的合并策略用法
  
  `VUE`官网中提到可以使用`Vue.config.optionMergeStrategies`进行全局配置。通过这个我们可以给指定的对象提供合并策略
  ```js
  Vue.config.optionMergeStrategies.customOption = function (parentVal, childVal) {
      return parentVal ? (parentVal + childVal) : childVal
  }
  ```
  创建一个子类
  ```js
  const Sub = Vue.extend({
      customOption: 1
  })
  ```
  以子类创建实例
  ```js
  const v = new Sub({
      customOption: 2,
      created () {
          console.log(this.$options.customOption) // 这个时候根据我们呢自己策略 那么返回的就是3
      }
  })
  ```


