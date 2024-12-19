# Docker Notes(记录中)

[toc]

> Docker中一些重要的基础性概念，非常欢迎大家一起补充。

## 基础

### Docker是什么

概念： **`Docker`属于Linux容器的一种封装，提供简单易用的容器使用接口**。

`Docker`将应用程序与该程序的依赖，打包在一个文件中。运行这个文件，就会生成一个虚拟容器。程序在这个虚拟容器中运行，就好像在真实的物理机上运行一样。

### Docker的用途

1. **提供一次性的环境**： 例如：本地测试他人的软件，持续集成的时候提供单元测试和构建的环境。
2. **提供弹性的云服务**：由于`Docker`容器可以随时开关，非常时候动态扩容和缩容。
3. **组建微服务架构**：通过多个容器，一台机器可以跑多个服务，因此在本机就可以模拟出微服务架构。

### 容器(Container)和镜像(Image)

**镜像**：Docker镜像可以理解为包含应用程序及其相关依赖的一个基础文件系统，在Docker容器的启动过程中，它以只读的方式用于**创建容器的运行环境。**

**容器**： 容器的实质就是一个**进程**， 但是容器进程运行于属于自己的独立命名空间中。因此容器可以拥有自己的 `root` 文件系统、自己的网络配置、自己的进程空间，甚至自己的用户 ID 空间。容器内的进程是运行在一个隔离的环境里，使用起来，就好像是在一个独立于宿主的系统下操作一样。

所以镜像指的是包含运行环境的只读文件系统，而容器可以说是在镜像环境下运行的程序。

**关系**： 镜像和容器的关系，就像是面向对象程序设计中的类和实例的关系。镜像是静态的类，而容器是镜像运行时的实体。

### 容器基本操作

首先我们需要了解容器存在的状态：

- **Created**： 已经创建完毕，相关资源已经准备就绪，但是容器中的程序还处于未运行的状态。
- **Running**： 容器（容器中的应用）正在运行。
- **Paused**： 容器中的所有程序处于**暂停**的状态。
- **Stopped**： 容器中的程序处理停止状态。
- **Created**：容器已经删除，相关占用的资源及存储，在Docker中的管理信息都已经被释放和删除。

操作命令

<small>备注： 下面命令中的`name`指的是容器名称， `image`指的是镜像名称。</small>

- 创建： `docker create --name name image`
- 启动： `docker start name`
- 创建+启动： `docker run --name name --detach image`   --detach: 启动后将程序和控制台分离，使之进入后台模式。
- 查看：`docker ps --all`
- 停止：`docker stop name` (沙盒系统还存在，修改的内容也都保存)
- 删除： `docker rm name --force`
- 进入容器： `docker exec -it name bash`   其中 `-i` ( `--interactive` ) 表示保持我们的输入流。 `-t` ( `--tty` ) 表示启用一个伪终端，形成我们与 bash 的交互 。
- 衔接容器： `docker attach name` 这个命令最直观的效果可以理解为我们将容器中的主程序转为了“前台”运行 ( 与 `docker run` 中的 `-d` 选项有相反的意思 )。

以启动一个`nginx`容器服务作为示例

``` shell
docker run -d -rm -p 8800:80 -v /home/nginx/html:/usr/share/nginx/html --name nginx nginx

-d: 在后台云运行
-p 映射端口号
-rm 容器停止后自动自动删除容器文件
--name 容器名称
--volume/-v /home/nginx/html :/usr/share/nginx/html
```

### 常用命令一览

下面图中列出来的是一些`docker`的高频使用命令。

![Docker命令](./images/docker.jpg 'Docker命令')

### 容器网络

> 容器网络实质上也是由 Docker 为应用程序所创造的虚拟环境的一部分，它能让应用从宿主机操作系统的网络环境中独立出来，形成容器自有的**网络设备、IP 协议栈、端口套接字、IP 路由表、防火墙等等与网络相关的模块**。

核心概念

- **沙盒 ( Sandbox )**: 提供容器网络栈，主要是隔离了容器网络和宿主机网络，形成完全独立的容器网络环境。
- **网络 ( Network )**: `Docker`内部的虚拟子网，与宿主机网络存在隔离关系，主要目的是形成容器间的安全通讯环境。
- **端点 ( Endpoint )**: 主要目的是形成一个可以控制的突破封闭网络环境的出入口。当容器的端点与网络的断点匹配之后，就如同在两者之间搭建了桥梁，就能进行数据传输了。

### 数据管理

数据是应用程序的重要产出，数据可以说是最重要的资产。但是在`docker`中容器运行的文件系统处在于沙盒之中，与外界是隔离的，所以必须是有合理的方式与外界进行数据交换。

**数据卷**

数据卷是一个可供一个或者多个容器使用的特殊目录，它绕过UFS，可以提供很多有用的特征。数据卷可以在容器之中共享和重用，对数据卷的修改立马生效，对数据卷的更新，不会影响镜像，数据卷是持久的，即使容器被删除。

**挂载方式**

- **Bind Mount**： 直接把宿主操作系统中的目录和文件挂载到容器中的文件系统中，通过指定容器外的路径和容器内的路径，就形成了挂载映射关系，在容器内外对文件的读写都是相互可见的。
- **Volume**：从宿主操作系统中挂载目录到容器内，只不过目录是由`Docker`进行管理，我们只需指定容器内的目录，不需要关心具体挂载到宿主操作系统中的哪里。
- **Tmpfs Mount**：支持挂载系统内存中的一部分到容器的文件系统里，不过由于内存和容器的特征。这个存储并不会持久，其中的内容会随着容器的停止而消失。一般用于挂载临时文件目录。

下面是`nginx`挂载目录的示例：

```bash
# 示例命令
docker container run 
-d 
-p 8800:80 
--rm 
--name nginx
-v /home/nginx/html:/usr/share/nginx/html 
nginx

# 命令解释
-d：在后台运行
-p ：容器的80端口映射到127.0.0.2:8080
--rm：容器停止运行后，自动删除容器文件
--name：容器的名字为nginx
-v：--volume 的缩写形式
```

### `Dockerfile`

这是`docker`特有的镜像构建定义文件，通过了解它就能真正体验一种秒级镜像构建的乐趣。

`Dockerfile`是`Docker`中用于定义镜像自动化构建的流程的配置文件，文件中包含了构建镜像过程中需要执行的命令和其他的操作。

**优点**

- 体积远小于镜像包，更容易进行快速迁移和部署。
- 环境构建流程在文件中展示，能够直观的看见镜像构建的顺序和逻辑。
- 能够更轻松的实现自动部署等自动化流程。
- 修改环境仅仅修改文件，要比从新提交镜像要简单的多。

**指令**

- **基础指令**：用于定义新镜像的基础和性质。
- **控制指令**：指导镜像构建的核心部分，用于描述镜像在构建过程中需要执行的命令。
- **引入指令**：用于将外部文件直接引入到构建镜像内部。
- **执行指令**：能够为基于镜像所构建的的容器，指定在启动时需要执行的脚本或命令。
- **配置指令**：对镜像以及基于镜像所创建的容器，可以通过配置指令对其网络用户等内容进行配置。

**常见指令**

- **FROM**：指定一个基础镜像，接下来的指令都是基于这个镜像所展开的。

  ```dockerfile
  FROM <image> [AS <name>]
  FROM <image>[:<tag>] [AS <name>]

FROM <image>[@<digest>] [AS <name>]

  ```
  
  `Dockerfile`第一条指令必须是`FROM`指令，因为一切构建过程都基于基础镜像。
  
  `FROM`可以多次出现，当 FROM 第二次或者之后出现时，表示在此刻构建时，要将当前指出镜像的内容合并到此刻构建镜像的内容里。
  
- **RUN**：指令本质上来说只是一种引导作用，`RUN`指令用于向控制台发送命令的指令。

  ```dockerfile
  RUN <command>
  RUN ["executable", "param1", "param2"]
  ```

- **ENTRYPOINT/CMD**：用来启动容器在运行时会根据镜像所定义的一条命令来启动容器中进程号为1的进程。

  ```dockerfile
  ENTRYPOINT ["executable", "param1", "param2"]
  ENTRYPOINT command param1 param2
  
  CMD ["executable","param1","param2"]
  CMD ["param1","param2"]
  CMD command param1 param2
  ```

  这两个命令用法类似，都可以为空。当两个命令同时给出的时候，`CMD`中的内容会作为`ENTRYPOINT`中的参数去执行`ENTRYPOINT`指令。

  ****

  这两个指令的区别：

  **ENTRYPOINT**： 主要对容器进行一些初始化，常见的使用方式是使用外部引入的一个脚本文件。

  **CMD**：用于定义真正容器中的主程序的启动命令。

  例如`Redis`中对于中对于这两个命令的使用

  ```dockerfile
  ## ......
  COPY docker-entrypoint.sh /usr/local/bin/
  ENTRYPOINT ["docker-entrypoint.sh"]
  ## ......
  CMD ["redis-server"]
  ```

- **EXPOSE**：为镜像指定要暴露的端口。

  ```dockerfile
  EXPOSE <port> [<port>/<protocol>...]
  ```

- **VOLUME**：用来定义基于此镜像的的容器所自动建立的容器卷。

  ```dockerfile
  VOLUME ["/data"]
  ```

- **COPY/ADD**：从宿主机器中将一些配置文件，程序代码，执行脚本等文件内容**直接导入到镜像内的文件系统中**。

  ```dockerfile
  COPY [--chown=<user>:<group>] <src>... <dest>
  # ADD 能够支持使用网络端的 URL 地址作为 src 源，并且在源文件被识别为压缩包时，自动进行解压，而 COPY 没有这两个能力
  ADD [--chown=<user>:<group>] <src>... <dest>
  
  #COPY 与 ADD 指令的定义方式完全一样，需要注意的仅是当我们的目录中存在空格时，可以使用下面的格式避免空格产生歧义。
  COPY [--chown=<user>:<group>] ["<src>",... "<dest>"]
  ADD [--chown=<user>:<group>] ["<src>",... "<dest>"]
  ```

- **WORKDIR**：使用 `WORKDIR` 指令可以来指定工作目录（或者称为当前目录），以后各层的当前目录就被改为指定的目录，如该目录不存在，`WORKDIR` 会帮你建立目录。

**构建镜像**

在编写完`dockerfile`之后，就可以对我们的镜像进行构建了。 构建命令为 `docker build <src>`

`docker build` 可以接收一个参数，需要特别注意的是，**这个参数为一个目录路径 ( 本地路径或 URL 路径 )，而并非 Dockerfile 文件的路径**。在 `docker build` 里，这个我们给出的目录会作为构建的环境目录，我们很多的操作都是基于这个目录进行的。

```shell
sudo docker build -t webapp:latest -f ./webapp/a.Dockerfile ./webapp
```

**-t webapp:latest** 指定镜像名称。

**-f ./webapp/a.Dockerfile** 指定dockerfile地址，如果dockerfile不在./webapp中，那么你需要这个选项。

### 高频容器

- `Nginx`

  ```dockerfile
  docker container run 
  -d 
  -p 8800:80 
  --rm 
  --name nginx
  -v /home/nginx/html:/usr/share/nginx/html 
  nginx
  ```

- `MySQL`

  ```dockerfile
  docker container run 
  -d
  -p 3306:3306 
  -v /home/mysql/conf.d:/etc/mysql/conf.d # 配置文件
  -v /home/mysql/data:/var/lib/mysql # 数据文件
  -e MYSQL_ROOT_PASSWORD=123456
  --rm
  --name mysql 
  mysql:5.7.21
  ```

## 基础进阶

基础弄完了总要来点高级的

### Docker Compose

`Docker Compose` 是 Docker 官方编排（Orchestration）项目之一，负责快速的部署分布式应用。

先了解一下专业术语：

- 服务（`service`）：一个应用容器，实际上可以运行多个相同镜像的实例。
- 项目（`project`）: 由一组关联的应用容器组成的一个完整业务单元。

一个项目可以由多个服务（容器）关联而成，`Compose`是面向项目进行管理。

**使用**：

使用基本上就分为三步

1. 准备好容器。如果需要的话就编写容器所需镜像的`Dockerfile`。

2. 编写用于配置容器的`docker-compose.yml`。例如：

   ```yaml
   # 版本标识
   version: '3'
   # 服务内容器的细节
   services:
    # 一个service是配置的最小单元
     webapp:
       build: ./image/webapp
       ports:
         - "5000:5000"
       volumes:
         - ./code:/code
         - logvolume:/var/log
       links:
         - mysql
         - redis
   
     redis:
       image: redis:3.2
     
     mysql:
       image: mysql:5.7
       environment:
         - MYSQL_ROOT_PASSWORD=my-secret-pw
   
   volumes:
     logvolume: {}
   ```

3. 启动项目/应用。

## 最佳实践

```shell
# 1. 清理容器，镜像和缓存
docker system prune -a --volumes
```

## 总结

话不多说，`Docker`是真香。
