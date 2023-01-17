---
title: Node10 同时使用不同版本的node
top: false
date: 2017-11-03 15:10:17
updated: 2019-04-30 11:07:47
tags:
- n
- nvm
categories: Node
---

重新学习Node，整理以前的日志。管理Node版本的笔记。

<!-- more -->

## 方法1：利用n工具

利用n工具可以创建不同版本的node并且在不同版本切换

```BASH
npm install -g n   

n ls                    //查看可用版本  
n 5.8.0                 //安装5.8.0版本
n stable                //安装最新稳定版
n lastest               //安装最新版
n rm 0.10.1             //删除某个版本
n use 0.10.21 some.js   //以指定的版本来执行脚本
```
使用最后一条命令就可以使用指定版本的node来运行文件， 比如，etutorweb使用的node是5.8.0， 所以当前node全局版本是5.8.0，exam的使用的node版本是6.9.1，原本的前端服务的启动命令是：

```BASH
NODE_ENV=development node ./bin/devServer.js
```

想要使用6.9.1版本的node需要把启动命令改为：

```BASH
NODE_ENV=development n use 6.9.1 ./bin/devServer.js
```

ok了！并且不需要更改项目中任何关于node版本的信息

## n无效的解决方法

有时候通过n来切换版本会出现不生效的情况，原因可能就是node的安装目录和n默认的路径和不同

查看node当前安装路径：

```
which node

/opt/node/bin/node  #举个例子
```
而n的默认安装路径是`/usr/local`，若你的node不是在此路径下，n切换版本就不能把文件复制到改路径中，所以我们必须通过`N_PREFIX`变量来修改n的默认的node安装路径

编辑环境配置文件：

```
vim ~/.bash_profile
```
将下面两行代码插入到文件末尾：

```
export N_PREFIX=/opt/node #node实际安装位置
export PATH=$N_PREFIX/bin:$PATH
```
`:wq`保存退出

执行source使修改生效

```
source ~/.bash_profile
```
确认一下环境变量是否生效：

```
echo $N_PREFIX
/opt/node
```
然后再重新执行切换版本的命令就ok了

```
$ n 4.4.4
install : node-v4.4.4
       mkdir : /opt/node/n/versions/node/4.4.4
       fetch : https://nodejs.org/dist/v4.4.4/node-v4.4.4-linux-x64.tar.gz
##############100.0%
   installed : v4.4.4
```
再查看当前 node 版本：

```
$ node -v
v4.4.4
```


## 方法2 使用ln -s命令

ln -s是linux系统中一个非常重要的命令，它的功能是为某一个文件在另外一个位置建立一个同不的链接，使用方法：

```
ln -s 源文件 目标文件
```
这里有两点要注意：

第一，ln命令会保持每一处链接文件的同步性，也就是说，不论你改动了哪一处，其它的文件都会发生相同的变化；

第二，ln的链接又软链接 和硬链接两种，软链接就是ln -s ** **,它只会在你选定的位置上生成一个文件的镜像，不会占用磁盘空间，硬链接ln ** **,没有参数-s, 它会在你选定的位置上生成一个和源文件大小相同的文件，无论是软链接还是硬链接，文件都保持同步变化。

这里就是利用ln -s将项目中的node文件连接到指定版本的node文件

首先找到n创建的不同版本的node安装位置（也可以不使用n安装不同版本的node），找到要使用的版本的node的启动文件：

```
/usr/local/n/versions/node/6.0.0/bin/node
```
然后将这个文件链接到全局，不过为了全局的node已经被node5.8.0版本占用，所以连接到node6，：


```
ln -s /usr/local/n/versions/node/6.9.1/bin/node /usr/local/bin/node6
```
也可以将对应版本的npm链接过去
```
ln -s /usr/local/n/versions/node/6.9.1/bin/npm /usr/local/bin/npm6
```
这样执行node6就对应到6.9.1的node上了

```
ubuntu@et-zhouhao:~$ node -v
v5.8.0
ubuntu@et-zhouhao:~$ node6 -v
v6.9.1
```
然后将项目的启动命令中的node改为node6即可：

```
NODE_ENV=development node6 ./bin/devServer.js
```
但是要注意，这种方法，需要在项目中将对应的node命令都改为node6，否则会报错，比如nodemon.json中的execMap对应的node要改为node6，并且只能在本机更改，不能上传到代码仓库中：

```
{
  "restartable": "rs",
  "ignore": [
    ".git",
    "node_modules/**/node_modules",
    "test",
    "view"
  ],
  "verbose": true,
  "execMap": {
    "js": "node6"
  },
  "events": {
    "restart": "echo \"App restarted\""
  },
  "watch": [
    "config",
    "server",
    "shared"
  ],
  "env": {
    "NODE_ENV": "development"
  },
  "ext": "js json"
}
```
## 参考
- https://segmentfault.com/a/1190000007567870
