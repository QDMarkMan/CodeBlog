# Electron-builder打包详解

开发electron客户端程序，打包是绕不开的问题。下面就我在工作中的经验以及目前对```electron-builder```的了解来分享一些心得。

## 基本概念
[官网](https://www.electron.build/)的定义
> A complete solution to package and build a ready for distribution Electron app for macOS, Windows and Linux with “auto update” support out of the box.

关于```electron```和```electron-builder```的基础部分这篇文章就跳过了，有兴趣的话可以看[这篇文章](https://github.com/QDMarkMan/CodeBlog/tree/master/Electron)

## 如何使用

builder的使用和配置都是很简单的
builder配置有两种方式
- ```package.json```中直接配置使用（比较常用，我们下面着重来讲这个）
- 指定```electron-builder.yml```文件

demo地址会在文章末尾给出（demo项目中`electron`使用得是`V2.0.7`版本,目前更高得是`2.0.8`版本）。

下面是一个简单的```package.js```中带注释的配置
1. 基础配置
``` js
"build": {  // 这里是electron-builder的配置
    "productName":"xxxx",//项目名 这也是生成的exe文件的前缀名
    "appId": "com.xxx.xxxxx",//包名  
    "copyright":"xxxx",//版权  信息
    "directories": { // 输出文件夹
      "output": "build"
    }, 
    // windows相关的配置
    "win": {  
      "icon": "xxx/icon.ico"//图标路径 
    }  
  }
```
在配置文件中加入以上的文件之后就可以打包出来简单的<font clolor="red">文件夹</font>，文件夹肯定不是我们想要的东西。下一步我们来继续讲别的配置。
2. 打包目标配置
要打包成**安装程序**的话我们有两种方式，
  1. 使用NSIS工具对我们的文件夹再进行一次打包，打包成exe
  2. 通过electron-builder的nsis直接打包成exe，配置如下
```js
"win": {  // 更改build下选项
    "icon": "build/icons/aims.ico",
    "target": [
      {
        "target": "nsis" // 我们要的目标安装包
      }
    ]
  },
```
3. 其他平台配置
```js
  "dmg": { // macOSdmg
    "contents": [
      ...
    ]
    },
    "mac": {  // mac
      "icon": "build/icons/icon.icns"
    },
    "linux": { // linux
      "icon": "build/icons"
    }
```

4. **nsis配置**

这个要详细的讲一下，这个nsis的配置指的是安装过程的配置，其实还是很重要的，如果不配置nsis那么应用程序就会自动的安装在C盘。没有用户选择的余地，这样肯定是不行的

关于nsis的配置是在build中nsis这个选项中进行配置，下面是部分nsis配置
```js
"nsis": {
  "oneClick": false, // 是否一键安装
  "allowElevation": true, // 允许请求提升。 如果为false，则用户必须使用提升的权限重新启动安装程序。
  "allowToChangeInstallationDirectory": true, // 允许修改安装目录
  "installerIcon": "./build/icons/aaa.ico",// 安装图标
  "uninstallerIcon": "./build/icons/bbb.ico",//卸载图标
  "installerHeaderIcon": "./build/icons/aaa.ico", // 安装时头部图标
  "createDesktopShortcut": true, // 创建桌面图标
  "createStartMenuShortcut": true,// 创建开始菜单图标
  "shortcutName": "xxxx", // 图标名称
  "include": "build/script/installer.nsh", // 包含的自定义nsis脚本 这个对于构建需求严格得安装过程相当有用。
},
```
关于`include` 和 `script` 到底选择哪一个 ？

在对个性化安装过程需求并不复杂，只是需要修改一下安装位置，卸载提示等等的简单操作建议使用`include`配置,如果你需要炫酷的安装过程，建议使用`script`进行完全自定义。

`NSIS`对于处理安装包这种东西，功能非常的强大。但是学习起来并不比一门高级语言要容易。其中的奥秘还要各位大佬自行探索

这里上一些学习资源
- [NSIS初级篇](https://blog.csdn.net/ddjj_1980/article/details/7843944)
- [NSIS 打包脚本基础](http://www.cnblogs.com/jingmoxukong/p/5033622.html)
- [示例脚本](http://www.cppblog.com/momoxiao/archive/2010/02/06/107326.html)
- [NSIS论坛](http://www.nsisfans.com/forum-2-1.html)

5. 关于操作系统的配置

主要是windows中64和32位的配置

CLI参数
```bash
electron-builder --ia32 // 32位
electron-builder        // 64位(默认)
```
nsis中配置
```js
"win": {
  "icon": "build/icons/aims.ico",
  "target": [
    {
      "target": "nsis",
      "arch": [ // 这个意思是打出来32 bit + 64 bit的包，但是要注意：这样打包出来的安装包体积比较大，所以建议直接打32的安装包。
        "x64", 
        "ia32"
      ]
    }
  ]
}
```
6. 更新配置

下面这个是给更新用的配置，主要是为了生成```lastest.yaml```配置文件
```js
"publish": [
  {
    "provider": "generic", // 服务器提供商 也可以是GitHub等等
    "url": "http://xxxxx/" // 服务器地址
  }
],
```
## 完整配置
基本上可用的完整的配置
```js
"build": {
    "productName":"xxxx",//项目名 这也是生成的exe文件的前缀名
    "appId": "com.leon.xxxxx",//包名  
    "copyright":"xxxx",//版权  信息
    "directories": { // 输出文件夹
      "output": "build"
    }, 
    "nsis": {
      "oneClick": false, // 是否一键安装
      "allowElevation": true, // 允许请求提升。 如果为false，则用户必须使用提升的权限重新启动安装程序。
      "allowToChangeInstallationDirectory": true, // 允许修改安装目录
      "installerIcon": "./build/icons/aaa.ico",// 安装图标
      "uninstallerIcon": "./build/icons/bbb.ico",//卸载图标
      "installerHeaderIcon": "./build/icons/aaa.ico", // 安装时头部图标
      "createDesktopShortcut": true, // 创建桌面图标
      "createStartMenuShortcut": true,// 创建开始菜单图标
      "shortcutName": "xxxx", // 图标名称
      "include": "build/script/installer.nsh", // 包含的自定义nsis脚本
    },
    "publish": [
      {
        "provider": "generic", // 服务器提供商 也可以是GitHub等等
        "url": "http://xxxxx/" // 服务器地址
      }
    ],
    "files": [
      "dist/electron/**/*"
    ],
    "dmg": {
      "contents": [
        {
          "x": 410,
          "y": 150,
          "type": "link",
          "path": "/Applications"
        },
        {
          "x": 130,
          "y": 150,
          "type": "file"
        }
      ]
    },
    "mac": {
      "icon": "build/icons/icon.icns"
    },
    "win": {
      "icon": "build/icons/aims.ico",
      "target": [
        {
          "target": "nsis",
          "arch": [
            "ia32"
          ]
        }
      ]
    },
    "linux": {
      "icon": "build/icons"
    }
  }
```

## 命令行参数（CLI）
Commands(命令):
```bash
  electron-builder build                    构建命名                      [default]
  electron-builder install-app-deps         下载app依赖
  electron-builder node-gyp-rebuild         重建自己的本机代码
  electron-builder create-self-signed-cert  为Windows应用程序创建自签名代码签名证书
  electron-builder start                    使用electronic-webpack在开发模式下运行应用程序(须臾要electron-webpack模块支持)
```
Building(构建参数):
```bash
  --mac, -m, -o, --macos   Build for macOS,                              [array]
  --linux, -l              Build for Linux                               [array]
  --win, -w, --windows     Build for Windows                             [array]
  --x64                    Build for x64 (64位安装包)                     [boolean]
  --ia32                   Build for ia32(32位安装包)                     [boolean]
  --armv7l                 Build for armv7l                              [boolean]
  --arm64                  Build for arm64                               [boolean]
  --dir                    Build unpacked dir. Useful to test.           [boolean]
  --prepackaged, --pd      预打包应用程序的路径（以可分发的格式打包）
  --projectDir, --project  项目目录的路径。 默认为当前工作目录。
  --config, -c             配置文件路径。 默认为`electron-builder.yml`（或`js`，或`js5`)

```
Publishing(发布):
```bash
  --publish, -p  发布到GitHub Releases [choices: "onTag", "onTagOrDraft", "always", "never", undefined]
```

<font color="red">**Deprecated(废弃):**</font>
```bash
  --draft       请改为在GitHub发布选项中设置releaseType                 [boolean]
  --prerelease  请改为在GitHub发布选项中设置releaseType                 [boolean]
  --platform    目标平台 (请更改为选项 --mac, --win or --linux)
           [choices: "mac", "win", "linux", "darwin", "win32", "all", undefined]
  --arch        目标arch (请更改为选项 --x64 or --ia32)
                   [choices: "ia32", "x64", "armv7l", "arm64", "all", undefined]
```
Other(其他):
```bash
  --help     Show help                                                 [boolean]
  --version  Show version number                                       [boolean]
```
Examples(例子):
```bash
  electron-builder -mwl                        为macOS，Windows和Linux构建（同时构建）
  electron-builder --linux deb tar.xz          为Linux构建deb和tar.xz
  electron-builder -c.extraMetadata.foo=bar    将package.js属性`foo`设置为`bar`
  electron-builder --config.nsis.unicode=false 为NSIS配置unicode选项
    
```
TargetConfiguration(构建目标配置):
```js
target:  String - 目标名称，例如snap.
arch “x64” | “ia32” | “armv7l” | “arm64”> | “x64” | “ia32” | “armv7l” | “arm64”  -arch支持列表
```

## 常见的错误
- `NPM`下载的问题
  
  因为`NPM`在国内比较慢。导致`electron-V.xxxx.zip`下载失败。这些东西如果是第一次打包的话是需要下载对应`electron`版本的支持文件。解决办法有两个
  1. 设置镜像：在C盘User中找到`.npmrc`文件。然后加入下面这句代码,但是这个有时候也不是很好用
  ```
  ELECTRON_MIRROR=http://npm.taobao.org/mirrors/electron/
  ```
  2. 直接去淘宝镜像文件库找到对应的文件并下载，放到指定的目录下，electron的淘宝[镜像地址](https://npm.taobao.org/mirrors/electron/)。下载完之后放到指定的文件。一般文件得地址在`C:\Users\Administrator\AppData\Local\electron\Cache`。例如我要下载1.8.4版本的`electron`，那么找到镜像下得文件然后放到指定文件夹中。
  ![electron](./error1.png 'electron')
  ![electron](./error2.png 'electron')

  <small>(如果是在执行`npm install`时下载不下来)直接在https://npm.taobao.org/mirrors/electron下载对应版本的zip 直接扔到C:\Users\YourUserName\.electron就行</small>
  这就解决了这个问题，简单又暴力。

- `NSIS`下载问题

  如果你要打`NSIS`得包还需要西再下载`nsis-resources-xxx`等等包。经过上面得经验这下我们知道缺什么就填什么呗，通过错误日志我们可以得到我们要下载得版本，一般错误中通常会展示`github`下载地址，直接点开连接去下载。但是位置这次不一样了。因为这是`electron-builder`的支持环境所以我们要放在`C:\Users\Administrator\AppData\Local\electron-builder\cache\nsis\`下了。
  ![electron](./nsiserror.png 'electron')

一般情况下解决这些问题的思路就是，缺什么拿什么😄。

# 总结
electron-builder是一个简单又强大的库。反正我是很服

[Demo地址](https://github.com/QDMarkMan/electron-builder-start)

[原文地址](https://github.com/QDMarkMan/CodeBlog/tree/master/Electron/electron-builder打包详解.md)  如果觉得有用得话给个⭐吧