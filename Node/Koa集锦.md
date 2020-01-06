## 使用Koa开发API Server

写作中（）

## 写在前面



## 初始化项目

### 文件结构介绍

本项目是通过`koa-generator`创建的工程，非常的简单。希望对`koa`了解更深一点的朋友可以自己尝试着手动搭建一下，也不是很麻烦。下面是经过改造之后的文件目录结构。

```js
koa-server
│   config                  // 配置文件 包括 日志/项目配置
│   error                   // 错误处理
│   middlewares             // koa中间件
│   models                  // MySQL数据模型
│   public                  // koa2保留文件（后来会删除）
└───routes                  // API抛出层/ controller层
│   │   index               // route自动注入文件
└───service                 // 业务逻辑层（承上启下， 操作models为controller层服务）
└───sql                     // MySQL Helper文件 初始化/导出/倒入表
│   │   sql_file            // sql文件
│   │   async.js            // 数据库同步脚本文件
│   │   index.js            // MySQL 使用SQL语句
│   │   utils               // 工具类
│   │   validators          // 验证器
│   │   views               // koa2保留文件（后来会删除）
│   utils                   // 帮助文件
│   views                   // MySQL数据模型
│   app.js                  // koa项目初始化
│   main.js                 // 项目启动文件

```



### 配置文件



## 中间件

### 日志中间件



### JWT



### CORS

```js
// 我们可以用下面的中间件理解app.use(cors({}))
app.use(async (ctx, next) => {
    // 允许来自所有域名请求
    ctx.set("Access-Control-Allow-Origin", "*");
    // 这样就能只允许 http://localhost:8080 这个域名的请求了
    // ctx.set("Access-Control-Allow-Origin", "http://localhost:8080"); 

    // 设置所允许的HTTP请求方法
    ctx.set("Access-Control-Allow-Methods", "OPTIONS, GET, PUT, POST, DELETE");

    // 字段是必需的。它也是一个逗号分隔的字符串，表明服务器支持的所有头信息字段.
    ctx.set("Access-Control-Allow-Headers", "x-requested-with, accept, origin, content-type");

    // 服务器收到请求以后，检查了Origin、Access-Control-Request-Method和Access-Control-Request-Headers字段以后，确认允许跨源请求，就可以做出回应。

    // Content-Type表示具体请求中的媒体类型信息
    ctx.set("Content-Type", "application/json;charset=utf-8");

    // 该字段可选。它的值是一个布尔值，表示是否允许发送Cookie。默认情况下，Cookie不包括在CORS请求之中。
    // 当设置成允许请求携带cookie时，需要保证"Access-Control-Allow-Origin"是服务器有的域名，而不能是"*";
    ctx.set("Access-Control-Allow-Credentials", true);

    // 该字段可选，用来指定本次预检请求的有效期，单位为秒。
    // 当请求方法是PUT或DELETE等特殊方法或者Content-Type字段的类型是application/json时，服务器会提前发送一次请求进行验证
    // 下面的的设置只本次验证的有效时间，即在该时间段内服务端可以不用进行验证
    ctx.set("Access-Control-Max-Age", 300);

    /*
    CORS请求时，XMLHttpRequest对象的getResponseHeader()方法只能拿到6个基本字段：
        Cache-Control、
        Content-Language、
        Content-Type、
        Expires、
        Last-Modified、
        Pragma。
    */
    // 需要获取其他字段时，使用Access-Control-Expose-Headers，
    // getResponseHeader('myData')可以返回我们所需的值
    ctx.set("Access-Control-Expose-Headers", "myData");

    await next();
})
```



## 开发分层



### Route

### Service

### Dao



## MySQL 使用





