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

## 3：Tomcat下SpringMVC中发布

这个发布到SpringMVC中的配置相对来说就比较简单了。
只需要在发布的文件夹中新增`WEB-INF`配置文件夹中就行。如下图
![tomcat中配置](tomcat.png 'tomcat中文件')

`WEB-INF` 文件夹放在项目中那么`tomcat`会自动扫描文件夹中的`web.xml`然后重写web配置
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

这就配置完了，可以说是贼简单了。
