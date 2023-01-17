---
title: Node13 文件路径
top: false
date: 2018-07-11 09:12:17
updated: 2019-04-30 11:10:46
tags:
- process.cwd
categories: Node
---

重新学习Node，整理以前的日志。文件路径的笔记。

<!-- more -->

## 路径表示

Node.js中的文件路径主要有以下几种：

1. `__dirname`
2. `__filename`
3. `proces.cwd()`
4. `./`
5. `../`

其中前三个是绝对路径，后两个是相对路径 ，可以通过`path.resolve`转换为绝对路径

我现在的目录结构是这样的：

```TEXT
D:/
    -projects/
        path-test/
            path.js
```
path.js：

```JS
const path = require('path')
console.log('__dirname：', __dirname)
console.log('__filename：', __filename)
console.log('process.cwd()：', process.cwd())
console.log('./：', path.resolve('./'))
```
在`path-test`文件夹下用Node执行`path.js`，输出结果：

```TEXT
__dirname：     D:\projects\path-test
__filename：    D:\projects\path-test\path.js
process.cwd()： D:\projects\path-test
./：            D:\projects\path-test
```

在`projectst`文件夹下用Node执行`path.js`：

```TEXT
__dirname：     D:\projects\path-test
__filename：    D:\projects\path-test\path.js
process.cwd()： D:\projects
./：            D:\projects
```
关于他们的区别：

1. `process.cwd()`是程序的执行路径，`./`相同
2. `__dirname`是被执行的JS文件所在文件夹的绝对路径
3. `__filename`是被执行的JS文件的绝对路径，与`__dirname`一样，都是JS文件本身的属性

## path模块

### `path.join()`

用于连接路径，主要用于针对不同系统（windows/unix）使用当前系统的路径分隔符

```JS
var path = require('path');
path.join(mydir, "foo");
```
上面代码在Unix系统下，会返回路径`mydir/foo`。

### `path.join()`

用于将相对路径转为绝对路径

它可以接受多个参数，依次表示所要进入的路径，直到将最后一个参数转为绝对路径。**如果根据参数无法得到绝对路径，就以当前所在路径作为基准**。除了根目录，该方法的返回值都不带尾部的斜杠。

```JS
// 实例
path.resolve('foo/bar', '/tmp/file/', '..', 'a/../subfile')
```

上面代码的实例，执行效果类似下面的命令。

```BASH
$ cd foo/bar
$ cd /tmp/file/
$ cd ..
$ cd a/../subfile
$ pwd
```

更多例子：

```JS
path.resolve('/foo/bar', './baz')
// '/foo/bar/baz'

path.resolve('/foo/bar', '/tmp/file/')
// '/tmp/file'

path.resolve('wwwroot', 'static_files/png/', '../gif/image.gif')
// 如果当前目录是/home/myself/node，返回
// /home/myself/node/wwwroot/static_files/gif/image.gif
```



## 参考
- [Node.js的__dirname，__filename，process.cwd()，./的一些坑@github](https://github.com/jawil/blog/issues/18)
- [Path模块@JavaScript标准参考教程](http://javascript.ruanyifeng.com/nodejs/path.html#toc0)
- [process对象@JavaScript标准参考教程](http://javascript.ruanyifeng.com/nodejs/process.html#toc7)
