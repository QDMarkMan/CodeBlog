# Electron 实例检测
> 这个功能学名叫什么我也不太清楚，暂且就叫她实例检测吧。有时候我们想点击了快捷方式之后只打开一个应用程序，但是显示总是残酷的，electron默认的是完成打包之后每次点击快捷方式都会生成一个进程实例，这个时候稳住不要慌，别人实现了我们肯定也能实现而且还很简单。

- 首先 我们需要了解Electron的基本知识，<small> [点击看基础](https://github.com/QDMarkMan/CodeBlog/tree/master/Electron) </small>。这里就不赘述了，然后我们就需要知道我们用Elenton中提供的API了。

- 接下来我们的主角 <font face="微软雅黑" color="red">app.makeSingleInstance(callback)</font> 就粉墨登场了
    > 此方法使应用程序成为单个实例应用程序, 而不是允许应用程序的多个实例运行, 这将确保只有一个应用程序的实例正在运行, 其余的实例全部会被终止并退出。          当执行第二个实例时, 第一个实例将使用 callback (argv，workingDirectory) 调用 callback。 argv 是第二个实例的命令行参数的数组, workingDirectory 是这个实例当前工作目录。 通常, 应用程序会激活窗口并且取消最小化来响应。
    ``` js
      /*
        callback(): Function 回掉函数
      */
      app.makeSingleInstance(callback)
        argv String[] // 第二个实例的命令行参数数组
        workingDirectory String // 第二个实例的工作目录
       // 返回 Boolean.
    ```
    
    上面是官方文档给出的解释，这个API就是为了生成单个实例而存在的。实际上更主要的用处是用来处理在命令行打开应用是的限制，其实在mac上通过图标打开尝试启动第二个实例的时候，系统会自动强制执行单个实例。并且发出 open-file 和 open-url 事件。但是当用户在命令行中启动应用程序时, 系统的单实例机制将被绕过, 所以我们要用这个方法来保证单实例。

- 本质来讲就是在创建electron实例窗口之前，去检查是否已经存在一个实例，如果有那么就在启动时激活主实例。废话少说上代码
``` js
const {app} = require('electron') // 引入主线程
  let mianInstance = null
  /**
  * 实例检测
  */
  const moreInstance = app.makeSingleInstance((commandline, workingDirectory) => {
    if(mainWindow) { // 如果存在执行以下
      // 判断主实例窗口是否最小化 如果是的话 恢复到之前的状态
      if (mainWindow.isMaximized()) mainWindow.restore()
      mainWindow.focus() // 主实例窗口focus
    }
  })
  // 判断是否存在主实例
  if (moreInstance) {
    // 离开当前的进程
    app.quit()
  }
  // 主进程准备完毕
  app.on('ready', () => {
    // 创建窗口的方法
  })
```
 # 总结
其实这个方面还是很容易的，我们也可以发散一下思维，如果仅仅是检测有没有实例 是不是通过Node环境变量也也就而已实现呢？ 这个是写文章的时候想到的，有空就试一试咯

