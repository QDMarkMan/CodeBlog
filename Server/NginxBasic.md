# Nginx安装部署手册

下载地址：<https://nginx.org/en/download.html>

本手册版本：

​ Linux：[nginx-1.14.2](https://nginx.org/download/nginx-1.14.2.tar.gz)

​ Windows： [nginx/Windows-1.14.2](https://nginx.org/download/nginx-1.14.2.zip)

参考引用：[Nginx安装及配置详解包括windows环境](https://www.cnblogs.com/loong-hon/p/9060515.html)

## 安装Nginx

### Windows

#### 1、安装

下载对应的版本的nginx压缩包，解压到自己电脑上存放软件的文件夹中，例如：C:\Users\hi.li\Desktop\opensource\nginx\nginx-1.14.2

**注：要给执行nginx的命令窗口管理员执行权限，否则可能因为读写权限问题无法正常关闭**

```
引申：默认的Nginx安装文件并不包括一些nginx模块，例如lua，如果需要使用lua，建议直接使用OpenResty打包的Nginx，已直接集成了通用的lua模块并支持lua。
OpenResty下载地址：http://openresty.org/cn/download.html
```

#### 2、启动nginx

```
>cd C:\Users\hi.li\Desktop\opensource\nginx\nginx-1.14.2
>nginx
# or启动后台任务模式
>start nginx
# or 指定配置文件启动
>nginx -c test_nginx.conf
```

#### 3、访问nginx

打开浏览器，输入地址：<http://localhost，访问页面，出现如下页面表示访问成功>

![01](media/nginx-install/01.png)

#### 4、停止nginx

命令行进入nginx根目录，执行如下命令，停止服务器：

```
>cd C:\Users\hi.li\Desktop\opensource\nginx\nginx-1.14.2

# 强制停止nginx服务器，如果有未处理的数据，丢弃
> nginx -s stop

# 优雅的停止nginx服务器，如果有未处理的数据，等待处理完成之后停止
> nginx -s quit
```

#### 5、其他nginx命令

```
nginx -s reload  ：修改配置后重新加载生效
nginx -s reopen  ：重新打开日志文件
nginx -t -c /path/to/nginx.conf : 测试nginx配置文件是否正确
nginx -h : 打开命令帮助
```

### CentOS(待检验)

#### 1、安装准备

由于nginx的一些模块依赖一些lib库，所以在安装nginx之前，必须先安装这些lib库，这些依赖库主要有g++、gcc、openssl-devel、pcre-devel和zlib-devel 所以执行如下命令安装

```
yum install gcc-c++
yum install pcre pcre-devel
yum install zlib zlib-devel
yum install openssl openssl--devel
```

如果之前有安装国nginx，先删除

```
$ find -name nginx

# 如果系统已经安装了nginx，那么就先卸载
$ yum remove nginx
```

#### 2、安装

下载对应的版本的nginx压缩包，放置到/usr/local目录中，并解压缩：

```
$ cd /usr/local

# 从官网下载最新版的nginx
$ wget https://nginx.org/download/nginx-1.14.2.tar.gz

# 解压nginx压缩包
$ tar -zxvf nginx-1.14.2.tar.gz

# 接下来安装，使用--prefix参数指定nginx安装的目录,make、make install安装
$ cd nginx-1.14.2
# 默认安装在/usr/local/nginx
$ ./configure
$ make
$ make install

# 如果没有报错，顺利完成后，最好看一下nginx的安装目录
$ whereis nginx
```

安装完成后，在/usr/sbin/目录下是nginx命令所在目录，在/etc/nginx/目录下是nginx所有的配置文件，用于配置nginx服务器以及负载均衡等信息

#### 3、查看nginx进程是否启动

```
ps -ef|grep nginx
```

nginx会自动根据当前主机的CPU的内核数目创建对应的进程数量，启动的服务进程为CPU内核数+1，因为nginx进程在启动的时候，会附带一个守护进程，用于保护正式进程不被异常终止；如果守护进程一旦返现nginx继承被终止了，会自动重启该进程。

守护进程一般会称为master进程，业务进程被称为worker进程

#### 4、启动、关闭nginx的命令和windows一致

## Nginx配置

nginx是一个功能非常强大的web服务器加反向代理服务器，同时又是邮件服务器等等。

在项目使用中，使用最多的三个核心功能是反向代理、负载均衡和静态服务器。这三个不同的功能的使用，都跟nginx的配置密切相关，nginx服务器的配置信息主要集中在nginx.conf这个配置文件中，并且所有的可配置选项大致分为以下几个部分：

```

```

如上述配置文件所示，主要由6个部分组成：

1. main：用于进行nginx全局信息的配置
2. events：用于nginx工作模式的配置
3. http：用于进行http协议信息的一些配置
4. server：用于进行服务器访问信息的配置
5. location：用于进行访问路由的配置
6. upstream：用于进行负载均衡的配置

### main模块

```
# user nobody nobody;
worker_processes 2;
# error_log logs/error.log
# error_log logs/error.log notice
# error_log logs/error.log info
# pid logs/nginx.pid
worker_rlimit_nofile 1024;
```

上述配置都是存放在main全局配置模块中的配置项

- user用来指定nginx worker进程运行用户以及用户组，默认nobody账号运行

- worker_processes指定nginx要开启的子进程数量，运行过程中监控每个进程消耗内存(一般几M~几十M不等)根据实际情况进行调整，通常数量是CPU内核数量的整数倍

- error_log定义错误日志文件的位置及输出级别【debug / info / notice / warn / error / crit】

- pid用来指定进程id的存储文件的位置

- worker_rlimit_nofile用于指定一个进程可以打开最多文件数量的描述

### event 模块

```
event {
    worker_connections 1024;
    multi_accept on;
    use epoll;
}
```

上述配置是针对nginx服务器的工作模式的一些操作配置

- worker_connections 指定最大可以同时接收的连接数量，这里一定要注意，最大连接数量是和worker processes共同决定的。
- multi_accept 配置指定nginx在收到一个新连接通知后尽可能多的接受更多的连接
- use epoll 配置指定了线程轮询的方法，如果是linux2.6+，使用epoll，如果是BSD如Mac请使用Kqueue

### http模块

作为web服务器，http模块是nginx最核心的一个模块，配置项也是比较多的，项目中会设置到很多的实际业务场景，需要根据硬件信息进行适当的配置，常规情况下，使用默认配置即可！

```
http {
    ##
    # 基础配置
    ##

    sendfile on;
    tcp_nopush on;
    tcp_nodelay on;
    keepalive_timeout 65;
    types_hash_max_size 2048;
    # server_tokens off;

    # server_names_hash_bucket_size 64;
    # server_name_in_redirect off;

    include /etc/nginx/mime.types;
    default_type application/octet-stream;

    ##
    # SSL证书配置
    ##

    ssl_protocols TLSv1 TLSv1.1 TLSv1.2; # Dropping SSLv3, ref: POODLE
    ssl_prefer_server_ciphers on;

    ##
    # 日志配置
    ##

    access_log /var/log/nginx/access.log;
    error_log /var/log/nginx/error.log;

    ##
    # Gzip 压缩配置
    ##

    gzip on;
    gzip_disable "msie6";

    # gzip_vary on;
    # gzip_proxied any;
    # gzip_comp_level 6;
    # gzip_buffers 16 8k;
    # gzip_http_version 1.1;
    # gzip_types text/plain text/css application/json application/javascript
 text/xml application/xml application/xml+rss text/javascript;

    ##
    # 虚拟主机配置
    ##

    include /etc/nginx/conf.d/*.conf;
    include /etc/nginx/sites-enabled/*;
```

1) 基础配置

- sendfile on：配置on让sendfile发挥作用，将文件的回写过程交给数据缓冲去去完成，而不是放在应用中完成，这样的话在性能提升有有好处
- tc_nopush on：让nginx在一个数据包中发送所有的头文件，而不是一个一个单独发
- tcp_nodelay on：让nginx不要缓存数据，而是一段一段发送，如果数据的传输有实时性的要求的话可以配置它，发送完一小段数据就立刻能得到返回值，但是不要滥用哦
- keepalive_timeout 10：给客户端分配连接超时时间，服务器会在这个时间过后关闭连接。一般设置时间较短，可以让nginx工作持续性更好
- client_header_timeout 10：设置请求头的超时时间
- client_body_timeout 10:设置请求体的超时时间
- send_timeout 10：指定客户端响应超时时间，如果客户端两次操作间隔超过这个时间，服务器就会关闭这个链接
- limit_conn_zone $binary_remote_addr zone=addr:5m ：设置用于保存各种key的共享内存的参数，
- limit_conn addr 100: 给定的key设置最大连接数
- server_tokens：虽然不会让nginx执行速度更快，但是可以在错误页面关闭nginx版本提示，对于网站安全性的提升有好处哦
- include /etc/nginx/mime.types：指定在当前文件中包含另一个文件的指令
- default_type application/octet-stream：指定默认处理的文件类型可以是二进制
- type_hash_max_size 2048：混淆数据，影响三列冲突率，值越大消耗内存越多，散列key冲突率会降低，检索速度更快；值越小key，占用内存较少，冲突率越高，检索速度变慢

2) 日志配置

- access_log logs/access.log：设置存储访问记录的日志
- error_log logs/error.log：设置存储记录错误发生的日志

3) SSL证书加密

- ssl_protocols：指令用于启动特定的加密协议，nginx在1.1.13和1.0.12版本后默认是ssl_protocols SSLv3 TLSv1 TLSv1.1 TLSv1.2，TLSv1.1与TLSv1.2要确保OpenSSL >= 1.0.1 ，SSLv3 现在还有很多地方在用但有不少被攻击的漏洞。
- ssl prefer server ciphers：设置协商加密算法时，优先使用我们服务端的加密套件，而不是客户端浏览器的加密套件

4) 压缩配置

- gzip 是告诉nginx采用gzip压缩的形式发送数据。这将会减少我们发送的数据量。
- gzip_disable 为指定的客户端禁用gzip功能。我们设置成IE6或者更低版本以使我们的方案能够广泛兼容。
- gzip_static 告诉nginx在压缩资源之前，先查找是否有预先gzip处理过的资源。这要求你预先压缩你的文件（在这个例子中被注释掉了），从而允许你使用最高压缩比，这样nginx就不用再压缩这些文件了（想要更详尽的gzip_static的信息，请点击这里）。
- gzip_proxied 允许或者禁止压缩基于请求和响应的响应流。我们设置为any，意味着将会压缩所有的请求。
- gzip_min_length 设置对数据启用压缩的最少字节数。如果一个请求小于1000字节，我们最好不要压缩它，因为压缩这些小的数据会降低处理此请求的所有进程的速度。
- gzip_comp_level 设置数据的压缩等级。这个等级可以是1-9之间的任意数值，9是最慢但是压缩比最大的。我们设置为4，这是一个比较折中的设置。
- gzip_type 设置需要压缩的数据格式。上面例子中已经有一些了，你也可以再添加更多的格式。

5) 文件缓存配置

- open_file_cache 打开缓存的同时也指定了缓存最大数目，以及缓存的时间。我们可以设置一个相对高的最大时间，这样我们可以在它们不活动超过20秒后清除掉。
- open_file_cache_valid 在open_file_cache中指定检测正确信息的间隔时间。
- open_file_cache_min_uses 定义了open_file_cache中指令参数不活动时间期间里最小的文件数。
- open_file_cache_errors 指定了当搜索一个文件时是否缓存错误信息，也包括再次给配置中添加文件。我们也包括了服务器模块，这些是在不同文件中定义的。如果你的服务器模块不在这些位置，你就得修改这一行来指定正确的位置。

### server模块

srever模块配置是http模块中的一个子模块，用来定义一个虚拟访问主机，也就是一个虚拟服务器的配置信息:

```
server {
    listen        80;
    server_name localhost    192.168.1.100;
    root        /nginx/www;
    index        index.php index.html index.html;
    charset        utf-8;
    access_log    logs/access.log;
    error_log    logs/error.log;
    ......
}
```

核心配置信息如下：

- server：一个虚拟主机的配置，一个http中可以配置多个server
- server_name：用力啊指定ip地址或者域名，多个配置之间用空格分隔
- root：表示整个server虚拟主机内的根目录，所有当前主机中web项目的根目录
- index：用户访问web网站时的全局首页
- charset：用于设置www/路径中配置的网页的默认编码格式
- access_log：用于指定该虚拟主机服务器中的访问记录日志存放路径
- error_log：用于指定该虚拟主机服务器中访问错误日志的存放路径

### location模块

location模块是nginx配置中出现最多的一个配置，主要用于配置路由访问信息 。

在路由访问信息配置中关联到反向代理、负载均衡等等各项功能，所以location模块也是一个非常重要的配置模块

基本配置:

```
location / {
    root    /nginx/www;
    index    index.php index.html index.htm;
}
```

- location /：表示匹配访问根目录
- root：用于指定访问根目录时，访问虚拟主机的web目录
- index：在不指定访问具体资源时，默认展示的资源文件列表

反向代理配置方式

通过反向代理代理服务器访问模式，通过proxy_set配置让客户端访问透明化

```
location / {
    proxy_pass http://localhost:8888;
    proxy_set_header X-real-ip $remote_addr;
    proxy_set_header Host $http_host;
}
```

uwsgi配置

wsgi模式下的服务器配置访问方式

```
location / {
    include uwsgi_params;
    uwsgi_pass localhost:8888
}
```

### upstream模块

upstream模块主要负责负载均衡的配置，通过默认的轮询调度方式来分发请求到后端服务器。

简单的配置方式如下：

```
upstream name {
    ip_hash;
    server 192.168.1.100:8000;
    server 192.168.1.100:8001 down;
    server 192.168.1.100:8002 max_fails=3;
    server 192.168.1.100:8003 fail_timeout=20s;
    server 192.168.1.100:8004 max_fails=3 fail_timeout=20s;
}
```

核心配置信息如下

- ip_hash：指定请求调度算法，默认是weight权重轮询调度，可以指定
- server host:port：分发服务器的列表配置
- -- down：表示该主机暂停服务
- -- max_fails：表示失败最大次数，超过失败最大次数暂停服务
- -- fail_timeout：表示如果请求受理失败，暂停指定的时间之后重新发起请求

## nginx location正则写法

### 语法（Syntax）

```
location [ = | ~ | ~* | ^~ ] uri { ... }
location @name { ... }
```

- 普通逻辑（uri前不带指令符）的匹配规则是最大匹配，如果是严格匹配（正好完全精确匹配），则匹配上不会再匹配其他规则；如果不是严格匹配，则会继续匹配其他规则；例如

  ```
  location /login/a.html {
      # 匹配/login/a.html开头的uri
      proxy_pass http://www.baidu.com;
  }
  
  location /login/ {
      # 匹配/login/开头的uri
      规则B
  }
  
  # uri localhost/login/a.html 匹配到第1个规则后停止匹配其他正则规则
  ```

- 以" = "（注意前后要有空格）开头表示精确匹配，匹配上会中止其他匹配；例如

  ```
  location = /login {
      # 精确匹配后面的字符， 例如“localhost/login”可以匹配上
  }
  ```

- `^~` 开头表示uri以某个常规字符串开头，不是正则匹配，匹配上会中止其他匹配；例如

  ```
  location ^~ /pic/ {
      # 匹配/pic/开头的uri，如果匹配上但找不到资源，默认取/pic/ios.jpg
      root ../static_server;
      try_files $uri /pic/ios.jpg;
  }
  ```

- ~ 开头表示区分大小写的正则匹配；正则匹配只要匹配上就会中止其他匹配；

- ~* 开头表示不区分大小写的正则匹配；正则匹配只要匹配上就会中止其他匹配；

- / 通用匹配, 如果没有其它匹配,任何请求都会匹配到，例如

  ```
  location / {
      # 首页，指定主页，如果没有其他匹配，根据uri在../static_server下找资源
      root   ../static_server;
      index  login.html;
  }
  ```

- " = / "和“ / ”的区别

  ```
  " = / "： 精确匹配 "/" ，后面带任何字符都不能匹配上，也就是严格匹配http://host_name/这种情况
  " / ": 通用匹配，可以匹配上所有uri
  ```

### uri匹配规则

- 普通匹配的uri匹配规则是从前往后最大字符匹配（区分大小写）

- 正则匹配需要指令符“ ~ ”和“ ~* ”标记，后面的uri使用标准正则表达式即可，例如

  ```
  location ~* \.(mp3|mp4) {
      # 匹配所有以mp3或mp4结尾的uri，不区分大小写
  }
  ```

- 注意配置中的uri无需进行URLDecode处理，例如要匹配浏览器中的URL“/images/%20/test ”，你写location 的时候匹配目标应该是：“/images/ /test ”

## Rewrite用法

Rewrite功能就是使用nginx提供的全局变量或自己设置的变量，结合正则表达式和标志位实现url重写以及重定向。

```
限制：
rewrite只能放在 server{}, location{}, if{}中，并且只能对域名后边的除去传递的参数外的字符串起作用。
例如 http://seanlook.com/a/we/index.php?id=1&u=str 只对/a/we/index.php重写。

如果想对域名或参数字符串起作用，可以使用全局变量匹配，也可以使用proxy_pass反向代理。
```

rewrite和location功能有点像，都能实现跳转。主要区别在于rewrite是在**同一域名内**更改获取资源的路径，而location是对一类路径做控制访问或反向代理，**可以proxy_pass到其他机器**。

```
很多情况下rewrite也会写在location里，它们的执行顺序是：
1 执行server块的rewrite指令
2 执行location匹配
3 执行选定的location中的rewrite指令
```

### 语法

**Syntax ： rewrite regex replacement [flag];**

- regex ：要进行替换的匹配正则表达式（匹配上才会执行rewrite操作），如果表达式中含{}，则应使用单引号(')引用起来；同时可以基于该表达式获取原uri中的指定匹配项用于replacement 进行替换

- replacement :  要将uri替换后的字符串，可使用“$1”、“$2”等引用前面表达式的匹配项

- flag ：处理标记位

  - - last : 相当于Apache的[L]标记，表示完成rewrite

  - break : 停止执行当前虚拟主机的后续rewrite指令集

  - redirect : 返回302临时重定向，地址栏会显示跳转后的地址

  - permanent : 返回301永久重定向，地址栏会显示跳转后的地址

    **注：因为301和302不能简单的只返回状态码，还必须有重定向的URL，这就是return指令无法返回301,302的原因了。**

    ```
    这里 last 和 break 区别有点难以理解：
    1. last一般写在server和if中，而break一般使用在location中
    2. last不终止重写后的url匹配，即重写后的url会继续走下面的location匹配处理过程；而break将直接终止重写后的location匹配；
    3. break和last都能阻止继续执行后面（同一个域内，例如同一个location下）的rewrite指令；
    ```

### rewrite_log

开启rewrite_log，这样在/var/log/nginx/error.log中显示匹配的规则，便于debug，理解rewrite的过程。

```
rewrite_log on;
access_log /var/log/nginx/access.log;
error_log /var/log/nginx/error.log info;
```

### 实例

**server中**

```
# 如果UA包含"MSIE"（浏览器为IE），rewrite请求到/msie/目录下
if ($http_user_agent ~ MSIE) {
    rewrite ^(.*)$ /msie/$1 break;
}
```

**location中**

```
# 对形如/images/ef/uh7b3/test.png的请求，重写到/data?file=test.png，于是匹配到location 
# /data，先看/data/images/test.png文件存不存在，如果存在则正常响应，如果不存在则重写tryfiles
# 到新的image404 location，直接返回404状态码。

location / {
    # 注意这里要用‘’单引号引起来，避免{}
    rewrite '^/images/([a-z]{2})/([a-z0-9]{5})/(.*)\.(png|jpg|gif)$' /data?file=$3.$4;
    # 注意不能在上面这条规则后面加上“last”参数，否则下面的set指令不会执行
    set $image_file $3;
    set $image_type $4;
    
    # 对形如/images/bla_500x400.jpg的文件请求，重写到/resizer/bla.jpg?width=500&height=400地址，并会继续尝试匹配location
    rewrite ^/images/(.*)_(\d+)x(\d+)\.(png|jpg|gif)$ /resizer/$1.$4？width=$2&height=$3? last;
}

location /data {
    # 指定针对图片的日志格式，来分析图片类型和大小
    access_log logs/images.log mian;
    root /data/images;
    # 应用前面定义的变量。判断首先文件在不在，不在再判断目录在不在，如果还不在就跳转到最后一个url里
    try_files /$arg_file /image404.html;
}

location = /image404.html {
    # 图片不存在返回特定的信息
    return 404 "image not found\n";
}
```

## if指令与全局变量

### 语法

**语法：if (condition) {...}**

对给定的条件condition进行判断。如果为真，大括号内的指令将被执行（如rewrite或set），if条件(conditon)可以是如下任何内容：

- 当表达式只是一个变量时，如果值为空或任何以0开头的字符串都会当做false
- 直接比较变量和内容时，使用=或!=
- ~  正则表达式匹配
- ~* 不区分大小写的匹配
- !~  区分大小写的不匹配
- -f和!-f  用来判断是否存在文件
- -d和!-d  用来判断是否存在目录
- -e和!-e  用来判断是否存在文件或目录
- -x和!-x  用来判断文件是否可执行

**示例**

```
# 如果用户设备为IE浏览器的时候，重定向
if ($http_user_agent ~ MSIE) {
    rewrite ^(.*)$ /msie/$1 break;
}

# 如果cookie匹配正则，设置变量$id等于正则引用部分
if ($http_cookie ~* "id=([^;]+)(?:;|$)") {
    set $id $1;
}

# 如果提交方法为POST，则返回状态405（Method not allowed）。return不能返回301,302
if ($request_method = POST) {
    return 405;
}

# 限速，$slow可以通过 set 指令设置
if ($slow) {
    limit_rate 10k;
}

# 如果请求的文件名不存在，则反向代理到localhost 。这里的break也是停止rewrite检查
if (!-f $request_filename){
    break;
    proxy_pass http://127.0.0.1;
}

# 如果query string中包含"post=140"，永久重定向到example.com
if ($args ~ post=140){
    rewrite ^ http://example.com/ permanent;
}

# 防盗链
location ~* \.(gif|jpg|png|swf|flv)$ {
    valid_referers none blocked www.jefflei.comwww.leizhenfang.com;
    if ($invalid_referer) {
        return 404;
    }
}
```

### 全局变量

下面是可以用作if判断的全局变量

- $args ： #这个变量等于请求行中的参数，同$query_string
- $content_length ： 请求头中的Content-length字段。
- $content_type ： 请求头中的Content-Type字段。
- $document_root ： 当前请求在root指令中指定的值。
- $host ： 请求主机头字段，否则为服务器名称。
- $http_user_agent ： 客户端agent信息
- $http_cookie ： 客户端cookie信息
- $limit_rate ： 这个变量可以限制连接速率。
- $request_method ： 客户端请求的动作，通常为GET或POST。
- $remote_addr ： 客户端的IP地址。
- $remote_port ： 客户端的端口。
- $remote_user ： 已经经过Auth Basic Module验证的用户名。
- $request_filename ： 当前请求的文件路径，由root或alias指令与URI请求生成。
- $scheme ： HTTP方法（如http，https）。
- $server_protocol ： 请求使用的协议，通常是HTTP/1.0或HTTP/1.1。
- $server_addr ： 服务器地址，在完成一次系统调用后可以确定这个值。
- $server_name ： 服务器名称。
- $server_port ： 请求到达服务器的端口号。
- $request_uri ： 包含请求参数的原始URI，不包含主机名，如：”/foo/bar.php?arg=baz”。
- $uri ： 不带请求参数的当前URI，$uri不包含主机名，如”/foo/bar.html”。
- $document_uri ： 与$uri相同。

**示例**

```
http://localhost:88/test1/test2/test.php
$host：localhost
$server_port：88
$request_uri：http://localhost:88/test1/test2/test.php
$document_uri：/test1/test2/test.php
$document_root：/var/www/html
$request_filename：/var/www/html/test1/test2/test.php
```

## Nginx内置变量以及日志格式变量参数详解

```
$args                    #请求中的参数值
$query_string            #同 $args
$arg_NAME                #GET请求中NAME的值
$is_args                 #如果请求中有参数，值为"?"，否则为空字符串
$uri                     #请求中的当前URI(不带请求参数，参数位于$args)，可以不同于浏览器传递的$request_uri的值，它可以通过内部重定向，或者使用index指令进行修改，$uri不包含主机名，如"/foo/bar.html"。
$document_uri            #同 $uri
$document_root           #当前请求的文档根目录或别名
$host                    #优先级：HTTP请求行的主机名>"HOST"请求头字段>符合请求的服务器名.请求中的主机头字段，如果请求中的主机头不可用，则为服务器处理请求的服务器名称
$hostname                #主机名
$https                   #如果开启了SSL安全模式，值为"on"，否则为空字符串。
$binary_remote_addr      #客户端地址的二进制形式，固定长度为4个字节
$body_bytes_sent         #传输给客户端的字节数，响应头不计算在内；这个变量和Apache的mod_log_config模块中的"%B"参数保持兼容
$bytes_sent              #传输给客户端的字节数
$connection              #TCP连接的序列号
$connection_requests     #TCP连接当前的请求数量
$content_length          #"Content-Length" 请求头字段
$content_type            #"Content-Type" 请求头字段
$cookie_name             #cookie名称
$limit_rate              #用于设置响应的速度限制
$msec                    #当前的Unix时间戳
$nginx_version           #nginx版本
$pid                     #工作进程的PID
$pipe                    #如果请求来自管道通信，值为"p"，否则为"."
$proxy_protocol_addr     #获取代理访问服务器的客户端地址，如果是直接访问，该值为空字符串
$realpath_root           #当前请求的文档根目录或别名的真实路径，会将所有符号连接转换为真实路径
$remote_addr             #客户端地址
$remote_port             #客户端端口
$remote_user             #用于HTTP基础认证服务的用户名
$request                 #代表客户端的请求地址
$request_body            #客户端的请求主体：此变量可在location中使用，将请求主体通过proxy_pass，fastcgi_pass，uwsgi_pass和scgi_pass传递给下一级的代理服务器
$request_body_file       #将客户端请求主体保存在临时文件中。文件处理结束后，此文件需删除。如果需要之一开启此功能，需要设置client_body_in_file_only。如果将次文件传 递给后端的代理服务器，需要禁用request body，即设置proxy_pass_request_body off，fastcgi_pass_request_body off，uwsgi_pass_request_body off，or scgi_pass_request_body off
$request_completion      #如果请求成功，值为"OK"，如果请求未完成或者请求不是一个范围请求的最后一部分，则为空
$request_filename        #当前连接请求的文件路径，由root或alias指令与URI请求生成
$request_length          #请求的长度 (包括请求的地址，http请求头和请求主体)
$request_method          #HTTP请求方法，通常为"GET"或"POST"
$request_time            #处理客户端请求使用的时间,单位为秒，精度毫秒； 从读入客户端的第一个字节开始，直到把最后一个字符发送给客户端后进行日志写入为止。
$request_uri             #这个变量等于包含一些客户端请求参数的原始URI，它无法修改，请查看$uri更改或重写URI，不包含主机名，例如："/cnphp/test.php?arg=freemouse"
$scheme                  #请求使用的Web协议，"http" 或 "https"
$server_addr             #服务器端地址，需要注意的是：为了避免访问linux系统内核，应将ip地址提前设置在配置文件中
$server_name             #服务器名
$server_port             #服务器端口
$server_protocol         #服务器的HTTP版本，通常为 "HTTP/1.0" 或 "HTTP/1.1"
$status                  #HTTP响应代码
$time_iso8601            #服务器时间的ISO 8610格式
$time_local              #服务器时间（LOG Format 格式）
$cookie_NAME             #客户端请求Header头中的cookie变量，前缀"$cookie_"加上cookie名称的变量，该变量的值即为cookie名称的值
$http_NAME               #匹配任意请求头字段；变量名中的后半部分NAME可以替换成任意请求头字段，如在配置文件中需要获取http请求头："Accept-Language"，$http_accept_language即可
$http_cookie
$http_host               #请求地址，即浏览器中你输入的地址（IP或域名）
$http_referer            #url跳转来源,用来记录从那个页面链接访问过来的
$http_user_agent         #用户终端浏览器等信息
$http_x_forwarded_for
$sent_http_NAME          #可以设置任意http响应头字段；变量名中的后半部分NAME可以替换成任意响应头字段，如需要设置响应头Content-length，$sent_http_content_length即可
$sent_http_cache_control
$sent_http_connection
$sent_http_content_type
$sent_http_keep_alive
$sent_http_last_modified
$sent_http_location
$sent_http_transfer_encoding
```

**正确设置nginx中remote_addr和x_forwarded_for参数**

什么是remote_addr：
remote_addr代表客户端的IP，但它的值不是由客户端提供的，而是服务端根据客户端的ip指定的，当你的浏览器访问某个网站时，假设中间没有任何代理，那么网站的web服务器（Nginx，Apache等）就会把remote_addr设为你的机器IP，如果你用了某个代理，那么你的浏览器会先访问这个代理，然后再由这个代理转发到网站，这**样web服务器就会把remote_addr设为这台代理机器的IP**

什么是x_forwarded_for：
正如上面所述，当你使用了代理时，web服务器就不知道你的真实IP了，为了避免这个情况，代理服务器通常会增加一个叫做x_forwarded_for的头信息，把连接它的客户端IP（即你的上网机器IP）加到这个头信息里，这样就能**保证网站的web服务器能获取到真实IP**

**使用HAProxy做反向代理时：**
通常网站为了支撑更大的访问量，会增加很多web服务器，并在这些服务器前面增加一个反向代理（如HAProxy），它可以把负载均匀的分布到这些机器上。你的浏览器访问的首先是这台反向代理，它再把你的请求转发到后面的web服务器，这就使得web服务器会把remote_addr设为这台反向代理的IP，为了能让你的程序获取到真实的客户端IP，你需要给HAProxy增加以下配置

```
option forwardfor
```

它的作用就像上面说的，增加一个x_forwarded_for的头信息，把客户端的ip添加进去，否则的话经测试为空值
如上面的日志格式所示：**$http_x_forwarded_for 是客户端真实的IP地址，$remote_addr是前端Haproxy的IP地址**

**或者：**

当Nginx处在HAProxy后面时，就会把remote_addr设为HAProxy的IP，这个值其实是毫无意义的，你可以通过nginx的realip模块，让它使用x_forwarded_for里的值。使用这个模块需要重新编译Nginx，增加**--with-http_realip_module**参数
**./configure  --user=www --group=www --prefix=/usr/local/nginx --with-http_stub_status_module --with-http_ssl_module \                                                              --with-http_realip_module --http-log-path=/data/logs/nginx/access.log --error-log-path=/data/logs/nginx/error.log**

```
     set_real_ip_from 10.1.10.0/24;
     real_ip_header X-Forwarded-For;
```

上面的两行配置就是把从10.1.10这一网段过来的请求全部使用X-Forwarded-For里的头信息作为remote_addr，这样此时remote_addr就是客户端真实的IP地址

**X-Forwarded-For 和 X-Real-IP 获取客户端的ip的区别：**
一般来说，X-Forwarded-For是用于记录代理信息的，每经过一级代理(匿名代理除外)，代理服务器都会把这次请求的来源IP追加在X-Forwarded-For中 来自4.4.4.4的一个请求，header包含这样一行 X-Forwarded-For: 1.1.1.1, 2.2.2.2, 3.3.3.3 代表 请求由1.1.1.1发出，经过三层代理，第一层是2.2.2.2，第二层是3.3.3.3，而本次请求的来源IP 4.4.4.4是第三层代理 **而X-Real-IP，一般只记录真实发出请求的客户端IP，上面的例子，如果配置了X-Read-IP，将会是 X-Real-IP: 1.1.1.1 所以 ，如果只有一层代理，这两个头的值就是一样的**

​

## ngx_lua 模块API说明

<https://blog.csdn.net/hwhjava/article/details/47722309>

## Nginx实战

### 静态资源Web服务器

1、设置资源目录，示例：

```
C:\Users\hi.li\Desktop\opensource\nginx\static_server\html
C:\Users\hi.li\Desktop\opensource\nginx\static_server\login.html
C:\Users\hi.li\Desktop\opensource\nginx\static_server\mp3
C:\Users\hi.li\Desktop\opensource\nginx\static_server\pic
C:\Users\hi.li\Desktop\opensource\nginx\static_server\html\a.html
C:\Users\hi.li\Desktop\opensource\nginx\static_server\mp3\1
C:\Users\hi.li\Desktop\opensource\nginx\static_server\mp3\kaluli.mp3
C:\Users\hi.li\Desktop\opensource\nginx\static_server\mp3\1\cymx.mp3
C:\Users\hi.li\Desktop\opensource\nginx\static_server\pic\abc
C:\Users\hi.li\Desktop\opensource\nginx\static_server\pic\android.jpg
C:\Users\hi.li\Desktop\opensource\nginx\static_server\pic\bootstrap.png
C:\Users\hi.li\Desktop\opensource\nginx\static_server\pic\ios.jpg
C:\Users\hi.li\Desktop\opensource\nginx\static_server\pic\abc\redis.png
```

2、编辑conf目录下的nginx.conf文件，增加一个虚拟服务器配置(端口为81)：

```
    # 静态资源服务器
    server {
        listen       81;
        server_name  localhost;

        charset utf-8;

        #access_log  logs/host.access.log  main;
  
        # 首页，指定主页，如果没有其他匹配，根据uri在../static_server下找资源
        location / {
            root   ../static_server;
            index  login.html;
        }
        
        # 图片资源请求, 匹配以/pic/开头的Url, 如果找不到文件，默认显示/pic/ios.jpg
        location ^~ /pic/ {
            # 设置本地缓存过期时间为30天，充分利用客户端缓存提升性能
            expires 30d;
            root ../static_server;
            try_files $uri /pic/ios.jpg;
        }
        
        # 音频资源请求，文件后缀为mp3或mp4这类文件，统一到../static_server/mp3下查找, 且进行传输优化
        location ~* \.(mp3|mp4) {
            root ../static_server/mp3;
            try_files $uri /pic/android.jpg;
            sendfile           on;
            sendfile_max_chunk 1m;
            tcp_nodelay       on;
            keepalive_timeout 65;
        }
        
        # 图片防盗链设置，只有在valid_referers指令列表中的域名过来的链接才能访问：
        # none : Referer” 来源头部为空
        # blocked ：Referer”来源头部不为空，但是里面的值被代理或者防火墙删除了，这些值都不以http://或者https://开头
        # server_names : “Referer”来源头部包含当前的server_names（当前域名）
        # *.ttlsa.com : 任意字符串,定义服务器名或者可选的URI前缀.主机名可以使用*开头或者结尾，在检测来源头部这个过程中，来源域名中的主机端口将会被忽略掉
        # ~\.google\. ~\.baidu\. : 正则表达式,~表示排除https://或http://开头的字符串
        location ~* \.(gif|jpg|png|bmp)$ {
            valid_referers none blocked *.ttlsa.com server_names ~\.google\. ~\.baidu\.;
            if ($invalid_referer) {
                return 403;
                #rewrite ^/ http://www.ttlsa.com/403.jpg;
            }
        }
        
        # 指定代理转向，uri为/html/b.html时重定向至百度网站
        location /html/b.html {
            proxy_pass http://www.baidu.com;
        }
        
        # 动态页面代理转发至其他地址处理 
        location ~ .(jsp|do)$ {  
            proxy_pass  http://localhost;  
        }  


        error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        #
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }

    }
```

3、检测配置是否正确：

```
# 检测默认配置文件
> nginx -t
# 检测指定配置文件
> nginx -t -c conf/nginx.conf
```

4、启动服务器

5、在浏览器打开以下地址验证配置

```
# html访问
http://localhost:81/  # 显示login.html
http://localhost:81/index.html  # 找不到文件404
http://localhost:81/html/a.html  # 显示a.html
http://localhost:81/html/b.html  # 跳转到百度

# 图片资源获取
http://localhost:81/pic/android.jpg  # 显示android.jpg
http://localhost:81/pic/bootstrap.png  # 显示bootstrap.png
http://localhost:81/pic/abc/redis.png  # 显示redis.png
http://localhost:81/pic/  # 显示默认的iso.jpg
http://localhost:81/pic/1.jpg  # 显示默认的iso.jpg

# 音频资源请求
http://localhost:81/kaluli.mp3  # 下载kaluli.mp3
http://localhost:81/1/cymx.mp3  # 下载cymx.mp3
http://localhost:81/cymx.mp3  # 找不到路径，显示默认的android.jpg
http://localhost:81/test/a.mp4  # 找不到路径，显示默认的android.jpg
http://localhost:81/pic/1.mp3  # 优先匹配到/pic/规则，显示默认的iso.jpg
```

### Https访问设置

#### 申请证书

1、购买证书：可以在阿里云控制台-产品与服务-安全(云盾)-CA证书服务(数据安全)，点击购买证书；申请完成后可将证书下载到本地；

2、通过openssl制作证书

openssl是目前最流行的SSL密码库工具，其提供了一个通用、健壮、功能完备的工具套件，用以支持SSL/TLS协议的实现。

比如生成到：/usr/local/ssl

```
openssl req -x509 -nodes -days 36500 -newkey rsa:2048 -keyout /usr/local/ssl/nginx.key -out /usr/local/ssl/nginx.crt
```

生成过程：

```
# openssl req -x509 -nodes -days 36500 -newkey rsa:2048 -keyout /u    sr/local/ssl/nginx.key -out /usr/local/ssl/nginx.crt
Generating a 2048 bit RSA private key
...............................................................................+    ++
...............+++
writing new private key to '/usr/local/ssl/nginx.key'
-----
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [XX]:CN
State or Province Name (full name) []:beijing
Locality Name (eg, city) [Default City]:beijing
Organization Name (eg, company) [Default Company Ltd]:xxxx
Organizational Unit Name (eg, section) []:xxxx
Common Name (eg, your name or your server's hostname) []:xxxx（一般是域名）
Email Address []:xxxx@xxxx.com
# ll
total 8
-rw-r--r--. 1 root root 1391 Apr 21 13:29 nginx.crt
-rw-r--r--. 1 root root 1704 Apr 21 13:29 nginx.key
```

#### 设置服务器配置

1、将两个证书复制到服务器目录，可以自己在nginx目录中创建cert目录，放置证书（nginx.crt、nginx.key）；

2、修改nginx服务器配置，默认为nginx.conf

```
# https服务
server {
    listen 443;
    server_name hivenet.com; // 你的域名
    ssl on;
    root ../static_server; // 前台文件存放文件夹，可改成别的
    index login.html;// 上面配置的文件夹里面的login.html
    ssl_certificate  cert/nginx.crt;// 改成你的证书的名字
    ssl_certificate_key cert/nginx.key;// 你的证书的名字
    ssl_session_timeout 5m;
    # ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    # ssl_prefer_server_ciphers on;
    location / {
        index login.html;
    }
}

# http服务，跳转到https访问
server {
    listen 80;
    server_name hivenet.com;// 你的域名
    rewrite ^(.*)$ https://$host$1 permanent;// 把http的域名请求转成https
}
```

### 负载均衡+高可用配置

#### 环境信息

192.168.0.221：nginx + keepalived   master

192.168.0.222：nginx + keepalived   backup

192.168.0.223：nginx（后台静态资源Web服务器，也可替换为Tomcat等应用服务器）

192.168.0.224：nginx（后台静态资源Web服务器，也可替换为Tomcat等应用服务器）

虚拟ip(VIP):192.168.0.200，对外提供服务的ip，也可称作浮动ip

注：keepalived的安装自行参考其他材料

#### Nginx负载均衡配置

nginx.conf的配置参考如下（两台服务器的配置完全一样，暂时无主从之分）：

```
user  root;            #运行用户
worker_processes  1;        #启动进程,通常设置成和cpu的数量相等

#全局错误日志及PID文件
error_log  /usr/local/nginx/logs/error.log;
error_log  /usr/local/nginx/logs/error.log  notice;
error_log  /usr/local/nginx/logs/error.log  info;
pid        /usr/local/nginx/logs/nginx.pid;

# 工作模式及连接数上线
events 
{
    use epoll;            #epoll是多路复用IO(I/O Multiplexing)中的一种方式,但是仅用于linux2.6以上内核,可以大大提高nginx的性能

    worker_connections  1024;    #单个后台worker process进程的最大并发链接数
}

#设定http服务器,利用它的反向代理功能提供负载均衡支持
http 
{
    include       mime.types;
    default_type  application/octet-stream;

    #设定请求缓冲
    server_names_hash_bucket_size  128;
    client_header_buffer_size   32K;
    large_client_header_buffers  4 32k;
    # client_max_body_size   8m;
    
    #sendfile 指令指定 nginx 是否调用 sendfile 函数（zero copy 方式）来输出文件，对于普通应用，
    #必须设为 on,如果用来进行下载等应用磁盘IO重负载应用，可设置为 off，以平衡磁盘与网络I/O处理速度，降低系统的uptime.
    sendfile        on;
    tcp_nopush     on;
    tcp_nodelay    on;

    #连接超时时间
    keepalive_timeout  65;

    #开启gzip压缩，降低传输流量
    gzip  on;
    gzip_min_length    1k;
    gzip_buffers    4 16k;
    gzip_http_version  1.1;
    gzip_comp_level  2;
    gzip_types  text/plain application/x-javascript text/css  application/xml;
    gzip_vary on;

    #添加后端应用服务器列表，真实应用服务器都放在这
    upstream appserver_pool 
    {
       #server 后端服务器地址:端口号 weight表示权值，权值越大，被分配的几率越大;
　　　　server 192.168.0.223:8080 weight=4 max_fails=2 fail_timeout=30s;
    　 server 192.168.0.224:8080 weight=4 max_fails=2 fail_timeout=30s;
    }

    server 
    {
        listen       80;        #监听端口    
        server_name  localhost;
    
    #默认请求设置
    location / {
        proxy_pass http://appserver_pool;    #转向后端应用服务器处理
    }
    
    #所有的jsp页面均由后端应用服务器处理
    location ~ \.(jsp|jspx|dp)?$
    {
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_pass http://appserver_pool;    #转向后端应用服务器处理
    }
    
    #所有的静态文件直接读取不经过后端,nginx自己处理
    location ~ .*\.(htm|html|gif|jpg|jpeg|png|bmp|swf|ioc|rar|zip|txt|flv|mid|doc|ppt|pdf|xls|mp3|wma)$ 
    { 
        expires  30d;
    }
    location ~ .*\.(js|css)?$
    {
       expires  1h;
    }

    #定义错误提示页面
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }

    }
}
```

**重点配置项为“upstream appserver_pool ”和“proxy_pass”**，上例中采用了权重（weight）的策略，各种分配策略分别说明如下：

##### **轮询**

最基本的配置方法，它是upstream模块默认的负载均衡默认策略。每个请求会按时间顺序逐一分配到不同的后端服务器。

```
    upstream appserver_pool 
    {
       #server 后端服务器地址:端口号 参数=参数值 ...
　　　　server 192.168.0.223:8080 max_fails=2 fail_timeout=30s;
    　 server 192.168.0.224:8080 max_fails=2 fail_timeout=30s;
    }
```

有如下参数：

| fail_timeout | 与max_fails结合使用。                                        |
| ------------ | ------------------------------------------------------------ |
| max_fails    | 设置在fail_timeout参数设置的时间内最大失败次数，如果在这个时间内，所有针对该服务器的请求都失败了，那么认为该服务器会被认为是停机了， |
| fail_time    | 服务器会被认为停机的时间长度,默认为10s。                     |
| backup       | 标记该服务器为备用服务器。当主服务器停止时，请求会被发送到它这里。 |
| down         | 标记服务器永久停机了。                                       |

 注意：

- 在轮询中，如果服务器down掉了，会自动剔除该服务器。
- 缺省配置就是轮询策略。
- 此策略适合服务器配置相当，无状态且短平快的服务使用。

##### 权重

权重方式，在轮询策略的基础上增加weight参数指定轮询的几率。weight参数用于指定轮询几率，默认值为1；weight的数值与访问比率成正比。比如下例中Tomcat 7.0被访问的几率为其他服务器的两倍。

```
#动态服务器组
upstream dynamic_zuoyu {
  server localhost:8080  weight=2; #tomcat 7.0
  server localhost:8081; #tomcat 8.0
  server localhost:8082  backup; #tomcat 8.5
  server localhost:8083  max_fails=3 fail_timeout=20s; #tomcat 9.0
}
```

注意：

- 权重越高分配到需要处理的请求越多。
- 此策略可以与least_conn和ip_hash结合使用。
- 此策略比较适合服务器的硬件配置差别比较大的情况。

##### 客户端IP分配

指定负载均衡器按照基于客户端IP的分配方式，这个方法确保了相同的客户端的请求一直发送到相同的服务器，以保证session会话。这样每个访客都固定访问一个后端服务器，可以解决session不能跨服务器的问题。

```
#动态服务器组
  upstream dynamic_zuoyu {
    ip_hash;  #保证每个访客固定访问一个后端服务器
    server localhost:8080  weight=2; #tomcat 7.0
    server localhost:8081; #tomcat 8.0
    server localhost:8082; #tomcat 8.5
    server localhost:8083  max_fails=3 fail_timeout=20s; #tomcat 9.0
  }
```

注意：

- 在nginx版本1.3.1之前，不能在ip_hash中使用权重（weight）。
- ip_hash不能与backup同时使用。
- 此策略适合有状态服务，比如session。
- 当有服务器需要剔除，必须手动down掉。

##### 连接数较少优先

把请求转发给连接数较少的后端服务器。轮询算法是把请求平均的转发给各个后端，使它们的负载大致相同；但是，有些请求占用的时间很长，会导致其所在的后端负载较高。这种情况下，least_conn这种方式就可以达到更好的负载均衡效果。

```
#动态服务器组
upstream dynamic_zuoyu {
  least_conn;  #把请求转发给连接数较少的后端服务器
  server localhost:8080  weight=2; #tomcat 7.0
  server localhost:8081; #tomcat 8.0
  server localhost:8082 backup; #tomcat 8.5
  server localhost:8083  max_fails=3 fail_timeout=20s; #tomcat 9.0
}
```

注意：

- 此负载均衡策略适合请求处理时间长短不一造成服务器过载的情况。

##### 第三方策略

第三方的负载均衡策略的实现需要安装第三方插件。

**①fair**

按照服务器端的响应时间来分配请求，响应时间短的优先分配。

```
#动态服务器组
upstream dynamic_zuoyu {
  server localhost:8080; #tomcat 7.0
  server localhost:8081; #tomcat 8.0
  server localhost:8082; #tomcat 8.5
  server localhost:8083; #tomcat 9.0
  fair;  #实现响应时间短的优先分配
}
```

**②url_hash**

按访问url的hash结果来分配请求，使每个url定向到同一个后端服务器，要配合缓存命中来使用。同一个资源多次请求，可能会到达不同的服务器上，导致不必要的多次下载，缓存命中率不高，以及一些资源时间的浪费。而使用url_hash，可以使得同一个url（也就是同一个资源请求）会到达同一台服务器，一旦缓存住了资源，再此收到请求，就可以从缓存中读取。

```
#动态服务器组
upstream dynamic_zuoyu {
  hash $request_uri;  #实现每个url定向到同一个后端服务器
  server localhost:8080; #tomcat 7.0
  server localhost:8081; #tomcat 8.0
  server localhost:8082; #tomcat 8.5
  server localhost:8083; #tomcat 9.0
}
```

#### keepalived实现nginx高可用(HA)

keepalived主要起到两个作用：实现VIP到本地ip的映射； 以及检测nginx状态。

master上的keepalived.conf内容如下：

```
global_defs {
    notification_email {
        997914490@qq.com
    }
    notification_email_from sns-lvs@gmail.com
    smtp_server smtp.hysec.com
    smtp_connection_timeout 30
    router_id nginx_master        # 设置nginx master的id，在一个网络应该是唯一的
}
vrrp_script chk_http_port {
    script "/usr/local/src/check_nginx_pid.sh"    #最后手动执行下此脚本，以确保此脚本能够正常执行
    interval 2                          #（检测脚本执行的间隔，单位是秒）
    weight 2
}
vrrp_instance VI_1 {
    state MASTER            # 指定keepalived的角色，MASTER为主，BACKUP为备
    interface eth0            # 当前进行vrrp通讯的网络接口卡(当前centos的网卡)
    virtual_router_id 66        # 虚拟路由编号，主从要一直
    priority 100            # 优先级，数值越大，获取处理请求的优先级越高
    advert_int 1            # 检查间隔，默认为1s(vrrp组播周期秒数)
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    track_script {
    chk_http_port            #（调用检测脚本）
    }
    virtual_ipaddress {
        192.168.0.200            # 定义虚拟ip(VIP)，可多设，每行一个
    }
}
```

backup上的keepalived.conf内容如下：

```
global_defs {
    notification_email {
        997914490@qq.com
    }
    notification_email_from sns-lvs@gmail.com
    smtp_server smtp.hysec.com
    smtp_connection_timeout 30
    router_id nginx_backup              # 设置nginx backup的id，在一个网络应该是唯一的
}
vrrp_script chk_http_port {
    script "/usr/local/src/check_nginx_pid.sh"
    interval 2                          #（检测脚本执行的间隔）
    weight 2
}
vrrp_instance VI_1 {
    state BACKUP                        # 指定keepalived的角色，MASTER为主，BACKUP为备
    interface eth0                      # 当前进行vrrp通讯的网络接口卡(当前centos的网卡)
    virtual_router_id 66                # 虚拟路由编号，主从要一直
    priority 99                         # 优先级，数值越大，获取处理请求的优先级越高
    advert_int 1                        # 检查间隔，默认为1s(vrrp组播周期秒数)
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    track_script {
        chk_http_port                   #（调用检测脚本）
    }
    virtual_ipaddress {
        192.168.0.200                   # 定义虚拟ip(VIP)，可多设，每行一个
    }
}
```

nginx检测脚本check_nginx_pid.sh内容如下：

```
#!/bin/bash
A=`ps -C nginx --no-header |wc -l`        
if [ $A -eq 0 ];then                            
      /usr/local/nginx/sbin/nginx                #重启nginx
      if [ `ps -C nginx --no-header |wc -l` -eq 0 ];then    #nginx重启失败，则停掉keepalived服务，进行VIP转移
              killall keepalived                    
      fi
fi
```
