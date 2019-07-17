# NodeJS发光发热之打包hooks

[让NodeJS在项目中发光发热](https://github.com/QDMarkMan/CodeBlog/blob/master/Node/%E8%AE%A9NodeJS%E5%9C%A8%E9%A1%B9%E7%9B%AE%E4%B8%AD%E5%8F%91%E5%85%89%E5%8F%91%E7%83%AD.md)系列文章

## 写在前面

最近离职了，闲着也是闲着。就想起来了之前`Node`相关的文章还有一部分没写，刚好有时间，今天给续上。现如今的前端开发，通过`Node`可以高度自定义的为我们的项目打造**一条龙服务**。既然一条龙那么不仅仅是开发阶段，打包之后的事情，我们也要处理。这篇文章就聊一聊打包之后的一些用得上的`hooks`。同样的这篇文章也是抛砖引玉的作用，文章里涉及的都是一些简单的部分，深层次的骚操作还要各位朋友自己去挖掘。

首先来看一下大致的效果

![](./images/build-hooks.png '打包hooks')


## 准备工作

准备一个项目，`Vue/React/Angular`的都可以，我们这里以`vue-base-template`为项目模板


## 改造打包命令

因为我使用的是`Vue-cli3.0`版本的脚手架，它打包的命令是通过`vue-cli-service build`命令来执行。`npm`直接运行的话我们就没法加`hooks`了， 所有我们自己写一个`Node`脚本来执行打包命令, 在`scripts`文件夹下新建`build.js`，这个脚本用来组织我们所有的打包相关东西。

`build.js`的职责其实很简单： 

1. 组织打包参数
2. 运行打包命令
3. 检测打包完成之后运行自定义`hooks`

有了需求再去写功能，对大家来说都是小case，下面就来实现这三个需求。

1. 打包参数

这个参数就是我们在命令行中输入的参数，也就是`package.json scripts`中写好的参数，例如
```bash
"scripts": {
  "build": "node scripts/build.js --mode production"
}
```
这里我们暂时不对参数做特殊的处理，仅仅取出来给打包命令用，如果你需要复杂的参数建议你使用[commander.js](https://github.com/tj/commander.js)，我这个项目现在还没涉及到复杂的打包参数。所以直接使用`process.argv`取出来就行了，关于`process`具体使用感兴趣可以自己看看[文档](http://nodejs.cn/api/process.html#process_process_argv)
```js
const scriptArgv = process.argv.slice(2) // 删除node scripts/build.js
const args = scriptArgv.join(' ') // 组装成正常格式
```

2. 运行命令

这个对`Node`来说，这个很简单。通过`child_process`就能完美实现，我比较懒，直接用了[tasksfile](https://www.npmjs.com/package/taskfile)这个库。你也可以自己采用原生`Node`写，不过要注意异步的问题。

```js
const { sh } = require('tasksfile')
// 同步执行打包命令
sh(`vue-cli-service build ${args}`, {
  silent: false
})
```

3. 打包完成执行`Hooks`，这就是执行个方法。

下面是`build.js`中全部的代码。

```js
const ora = require('ora')
const { sh } = require('tasksfile')
const { Notify } = require('./util') 
const builtHooks = require('./build-hooks')
const scriptArgv = process.argv.slice(2)
const args = scriptArgv.join(' ')

const spinner = ora(`building for ${process.env.NODE_ENV}...\n`)
spinner.start()
// real pack command
sh(`vue-cli-service build ${args}`, {
  silent: false
})
// build success
spinner.succeed("打包完成")
// notify
Notify.showNotify("打包完成", "即将进行下一步操作")
// delay 2s
setTimeout(() =>{
  // run hooks
  builtHooks()
},2000)
```

## 自定义hooks

这个发挥的空间有点大，大部分都是为了提升我们的效率的，我就写几个我自己认为常用一点的。

1. 发布到服务器
2. 本地预览
3. 生成Zip文件
4. 备份Zip文件到本地

在执行完`build.js`的前两步之后，接着就会运行`hooks`。因为`hooks`脚本存在的目的是为开发者最大可能性的节约时间。所以就设计为了非强制性的选项。

首先新建文件`build-hooks.js`

第一步设计`hooks`选项，同样的使用我们的老朋友`inquirer`
```js
const builtHooks = () => {
  inquirer.prompt([
    {
      type: 'list',
      message: `检测到production环境打包完成，请选择下一步操作`,
      name: 'next',
      choices: [
        {
          name: '退出脚本',
          value: 0
        },
        {
          name: '发布到服务器',
          value: 1
        },
        {
          name: '本地预览',
          value: 2
        },
        {
          name: '生成Zip文件',
          value: 3
        },
        {
          name: '备份Zip文件到本地',
          value: 4
        }
      ]
    }
  ]).then(answers => {
    afterHooks.get(answers.next)()
  })
}
```

第二步设计不同选项对应的行为
```js
const afterHooks = new Map([
  [0, () => {
    Log.logger('退出程序')
    process.exit(0)
  }],
  [1, () => {
    Log.logger('即将进行发布🎈')
    require('./deploy')
  }],
  [2, () => {
    Log.logger('开始本地预览💻')
    require('./server')
  }],
  [3, async () => {
    Log.logger('开始压缩zip文件👜')
    await FileUtil.zipDir()
  }],
  [4, async () => {
    Log.logger('开始备份Zip文件到本地📦')
    await Backup.doBackup()
  }]
])
```

是不是非常简单，简洁易懂哈哈哈。 下面我们来看一下具体的操作实现细节

### 发布到服务器🎈

这个在这里就不说了， 可以直接看[这里](https://github.com/QDMarkMan/CodeBlog/blob/master/Node/%E8%AE%A9NodeJS%E5%9C%A8%E9%A1%B9%E7%9B%AE%E4%B8%AD%E5%8F%91%E5%85%89%E5%8F%91%E7%83%AD.md)。

### 本地预览💻

简单问题了，无非是在本地搭建一个轻量级的`Web`服务器， 用第三方库实现的话就更简单了。代码如下`server.js`

```js
const http = require('http')
const fs = require('fs')
const path = require('path')
const { Log } = require('./util')
const httpPort = 8088
const filePath = path.resolve(__dirname, '../dist/index.html')
const open = require('open')
// create current http server
http.createServer((req, res) => {
  Log.logger(req.url)
  try {
    const content = fs.readFileSync(filePath, 'utf-8')
    // deal resource
    if (req.url.indexOf('static') !== -1 || req.url.indexOf('vendor') !== -1 || req.url == "/favicon.ico") {
      const data = fs.readFileSync(path.resolve(__dirname, `../dist${req.url}`))
      return res.end(data)
    }
    // index.html
    res.setHeader('Content-Type','text/html;charset=utf-8')
    res.writeHead(200, {
      'Content-Type': 'text/html; charset=utf-8'
    })
  res.end(content)
  } catch (error) {
    Log.error('We cannot open "index.htm" file.')
  }
}).listen(httpPort, async () => {
  const location = `http://localhost:${httpPort}`
  Log.success(`Server listening on: ${location}, Open in the browser after 3 seconds`)
  // open default browser
  setTimeout(async ()=> {
    // 自动打开默认浏览器
    await open(location)
  }, 3000)
})
```

效果如下：

![](./images/preview.png '')


### 压缩zip文件👜

这个也没什么好说的😂， 就是通过`Node`压缩`dist`文件夹，在这里我用了[zip-local]()来实现需求（你也可以使用`Node`实现， 但是我比较懒哈哈哈）代码如下

```js
/**
   * 压缩文件夹
   * @param {*} dir 要压缩的文件夹 默认 ROOTPATH.distDir
   * @param {*} zipedPath 压缩之后 的zip存放路径
   */
  static async zipDir (dir = ROOTPATH.distDir, zipedPath = ROOTPATH.distZipPath) {
    try {
      if(fs.existsSync(zipedPath)) {
        Log.logger('zip已经存在, 即将删除压缩包')
        fs.unlinkSync(zipedPath)
      } else {
        Log.logger('即将开始压缩zip文件')
      }
      await zipper.sync.zip(dir).compress().save(zipedPath);
      Log.success('文件夹压缩成功')
    } catch (error) {
      Log.error(error)
      Log.error('压缩dist文件夹失败')
    }
  }
```

### 备份Zip文件到本地📦

压缩+备份（就是复制一份文件到指定文件夹）

我在深思熟虑之后还是决定把备份给加上，我反正是吃过没备份的亏😂， 这个选项会为我们在`backups`文件目录下生成一个新的以当前日期命名的压缩文件。

熟悉`Node`的文件系统的话这个对大家来说就很简单了。`backup.js`代码如下:

```js
const path = require('path')
const { FileUtil, StringUtil, DateUtil, ROOTPATH, Log, Notify } = require('./util')

class Backup {
  /**
   * @author: etongfu
   * @description: 清空所有备份
   */
  static clearBackups () {

  }
  /**
   * @author: etongfu
   * @description: 执行备份
   * @param {type}  {*}
   * @returns:  {*}
   */
  static async doBackup () {
    try {
      // 文件名是当前日期 精确到秒
      let date = StringUtil.trim(DateUtil.getCurrentDate("YYYY-MM-DD hh:mm:ss"), 1)
      date = StringUtil.replaceAll(date,"-", "")
      date = StringUtil.replaceAll(date,":", "")
      let targetPath = path.resolve(__dirname, `../backups/${date}.backup.zip`)
      if(FileUtil.fileExist(targetPath)) {
        return Log.warning(`${targetPath}已存在，已放弃备份`)
      }
      // Zip File
      await FileUtil.zipDir(ROOTPATH.distDir, targetPath)
      Log.success(`本地备份完成, 文件：${targetPath}`)
      Notify.showNotify("本地备份", `本次备份完成, 文件地址：${targetPath}`)
    } catch (error) {
      Log.error(`备份文件失败：${error}`)
    }
  }
}
module.exports = Backup
```

效果如下：

![](./images/backup.png '')

# 总结

其实hooks完全不必和打包耦合在一起，完全可以拆出来使用，不过这个是下一个版本的需求哈哈哈😀。完这些再亲身使用了一段时间之后，真的感觉可以节省下来不少时间。免去了一些繁琐的手工操作。推荐大家尝试一下。


[示例代码](https://github.com/QDMarkMan/vue-base-template)

[原文地址](https://github.com/QDMarkMan/CodeBlog/blob/master/Node/NodeJS%E5%8F%91%E5%85%89%E5%8F%91%E7%83%AD%E4%B9%8B%E6%89%93%E5%8C%85hooks.md)  如果觉得有用得话给个⭐吧
