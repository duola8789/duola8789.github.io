---
title: Docker04 Docker最佳实践
top: false
date: 2018-10-18 15:48:45
updated: 2018-10-18 15:48:45
tags:
- Docker
categories: Docker
---

Docker最佳实践。

<!-- more -->

## 使用`.dockerignore`

在大多数情况下，最好将每个Dockerfile放在一个空目录中。然后，仅添加构建Dockerfile所需的文件。要增加构建的性能，可以通过将`.dockerignore`文件添加到该目录来排除文件和目录。

## 避免安装不必要的安装包

应该尽量减少容器的复杂性，依赖性，文件的大小，构建的次数，所以应该尽量避免安装不必要的安装包。

## 每个容器应该只有一个进程

将应用程序解耦到多个容器中可以更轻松地水平扩展和重新使用容器。

例如，Web应用程序可能由三个独立的容器组成，每个容器具有自己独特的映像，以解耦的方式管理应用、数据库、数据缓存。

如果容器之间有依赖关系，应该使用Docker Network解决容器之间的通信。

## 构建缓存

在构建image的过程中，Docker将按照指定的顺序逐步执行你的Dockerfile中的指令。随着每条指令的检查，Docker将在其缓存中查找可重用的现有image，而不是创建一个新的（重复）image。如果你不想使用缓存，可以在`docker build`命令中使用`–no-cache=true`选项。

## FROM 

只要有可能，使用当前的官方存储库作为你的image的基础。建议使用Debian镜像，因为它是非常严格的控制，并保持最小（目前在150 mb），而仍然是一个完整的分布。

## RUN

可以在多行上分隔长度或复杂的RUN语句，并以反斜杠分隔。

请务必将`RUN apt-get update`与`apt-get install`组合在同一个RUN语句中。例如：

```
RUN apt-get update && apt-get install -y \
  package-bar \
  package-baz \
  package-foo
```
如果在RUN语句中单独使用`apt-get update`会导致缓存问题。例如：

```
FROM ubuntu:14.04
RUN apt-get update
RUN apt-get install -y curl
```

构建image后，所有内容都在Docker缓存中。如果以后通过添加额外的程序：

```
FROM ubuntu:14.04
RUN apt-get update
RUN apt-get install -y curl nginx
```

Docker将认为第一个RUN是相同的命令，会重复使用先前步骤的缓存，因此第一个RUN并不会实际执行，因此有可能得到的curl和nginx不是最新版本的。

将`RUN apt-get update`与`apt-get install`组合在同一个RUN语句中确保Dockerfile安装最新的软件包版本，无需进一步的编码或手动干预。这种技术被称为“缓存破解”。

还可以通过指定包版本来实现缓存清除。这被称为版本固定，例如：

```Dockerfile
RUN apt-get update && apt-get install -y \
  package-bar \
  package-baz \
  package-foo=1.3.*
```

## CMD

CMD指令应用于运行image中包含的软件以及任何参数。 CMD几乎总是以`CMD [“executable”, “param1”, “param2”…]`的形式使用。 

因此，如果image用于服务，例如Apache和Rails，则可以运行类似于`CMD [“apache2”,”-DFOREGROUND”]`的内容。 实际上，这种形式的指令是推荐用于任何基于服务的image。

## ADD or COPY

虽然ADD和COPY在功能上是相似的，但一般来说，COPY是首选的。这是因为它比ADD更透明。

COPY只支持将本地文件复制到容器中，而ADD具有一些不是很明显的功能（如本地的tar提取和远程URL支持）。因此，ADD的最佳用途是将本地tar文件自动提取到镜像中，如：`ADD rootfs.tar.xz /`。

## 使用`docker exec`而不是`sshd`

需要进入容器要使用`docker exec`命令，而不要单独安装`sshd`

## 参考
- [birdben - Docker实战（三十）Dockerfile最佳实践总结](https://birdben.github.io/2017/05/07/Docker/Docker%E5%AE%9E%E6%88%98%EF%BC%88%E4%B8%89%E5%8D%81%EF%BC%89Dockerfile%E6%9C%80%E4%BD%B3%E5%AE%9E%E8%B7%B5%E6%80%BB%E7%BB%93/)
