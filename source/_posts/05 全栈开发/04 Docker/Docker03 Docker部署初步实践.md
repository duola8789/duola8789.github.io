---
title: Docker03 Docker部署初步实践
top: false
date: 2018-10-11 15:48:45
updated: 2019-10-22 11:15:59
tags:
- Docker
categories: Docker
---

一次迁移到Docker的小尝试。

<!-- more -->

## 问题描述

服务A是一个静态博客网站，由Nginx提供HTTP服务（`80`端口），代码仓库为GitA。当向GitA中提交新的文件时，会触发Gitlab的Webhook的Push Events，向另一个端口`8888`提交一个POST请求。

服务B利用NodeJS，监听了`8016`端口，当收到Webhook触发的POST请求后，会进行一系列的动作,拉取GitA中代码，清空文件夹，利用Hexo进行编译，将编译好的文件提供给服务A使用。

现在我的工作就是要将在传统服务器上的这两个服务迁移到Docker上来。由于这个博客的访问量很小，不用考虑太多优化的问题，所以只能算的上是Dokcer部署的“初步实践”。

如果保持原来的代码不做任何修改，也就是需要同时使用Nginx提供静态服务+NodeJS提供监听编译服务，有下面几种方案：

1. 方案一：构建两个镜像，手动控制端口暴露
2. 方案二：在同一个容器中，通过NPM命令同时启动两个服务
3. 方案三：构建两个容器，通过`docker compose`控制端口

如果对现有的代码进行修改，完全使用NodeJS提供静态服务和监听编译服务，那么就有方案四：完全使用NodeJS进程。

相应的代码在我的[代码仓库](https://github.com/duola8789/docker-demos)里。

## 准备工作

### 准备`.dockerignore`文件

`.dockerignore`文件和`.gitrignore`文件，也就是制作镜像时排除在外的文件

```TEXT
node_modules
.DS_Store
public
source/_posts

npm-debug.log*
yarn-debug.log*
yarn-error.log*

.idea
.vscode
*.suo
*.ntvs*
*.njsproj
*.sln
*.sw*

public
source/_posts
```

###  Win7下docker镜像的ip地址

我的工作电脑是Win7系统，使用的是Docker官方提供的ToolBox的工具，工具使用没有问题，但是遇到了一个小坑，开发完了镜像，通过`localhost`访问指定端口，无论如何也连接不上。

后来发现是IP的问题，其实Docker一启动时就告诉了我暴露出来的地址了，奈何我自己眼瞎：

![IP地址](http://image.oldzhou.cn/18-10-18/83007639.jpg)

通过`docker-machine env`这个命令也可以查看分配的IP，其中`export DOCKER_HOST`就是docker镜像分配的IP

```BASH
$ docker-machine env

export DOCKER_TLS_VERIFY="1"
export DOCKER_HOST="tcp://192.168.99.100:2376"
export DOCKER_CERT_PATH="C:\Users\zhouhao1\.docker\machine\machines\
export DOCKER_MACHINE_NAME="default"
export COMPOSE_CONVERT_WINDOWS_PATHS="true"
# Run this command to configure your shell:
# eval $("C:\Program Files\Docker Toolbox\docker-machine.exe" env)
```

所以，应该通过`192.168.99.100`加上对应的端口号去访问镜像

准备好之后开始逐个方案进行介绍。

## 方案一：构建两个镜像，手动控制端口暴露

### （1）Nginx容器

在Dockerfile中暴露出`80`端口，映射为本机IP的（`http://192.168.99.100`）的`80`端口。

同时在`global.conf`设定转发规则，将访问`api`地址的请求转发到本机IP的`8016`端口。

最后，在`nginx.conf`中设定一些基本的nginx配置项，关键点是`daemon off`，将Nginx服务设定为前台方式运行，这是因为在Docker中服务要以前台方式启动。

Docker容器默认把容器内部第一个进程，也就是`pid=1`的进程作为Docker容器是否正在运行的依据。如果此程序挂了，那么Docker便会直接退出。当Nginx在后台运行时，Nginx并不是`pid`为`1`的程序，而是正在执行的`bash`。`bash`执行完了nginx后便结束了，容器也就退出了。

同理，`forever start app.js`后，`bash`的`pid`为1，此时`bash`执行后退出，容器也就退出了。正确的使用方式应该是`forever app.js`，保证进程处于前台。

Dockerfile：

```Dockerfile
FROM nginx:latest

# 拷贝文件
COPY . /var/www/html

# 设置工作目录
WORKDIR /var/www/html

# 把nginx.conf配置文件复制到镜像中
ADD nginx.conf /etc/nginx/nginx.conf
ADD global.conf /etc/nginx/conf.d/

# 删除默认配置文件
RUN cd /etc/nginx/conf.d/; rm default.conf

EXPOSE 80

CMD ["nginx"]
```

global.conf：

```NGINX
server {
  listen 0.0.0.0:80;
  server_name _;

  root /var/www/html/public;
  index index.html index.htm;

  access_log /var/log/nginx/default_access.log;
  error_log /var/log/nginx/default_error.log;

  location /api {
    proxy_pass http://192.168.99.100:8016;
  }
}
```

nginx.conf:

```NGINX
user www-data;
worker_processes 4;
pid /run/nginx.pid;

# 将Nginx服务设定为前台方式运行
daemon off;

events {  }

http {
  sendfile on;
  tcp_nopush on;
  tcp_nodelay on;
  keepalive_timeout 65;
  types_hash_max_size 2048;
  include /etc/nginx/mime.types;
  default_type application/octet-stream;
  access_log /var/log/nginx/access.log;
  error_log /var/log/nginx/error.log;
  gzip on;
  gzip_disable "msie6";
  include /etc/nginx/conf.d/*.conf;
}
```

然后开始操作：

```BASH
# 构建镜像
docker image build -t nginx01 .

# 运行镜像
# 将 Nginx 容器的 80 端口映射为本机（http://192.168.99.100）的 80 端口
docker container run -d -p 80:80 --rm --name nginxC01 nginx01
```

运行成功后，可以对当前容器进行操作：

```BASH
# 进入某个容器的控制台
$ docker container exec -it nginxC01 /bin/bash

# 停止容器进程
$ docker container kill nginxC01
```

### （2）NodeJS容器

在Dockerfile中，暴露出`8016`端口

Dockerfile文件：

```Dockerfile
FROM node:latest

# 拷贝文件
COPY ./src /var/www/api

# 设置工作目录
WORKDIR /var/www/api

# 安装依赖
# RUN cd /var/www/api; npm install

EXPOSE 8016

CMD ["npm", "run", "start"]
```

在`packjage.json`文中定义NPM脚本：

```JSON
{
  "name": "lede-tech",
  "version": "1.0.0",
  "private": true,
  "scripts": {
    "start": "node app.js"
  }
}
```
然后同样开始构建并运行镜像：

```BASH
# 构建镜像
docker image build -t node01 .

# 运行镜像
# 将 NodeJS 容器的 8016 端口映射为本机（http://192.168.99.100）的 8016 端口
docker container run -d -p 8016:8016 --rm --name nodeC01 node01
```

如果需要去在docker中使用node，那么就没必要去安装pm2等工具了，直接使用`node`命令运行脚本，如果你怕你的容器会挂掉，可以加上`restart`等相关参数比如`docker run .... --restart=always`

### （3）总结

这种方法比较简单，通过分别构建了两个镜像来实现Nginx和NodeJS服务的共存。

优点是比较简单，并且遵循了一个容器一个进程的最佳实践，充分解耦

缺点是端口的转发实际上是在宿主外层实现的，并且需要手动控制端口暴露，手动向`global.conf`中传入宿主的IP地址，实现两个服务的连接。如果服务比较复杂的时候，手动控制的难度和可维护性的难度也会大大提高。

## 方案二：在同一个容器中，通过NPM命令同时启动两个服务

这个Dockerfile里，我们首先继承自官方的Node镜像，然后通过`RUN apt-get update`和`RUN apt-get -y -q install nginx`安装Nginx，其他的内容没有什么特殊

Dockerfile：

```Dockerfile
FROM node:latest

# 如果上个步骤已经更新软件源，这步可以忽略
RUN apt-get update

# 安装 Nginx
RUN apt-get -y -q install nginx

# 拷贝文件
COPY . /var/www/html

# 设置工作目录
WORKDIR /var/www/html

# 把nginx.conf配置文件复制到镜像中
ADD nginx.conf /etc/nginx/nginx.conf
ADD global.conf /etc/nginx/conf.d/

EXPOSE 80

# 安装依赖
# RUN npm install

CMD ["npm", "run", "start"]
```

关键点是在`package.json`的`scripts`中，我们通过`&`来连接两个命令，就可以做到同时启动Nginx和Node

```JSON
{
  "scripts": {
    "start": "service nginx start & node app.js"
  }
}
```

这是因为：

> 如果是并行执行（即同时的平行执行），可以使用`&`符号。如果是继发执行（即只有前一个任务成功，才执行下一个任务），可以使用`&&`符号。 --- [《阮一峰 - npm scripts 使用指南》](http://www.ruanyifeng.com/blog/2016/10/npm_scripts.html) 

然后执行容器的构建和运行命令：

```BASH
docker image build -t web01 .
docker container run -d -p 80:80 --rm --name webC01 web01
```

也可以通过使用pm2进行守护启动多个进程，我还没有尝试。

### 总结

这种方案在一个容器中同时启动了Nginx和NodeJS进程，端口的转发实在容器内实现，不再需要传入宿主IP，同时代码不需要有任何改动，还是一种比较经济的实现方法。

## 方案三：构建两个容器，通过Docker Compose控制端口

[Docker Compose](https://docs.docker.com/compose/)是用来管理多容器的Docker应用，Win7下的Toolbox已经自带了Docker Compose，所以直接使用即可。

Docker Compose的使用需要编写`docker-compose.yml`：

```YML
version: "3"
services:
  nginx:
    # if images has been build, use it
    # image: "nginx01"
    build: "./nginx"
    ports:
      - "80:80"
    links:
      # writ the host in the container
      - nodejs:nodeServer
  nodejs:
    # if images has been build, use it
    # image: "node01"
    build: "./nodejs"
    ports:
      - "8016:8016"
```

上面这个文件定义了两个容器服务。首先明确使用的`compose file`版本是3，然后在`services`中定义服务。第一个是Ngxin服务。
可以用`build`指定对应镜像的Dockerfile的地址。Compose会利用它自动构建这个镜像，然后使用这个镜像启动服务。

这样生成的容器名称是`[yml所在文件夹名称]_[服务名]_1`，我把`docker-compose.yml`放在了`compo`文件夹下，那么上面自动构建的两个容器的名字分别就是`compo_nginx_1`和`compo_nodejs_1`。

如果镜像已经构建好，可以直接使用，那么就无需使用`build`字段，改用`image`字段即可，内容是对应的`image`的名称。

如果同时指定`build`和`image`：

```
build: ./dir
image: webapp:tag
```

那么就会在`./dir`目录下生成名称为`webapp`，标签为`tag`的镜像。
`ports`字段指定容器暴露出的端口。

`links`在两个容器之间建立连接，实际上相当于在容器内部改写host文件，`nodejs:nodeServer`意味着在Nginx内部使用`nodeServer`这个`host`时就会指向NodeJS这个container所在的IP地址。这样在Nginx的配置文件中，配置转发规则时就可以这样配置了：

```NGINX
location /api {
  proxy_pass http://nodeServer:8016;
}
```

如果直接改写容器的Host文件是不会生效的，而且容器的IP不是固定的，所以采用这种方式就避免了直接在两个容器之间手动关联IP。

`docker-compose.yml`文件编写完成后，就可以启动两个容器了。

```BASH
docker-compose up --build -d

# stop
# docker-compose stop
```

`--build`会强制在运行容器前重新构建镜像，`-d`让容器在后台运行。启动之后：

```BASH
Starting compo_nodejs_1 ... done
Starting compo_nginx_1  ... done
```

启动成功，访问`80`端口会指向`index.html`文件，访问`/api`会转发到Nodejs监听的`8016`端口。

### 总结

相比直接构建两个容器手动控制端口，使用Compose更加的合理、清晰。

但是官方文档对于`links`字段有[警告](https://docs.docker.com/compose/compose-file/#labels-2)，它属于一个遗留功能，未来将被移除，所以建议使用[自定义网络（user-defined networks）](https://docs.docker.com/network/)来代替`links`实现两个容器之间的通信。

但是自定义网络不支持容器间共享环境变量（而`links`支持）。如果需要共享环境变量，需要使用`volumes`。

## 方案四：完全使用NodeJS进程

Nginx提供的静态文件服务完全可以有NodeJS实现。我这里面使用了Koa监听端口，`koa-static`中间件实现静态服务，同时通过`request`实现了请求的转发

对`app.js`改造如下：

```JS
const path = require('path');
const request = require('request');
const Koa = require('koa');
const KoaBodyParser = require('koa-bodyparser');
const Compress = require('koa-compress');
const staticServer = require('koa-static');

const app = new Koa();
const app2 = new Koa();

app.use(Compress({
  threshold: 2048 // 要压缩的最小响应字节
}));

app.use(KoaBodyParser());

// log
app.use(async (ctx, next) => {
  const start = new Date();
  await next();
  let ms = new Date() - start;
  console.log('%s %s - %s', ctx.method, ctx.url, ms); // 显示执行的时间
});

// Koa静态文件服务
app.use(staticServer(path.resolve('dist')));

// 端口转发
app.use((ctx) => {
  if(ctx.path === '/api') {
    ctx.body = ctx.req.pipe(request(`${ctx.protocol}://0.0.0.0:8016`));
  }
});

// 80 端口
app.listen(80, () => {
  console.log(`Koa is listening in 80`);
});

// 8016 端口
app2.listen(8016, () => {
  app2.use(ctx => {
    ctx.body = '哦也，/api的请求转发过来了';
  });
  console.log(`Koa is listening in 8016`);
});
```

Dockerfile基本上没有变化：

```Dockerfile
FROM node:latest

# 拷贝文件
COPY ./src /var/www/api

# 设置工作目录
WORKDIR /var/www/api

# 安装依赖
RUN cd /var/www/api; npm install

EXPOSE 80

CMD ["npm", "run", "start"]
```

然后执行命令，创建并运行容器：

```BASH
docker image build -t koa01 .
docker container run -d -p 80:80 --rm --name koaC01 koa01
```

服务正常开启，访问`http://192.168.99.100/`就可以获得静态页面，访问`http://192.168.99.100/api`，Koa会将请求转发到容器的`8016`端口，执行相应的代码。

### 总结

这种方法只用了NodeJS，抛弃了Nginx，对于前端来说实现更容易一点，但是也就没有办法享受Nginx易配置、实现负载均衡等功能了，并且需要对代码进行改造。

最终采取的也是这种方案，只不过在此基础上进行了简化，编译的触发不再通过不同端口实现，都是在`80`端口上实现，更加简单。


## 参考
- [Nginx and Node.js with Docker](https://schempy.com/2015/08/25/docker_nginx_nodejs/)
- [Using Docker with nginx and NodeJS](https://cloudbuilder.in/blogs/2017/11/19/using-docker-nginx-nodejs/)
- [简书 - Docker多容器部署实践(nginx+node.js)](https://www.jianshu.com/p/1f7ccba1d65f)
- [Robby - 容器化 Node.js express（Mac）](https://dotblogs.com.tw/explooosion/2018/09/15/194754)
- [Sean's Notes - Dockerfile指令详解](http://seanlook.com/2014/11/17/dockerfile-introduction/)
- [medium - Docker Compose 讓 Nginx 與 Nodejs web server共舞](https://medium.com/@Whien/docker-compose-%E8%AE%93-nginx-%E8%88%87-nodejs-web-server%E5%85%B1%E8%88%9E-load-balancing-5d3fc41bab9e)
- [segmentfault -为什么在docker中服务要以前台方式启动？ ](https://segmentfault.com/q/1010000009581818)
- [stackoverflow - Error starting node with forever in docker container](https://stackoverflow.com/questions/26237044/error-starting-node-with-forever-in-docker-container)
