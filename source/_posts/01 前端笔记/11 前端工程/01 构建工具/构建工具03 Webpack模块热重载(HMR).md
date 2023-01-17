---
title: 构建工具03 Webpack模块热重载(HMR)
top: false
date: 2018-04-24 16:43:00
updated: 2019-08-05 16:43:00
tags: 
- HMR
- Webpack
categories: 前端工程
---

使用[`webpack-dev-server`](https://webpack.js.org/configuration/dev-server/#devserver) 实现的Hot Moudle Replacement（HMR）让我们在开发时修改代码并保存后，不必手动刷新浏览器，而是让浏览器通过新的模块替换老的模块。这样可以让我们在保证当前页面状态的前提下，让新的代码生效，就如同在Chrome的控制台修改CSS样式一样。

<!-- more -->

## 使用

安装`webpack-dev-server`

```BASH
npm install webpack-dev-server --save-dev
```

在`webpack.config.js`中进行配置

```JS
devServer: {
  contentBase: path.resolve(__dirname, 'dist'),
  host: 'localhost',
  compress: true,
  port: 8080
}
```

其中：

- `contentBase`：服务器基本运行路径
- `host`：服务器运行地址
- `compress`：服务器压缩式，一般为`true`
- `port`：服务运行端口

在`package.json`中定义相关命令：

```JS
"scripts": {
  "dev": "webpack-dev-server --hot --open",
},
```

然后执行`npm run dev`就可以开启webpack的服务，并且实现模块热重载，并且自动打开浏览器。

增加`--open`属性可以自动打开浏览器。

## 原理解析

原来只是在各种Cli工具中使用了模块热重载，知道是利用了Webpack的HMR特性，但是它是怎么实现的却不了解。今天在清理收藏夹攒的知识时看到了饿了么前端专栏的[这篇文章Webpack HMR 原理解析](https://zhuanlan.zhihu.com/p/30669007)，写的非常好，简单易懂，把道理也说的很明白。

![](http://image.oldzhou.cn/FjAWv8-u5cSSC1-0mOBQb1HfWw9V)

上图展示了从修改代码到模块热更新完成的一个周期：

### 第一步：Webpack在`watch`模式下打包更改的文件到内存中（对应图中的①②③）

Webpack-dev-middleware调用Webpack的API对文件系统watch，监听到文件变化时，根据配置文件对模块重新编译打包，将打包后的代码以JavaScript对象的形式保存在**内存**中。

```JS
// webpack-dev-middleware/lib/Shared.js
if (!options.lazy) {
  var watching = compiler.watch(options.watchOptions, share.handleCompilerCallback);
  context.watching = watching;
}
```

Webpack会将打包的文件保存在内存中，而不是打包到`output.path`目录下，是因为访问内存中的代码比访问文件系统中的代码更快，也减少了写入文件的开销。这个过程利用了[memory-fs](https://github.com/webpack/memory-fs)这个库，它提供了一个简单的基于内存的文件系统，所有数据都保存在JavaScript对象中。

图中的第③步也是对文件变化的监控，只不过这一步监听的不是代码，而是在配置文件制定的静态文件目录下的静态文件的变化（当配置文件中配置了`devServer.watchContentBase`为`true`的时候），当静态文件发生变化时通知浏览器对应用进行刷新（注意是浏览器刷新，而非HRM）


### 第二步：webpack-dev-Server通知浏览器端文件发生变化（对应④）

浏览器端和服务端之间是通过Websocket长连接进行通信的，利用的是[sockjs](https://github.com/sockjs/sockjs-client)建立的。通过Websocket长连接，webpack-dev-Server将编译打包的各个阶段状态告知浏览器（包括第③步中监听的静态文件的变化）。

同时webpack-dev-Server调用Webpack的API监听complie的`done`事件，在编译完成后，webpack-dev-Server通过`_sendStatus`方法将编译打包后的新模块的**hash值**发送给浏览器，后面的步骤都会利用这个hash值来进行模块热替换。

```JS
// webpack-dev-server/lib/Server.js
compiler.plugin('done', (stats) => {
  // stats.hash 是最新打包文件的 hash 值，发送给浏览器
  this._sendStats(this.sockets, stats.toJson(clientStats));
  this._stats = stats;
});
// ...
Server.prototype._sendStats = function (sockets, stats, force) {
  if (!force && stats &&
  (!stats.errors || stats.errors.length === 0) && stats.assets &&
  stats.assets.every(asset => !asset.emitted)
  ) { 
    return this.sockWrite(sockets, 'still-ok'); 
  }
  // 调用 sockWrite 方法将 hash 值通过 websocket 发送到浏览器端
  this.sockWrite(sockets, 'hash', stats.hash);
  if (stats.errors.length > 0) { 
    this.sockWrite(sockets, 'errors', stats.errors); 
  } 
  else if (stats.warnings.length > 0) { 
    this.sockWrite(sockets, 'warnings', stats.warnings); 
  } else { 
    this.sockWrite(sockets, 'ok'); 
  }
};
```

### 第三步：webpack-dev-server/client接收到服务端消息做出响应（对应⑤⑪）

webpack-dev-server/client端并不能够请求更新的代码，也不会执行热更模块操作，而是在接收到通过长连接收到的服务端的消息后，对信息进行处理，而具体的更新操作又交回给了Webpack。

webpack/hot/dev-server的工作就是根据webpack-dev-server/client传给它的信息以及dev-server的配置决定是刷新浏览器呢还是进行模块热更新。当然如果仅仅是刷新浏览器，也就没有后面那些步骤了。

我们并没有在业务代码里添加Websocket客户端的代码，也没有在`webpack.config.js`中的`entry`属性中添加新的入口文件，那么`bundle.js`中的接受Websocket信息的代码是从哪来的呢？答案是webpack-dev-server会自动修改Webpack配置中的`entry`属性，在里面添加了webpck-dev-client的代码。


具体来看，webpack-dev-server/client接收到`type`为`hash`的消息后会将`hash`保存起来，接收到`type`为`ok`的消息后会执行`relooad`操作，在`reload`操作中会根据`hot`的配置是刷新浏览器还是执行热更新（HMR）:


```JS
// webpack-dev-server/client/index.js
hash: function msgHash(hash) {
    currentHash = hash;
},
ok: function msgOk() {
    // ...
    reloadApp();
},
// ...
function reloadApp() {
  // ...
  if (hot) {
    log.info('[WDS] App hot update...');
    const hotEmitter = require('webpack/hot/emitter');
    hotEmitter.emit('webpackHotUpdate', currentHash);
    // ...
  } else {
    log.info('[WDS] App updated. Reloading...');
    self.location.reload();
  }
}
```

在上面的代码中，webpack-dev-server/client首先将接收到的`hash`值存储到`currentHash`变量中，当接收到`ok`消息后调用`reloadApp`方法，在其内部根据`hot`配置，决定是调用`webpack/hot/emitter`将最新的`hash`值发送给Webpack执行热更新，还是直接调用`location.reload`刷新页面。


### 第四步：Webpack接收新的`hash`值并请求模块代码（对应⑥⑦⑧⑨）

首先webpack/hot/dev-server监听上一步webpack-dev-server/client发送的`webpackHotUpdate`消息，然后调用webpack/lib/HotModuleReplacement.runtime（简称HMR runtime），HMR runtime是客户端HMR的**中枢**，它首先通过JsonpMainTemplate.runtime调用`hotDownloadManifest`方法向server端发送JSONP请求，检查是否有更新的文件，如果有的话服务端返回一个JSON响应，包含了所有要更新的模块的hash值。

获取到更新列表后，该模块通过`hotDownloadUpdateChunk`再次发送JSONP请求，获取到最新的模块代码，并返回给HMR runtime。

上面为了获取最新的Hash值和最新的代码，HMR runtime向服务端发送了两次Ajax请求，为什么不在第三步的Websocket长连接中发送给浏览器呢？可能的原因：

（1）包括了**功能模块的解耦**，webpack-dev-server/client只负责消息的传递而不负责新模块的拉取，HRM runtime来负责获取新代码

（2）可以使用webpack-hot-middleware来代替webpack-dev-server实现HMR，webpack-hot-middleware没有使用Websocket，而是使用[EventSource](https://www.ruanyifeng.com/blog/2017/05/server-sent_events.html)来实现客户端与服务端通信。

### 第五步：HMR runtime对模块进行热更新（对应⑩）

HMR runtime会对新旧模块进行对比，决定是否更新模块，在决定更新模块后，检查模块之间的依赖关系，更新模块的同时更新模块间的依赖引用。

这一切都发生在HMR runtime的`hotApply`方法中：


```JS
// webpack/lib/HotModuleReplacement.runtime
function hotApply() {
  // ...
  var idx;
  var queue = outdatedModules.slice();
  while (queue.length > 0) {
    moduleId = queue.pop();
    module = installedModules[moduleId];
    // ...
    
    // remove module from cache
    delete installedModules[moduleId];
    // when disposing there is no need to call dispose handler
    delete outdatedDependencies[moduleId];
    // remove "parents" references from all children
    
    for (j = 0; j < module.children.length; j++) {
      var child = installedModules[module.children[j]];
      if (!child) continue;
      idx = child.parents.indexOf(moduleId);
      if (idx >= 0) {
        child.parents.splice(idx, 1);
      }
    }
  }
  // ...
  // insert new code
  for (moduleId in appliedUpdate) {
    if (Object.prototype.hasOwnProperty.call(appliedUpdate, moduleId)) {
      modules[moduleId] = appliedUpdate[moduleId];
    }
  }
  // ...
}
```

`hotApply`方法主要分为了三个阶段：

1. 找出陈旧的模块`outdatedModules`和依赖`outdatedDependencies`
2. 从缓存中删除过期的模块和依赖
3. 将新的模块和依赖添加到`moudles`中，当下次调用`_webpack_require`方法时就获取到新的代码

如果HMR失败后，回退到`live reload`操作，也就是进行浏览器刷新来获取最新打包代码，相关的代码在dev-server中：

```JS
module.hot.check(true).then(function(updatedModules) {
  if (!updatedModules) {
    return window.location.reload();
  }
  // ...
}).
catch (function(err) {
  var status = module.hot.status();
  if (["abort", "fail"].indexOf(status) >= 0) {
    window.location.reload();
  }
});
```

### 第六步：业务代码改造

当新的模块代替老的模块后，旧的业务代码并不能知道代码发生变化，所以需要在业务代码的入口调用HMR的`accept`方法，添加模块更新后的处理函数：

```JS
// index.js
if (module.hot) {
  module.hot.accept('./hello.js', function() {
    // 更新后的处理函数
  })
}
```



## 参考

- [模块热替换@webpack](https://webpack.docschina.org/guides/hot-module-replacement/#-hmr/)
- [webpack/webpack-dev-server@github](https://github.com/webpack/webpack-dev-server)
- [Webpack HMR 原理解析@知乎](https://zhuanlan.zhihu.com/p/30669007)

