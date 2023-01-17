---
title: Docker01 入门
top: false
date: 2019-10-10 09:32:00
updated: 2019-10-10 09:32:00
tags:
- docker
categories: Docker
---

Docker快速入门学习笔记。

<!-- more -->

## Docker 是什么

Docker属于Linux 容器的一种封装，提供简单易用的容器使用接口。它是目前最流行的 Linux 容器解决方案。

> Linux容器不是模拟一个完整的操作系统，而是对进程进行隔离。或者说，在正常进程的外面套了一个保护层。对于容器里面的进程来说，它接触到的各种资源都是虚拟的，从而实现与底层系统的隔离。

Docker将应用程序与该程序的依赖，打包在一个文件里面。运行这个文件，就会生成一个虚拟容器。程序在这个虚拟容器里运行，就好像在真实的物理机上运行一样。有了Docker，就不用担心环境问题。

## Docker的用途

Docker 的主要用途，目前有三大类。

1. 提供一次性的环境。比如，本地测试他人的软件、持续集成的时候提供单元测试和构建的环境。
2. 提供弹性的云服务。因为Docker容器可以随开随关，很适合动态扩容和缩容。
3. 组建微服务架构。通过多个容器，一台机器可以跑多个服务，因此在本机就可以模拟出微服务架构。

## Docker的安装

### windows

Docker提供了windows的安装版本[Docker for Windows](https://docs.docker.com/docker-for-windows/)，但是针对的版本是64位win10企业版、教育版：

> The current version of Docker for Windows runs on 64bit Windows 10 Pro, Enterprise and Education (1607 Anniversary Update, Build 14393 or later).

我的公司电脑是win7版本的，所以不能使用Docker for windows。可以使用Docker Toolbox来安装Docker。[下载地址](https://download.docker.com/win/stable/DockerToolbox.exe)。

在使用Docker Toolbox之前，必须确定电脑CPU支持VT-X，可以通过CPU-Z来确定。另外也必须到BIOS中确定VT-X开启。

更简单的方法是使用CPU-V来观察电脑是否支持并且开启了VT-X，如果不支持那就没办法了，如果支持但是未开启，则需要到BIOS中开启。

![c553cea4-8651-495c-aef3-daf1d29981ff.png](http://image.oldzhou.cn/c553cea4-8651-495c-aef3-daf1d29981ff.png)

我的电脑支持VT-X，但是未开启，所以需要到BIOS中开启。我的电脑主板是HP的，开启时进入到主板设置中的`高级选项`，选择`系统选项`,然后勾选`Virtualiztion Technology`保存即可

重启后在使用CPU-V查看，VT-X状态改为了开启


不同主板的开发方法略有不同，具体的开启方法可以参考[这篇文章](http://mumu.163.com/include/16v1/2016/06/27/21967_625825.html)。

开启成功后进行安装，一路点击下一步就可以了。安装完成后在桌面上会出现几个快捷方式，点击`Docker Quickstart Terminal`启动docker。

### macOS

macOS安装Docker要求系统最低为Sierra 10.12。

可以使用Homebrew安装：

```BASH
$ brew cask install docker
```

也可以手动下载安装，具体安装方式可以参考[官方文档](https://docs.docker.com/docker-for-mac/install/)。


## Image文件（镜像文件）

Docker把应用程序及其依赖，打包在image文件里面。只有通过这个文件，才能生成Docker容器。image文件可以看作是容器的模板。Docker 根据image文件生成容器的实例。同一个image文件，可以生成多个同时运行的容器实例。

image是二进制文件。实际开发中，一个image文件往往通过继承另一个image文件，加上一些个性化设置而生成。举例来说，你可以在Ubuntu的image基础上，往里面加入Apache服务器，形成你的image。

## 容器文件

image文件生成的容器实例，本身也是一个文件，称为容器文件。也就是说，一旦容器生成，就会同时存在两个文件：image文件和容器文件。而且关闭容器并不会删除容器文件，只是容器停止运行而已。


```BASH
# 列出本机正在运行的容器
$ docker container ls

# 列出本机所有容器，包括终止运行的容器
$ docker container ls --all
```

上面命令的输出结果之中，包括容器的ID。很多地方都需要提供这个ID，比如上一节终止容器运行的`docker container kill`命令。


终止运行的容器文件，依然会占据硬盘空间，可以使用`docker container rm`命令删除。

```BASH
$ docker container rm [containerID]
```

## Hello world

首先将image文件从仓库抓取到本地：

```BASH
$ docker image pull library/hello-world
```

上面代码中，`docker image pull`是抓取image文件的命令。`library/hello-world`是image文件在仓库里面的位置，其中`library`是image文件所在的组，`hello-world`是image文件的名字。

由于Docker官方提供的image文件，都放在library组里面，所以它的是默认组，可以省略。因此，上面的命令可以写成下面这样。

```BASH
$ docker image pull hello-world
```
运行这个image文件。

```BASH
$ docker container run hello-world
```
`docker container run`命令会从image文件，生成一个正在运行的容器实例。

注意，`docker container run`命令具有自动抓取image文件的功能。如果发现本地没有指定的image文件，就会从仓库自动抓取。因此，前面的`docker image pull`命令并不是必需的步骤。

如果运行成功，你会在屏幕上读到下面的输出。

```BASH
$ docker container run hello-world

$ Hello from Docker!
$ This message shows that your installation appears to be working correctly.

... ...
```
输出这段提示以后，`hello world`就会停止运行，容器自动终止。

对于那些不会自动终止的容器，必须使用docker container kill 命令手动终止。

```BASH
$ docker container kill [containID]
```

## 制作自己的 Docker 容器

### （1）编写Dockerfile文件

要制作自己的image文件，需要用到Dockerfile文件，它是一个文本文件，用来配置image。Docker根据该文件生成二进制的image文件。

首先创建一个`.dockerignore`文件，排除相关路径，不要打包进入image文件：

```TEXT
.git
node_modules
npm-debug.log
```

然后在项目的根目录下，新建文本文件`Dockerfile`，写入下面内容：

```Dockerfile
FROM node:8.4
COPY . /app
WORKDIR /app
RUN ["npm", "install"]
EXPOSE 3000
```
上面代码一共五行，含义如下。

- `FROM node:8.4`：该image文件继承官方的Node Image，冒号表示标签，这里标签是`8.4`，即8.4版本的node。
- `COPY . /app`：将当前目录下的所有文件（除了`.dockerignore`排除的路径），都拷贝进入image文件的`/app`目录。
- `WORKDIR /app`：指定接下来的工作路径为`/app`。
- `RUN ["npm", "install"]`：在`/app`目录下，运行`npm install`命令安装依赖。注意，安装后所有的依赖，都将打包进入image文件。
- `EXPOSE 3000`：将容器3000端口暴露出来， 允许外部连接这个端口。

### （2）创建image文件

有了Dockerfile文件后，就可以使用`docker image build`命令创建image文件

```BASH
docker image build -t koa-demo .
# 或者
docker image build -t koa-demo:0.0.1 .
```

上面代码中，`-t`参数用来指定image文件的名字，后面还可以用冒号指定标签。如果不指定，默认的标签就是`latest`。最后的那个点表示Dockerfile文件所在的路径，上例是当前路径，所以是一个点。

如果运行成功，就可以看到新生成的image文件`koa-demo`了。

### （3）生成容器实例

`docker container run`命令会从image文件生成容器。

```BASH
$ docker container run -p 8000:3000 -it koa-demo /bin/bash
# 或者
$ docker container run -p 8000:3000 -it koa-demo:0.0.1 /bin/bash
```

上面命令的各个参数含义如下：

- `-p`参数：容器的3000端口映射到本机的8000端口。
- `it`参数：容器的Shell映射到当前的Shell，然后你在本机窗口输入的命令，就会传入容器。
- `koa-demo:0.0.1`：image文件的名字（如果有标签，还需要提供标签，默认是latest标签）。
- `/bin/bash`：容器启动以后，内部第一个执行的命令。这里是启动Bash，保证用户可以使用 Shell。


如果一切正常，运行上面的命令以后，就会返回一个命令行提示符。

```BASH
$ root@69f4c605f57b:/app
```

这表示你已经在容器里面了，返回的提示符就是容器内部的Shell提示符。执行下面的命令。

```BASH
$ root@69f4c605f57b:/app node demos/01.js
```

这时，Koa框架已经运行起来了。

这个例子中，Node进程运行在Docker容器的虚拟环境里面，进程接触到的文件系统和网络接口都是虚拟的，与本机的文件系统和网络接口是隔离的，因此需要定义容器与物理机的端口映射。

### （4）退出容器

在容器的命令行，按下`Ctrl + c`停止Node进程，然后按下`Ctrl + d`（或者输入`exit`）退出容器。此外，也可以用`docker container kill`终止容器运行。

```BASH
# 在本机的另一个终端窗口，查出容器的 ID
$ docker container ls

# 停止指定的容器运行
$ docker container kill [containerID]
```

容器停止运行之后，并不会消失，用下面的命令删除容器文件。

```BASH
# 查出容器的 ID
$ docker container ls --all

# 删除指定的容器文件
$ docker container rm [containerID]
```

也可以使用`docker container run`命令的`--rm`参数，在容器终止运行后自动删除容器文件。

```BASH
$ docker container run --rm -p 8000:3000 -it koa-demo /bin/bash
```

### （5）CMD命令

容器启动以后，需要手动输入命令`node demos/01.js`。我们可以把这个命令写在Dockerfile里面，这样容器启动以后，这个命令就已经执行了，不用再手动输入了。

```Dockerfile
FROM node:8.4
COPY . /app
WORKDIR /app
RUN ["npm", "install"]
EXPOSE 3000
CMD node demos/01.js
```

`CMD node demos/01.js`，它表示容器启动后自动执行`node demos/01.js`。

`RUN`命令与`CMD命令`的区别在哪里？简单说，**`RUN`命令在image文件的构建阶段执行，执行结果都会打包进入image文件；CMD命令则是在容器启动后执行**。

另外，一个Dockerfile可以包含多个`RUN`命令，但是只能有一个`CMD`命令。

注意，指定了`CMD`命令以后，`docker container run`命令就不能附加命令了（比如前面的`/bin/bash`），否则它会覆盖CMD命令。现在，启动容器可以使用下面的命令。

```BASH
$ docker container run --rm -p 8000:3000 -it koa-demo
```

### （6）发布image文件

容器运行成功后，就确认了image文件的有效性。这是就可以考虑将iamge文件分享到网上，让其他人使用。

首先，去[hub.docker.com](https://hub.docker.com/)或[cloud.docker.com](https://cloud.docker.com/)注册一个账户。然后，用下面的命令登录。

登陆：

```BASH
$ docker login
```

接着，为本地的 image 标注用户名和版本。

```BASH
$ docker image tag [imageName] [username]/[repository]:[tag]
# 实例
$ docker image tag koa-demos:0.0.1 ruanyf/koa-demos:0.0.1
```

## 其他有用的命令

### `docker container start`

`docker container run`命令是新建容器，每运行一次，就会新建一个容器。同样的命令运行两次，就会生成两个一模一样的容器文件。

如果希望重复使用容器，就要使用`docker container start`命令，它用来启动已经生成、已经停止运行的容器文件。

```BASH
$ docker container start [containerID]
```

### `docker contarin stop`

`docker container kill`命令终止容器运行，相当于向容器里面的主进程发出SIGKILL信号。而`docker container stop`命令也是用来终止容器运行，相当于向容器里面的主进程发出SIGTERM信号，然后过一段时间再发出SIGKILL信号。

```BASH
$ docker container stop [containerID]
```

这两个信号的差别是，应用程序收到SIGTERM信号以后，可以自行进行收尾清理工作，但也可以不理会这个信号。如果收到SIGKILL信号，就会强行立即终止，那些正在进行中的操作会全部丢失。

我的理解就是，以PC进行类比，`stop`命令相当于点击关机键，PC会自动保存所有文件，妥当之后关机。而`kill`就是直接拔电源了。

### `docker container logs`

用来查看docker容器的输出，即容器里面Shell的标准输出。如果`docker run`命令运行容器的时候，没有使用`-it`参数，就要用这个命令查看输出。

```BASH
$ docker container logs [containerID]
```

### `docker container exec`

用于进入一个正在运行的docker容器。如果`docker run`命令运行容器的时候，没有使用`-it`参数，就要用这个命令进入容器。一旦进入了容器，就可以在容器的`Shell`执行命令了。

```BASH
$ docker container exec -it [containerID] /bin/bash
```

### `docker container cp`

用于从正在运行的Docker容器里面，将文件拷贝到本机。下面是拷贝到当前目录的写法。

```BASH
$ docker container cp [containID]:[/path/to/file] .
```

## 参考
- [docker docs - Get started with Docker for Windows](https://docs.docker.com/docker-for-windows/)
- [docker docs - Install Docker for Windows](https://docs.docker.com/docker-for-windows/install/#what-to-know-before-you-install)
- [mumu模拟器 - 如何设置VT](http://mumu.163.com/include/16v1/2016/06/27/21967_625825.html)
- [docker docs - Docker Toolbox overview ](https://docs.docker.com/toolbox/overview/#whats-in-the-box)
- [阮一峰的网络日志 - Docker 入门教程](http://www.ruanyifeng.com/blog/2018/02/docker-tutorial.html)
