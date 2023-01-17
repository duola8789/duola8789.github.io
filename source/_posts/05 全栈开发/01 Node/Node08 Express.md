---
title: Node08 Express
top: false
date: 2017-04-04 09:18:17
updated: 2019-04-30 11:06:03
tags:
- Express
categories: Node
---

重新学习Node，整理以前的日志。Express学习笔记。

<!-- more -->

## 简介

express是基于nodejs平台的web框架，它可以让我们快速开发出==web应用==。Express主要解决了 请求路由 和 视图模板 的问题

安装express项目的命令如下

```
express -e nodejs-product
//-e, --ejs add ejs engine support 
//-J, --jshtml add jshtml engine support (defaults to jade)

```
## 验证版本

如果是express 4.0之前版本，那么执行“express -V”就可以得到版本号了，可express 4.0之后还需要再安装express-generator包，如果没有安装还执行“express -V”命令会报错。


```
$ npm install -g express-generator
express -V
```


## 项目目录

![image](http://note.youdao.com/yws/public/resource/676feab4918c01e107b827b44bd71894/xmlnote/3FC2ADAAECA44BF5BE60E8A1038AA4D1/19981)

- bin:真实的执行程序（在命令行输入express命令时，其实对应的其实是去执行bin目录下的express程序）。

- app.js： 项目的启动文件(项目入口)，可以改成index.js或者main.js都成

- node_modules： 存放项目的依赖库

- package.json： 项目依赖配置及开发者信息

- public： 静态文件如 css,js,img (PS:俺其实习惯叫static)

- routes： 路由文件

- Views： 页面文件(Ejs或者jade的模板，默认是jade)

express() 表示创建express应用程序。简单几行代码其实就可以创建一个应用，如下：

![image](http://note.youdao.com/yws/public/resource/676feab4918c01e107b827b44bd71894/xmlnote/C0F2241788CF4B50A64846C77F449033/19933)


express默认的模版是jade模版，想要更改模版需要在app.js中更改：

```
// 原本是“app.set('view engine', 'ejs');
// 修改为
app.engine('.html', ejs.__express);
app.set('view engine', 'html');
```
app.engine方法用来重新设置模板文件的扩展名，上面的代码意思是用ejs模板引擎来处理“.html”后缀的文件

```
app.engine(ext, callback) 
//注册模板引擎的 callback 用来处理ext扩展名的文件。
```
PS：__express不用去care，其实就是ejs模块的一个公共属性，表示要渲染的文件扩展名。

app.use用来指定中间件function：
```
app.use([path], function)
```
可选参数path默认为"/"。使用 app.use() “定义的”中间件的顺序非常重要，它们将会顺序执行，use的先后顺序决定了中间件的优先级;

app.render用来渲染view同时传进对应的数据:
     

```
app.render(view, [options], callback)
// 渲染 view, callback 用来处理返回的渲染后的字符串。
```
## app.js

app.js是项目的主文件,为这个主文件里面有创建服务和监听端口的语句：

```
var express = require('express'); //引入express框架
var session = require('express-session'); //设置session的中间件
var redisStore = require('connect-redis')(session); //实现redis存储session
var glob = require('glob'); //使用类似shell的模式语法匹配文件路径
var path = require('path'); //path模块用于处理和转换文件路径
var bodyParser = require('body-parser'); //解析请求的body的中间件

var swig = require('swig'); //swig模板引擎
var staticTag = require('./swig/static'); //swig模板相关设置
var morgan = require("morgan"); //控制台日志

var app = express(); //创建一个express应用

app.locals.ENV = NODE_ENV; //将环境变量NODE_ENV存在app.locals里
app.locals.ENV_DEV = (NODE_ENV === 'dev'); //是否是dev环境


// view engine setup
app.engine('html', swig.renderFile); //使用swig渲染html文件
app.set('view engine', 'html'); //设置默认页面扩展名
app.set('view cache', false); //设置模板编译无缓存
app.set('views', path.join(__dirname, 'views')); //设置项目的页面文件，也就是html文件的位置
swig.setDefaults({cache: false}); //关闭swig模板缓存
swig.setDefaults({loader: swig.loaders.fs(__dirname + '/views')}); //从文件载入模板，请写绝对路径，不要使用相对路径

staticTag.init(swig); //这个init函数是自定义的，对swig模板做了一些自定义设置

app.use(session({ //设置session中间件的写法，session会存在服务端
    cookie: {
        maxAge: 2502000 * 1000 //设置最大生命周期，过了这个时间后cookie会失效，单位毫秒
    },
    name: 'lbn_sid', //用来保存session的cookie名称
    secret: 'what are you thinking?', //用来对session数据进行加密的字符串.这个属性值为必须指定的属性
    store: new redisStore({ //设置session的存储仓库为redis数据库
        ttl: 2502000, //redis session生命周期，单位秒
        url: REDIS_URL //redis缓存服务地址
    }),
    saveUninitialized: false, //false选项不会强制存储未初始化的session到redis里，未初始化意味着新的还没有修改的
    resave: false //如果是true选项，强制重新存储session到redis里，即使session没有被修改，false意味着如果没有变化就不用重新存
}));
// app.use(morgan('combined')); //morgan控制台日志，会在控制台输出所有http请求日志，combined是标准Apache日志格式
app.use(bodyParser.json()); //bodyParser.json是用来解析请求体的json数据格式
app.use(bodyParser.urlencoded({
    extended: true,
    limit: '10mb'
}));
/*bodyParser.urlencoded则是用来解析我们通常的form表单提交的数据，
 *也就是请求头中包含这样的信息： Content-Type: application/x-www-form-urlencoded
 *extended选项为true会使用qs library来解析数据，false会使用querystring来解析
 *limit选项限制请求体的大小
 */

app.use('/' + global.STATIC_URL, express.static(path.join(__dirname, STATIC_DIR)));
//为静态资源的请求添加虚拟路径，只有请求静态资源的路径前加了global.STATIC_URL前缀后，才可请求成功

var controllers = glob.sync('./controllers/*.js'); //获取到controllers文件夹下的所有js文件，这些文件里都是路由
controllers.forEach(function (controller) {
    require(controller)(app);
});
//将所有路由循环到主文件中使其生效

app.use(function (err, req, res, next) { //当请求出现500错误，渲染500错误页面
    res.locals = {env: NODE_ENV};
    // treat as 404
    if (err.message
        && (~err.message.indexOf('not found')
        || (~err.message.indexOf('Cast to ObjectId failed')))) {
        return next();
    }
    res.status(500).render('500', { error: err.stack });
});

app.use(function (req, res, next) { //当请求出现404错误，渲染404错误页面
    res.status(404).render('404', {
        url: req.originalUrl,
        error: 'Not found'
    });
});

module.exports = app; //将app应用导出成模块
```

这其中session的设置值得注意，session的设置写在了app.use()中，也就是中间件中，中间件也是路由，只是所有的请求都会经过它的处理。这里设置session时有一个cookie的设置，这个cookie就是session的唯一标示，是sessionId，也就是说，第一次访问网站的时候，在请求通过session设置的中间件时，响应头里会设置一个set-cookie来强制浏览器存储一个cookie，也就是在浏览器存下sessionId，然后会在node端新建一个session，这里浏览器存的sessionId和node端的session是对应关系，之后的请求也会经过session设置的中间件，此时的请求头里会自动带上浏览器的所有cookie，当中间件发现已经有sessionId的时候，就不会新建了，只用更新对应的session就可以了。


## bin
express4.0之后的版本，项目目录下会有bin/这个目录，这个目录专门用于自定义启动脚本，这样就把与启动服务的代码和主文件分离了，而且你可以定义多个启动脚本，而不用去修改app.js这个主文件。

文件内容：

```
#!/usr/bin/env node
/*这一句是写给类unix系统看的
 *如果用户没有将nodejs装在默认的/usr/bin路径里
 *当系统看到这一行的时候，首先会到env设置里查找nodejs的安装路径
 *再调用对应路径下的解释器程序完成操作
 */
var app = require('../app');
//引入app主应用

app.set('port', PORT || 3000);
//设置端口为环境变量.env文件里的PORT，如果.env里没有，就默认3000

var server = app.listen(app.get('port'), function() {
    console.log('Express server listening on port ', app.get('port'), " with pid ", process.pid);
});
/*app.listen(path,[callback])的写法是启动一个socket连接，然后在给定的端口上监听连接
 *app.get(name)的写法是获取app设置，设置的时候通过app.set('port', 'my port');来设置。
 */
```


## 路由

Web需要通过不同uri区分功能，如：/user/profile 表示用户信息，/about 表示网站简介，路由可以实现这一功能。

> 我个人理解的就是当在地址栏输入一个地址，要对应的页面去往何处，完成这一功能的命令就是路由。

路由用get去设置，默认的路由/对应routes.index,/user对应应routes.user

```
var express = require("express");
var http = require("http");
var app = express();

app.all("*", function(request, response, next) {
  console.log("step1");
  next();
});

app.get("/", function(request, response) {
  response.end("Home Page!");
});

app.get("/about", function(request, response) {
  response.end("About Page!");
});

app.get("*", function(request, response) {
  response.end("404!");
});

http.createServer(app).listen(1984);
```

## VIEW处理

Express加入了View处理机制，一起看看：

```
var express = require("express");
var app = express();

// 模板目录：./views
app.set("views", __dirname + "/views");

// 使用jade引擎
app.set("view engine", "jade");
 
// 寻址views/index，提交jade渲染，并返回结果
app.get("/", function(request, response) { response.render("index", { message: "I'm hyddd" }); });
```

## 参考
- http://www.cnblogs.com/Darren_code/p/node_express.html
- http://www.cnblogs.com/hyddd/p/4237099.html
- http://www.cnblogs.com/hahazexia/p/6027787.html
- http://www.cnblogs.com/hahazexia/p/6213286.html
