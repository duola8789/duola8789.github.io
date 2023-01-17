---
title: Nginx03 root、alias与try_files
top: false
date: 2020-07-27 11:47:32
updated: 2020-07-27 11:47:34
tags:
- Nginx
- vue-router
categories: 全栈开发
---

`root`、`alias`与`try_files`的配置与应用。

<!-- more -->

# `root`与`alias`的区别

`root`与`alias`的区别主要在于Nginx如何解释`location`后面的路径的URI，这会使两者分别以不同的方式将请求映射到服务器文件上。具体来看：

- `root`的处理结果是：`root`路径+`location`路径
- `alias`的处理结果是：使用`alias`路径替换`location`路径

例如，如下的Nginx配置：

```NGINX
server {
    listen              9001;
    server_name         localhost;
    location /hello {
        root            /usr/local/var/www/;
    }
}
```

在请求`http://localhost:9001/hello/`时，服务器返回的路径地址应该是`/usr/local/var/www/hello/index.html`

当使用`alias`时：

```NGINX
server {
    listen              9001;
    server_name         localhost;
    location /hello {
        alias           /usr/local/var/www/;
    }
}
```

在请求`http://localhost:9001/hello/`时，服务器返回的路径地址应该是`/usr/local/var/www/index.html`（用`alias`后面的路径将请求的`location`的地址`hello`替换掉）

另外，要注意的是，`alias`路径后面必须使用`/`结束，否则无法匹配正确的路径，而`root`后则可有可无。所以建议都加上`/`，反之出错

# `try_files`

接触到`try_files`，是在对vue-router进行`History`模式进行配置时，官网给出的[例子](https://router.vuejs.org/zh/guide/essentials/history-mode.html#%E5%90%8E%E7%AB%AF%E9%85%8D%E7%BD%AE%E4%BE%8B%E5%AD%90)中使用到了`try_files`：

```NGINX
location / {
    try_files $uri $uri/ /index.html;
}
```

`try_files`的作用就是在匹配`location`的路径时，如果没有匹配到对应的路径的话，提供一个回退的方案，在上面的例子中：`$uri`就代表匹配到的路径。例如我们访问`/demo`时，`$uri`就是`demo`

`try_files`的最后一个参数是回退方案，前两个参数的意思是，保证如果能够匹配到路径的话访问匹配到的路径，例如访问`/demo`时，如果根路径下没有`demo`目录，就会访问`index.html`

要注意到的是，`/index.html`的服务器路径，会到`root`路径下去寻找，上面的例子中，没有`root`选项，那么就会到Nginx的默认根路径下去寻找`index.html`，如果设置了`root: a`，那么就会到服务器的`/a/index.html`路径下去匹配

# 在非根路径下使用`try_files`

当我们希望在`/test`路径下部署一个路由使用History的Vue应用，那么可以使用如下的Nginx配置：

```NGINX
server {
    listen              9001;
    server_name         localhost;
    location /test {
        root            /usr/local/var/www/hello/;
        index           index.html;
        try_files       $uri $uri/ /test/index.html;
    }
}
```

这个时候：

- 非`/test`路径下的请求，不会收到上面的配置的影响
- 当访问`/test`时，会使用`root`的匹配规则，到服务器`/usr/local/var/www/hello/test`路径下寻找`index.html`文件
- 当访问`/test/demo1`时，会使用`try_files`的匹配规则，到`root`路径下去寻找最后一个参数`/test/index.html`的回退方案，也就是说去`/usr/local/var/www/hello/test`路径下寻找`index.html`文件

除了Nginx的配置外，Vue应用本身还需要配置两处：

（1）vue-router实例化时指定`history`模式，并添加`base`参数：

```JS
const router = new Router({
  routes,
  mode: 'history',
  base: '/test'
});
```

（2）静态资源的目录`publicPath`设置为相对路径（否则回去绝对路径下寻找JS等静态资源）

```JS
{
    assetsPublicPath: './'
}
```

# 参考

- [Nginx的location、root、alias指令用法和区别@腾讯云](https://cloud.tencent.com/developer/article/1535615/)
- [HTML5 History 模式@vue-router](https://router.vuejs.org/zh/guide/essentials/history-mode.html#%E5%90%8E%E7%AB%AF%E9%85%8D%E7%BD%AE%E4%BE%8B%E5%AD%90)
