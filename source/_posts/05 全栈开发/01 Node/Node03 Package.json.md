---
title: Node03 Package.json
top: false
date: 2017-07-06 10:19:17
updated: 2019-04-30 10:57:39
tags:
- Package.json
categories: Node
---

重新学习Node，整理以前的日志。Package.json的学习笔记。

<!-- more -->

## 1 概述

每个项目的根目录下面，一般都有一个`package.json`文件，定义了这个项目所需要的各种模块，以及项目的配置信息（比如名称、版本、许可证等元数据）。`npm install`命令根据这个配置文件，自动下载所需的模块，也就是配置项目所需的运行和开发环境。

命令提示符执行`npm install`，则会根据`package.json`下载所有需要的包，`npm install --production`只下载`dependencies`节点的包

一个完整的`package.json`文件实例如下：

```JS
{
  "name": "Hello World",
  "version": "0.0.1",
  "author": "张三",
  "description": "第一个node.js程序",
  "keywords": ["node.js", "javascript"],
  "repository": {
    "type": "git",
    "url": "https://path/to/url"
  },
  "license": "MIT",
  "engines": {
    "node": "0.10.x"
  },
  "bugs": {
    "url": "http://path/to/bug",
    "email": "bug@example.com"
  },
  "contributors": [{
    "name": "李四",
    "email": "lisi@example.com"
  }],
  "scripts": {
    "start": "node index.js"
  },
  "dependencies": {
    "express": "latest",
    "mongoose": "~3.8.3",
    "handlebars-runtime": "~1.0.12",
    "express3-handlebars": "~0.5.0",
    "MD5": "~1.2.0"
  },
  "devDependencies": {
    "bower": "~1.2.8",
    "grunt": "~0.4.1",
    "grunt-contrib-concat": "~0.3.0",
    "grunt-contrib-jshint": "~0.7.2",
    "grunt-contrib-uglify": "~0.2.7",
    "grunt-contrib-clean": "~0.5.0",
    "browserify": "2.36.1",
    "grunt-browserify": "~1.3.0",
  }
}
```
在`package.json`中可以使用通配符：

```TEXT
"lint": "jshint *.js"
"lint": "jshint **/*.js"
```

`*`表示任意文件名，`**`表示任意一层子目录

## 2 `name`字段

`name`字段不能含有`.`和`_`，可以使用`-`，不能含有非URL安全的字符

![非URL安全的字符](http://image.oldzhou.cn/WX20190430-101213.png)

## 3 `scripts`字段

### 3.1 概述

`scripts`字段指定了运行脚本命令时的npm命令缩写，比如`start`指定了运行`npm run start`时要执行的命令

注意的是当运行`test`、`start`、`restart`和`stop`命令时可以省略`run`

npm脚本中需要执行多个任务时，如果是并行执行，使用`&`符号：

```BASH
$ npm run script1.js & npm run script2.js
```
如果是继发执行，使用`&&`符号：

```BASH
$ npm run script1.js && npm run script2.js
```

更多的细节参考[阮一峰的文章](http://www.ruanyifeng.com/blog/2016/10/npm_scripts.html)。

### 3.2 自定义脚本

执行脚本时，npm会临时自动将目录的`node_modules/.bin`加入PATH变量。这意味着，可以使用`node_modules`中任何脚本，而无需添加`node_modules/.bin`前缀。比如，当前项目的依赖里面有Mocha，只要直接写`mocha test`就可以了。

例如执行`tap`命令，你可以直接写：

```JS
"scripts": {"test": "tap test/\*.js"}
```

而不是
```

"scripts": {"test": "node_modules/.bin/tap test/\*.js"}
```

### 3.3 传递参数

如果我们在执行`npm run xxx`操作的时候想给里面的脚本传参数可以使用`-- --`，如下所示：

```JS
"scripts": {
  "test": "mocha test/", 
  "test:xunit": "npm run test -- --reporter xunit" 
}
```
这种设置对于组合一些高级配置的命令是非常有用的。

```JS
“scripts”: {
  "lint": "jshint **.js", 
  "lint:checkstyle": "npm run lint -- --reporter checkstyle > checkstyle.xml"
}
```
### 3.4 生命周期钩子

npm也在不同的生命周期提供了一些钩子，可以方便你在项目运行的不同时间点进行一些脚本的编写。

它的钩子分为两类：`pre-`和`post-`，前者是在脚本运行前，后者是在脚本运行后执行。**所有的命令脚本都可以使用钩子（包括自定义的脚本**）。

例如：运行`npm run build`，会按以下顺序执行：

```
npm run prebuild --> npm run build --> npm run postbuild
```
pre脚本和post脚本也是出口代码敏感(exit-code-sensitive)的，这意味着如果您的pre脚本以非零出口代码退出，那么NPM将立即停止，并且不运行后续脚本。

通常可以在pre脚本上执行一些准备工作，在post脚本上执行一些后续操作。


```JS
{
  "clean": "rimraf ./dist && mkdir dist",
  "prebuild": "npm run clean",
  "build": "cross-env NODE_ENV=production webpack"  
}
```

另外，还有很多额外的生命周期钩子，可以方便使用，例如[husky](https://www.npmjs.com/package/husky)和[pre-commit](https://www.npmjs.com/package/pre-commit)包提供了有关git的commit的生命周期钩子。

### 3.5 使用环境变量

[根据官网的](https://docs.npmjs.com/misc/scripts)介绍，在"scripts"中编写的脚本还可以方便使用一些内置变量，这些内置变量会在Node运行的时候放在`process.env`下，如果是shell脚本，就直接使用环境变量`$…`， 这对你编写一些脚本工具特别有用。

在`package.json`中所有的配置项都可以通过`$npm_package_`前缀拿到：

```JS
"show": "echo $npm_package_name && echo $npm_package_version"
```

如果是使用Node：

```JS
const { log } = console;
log(process.env.npm_package_name);
log(process.env.npm_package_version);
```

配置参数放在`npm_config_`前缀的环境中（你可以通过`npm config set`设置一些配置变量，下面介绍`config`的时候会介绍)，例如：

![](http://image.oldzhou.cn/18-10-31/36600210.jpg)

### 3.6 一些常用的脚本配置

引用阮老师的一些配置，可以看到配合一定的插件，npm可是实现一些很实用的功能

```JS
{
  // 删除目录
  "clean": "rimraf dist/*",
  
  // 本地搭建一个HTTP服务
  "serve": "http-server -p 9090 dist/",
  
  // 打开浏览器
  "open:dev": "opener http://localhost:9090",
  
  // 实时刷新
  "livereload": "live-reload --port 9091 dist/",
  
  // 构建HTML文件
  "build:html": "jade index.jade > dist/index.html",
  
  // 只要CSS文件有变动，就重新执行构建
  "watch:css": "watch 'npm run build:css' assets/styles/",
  
  // 只要HTML文件有变动，就重新执行构建
  "watch:html": "watch 'npm run build:html' assets/html",
  
  // 部署到Amazon S3
  "deploy:prod": "s3-cli sync ./dist/ s3://example-com/prod-site/",
  
  // 构建favicon
  "build:favicon": "node scripts/favicon.js",
}
```
### 3.7 内置脚本

npm自带[了数十个内置命令](https://docs.npmjs.com/cli/)，这些命令都可以直接通过npm执行，除了`instal`，还有很多实用的

![](http://image.oldzhou.cn/18-10-31/92329660.jpg)

- `npm -l` 列举所有npm自带的命令简介，然后通过`npm help`可以详细查看某个命令
- `npm search` 快速查询npm中的相关包（和我们去npm官网查是一样的）
- `npm root` 查看全局的`node_modules`目录
- `npm audit fix` 这个命令很实用，自动扫描您的项目漏洞，并自动安装任何兼容更新到脆弱的依赖
- `npm restart` 重新启动模块
- `npm prune` 移除当前不在`package.json`中但是存在`node_modules`中的依赖
- `npm repo` 浏览器端打开项目地址（GitHub），省去打开浏览器查找的操作！
- `npm docs` 查看项目文档，同上
- `npm home` 在浏览器端查看项目（项目主页），同上
- `npm search` 查找包含该字符串的依赖包
- `npm view [field][--json]` 列出依赖信息，包括历史版本，可以指定field来查看某个具体信息，比如（versions) 可以添加`–json`参数输出全部结果

## 4 npx

npm v5.2.0之后还引入了npx，引入这个命令的目的是为了提升开发者使用包内提供的命令行工具的体验。（在`node_modules`中，所有可执行文件，也就是`package`中带`bin`的，都会放在`node_modules/.bin中`）

举例：使用`create-react-app`创建一个react项目，老方法：

```JS
npm install -g create-react-app  
// 实际就是把package.json中的bin命令连接到了/usr/local/bin中
create-react-app my-app
```

npx方式：

```BASH
npx create-react-app my-app 
// 执行本`node_modules/.bin`中的对应命令
```
这条命令会临时安装create-react-app包，命令完成后create-react-app 会删掉，不会出现在 global 中。下次再执行，还是会重新临时安装。

npx会帮你执行依赖包里的二进制文件。

举例来说，之前我们可能会写这样的命令：

```BASH
npm i -D webpack./node_modules/.bin/webpack -v
```

如果你对bash比较熟，可能会写成这样：

```BASH
npm i -D webpack
**npm bin**/webpack -v
```

有了npx，你只需要这样：

```BASH
npm i -D webpack
npx webpack -v
```

npx会自动查找当前依赖包中的可执行文件，如果找不到，就会去PATH里找。如果依然找不到，就会帮你安装！

npx甚至支持运行远程仓库的可执行文件：

```BASH
npx github:piuccio/cowsay hello
```

再比如`npx http-server`可以一句话帮你开启一个静态服务器！（第一次运行会稍微慢一些）

```BASH
npx http-server

npx: 1 安装成功，用时 3.032 秒
Path must be a string. Received undefined
npx: 25 安装成功，用时 5.241 秒
C:\Users\zhouhao1\AppData\Roaming\npm-cache\_npx\45232\node_modules\http-server\bin\http-server
Starting up http-server, serving ./
Available on:
  http://10.234.98.23:8080
  http://192.168.56.1:8080
  http://192.168.99.1:8080
  http://127.0.0.1:8080
Hit CTRL-C to stop the server
```
指定node版本来运行npm scripts：

```BASH
npx -p node@8 npm run build
```

npx的优点：

1. 临时安装可执行依赖包，不用全局安装，不用担心长期的污染。
2. 可以执行依赖包中的命令，安装完成自动运行。
3. 自动加载`node_modules`中依赖包，不用指定`$PATH`。
4. 可以指定node版本、命令的版本，解决了不同项目使用不同版本的命令的问题。

## 5 `dependencies`字段和`devDependencies`字段

### 5.1 使用

`dependencies`字段指定了项目运行所依赖的模块，`devDependencies`指定项目开发所需要的模块。

它们都指向一个对象。该对象的各个成员，分别由模块名和对应的版本要求组成，表示依赖的模块及其版本范围。

```JS
{
  "devDependencies": {
    "browserify": "~13.0.0",
    "karma-browserify": "~5.0.1"
  }
}
```

在单独安装某个模块时：

- 使用`--save`表示将该模块写入`dependencies`属性。缩写为`-S`
- 使用`--save-dve`表示将该模块写入`devDependencies`属性。缩写为`-D`

从命令行参数字面上，我们就能看出`dependencies`、`devDependencies`的区别：

- `dependencies`表示我们要在生产环境下使用该依赖，
- `devDependencies`则表示我们仅在开发环境使用该依赖。

举个例子，我要用Webpack构建代码，所以在开发环节，它是必需的，但对普通用户来说，它是不必要的，所以安装Wbpack时，我要执行：

```BASH
npm install webpack --save-dev
```

### 5.2 版本号

版本号 `major.minor.patch` ：其中：

- `patch`：修复bug，兼容老版本，
- `minor`：新增功能，兼容老版本
- `major`：新的架构调整，不兼容老版本

常用的主要有以下几种：

- 指定版本：比如`1.2.2`
- 波浪号（`~`）：表示安装`1.2.x`的最新版本
- 插入号（`^`）：表示安装`1.x.x`的最新版本，注意如果大版本号是0，则与波浪号使用相同，例如`^0.2.3`只会安装`0.2.x`的版本，这是因为此时处于开发阶段，即使是次要版本号变动，也可能带来程序的不兼容。
- `latest`：安装最新版本

推荐的做法：不锁版本，用 `^` 引入。

在package.json中确定版本号，只能锁定本身依赖的包，但是包自身所依赖的包没有办法锁定，解决方法就是`npm shrinkwrap`或者npm 5.0版本以后增加的`lock`功能：

- 如果你使用 lock 机制，则应该将 package-lock.json 提交到 repo 中。比如 Vue 采取了该策略。
- 如果你不使用 lock 机制，则应该加入 .npmrc 文件，内容为`package-lock=false` ，并提交到 repo 中。比如 ESLint 采取了该策略。

如果你需要在程序中做版本匹配，手写是不是很麻烦，其实[npm提供了一个现成的匹配版本号的工具](https://github.com/npm/node-semver#functions)

### 5.3 区别

在做项目的时候，两者可以认为没有实质的区别，但是在发布npm包的时候二者区别很大：`dependencies`下的模块会作为依赖，一起被下载；`devDependencies`下面的模块就不会自动下载了

一般来说，开发时依赖的东西需要安装在`devDependencies`字段中，比如转义用的`babel`，打包用的`webpack`等，如果发布后还需要使用的则要安装在`dependencies`，比如`vue`、`vue-router`等

> （2019.10.16）
> 
> 今天发现之前的理解有些片面，这二者在`install`的时候有差别，默认的`npm install`会同时安装`devDependencies`和`dependencies`的依赖，但是如果是发布到服务器上，依赖环境也会安装在服务器，那么可以执行`npm install -production`，那么只会安装`dependencies`下的依赖，`install`的速度会更快。
> 
> 但是在Webpack打包`build`文件时，并不会因为模块处于`devDependencies`就不会打包，也不会因为模块处于`dependencies`就一定打包，是否打包决定于是否在代码中引入`import`。只要`import`了，无论模块写在哪里，都会被打包。但是如果`import`之后，模块并没有使用，模块也会被打包，这就是Tree Shaking的作用，移除引入后但是没有使用到的代码。

## 6 其他字段

### 6.1 `main`字段

`main`字段指定了加载的入口文件，`require('moduleName')`就会加载这个文件。这个字段的默认值是模块根目录下面的`index.js`

### 6.2 `cinfig`字段

用于添加命令行的环境变量，例如：

```JS
{
  "name" : "foo",
  "config" : { "port" : "8080" },
  "scripts" : { "start" : "node server.js" }
}
```
然后，在`server.js`脚本就可以引用`config`字段的值。

```JS
http
  .createServer(...)
  .listen(process.env.npm_package_config_port)
```
用户可以改变这个值。

```
$ npm config set foo:port 80
```

### 6.3 `keywords`和`description`字段

`description`是字符串，`keywords`是字符串数组，简单地说，这两个东东是npm搜索系统中的搜索条件，所以。如果你试图发布的是一个开源插件，那么这两个字段你应该重视

### 6.4 `license` 字段

指定的项目的许可证，它告诉他人他们是否有权利使用你的包，以及，在使用你的包的时候他们应该受到怎样的限制

![license](https://ws1.sinaimg.cn/large/6483fd8bly1fszy98rplij20m80dw3zd.jpg)

### 6.5 `author`字段

可以是一个字符串，也可以是一个对象。如果传入对象，要包含三个属性：

- `name`属性(必填)
- `email`属性（选填）
- `URL`属性（选填）

### 6.5 `engines`字段

指明了该模块运行的平台，比如Node的某个版本或者浏览器

```JS
{ "engines" : { "node" : ">=0.10.3 <0.12" } }
```
也可以指定适用的npm版本。

```JS
{ "engines" : { "npm" : "~1.0.20" } }
```

### 6.6 `os`字段

指明了用户执行的操作系统：

```JS
{ "os": [ "darwin", "linux", "!win32" ] }
```
## 7 安装非NPM上发布的包

通常，我们安装的包都是在npm官网上，通过版本标明。但是，如果我想使用没上传到npm上的包怎么办？其实你可以直接加网址或git url。

git url可以是以下形式：

```TEXT
git://github.com/user/project.git#commit-ish
git+ssh://user@hostname:project.git#commit-is
hgit+http://user@hostname/project/blah.git#commit-ish
git+https://user@hostname/project/blah.git#commit-ish
```
其中，`commit-ish`可以是任意的`tag`，`branch`，`sha`。

## 8 更新

我们知道npm自带的`npm update`可以根据`pacaage.json`的版本号更新包，但是你==需要手动的更新版本号==，因此出现了升级插件`npm-check-updates`，可以自动搜索当前包的更新情况，并且修改`pacage.json`

```BASH
$ npm install -g npm-check-updates
```

`ncu`是`npm-check-updates`的缩写命令

```BASH
$ ncu  -v 
# 查询版本号

$ ncu  
# 直接输入ncu可以查看所有需要更新的包
```
![](http://image.oldzhou.cn/18-10-31/87493875.jpg)

```BASH
$ ncu -u  
# 更新所有的包，并修改package.json文件

$ ncu -f regex 
# 只匹配特定的正则格式的包

$ ncu -g 
# 更新全局包
```

## 参考
- http://javascript.ruanyifeng.com/nodejs/packagejson.html
- http://www.cnblogs.com/penghuwan/p/7134046.html
- http://guxinyan.github.io/2017/11/02/%E5%8C%85%E5%BA%94%E8%AF%A5%E6%94%BE%E5%9C%A8devDependencies%E8%BF%98%E6%98%AFdependencies/
- https://blog.zfanw.com/difference-between-dependencies-and-devdependencies/
