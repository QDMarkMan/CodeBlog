# 服务器发布Vue/Nuxt项目指南(多图)

很多前端朋友可能不是那么了解服务器配置。这篇文章我们一起大致了解一些常见的的Web服务器部署项目的方式。

## 写在前面

下面讲的每一种服务器深入进去都很复杂，在这篇文章**只是讨论一下基本的部署和使用**。更高级的知识和用法还需要各位朋友自行去探索和发现， 开始阅读之前希望大家能先了解一些`Linux`基础，就不至于看起来吃力了。

<font color="red">注</font>: **下面所有的例子大多基于`vue-router`的`history`模式下打包生成的静态文件，其他框架也都大同小异**


## Nginx服务器

`Nginx` 是一个高性能的`HTTP`和反向代理`web`服务器，在连接高并发的情况下，`Nginx`表现相当出色。

### Nginx安装

这里就不讲这个了吧， 有需要的朋友可以看[这个](http://www.nginx.cn/install)

### 配置修改

为了支持`history`模式， 我们要修改`nginx/conf/nginx.conf`文件

```bash
location / {
    root   html;
    try_files $uri $uri/ /index.html; # 只需要加上这么一行
    index  index.html index.htm;
}
```
![nginx](./images/nginx.png 'nginx')

然后把静态资源放在`html`文件夹内

![nginx_html](./images/nginx_html.png 'nginx_html')


然后启动`Nginx`服务器
```bash
cd usr/local/nginx/sbin
./nginx
```
接着访问你的服务器就行OK了。

### GZip支持

`nginx`实现资源压缩的原理是通过`ngx_http_gzip_module`模块拦截请求，并对需要做`gzip`的类型做`gzip`压缩，该模块是默认基础的，不需要重新编译，直接开启即可。大体配置如下
```bash
#开启和关闭gzip模式
gzip on|off;

#gizp压缩起点，文件大于1k才进行压缩
gzip_min_length 1k;

# gzip 压缩级别，1-9，数字越大压缩的越好，也越占用CPU时间
gzip_comp_level 1;

# 进行压缩的文件类型。
gzip_types text/plain application/javascript application/x-javascript text/css application/xml text/javascript ;

#nginx对于静态文件的处理模块，开启后会寻找以.gz结尾的文件，直接返回，不会占用cpu进行压缩，如果找不到则不进行压缩
gzip_static on|off

# 是否在http header中添加Vary: Accept-Encoding，建议开启
gzip_vary on;

# 设置压缩所需要的缓冲区大小，以4k为单位，如果文件为7k则申请2*4k的缓冲区 
gzip_buffers 2 4k;

# 设置gzip压缩针对的HTTP协议版本
gzip_http_version 1.1;
```

`Nginx`配置虽然简单，但是它本身是非常强大的，代理，负载等等都是非常具有实用性的。

## Apache服务器

`Apache`是世界使用排名第一的Web服务器软件, 使用非常广泛。由于VueRouter的`hash`模式本质上和静态资源没什么区别，在`Apache`上发布又比较简单，这里就跳过了发布直接进入配置支持History模式

1. *修改Apache默认配置*

首先要重新修改`\conf\httpd.conf`文件让文件支持`rewrite`

![引入模块](apacheconfig2.jpg '引入模块')

找到
```bash
// 这一行需要解开注释 引入这个模块
LoadModule rewrite_module modules/mod_rewrite.so 
```

然后新增或者修改下面得代码
![修改重写支持](apacheconfig1.jpg '修改重写支持')
```bash
# 重写文件根目录
DocumentRoot "/usr/local/apache/demo"
# 目录
<Directory "/usr/local/apache/demo">
    #
    # Possible values for the Options directive are "None", "All",
    # or any combination of:
    #   Indexes Includes FollowSymLinks SymLinksifOwnerMatch ExecCGI MultiViews
    #
    # Note that "MultiViews" must be named *explicitly* --- "Options All"
    # doesn't give it to you.
    Options Indexes FollowSymLinks

    #
    # 修改未允许重写
    AllowOverride all

    #
    # Controls who can get stuff from this server.
    #
    Require all granted
</Directory>
```

2. *新增htacces规则*

上面操作时修改服务器得默认配置让服务器支持`Rewrite`，下面来创建`Rewrite`规则

首先在和`index.html`同级得地方新建`.htacces`文件，具体内容可以参照`Vue-Router`官网给得[例子](https://router.vuejs.org/zh/guide/essentials/history-mode.html#%E5%90%8E%E7%AB%AF%E9%85%8D%E7%BD%AE%E4%BE%8B%E5%AD%90)

```bash
<IfModule mod_rewrite.c>
  RewriteEngine On
  RewriteBase /
  RewriteRule ^index\.html$ - [L]
  RewriteCond %{REQUEST_FILENAME} !-f
  RewriteCond %{REQUEST_FILENAME} !-d
  RewriteRule . /index.html [L]
</IfModule>
```
![.htacces文件](htac.png '.htacces文件')

### Apache开启Gzip压缩
```bash
/usr/local/apache/bin/apxs -i -c -n -a mod_deflate.so
```

## Tomcat服务器部署
现在这种前后端分离的大环境下，一般不会有太多人用`Tomcat`作`web`服务器。有的企业可能会配合着`SpringMVC`来一起使用，这里也来写一下。配置起来也很简单。

### 配置`server.xml`

首先找到`Tomcat`服务器目录中`tomcat/conf/server.xml`文件，找到`Host`，修改成你想要的配置。主要是`appBase`，它决定了静态文件查询的基础目录。

```xml
<Host name="localhost"  appBase="webapps"
      unpackWARs="true" autoDeploy="true">
  <Valve className="org.apache.catalina.valves.AccessLogValve" directory="logs"
          prefix="localhost_access_log." suffix=".txt"
          pattern="%h %l %u %t &quot;%r&quot; %s %b" />

</Host>
```

![tomcatxml](./images/tomcat_xml.png 'tomcatxml')


### 配置支持History模式

只需要在发布的文件夹中新增`WEB-INF`配置文件夹中就行。如下图

![tomcat中配置](tomcat.png 'tomcat中文件')


`WEB-INF` 文件夹放在项目中那么`tomcat`会自动扫描文件夹中的`web.xml`然后添加配置。
![tomcat中配置](webxml.png 'xml')

```xml
<?xml version="1.0" encoding="ISO-8859-1"?>
<web-app xmlns="http://java.sun.com/xml/ns/javaee"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://java.sun.com/xml/ns/javaee
                      http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd"
  version="3.0"
  metadata-complete="true">
  
  <!-- <display-name>webapp</display-name> -->
  <description>
    webapp
  </description>
  <error-page>  
    <!-- 重写404页面 -->
    <error-code>404</error-code>  
    <location>/index.html</location>
  </error-page>  
</web-app>
```

这就配置完了，可以说是贼简单了。至于怎么安装`Tomcat`服务器，大家自行解决吧😁

## Nuxt项目（服务端渲染）

发布`SSR`项目的时候官方推荐了两种方式**服务端渲染应用部署** 和 **静态应用部署**。静态应用部署的话基本上就失去了`SSR`的优势，而且部署方式也和上面讲的大同小异。这里只讲服务端渲染应用部署。

服务端渲染应用部署的话不同于静态部署，我们同时要在服务器上部署上`Node`环境。

### `Node环境部署`

1. cd到`/usr/local`文件夹并在这个文件夹下新建一个存放安装环境的文件夹`node`后进入`node`文件夹（名字可以随便起，这里做demo就用node了）
```bash
cd /usr/local
mkdir -p node
```
2. 下载`Node`我这里采用的是最新的长期服务版本，你想安装其他的版本，可以在[这里](https://nodejs.org/dist/)查找，还可以使用`nvm`来管理你的`Node`,简单[教程](https://www.jianshu.com/p/d0e0935b150a)
```bash
wget https://nodejs.org/dist/v10.13.0/node-v10.13.0-linux-x64.tar.gz
```
3. 下载完成之后解压并且为文件夹改名
```bash
# 解压到当前文件夹
tar -zxvf node-v10.13.0-linux-x64.tar.gz -C ./
# 改名
mv node-v10.13.0-linux-x64/ ./node10.13.0
```
4. 建立软连接，为`node,npm`注册环境变量(其他的`npm`全局包也要这样注册)
```bash
# 软链接指向到node npm
ln -s /usr/local/node/node10.13.0/bin/node  /usr/local/bin/node
ln -s /usr/local/node/node10.13.0/bin/npm  /usr/local/bin/npm 
```
5. 查看软链接建立是否成功
```bash
ls -al /usr/local/bin
```
显示如下就表示成功了

![node建立链接](./images/nodeinstall1.png 'node建立链接')
接着使用`node -v`
```bash
node -v
# 出现版本号就是安装成功了
v10.13.0
```

到这里为止我们的`Node`环境就安装成功了，接下来进行`nuxt`部署

### `Nuxt`部署
1. 全局安装`nuxt`，然后按照上面的方式建立软连接
```bash
# 安装
npm i nuxt -g
# 建立软连接
ln -s /usr/local/node/node10.13.0/bin/nuxt  /usr/local/bin/nuxt 
```
2. 代码部署

回到`/local`文件夹下，我们建立一个`nuxt`文件用来存放我的`nuxt`项目。然后进入`nuxt`文件夹
```bash
# 回到local
cd ../
mkdir nuxt
cd nuxt
```
在本地环境中执行`nuxt build`,然后会生成一个`.nuxt`文件夹。

然后修改`package.json`,为它加上新的内容，`nuxt`应用会根据下面的配置自动配置服务的端口号和地址。
```bash
"config": {
  "nuxt": {
    "host": "0.0.0.0", # 通过IPV4访问
    "port": xxxx
  }
},
```
然后将项目中的`.nuxt文件夹 static文件夹 package.json nuxt.config.js`上传到服务器的`nuxt`文件夹中，

然后在`nuxt`文件夹中执行
```bash
# 为了方便我这这里服务器上执行了install，正式项目不建议这么干
npm i
```
下载完成之后执行`nuxt start`出现下面的日志就表示成功启动了我们的`nuxt`应用

![成功启动](./images/nuxtstart.png '成功启动')

### `pm2`使用

既然使用了`Node`服务，那我们最好是使用[pm2](https://pm2.io/doc/en/runtime/overview/?utm_source=pm2&utm_medium=website&utm_campaign=rebranding)做进程守护，至于`pm2`是什么，以及`pm2`是干什么的，这篇文章我们不过多的赘述，有兴趣可以自己看看。

首先安装`pm2`
```bash
npm i pm2 -g
# 建立软连接
ln -s /usr/local/node/node10.13.0/bin/pm2  /usr/local/bin/pm2 
```

然后在`package.json`中加入`pm`启动指令或者直接像下面这样启动
```bash
# package.json
"scripts": {
    "pm2:nuxt": "pm2 start npm --name 'XXX' -- run nuxt:start", # 启动名字为xxx的进程
    "nuxt:start": "PORT=xxxx nuxt start",
    "start": "nuxt start",
    "generate": "nuxt generate",
    "pm2:stop:all": "pm2 stop all" # 停止所有进程
  }
# 直接启动命令
pm2 start npm --name 'XXX' -- run nuxt:start
```
出现这样的日志就表示成功

![成功启动](./images/pm2.png '成功启动')

下面是一些常用的`pm2`命令
```bash
pm2 start 0        # 启动 id为 0的指定应用程序
pm2 restart 0      # 重启 id为 0的指定应用程序
pm2 stop 0         # 停止 id为 0的指定应用程序
pm2 delete 0       # 删除 id为 0的指定应用程序

pm2 list           # 查看当前正在运行的进程
pm2 start all      # 启动所有应用
pm2 restart all    # 重启所有应用
pm2 stop all       # 停止所有的应用程序
pm2 delete all     # 关闭并删除所有应用
pm2 logs           # 控制台显示所有日志
 
pm2 logs 0         # 控制台显示编号为0的日志
pm2 show 0         # 查看执行编号为0的进程
pm2 monit xxx      # 监控名称为xxxx的进程
```

至此，一个`Nuxt`应用就算是部署完成了。

### 出现过的问题

1. 通过`koa server.js`启动的的时候外网无法访问。用的是阿里云的服务器，安全组规则都添加了。
   
   这个问题的具体出现原因我暂时还没找到，只是后来换了`nuxt start`外网才可以顺利访问。希望有了解到大佬能解答一下。

2. 权限不足使用`sudo su`找不到`node npm`等命令
   
   这个很简单， 我们再重新建立一边软链接就行了
  ```bash
  sudo ln -s /usr/local/bin/node /usr/bin/node
  sudo ln -s /usr/local/lib/node /usr/lib/node
  sudo ln -s /usr/local/bin/npm /usr/bin/npm
  ```

# 总结

项目做完了，总要整整齐齐的发布了才有成就感对吧😁， 文中的服务器种类有点多，不过多学点也不吃亏😭，学会了配置服务器，再配合上[`Node`的自动化脚本发布](https://segmentfault.com/a/1190000019502857#articleHeader5), 发布体验简直丝滑。

[原文地址](https://github.com/QDMarkMan/CodeBlog/blob/master/Vue/%E6%9C%8D%E5%8A%A1%E5%99%A8%E5%8F%91%E5%B8%83Vue%E9%A1%B9%E7%9B%AE%E6%8C%87%E5%8D%97.md)  如果觉得有用得话给个⭐吧