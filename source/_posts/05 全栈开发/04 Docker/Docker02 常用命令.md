---
title: Docker02 常用命令
top: false
date: 2018-10-10 16:48:45
updated: 2018-10-10 16:48:45
tags:
- Docker
categories: Docker
---

Docker 命令大全。

<!-- more -->

[Docker 命令大全](http://www.runoob.com/docker/docker-command-manual.html)

```BASH
# 列出所有在运行的容器信息
docker ps

# 列出本机的所有image文件。
docker image ls

# 删除image文件
docker image rm [imageName]

# 抓取image文件
# libraryName：image文件所在的组，默认值是library，Docker官方提供的image文件，都放在library组里
# imageName：image文件的名字
docker image pull [libraryName]/[imageName]

# 创建image文件
# -t参数用来指定image文件的名字，后面还可以用冒号指定标签
# 最后的那个点表示Dockerfile文件所在的路径，当前路径是一个点。
docker image build -t [imageName]:[tag] .

# 运行image文件，生成容器实例
# -p：容器的端口localPort映射到本机端口containerPort。
# -it：容器的Shell映射到当前的Shell，在本机窗口输入的命令，就会传入容器。
# imageName：image文件的名字（如果有标签，还需提供标签imageTag，默认是latest标签）。
# inner cmdh：容器启动以后，内部第一个执行的命令。例如启动Bash：/bin/bash。
# --rm：容器终止运行后自动删除容器文件
docker container run -p [localPort]:[containerPort] -it [imageName]:[imageTag] [inner cmd] --rm

# 列出本机正在运行的容器
docker container ls

# 列出本机所有容器，包括终止运行的容器
docker container ls --all

# 重复使用容器，启动已经生成、已经停止运行的容器文件
docker container start [containerID]

# 查看docker容器的输出，即容器里面Shell的标准输出
docker container logs [containerID]

# 进入一个正在运行的docker容器
docker container exec -it [containerID] /bin/bash

# 从正在运行的Docker容器里面，将文件拷贝到本机。
docker container cp [containID]:[/path/to/file] .

# 手动终止容器运行（强制）
docker container kill [containID]

# 手动终止容器运行（自动）
docker container stop [containID]

# 删除容器
docker container rm [containerID]

# 退出容器（ctrl+d）
exit

# 为本地的image标注用户名和版本。
docker image tag [imageName] [username]/[repository]:[tag]

# 发布image文件
docker image push [username]/[repository]:[tag]
```

