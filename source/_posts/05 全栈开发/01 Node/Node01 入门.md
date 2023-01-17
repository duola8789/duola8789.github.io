---
title: Node01 入门
top: false
date: 2017-03-31 15:19:17
updated: 2019-04-30 10:54:31
tags:
- Node
categories: Node
---

重新学习Node，整理以前的日志。一篇Node入门的笔记。

<!-- more -->

## 什么是Node.js

Node是一个服务器端JavaScript解释器。Node的目标是帮助程序员构建高度可伸缩的应用程序，编写能够处理数万条同时连接到一个物理机的连接代码。处理高并发和异步I/O是Node受到开发人员的关注的原因之一。

Node本身运行Google V8 JavaScript引擎，所以速度和性能非常好，而且Node对其封装的同时还改进了其处理二进制数据的能力。因此，Node不仅仅简单的使用了V8，还对其进行了优化，使其在各种环境下更加给力。

第三方的扩展和模块在Node的使用中起到重要的作用，npm就是模块的管理工具，用它安装各种Node的软件包并发布自己为Node写的软件包。

## 安装Node

### 在macOs下安装

可以通过homebrew来安装，可以通过`brew -v`来查看是否安装了homebrew，如果没有安装，则通过终端命令安装

```BASH
ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)" 
```
安装成功后，来安装node：

```BASH
brew link node
brew uninstall node
brew install node
```
也可以直接在[官网](https://nodejs.org/dist/)下载Node的安装包，为`pkg`格式，双击安装包安装即可。

### 在Windows下安装

直接在[官网](https://nodejs.org/dist/)下载Node的安装包，为`msi`格式，双击安装包安装即可。

### 在centos下安装

进入要存放下载资源的目录，比如`/usr/local/src/`目录，然后执行安装命令

```BASH
wget http://nodejs.org/dist/v6.9.4/node-v6.9.4-linux-x64.tar.gz
```
上述命令是下载6.9.4的64位nodejs版本，如果你想下载其他版本，可以将命令中的两处`v6.9.4`替换成其他版本号；

如果系统是32位(一般是64位)，也可以将`x64`改成`x32`

下载完成后，执行解压命令

```BASH
tar -zxvf node-v6.9.4-linux-x64.tar.gz
```

解压完成，可以看到当前目录解压后的文件夹`node-v6.9.4-linux-x64`，重命名一下

```BASH
mv node-v6.9.4-linux-x64 node
```
现在，node文件夹就是程序目录，但是还不能够全局使用，需要添加变量

首先在`root`目录下找到`.bash_profile`文件，编辑

```BASH
vim ~/.bash_profile
```

找到`PATH=$PATH:$HOME/bin`，在后面添加路径为：


```BASH
PATH=$PATH:$HOME/bin:/usr/local/src/node/bin
```

保存修改`:wq`，然后重载一下

```BASH
source ~/.bash_profile
```

OK！大功告成！现在可以在任何目录下执行`node`和`npm`命令了！

## Hello Node

```JS
var http = require('http');　

http.createServer(function(req, res) {　　
  res.writeHead(200, {
    'Content-Type': 'text/plain'
  }); // text/plain是无格式正文
  res.end('Hello World\n');　　
}).listen(1337, "127.0.0.1");　

console.log('Server running at http://127.0.0.1:1337/');
```

代码逻辑：

1. 全局方法`require()`是用来导入模块的，一般直接把`require()`方法的返回值赋值给一个变量，在JavaScript代码中直接使用此变量即可。`require("http")`就是加载系统预置的`http`模块；
2. `http.createServer`是模块的方法，目的就是创建并返回一个新的web server对象，并且给服务绑定一个回调，用以处理请求；
3. 通过`http.listen()`方法就可以让该HTTP服务器在特定端口监听
4. Node实现了`console.log`方法，执行`hello.js`代码，相关信息会显示在命令行中，

在命令行，成功启动会看见`console.log()`中的文本。


## Node Moudle

在Node中，不同的功能组件被划分成不同的模块。应用可以根据自己的需要来选择使用合适的模块。每个模块都会暴露一些公共的方法或属性。模块的使用者直接使用这些方法或属性即可，对于内部的实现细节就可以不用了解。除了Node本身提供的API外，开发人员也可以利用这个机制来将应用拆分成多个模块，以提高代码的可复用性。

### 如何使用模块

在Node中使用模块是非常方便的，在代码中可以直接使用全局函数`require()`来加载一个模块。

### 自己如何开发模块？

使用`require()`导入模块的时候，模块名称以`"./"`开始的这种，就是自己开发的模块文件。

代码中封装了模块的内部处理逻辑，一个模块一般都会暴露一些公开的方法或属性给其他的人使用。模块的内部代码需要把这些方法或属性给暴露出来。

`hello.js`的内容：

```JS
var method = {
  number: 50,
  add: function(a, b) {
    return a + b;
  }
};

exports.method = method;
```

在另外一个文件`myNode.js`中引用`hello.js`：

```JS
var hello = require("./hello.js");
var sum = hello.method.add(5, 10);

console.log(sum);
console.log(hello.method.number)
```

运行后结果：

```BASH
$ node myNode.js
15
50
```

## Node的优势

Node核心思想：

- **非阻塞**
- **单线程**
- **事件驱动**

在目前的web应用中，客户端和服务器端之间有些交互可以认为是基于事件的，那么Ajax就是页面及时响应的关键。每次发送一个请求时（不管请求的数据多么小），都会在网络里走一个来回。服务器必须针对这个请求作出响应，通常是开辟一个新的进程。那么越多用户访问这个页面，所发起的请求个数就会越来越多，就会出现内存溢出、逻辑交错带来的冲突、网络瘫痪、系统崩溃这些问题。

Node的目标是提供一种构建可伸缩的网络应用的方案，服务器可以同时处理很多客户端连接。

Node和操作系统有一种约定，如果创建了新的链接，操作系统通知Node后进入休眠。如果有人创建了新的链接，那么它（Node）执行一个回调，每一个链接只占用了非常小的（内存）堆栈开销。

Node异步执行的例子：

`hello.js`的内容：

```JS
var method = {
  number: 50,
  add: function(a, b) {
    return a + b;
  }
};

exports.method = method;
```
在另外一个JS文件`myNode.js`中引用`hello.js`：

```JS
var fs = require("fs");

fs.readFile("./hello.js", function(err, data) {
  if (err) {
    throw err
  }
  console.log("successful")
});

console.log("async")
```
文件执行结果首先执行打印`async`，然后打印异步执行的`successful`，运行后结果：

```BASH
$ node myNode.js
async
successful
```

Node是无阻塞的，新请求到达服务器时，不需要为这个请求单独作什么事情。Node仅仅是在那里等待请求的发生，有请求就处理请求。

因此，Node更擅长处理体积小的请求以及基于事件的I/O。Node不仅仅是做一个Web服务的框架，它可以做更多，比如它可以做Socket服务，可以做比方说基于文件的，然后基于像一些比方说可以有子进程，然后内部的，它是一个很完整的事件机制，包括一些异步非注射的解决方案，而不仅仅局限在网络一层。同时它可能，即使作为一个Web服务来说，它也提供了更多可以深入这个服务内核、核心的一些功能，比方说Node使用的Http Agent，这块就是它可以更深入这个服务内核来去做一些功能。


## Node的事件流

因为Node采用的是事件驱动的模式，其中的很多模块都会产生各种不同的事件，可由模块来添加事件处理方法，所有能够产生事件的对象都是事件模块中的EventEmitter类的实例。

```JS
var event = require("events");
var emitter = new event.EventEmitter();

emitter.on("myEvent",function(msg){
  console.log(msg)
});

emitter.emit("myEvent","this is my event")
```

上面的代码中：

1. 使用`require()`方法添加了`events`模块并把返回值赋给了一个变量

2. `new events.EventEmitter()`这句创建了一个事件触发器，也就是所谓的事件模块中的`EventEmitter`类的实例

3. `on(event, listener)`用来为某个事件`event`添加事件处理方法监听器

4. `emit(event, [arg1], [arg2], [...])`方法用来产生事件。以提供的参数作为监听器函数的参数，顺序执行监听器列表中的每个监听器函数。

在Node中，存在各式各样不同的数据流，Stream（流）是一个由不同对象实现的抽象接口。

例如请求HTTP服务器的`request`是一个流，类似于stdout（标准输出）；包括文件系统、HTTP 请求和响应、以及 TCP/UDP 连接等。流可以是可读的，可写的，或者既可读又可写。所有流都是EventEmitter的实例，因此可以产生各种不同的事件。

可读流主要会产生以下事件：

- `data`： 当读取到流中的数据时，此事件被触发
- `end`： 当流中没有数据可读时，此事件被触发
- `error`： 当读取数据出现错误时，此事件被触发
- `close`： 当流被关闭时，此事件被触发，可是并不是所有流都会触发这个事件。（例如，一个连接进入的HTTP `request`流就不会触发`close`事件）
- `fd`事件 ： 当在流中接收到一个文件描述符时触发此事件。只有UNIX流支持这个功能，其他类型的流均不会触发此事件。

## 强大的File System文件系统模块

Node中的`fs`模块用来对本地文件系统进行操作。文件的I/O是由标准POSIX函数封装而成。需要使用`require('fs')`访问这个模块。所有的方法都提供了异步和同步两种方式。

`fs`模块中提供的方法可以用来执行基本的文件操作，包括读、写、重命名、创建和删除目录以及获取文件元数据等。每个操作文件的方法都有同步和异步两个版本。

异步操作的版本都会使用一个回调方法作为最后一个参数。当操作完成的时候，该回调方法会被调用。而回调方法的第一个参数`err`总是保留为操作时可能出现的异常。如果操作正确成功，则第一个参数的值是`null`或`undefined`。

同步操作的版本的方法名称则是在对应的异步方法之后加上一个`Sync`作为后缀。比如异步的`rename()`方法的同步版本是`renameSync()`。
