# 服务器发布Vue项目指南

## 1：Apache服务器
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

上面操作时修改服务器得默认配置让服务器支持Rewrite，下面来创建Rewrite规则

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


## 2：Nginx服务器