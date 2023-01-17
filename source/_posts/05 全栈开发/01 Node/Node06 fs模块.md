---
title: Node06 fs模块
top: false
date: 2017-04-03 17:58:17
updated: 2019-04-30 11:03:51
tags:
- fs
- fs-extra
categories: Node
---

重新学习Node，整理以前的日志。fs模块和fs-extra的学习笔记。

<!-- more -->

## 概述

Node中的fs模块用来对本地文件系统进行操作。文件的I/O是由标准POSIX函数封装而成。需要使用`require('fs')`访问这个模块。

fs模块中提供的方法可以用来执行基本的文件操作，包括读、写、重命名、创建和删除目录以及获取文件元数据等。每个操作文件的方法都有同步和异步两个版本。

异步操作的版本都会使用一个回调方法作为最后一个参数。当操作完成的时候，该回调方法会被调用。而回调方法的第一个参数总是保留为操作时可能出现的异常。如果操作正确成功，则第一个参数的值是`null`或`undefined`。

同步操作的版本的方法名称则是在对应的异步方法之后加上一个`Sync`作为后缀。比如异步的`rename()` 方法的同步版本是`renameSync()`。
　　
## 使用方法：

`fs.readFile`属于`fs`模块，使用前需要首先引入`fs`模块

```JS
const fs= require(“fs”)
```

`fs.readFile`以异步的方式读取文件内容。

## API

### 读取文件`readFile`

```JS
fs.readFile(filename, [options], [callback(err,data)])
```

- `filename`：必选参数，文件路径，可以是绝对路径，也可以是相对路径。如果是相对路径，是相对于当前进程（`process.cwd`）的路径，而不是相对于当前脚本的路径。
- `options`：可选参数，可指定flag（文件操作选项，如r+ 读写；w+ 读写，文件不存在则创建）及`encoding`属性
- `callback`：回调函数，传递2个参数，异常`err`和文件内容`data`

注意，文件编码可取值包括'ascii'/'utf-8'/'base64'，如果没有指定文件编码，返回的是原始的缓存二进制数据，这时需要调用buffer对象的`toString`方法将其转换为字符串。

```JS
const fs = require('fs');

console.log('start');

fs.readFile('./test.txt', 'utf-8', (error, file) => {
  console.log(file);
});

// fs.readFile('./test.txt', (error, file) => {
//   console.log('before:', file.toString());
// });

console.log('finish');
```

输出结果：

```TEXT
start
finish
Content: 你好啊
```

### 写文件`writeFile`

```JS
fs.writeFile(filename, data, [options], callback(err));
```

- `filename`：必选参数，文件路径
- `data`：写入的数据，可以是字符或一个Buffer对象
- `options`：`flag`, `mode`(权限)，`encoding`
- `callback`：回调函数，传递1个参数，异常`err`


```JS
const fs = require('fs');

console.log('start');

fs.writeFile('./test.txt', '难判谁人错与对', 'utf-8', (error, file) => {
  console.log('write finish');
});

fs.readFile('./test.txt', 'utf-8', (error, file) => {
  console.log('Content:', file);
});

console.log('finish');
```

输出结果：

```TEXT
start
finish
write finish
Content: 难判谁人错与对
```
注意，在同一个文件多次使用`fs.writeFile`且不等待回调是不安全的，对于这种情况应该使用`fs.createWriteStrem`


### 以追加方式写文件`appendFile`

```JS
fs.appendFile(filename, data, [options], callback(err));
```
异步地将数据追加到文件，如果文件尚不存在则创建该文件。 `data`可以是字符串或`Buffer`。

```JS
const fs = require('fs');

fs.readFile('./test.txt', 'utf-8', (error, file) => {
  console.log('before:', file);
});

fs.appendFile('./test.txt', '你随河流而来', 'utf-8', (error) => {
  fs.readFile('./test.txt', 'utf-8', (error, file) => {
    console.log('after:', file);
  });
});
```
结果：

```JS
before: 难判谁人错与对

after: 难判谁人错与对
你随河流而来
```

### 打开文件`open`

```JS
fs.open(filename, flags, [mode], callback(err,fd));
```
- `filename`：必选参数，文件名
- `flags`：操作标识，如`r`,读方式打开
- `[mode]`：权限，如777，表示任何用户读写可执行
- `callback`：打开文件后回调函数，参数默认第一个`err`，第二个`fd`为一个整数，表示打开文件返回的文件描述符，Window中又称文件句柄


### 监视文件`watchFile`

```JS
fs.watchFile(filename, [options], listener(curr, prev));
```
对文件进行监视，并且在监视到文件被修改时执行处理

- `filename`, 完整路径及文件名；
- `[options]`, `persistent`为`true`表示持续监视，不退出程序；`interval`单位毫秒，表示每隔多少毫秒监视一次文件
- `listener`, 文件发生变化时回调，有两个参数：`curr`为修改后的`fs.Stat`对象，`prev`为修改之前的`fs.Sta`对象
 
```JS
fs.watchFile('./test.txt', (curr, prev) => {
  console.log('current', curr);
  console.log('previous', prev);
});
```
改动文件后，输出结果：

```TEXT
current Stats {
  dev: 16777218,
  mode: 33188,
  nlink: 1,
  uid: 501,
  gid: 20,
  rdev: 0,
  blksize: 4096,
  ino: 4636749,
  size: 46,
  blocks: 8,
  atimeMs: 1556524554000,
  mtimeMs: 1556524553000,
  ctimeMs: 1556524553000,
  birthtimeMs: 1556524553000,
  atime: 2019-04-29T07:55:54.000Z,
  mtime: 2019-04-29T07:55:53.000Z,
  ctime: 2019-04-29T07:55:53.000Z,
  birthtime: 2019-04-29T07:55:53.000Z }
previous Stats {
  dev: 16777218,
  mode: 33188,
  nlink: 1,
  uid: 501,
  gid: 20,
  rdev: 0,
  blksize: 4096,
  ino: 4636495,
  size: 42,
  blocks: 8,
  atimeMs: 1556524547000,
  mtimeMs: 1556524320000,
  ctimeMs: 1556524320000,
  birthtimeMs: 1556524319000,
  atime: 2019-04-29T07:55:47.000Z,
  mtime: 2019-04-29T07:52:00.000Z,
  ctime: 2019-04-29T07:52:00.000Z,
  birthtime: 2019-04-29T07:51:59.000Z }
```

`fs.unwatfile`方法用来接触对文件的监听。

### 判断路径是否存在

原本判断路径是否存在的`fs.exists`已经在V1.0.0被废弃，改为使用`fs.stat`或者`fs.access`

`fa.stat`用来查询文件信息

```JS
fs.stat('./test.txt', (err, stat) => {
  if (stat && stat.isFile()) {
    console.log(stat);
  } else {
    console.log('file does not exist');
  }
});
```
`fs.access`用来检查文件的访问权限

```JS
// 检查文件是否存在
fs.access('/etc/passwd', function(err) {
  console.log(err ? '文件存在' : '文件不存在');
});

// 检查文件是否有读写权限
fs.access('/etc/passwd', fs.R_OK | fs.W_OK, function(err) {
  console.lo(err ? '不可操作!' : '可以读/写');
});
```

### 新建目录`fs.mkdir()`

```JS
fs.mkdir(path[, options], callback)
```

`mkdir`接受三个参数吗，第一个是目录名，第二个是权限制，第三个是回调函数

```JS
fs.mkdir('./hello', '0777', err => console.log(err));
```

### 读取目录`readdir`

```JS
fs.readdir(path[, options], callback)
```

用来读取目录，返回一个所包含的文件和子目录的数组，不会递归寻找内部目录，需要手动递归实现：

```JS
const list = (path) => {
  fs.readdir(path, (err, files) => {
    console.log(files);
    files.forEach(file => {
      fs.stat(file, (error, stat) => {
        if(stat && stat.isDirectory()) {
          list(file)
        }
      })
    })
  })
};

list(process.cwd());
```

### 创建读/写操作的数据流`createReadStream`/`createWriteStream`

`createReadStrem`方法旺旺用来打开大型的文本文件，创建一个读取操作的数据流。大型文本文件体积很大，读取操作的缓存装不下，只能分成几次发送。每次发送会触发`data`时间，发送结束会触发`end`时间

```JS
const func = data => {
  console.log('Line: ' + data);
};

const readLines = (input, func) => {
  let remaining = '';

  input.on('data', data => {
    remaining += data;
    let index = remaining.indexOf('\n');
    let last = 0;
    while(index > -1) {
      let line = remaining.substring(last, index);
      last = index + 1;
      func(line);
      index = remaining.indexOf('\n', last);
    }

    remaining = remaining.substring(last);
  });

  input.on('end', () => {
    if(remaining.length > 0) {
      func(remaining)
    }
  })
};

const input = fs.createReadStream('./test.txt');

readLines(input, func);
```

`createWriteStream`用来创建一个写入数据流对象，该对象的`write`方法用于写入数据，`end`方法用于结束写入操作


```JS
const out = fs.createWriteStream(fileName, {
  encoding: 'utf8'
});
out.write(str);
out.end();
```

`createReadStream`和`createWriteStream`配合，可以实现拷贝大型文件。

```JS
const fileCopy = (filename1, filename2, done) => {
  const input = fs.createReadStream(filename1);
  const output = fs.createWriteStream(filename2);

  input.on('data', data => {
    output.write(data);
  });

  input.on('error', err => {
    console.log(err);
  });

  input.on('end', () => {
    output.end();
    if (done) {
      done()
    }
  })
};

fileCopy('./test.txt', './hello.text', () => {
  console.log('copy finish');
});
```
### 删除文件`unlink`和删除文件夹`rmdir`

```JS
// 删除文件
fs.unlink('./hello.text', (err) => {
  console.log('deleted');
});
```

```JS
// 删除文件夹只能删除空文件夹，如果里面有内容则不能直接删除，会报错
fs.rmdir('./world', (err) => {
  console.log(err, 'deleted');
});
```

如果`world`文件夹不为空，无法删除：

```TEXT
{ [Error: ENOTEMPTY: directory not empty, rmdir './world']
  errno: -66,
  code: 'ENOTEMPTY',
  syscall: 'rmdir',
  path: './world' }
```

### 重命名`rename`

```JS
// 重命名
fs.rename('./hello', './world', err => {
  console.log(err);
});
```

## 同步方法

上面提到的异步API都有着对应的同步方法，比如`readFile`对应的同步方法`readFileSync()`:

```JS
const fs = require("fs");
const file = fs.readFileSync("./test.txt", "utf-8");

console.log(file);
console.log("finish");
```
执行的顺序就是按照一条时间链由上至下执行：

```TEXT
你好哇~
finish
```

## fs-extra模块

fs-extra模块是fs模块的一个扩展，文档在[这里](https://www.npmjs.com/package/fs-extra)。

fs-extra集成了fs模块的所有方法，提供了很多遍历的API，并且添加了Promise的支持，可以使用它来代替fs模块。

### 安装使用

它不是Node自带的模块，需要单独安装

```BASH
npm i -S fs-extra
```

使用时完全可以替代fs模块，如果希望明确表示在使用fs-exra，可以将`fs`标示改为`fse`

```JS
const fse = require('fs-extra');
```

### 异步方法

fs-extra的异步方法可以使用回调函数的形式，更推荐的是使用Promise形式，也可以使用Async/Await的语法：

```JS
// 回调函数
fse.copy('./test.txt', './world/test.txt', err => {
  if (err) {
    console.error(err);
    return;
  }
  console.log('success!');
});

// Promise
fse.copy('./test.txt', './world/test.txt').then(() => {
  console.log('success');
}).catch(e => {
  console.error(e);
});

// Async/Await
const copyFile = async () => {
  try {
    await fse.copy('./test.txt', './world/test.txt');
    console.log('success');
  } catch (e) {
    console.error(e);
  }
};
copyFile()
```

### API

fs-extra提供了很多遍历的API，这些API都有异步和同步的两个版本，简单列举一部分异步的API，详细的API文档[看这里](https://github.com/jprichardson/node-fs-extra/tree/8e0879110fbdd597c36602fe3b81ef03a4b3ec7a/docs)。

```JS
fse.copy(src: string, dest: string, [options: object, callback: func])
// 复制文件或目录，目录可以包含内容

fse.emptyDir(dir: string, [callback: function])
// 确保目录为空，如果目录不为空，则删除目录下内容，如果目录不存在，则创建该目录

fse.ensureFile(file: string, [callback: func])
// 确保文件存在。如果请求创建的文件位于不存在的目录中，则会创建这些目录。如果该文件已存在，则不进行修改。（别名：createFile）

fse.ensureDir(dir: string, [callback: func])
// 确保目录存在，如果目录结构不存在，则创建它，如果目录存在则不进行操作

fse.ensureLink(srcpath: string, dstpath: string, [callback: func])
// 确保链接存在。如果目录结构不存在，则创建它。

fse.move(src: string, dest: string, [options: object, callback: func])
// 移动文件或目录

fse.outputFile(file: stirng, data: string|Buffer|Uint8Array, [options: string|object, callback: func])
// 几乎与writeFile相同，除了如果父目录不存在则创建它，file必须是文件路径（不允许使用缓存区或文件描述符）

fse.outputJson(file: string, object: object, [options: object, callback: func])
// 将对象写入JSON文件，几乎与writeJSON相同，除了如果目录不存在，则创建它

fse.writeJson(file, object, [options, callback])
// 将对象写入JSON文件，几乎与outputJSON相同，除了必须保证目录存在之外

fse.pathExists(file: string [, callback: func]);
// 检查文件系统来测试给定路径是否存在

fse.readJson(file: string, [options: object, callback: func])
// 读取JSON文件，并将其解析为对象

fse.remove(path: string, [callback: func])
// 删除文件或者目录，该目录可以包含内容，类似rm -rf
```

## 参考
- [fs 模块@JavaScript标准参考教程（alpha）](http://javascript.ruanyifeng.com/nodejs/fs.html#toc7)
- [Node.js v10.15.3 文档@nodejs.cn](http://nodejs.cn/api/fs.html#fs_fs_exists_path_callback)
- [使用Node.js的fs模块操作文件之检查文件是否存在](https://itbilu.com/nodejs/core/4JGAlesbl.html)
- [node-"fs-extra"模块代替fs使用@掘金](https://juejin.im/post/5b52fd21e51d4519234468f1/)
