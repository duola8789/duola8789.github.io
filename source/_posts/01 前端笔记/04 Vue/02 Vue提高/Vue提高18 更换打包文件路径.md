---
title: Vue提高18 更换打包文件路径
top: false
date: 2019-03-17 22:36:59
updated: 2019-03-17 22:37:07
tags:
- Vue
- Webpack
- 路径
categories: Vue
---

一个由更改Vue打包后路径引发的一系列问题。

<!-- more -->

## 引入字体文件

在Vue中，想要引入字体文件，需要，使用`@font-face`来引入本地的字体

```CSS


h1 {
  background: url('./assets/images/logo.png');
}
```
然后再`App.vue`中引入并使用

```HTML
<style>
  @font-face {
    font-family: 'hello';
    src: url('./assets/font/d.ttf');
    font-weight: bold;
    font-style: italic;
  }
  .hello-inner h1 {
    font-family: hello;
   }
</style>
```
这样就可以使用引入的字体了。

也可以单独建立一个CSS文件来统一管理引入的字体，然后通过`import`引入

```HTML
<style>
  @import 'font.css';
  .hello-inner h1 {
    font-family: hello;
   }
</style>
```
## url-loader

Webpack中的loader的目的是用来处理各种非JS之外的文件，可以使你在`import`或"加载"模块时预处理文件。

执行的顺序是从右至左，在Vue-cli2中，在`webpack.base.conf.js`中定义了一些基本的loader：

```JS
module: {
  rules: [
    {
      test: /\.vue$/,
      loader: 'vue-loader',
      options: vueLoaderConfig
    },
    {
      test: /\.js$/,
      loader: 'babel-loader',
      include: [resolve('src'), resolve('test'), resolve('node_modules/webpack-dev-server/client')]
    },
    {
      test: /\.(png|jpe?g|gif|svg)(\?.*)?$/,
      loader: 'url-loader',
      options: {
        limit: 10000,
        name: utils.assetsPath('img/[name].[hash:7].[ext]')
      }
    },
    {
      test: /\.(mp4|webm|ogg|mp3|wav|flac|aac)(\?.*)?$/,
      loader: 'url-loader',
      options: {
        limit: 10000,
        name: utils.assetsPath('media/[name].[hash:7].[ext]')
      }
    },
    {
      test: /\.(woff2?|eot|ttf|otf)(\?.*)?$/,
      loader: 'url-loader',
      options: {
        limit: 10000,
        name: utils.assetsPath('fonts/[name].[hash:7].[ext]')
      }
    },
	{
	  test: /\.pug$/,
	  loader: 'pug',
	},
    {
      test: /\.less$/,
      loader: "style-loader!css-loader!less-loader",
    },
  ]
},
```
比较重要的是`url-loader`这个插件。我们在处理引入的各种资源文件时，Webpack最终会将各个模块打包成一个文件，因此我们样式中的URL路径是相对入口HTML页面的，而不是相对于原始CSS文件所在的路径的。这就会导致图片引入失败。这个问题可以使用`file-loader`解决。

`url-loader`中封装了`file-loader`，它不依赖于`file-loader`，它除了可以完成`file-loader`的作用，还可以将符合要求的图片进行编码生成DataURL，减少HTTP请求数目。`url-loader`选项中提供了`limit`参数，小于这个参数的文件才会转换为DataURL

## 路径

明确一下关于路径的表示，

```BASH
./        表示当前目录，相对地址
../       表示上层目录，绝对地址
/         表示根目录，绝对地址
```
`__dirname`表示当前文件在系统中的绝对路径，比如我在`D:\projects\vue-cli-learning`文件夹下新建了一个`test.js`文件：

```JS
console.log(__dirname)
```
在Node环境下运行这个文件输出结果就是：

```BASH
D:\projects\vue-cli-learning
```
path.resolve的目的是用来将相对路径转为绝对路径，接受多册参数，一次表示要进入的路径，直到最后一个参数为止。除了根目录，该方法的返回值都不带尾部的斜杠

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

比如，在刚才的`test.js`中打印：

```JS
const path = require('path');
console.log(path.resolve(__dirname, 'a'));
console.log(path.resolve(__dirname, '../a'));
```
输出的结果就是两个路径：

```BASH
D:\projects\vue-cli-learning\a
D:\projects\a
```

## 打包路径的配置

在处理字体文件的`url-loader`的配置选项中：

```BASH
name: utils.assetsPath('fonts/[name].[hash:7].[ext]')
```
规定了Webpack打包后生成的资源名称和路径，其中，`utils.assetsPath`规定了跨系统平台输出文件路径，并且和环境变量相关：

```JS
exports.assetsPath = function (_path) {
  const assetsSubDirectory = process.env.NODE_ENV === 'production'
    ? config.build.assetsSubDirectory
    : config.dev.assetsSubDirectory

  return path.posix.join(assetsSubDirectory, _path)
}
```

在生产环境下的`config.build.assetsSubDirectory`配置实在`/config/index.js`中：

```JS
build: {
  // Template for index.html
  index: path.resolve(__dirname, '../dist/index.html'),
  
  // Paths
  assetsRoot: path.resolve(__dirname, '../dist/'),
  assetsSubDirectory: 'static',
  assetsPublicPath: '/',
  
  // ... 其他
}
```
其中，

`index`是生成的`index.html`在本机的地址，现在的配置下，生成的`index`文件会位于`D:\projects\vue-cli-learning\dist\`中（由`path.resolve`转换而来）

`assetsRoot`指明了构建后的资源的路径，Webpack会将所有打包后的资源（包括`index.html`在内都放到这个文件夹下

`assetsSubDirectory`是打包后的资源目录，除了`index.html`之外的所有模块都会放到这个文件夹中，当前配置后，所有的资源文件都会打包在`D:\projects\vue-cli-learning\dist\static`中

`assetsPublicPath`是`static`资源文件夹相对于HTTP服务器运行时的路径，比如我利用`http`模块构建了一个简单的静态服务器：

```JS
npx http-server -a 127.0.0.1 -p 7070
```
这个时候服务器运行的URL是`localhost:7070/`

当前目录的情况是这样：

![捕获333.PNG](http://image.oldzhou.cn/捕获333.PNG)

如果我在`App.vue`中引入的字体文件路径是`./assets/font/d.ttf)`，在打包之后首先由对应的`url-loader`处理，生成的字体文件将位于`/dist/static/fonts`中，打包后静态服务器请求的字体地址是：`http://127.0.0.1:7070/static/fonts/d.d30126a.ttf`

明白这些选项都是干什么的后，就可以更改了

（1）更改打包后的文件路径，更改`index`选项`assetsRoot`选项，比如更改为：

```JS
build: {
  // Template for index.html
  index: path.resolve(__dirname, '../dist/test/index.html'),
  
  // Paths
  assetsRoot: path.resolve(__dirname, '../dist/test/'),
  assetsSubDirectory: 'static',
  assetsPublicPath: '/',
  
  // ... 其他
}
```

这时候打包后文件将是这样的结构：

![捕获123.PNG](http://image.oldzhou.cn/捕获123.PNG)

（2）更改打包后静态资源所在目录（同时对资源的请求也会更改，因为打包后内联的URL地址也由`url-loader`一并改变了

更改为：

```JS
build: {
  // Template for index.html
  index: path.resolve(__dirname, '../dist/index.html'),
  
  // Paths
  assetsRoot: path.resolve(__dirname, '../dist/'),
  assetsSubDirectory: 'static/hello/',
  assetsPublicPath: '/',
  
  // ... 其他
}
```
此时打包生成的目录：

![捕获31.PNG](http://image.oldzhou.cn/捕获31.PNG)

发出的获取字体的网络请求的URL为`http://127.0.0.1:7070/static/hello/fonts/d.d30126a.ttf`

（3） 更改HTTP服务器获取静态资源的路径

这个一直把我搞混了，在默认的配置情况下是去网站的根目录下去寻找`assetsSubDirectory`中规定的资源目录，比如`http://127.0.0.1:7070/static/fonts/d.d30126a.ttf`

但是如果资源不是在根目录下呢，比如需要通过`http://127.0.0.1:7070/ok/static/fonts/d.d30126a.ttf`来请求字体时，就需要改动`assetsPublicPath`了，此时的配置：

```JS
build: {
  // Template for index.html
  index: path.resolve(__dirname, '../dist/index.html'),
  
  // Paths
  assetsRoot: path.resolve(__dirname, '../dist/'),
  assetsSubDirectory: 'static/',
  assetsPublicPath: '/ok/',
  
  // ... 其他
}
```
这个时候打包后的目录没有变化，但是请求字体的URL就变成了`http://127.0.0.1:7070/ok/static/fonts/d.d30126a.ttf`

如果要将`assetsPublicPath`改为`./`会发生什么呢？请求字体的URL发生了变化

这是因为`./`是相对地址，相对发出请求的文件所在的木而言的，这个请求是谁发出的呢？是打包后的CSS文件发出的

![捕获g.PNG](http://image.oldzhou.cn/捕获g.PNG)

转换为绝对路径就是`/static/css/`，然后再这个目录下再去寻找资源目录，所以请求的URL是`http://127.0.0.1:7070/static/css/static/fonts/d.d30126a.ttf`，这肯定不是我们想要的

所以在更改`assetsPublicPath`时，尽量采用绝对地址，从服务器的根节点域名触发，定位资源文件，不容易发生错误。


## 总结

回到初衷，本意只是简单的更改打包后的文件路径，想要打包在`/dist/project`中，只需要简单的更改`index`选项和`assertsRoot`即可

```JS
build: {
  index: path.resolve(__dirname, '../dist/project/index.html'),
  assetsRoot: path.resolve(__dirname, '../dist/project/'),
}
```
这两项只会影响打包后在本地的路径，而`assetsSubDirectory`会影响请求和本地打包后路径，`assetsPublicPath`只会影响上线后发送请求的路径。

简单的一点改动，引出了一堆东西要学习。


## 参考
- [webpack学习笔记-2-file-loader和url-loader@CSDN](https://blog.csdn.net/qq_38652603/article/details/73835153)
- [Path模块@JavaScript标准参考教程](http://javascript.ruanyifeng.com/nodejs/path.html#toc1)
- [webpack再入门，说一下那些不入流的知识点@segmentfault](https://segmentfault.com/a/1190000010627001)
- [webpack的3个路径配置项@博客园](https://www.cnblogs.com/cag2050/p/7300756.html)
