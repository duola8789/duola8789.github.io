---
title: VueRouter04 路由懒加载
top: false
date: 2019-09-18 19:30:15
updated: 2019-09-18 19:30:17
tags:
- Vue Router
- 懒加载
categories: Vue
---

Vue router 路由懒加载学习笔记。

<!-- more -->

## 路由懒加载

路由懒加载解决的是一次加载的JS包过大的问题，将整个的JS包按照不同的路由对应的组件分割成为不同的代码块，路由访问的时候再加载对应的组件

这样的好处就是首次加载时的速度很加快，但是路由切换的过程，由于组件本身也需要通过网络请求获取，所以性能会降低

实现需要借助Vue的[异步组件](https://cn.vuejs.org/v2/guide/components-dynamic-async.html#%E5%BC%82%E6%AD%A5%E7%BB%84%E4%BB%B6)和Webpack的[代码分割功能](https://webpack.docschina.org/guides/code-splitting/)

## 实践

（1）配置Babel

如果使用了Babel，需要安装`syntax-dynamic-import`插件，才可以使Babel正确解析下面的语法。Babel@6需要使用[babel-plugin-syntax-dynamic-import](https://www.npmjs.com/package/babel-plugin-syntax-dynamic-import)，Babel@7使用[@babel/plugin-syntax-dynamic-import](https://babeljs.io/docs/en/babel-plugin-syntax-dynamic-import)

同时为了防止Eslint报错：`Parsing error: Unexpected token import`，需要使用babel-eslint，方法参考[这里](https://stackoverflow.com/questions/47815775/dynamic-imports-for-code-splitting-cause-eslint-parsing-error-import)。


（2）需要将异步组件定义为一个返回Promise的工厂函数：

```JS
const Foo = () => Promise.resolve({ /* 组件定义 */);
```

也可以提供一个定义异步组件的工厂函数：


（3）使用Webpack 2 提供的动态`import`语法来定义代码分块点

```JS
import('./Foo.vue') // 返回 Promise
```

实际工作中更常见的是提供一个定义异步组件的工厂函数，我们只要传入组件名让Webpack找到

```JS
// 懒加载工厂函数
const lazyLoad = path => () => import(/* webpackChunkName: "view-[request]-[index]" */ `@/components/${path}.vue`);
```


（4）路由配置中，仍然直接引入`Foo`即可

```JS
const router = new VueRouter({
  routes: [
    { path: '/foo', component: Foo }
  ]
})
```

## 命名Chunk

如果想把某个路由下的所有组件都打包在通个异步chunk中，那么需要使用命名chunk（需要Webpack版本2.4+）。

首先需要在Webpack的`output`配置项中，可以设置打包出的代码块（chunk）的名称`chunkFilename`，不设置的话`chunkFilename`默认使用`[id].js`（[文档](https://webpack.docschina.org/configuration/output/#output-chunkfilename)），我们在`import`中采用的特殊注释注入的`webpackChunkName`是无法生效的

此时各个代码块的名字都是默认的`id`：

![](https://imgconvert.csdnimg.cn/aHR0cDovL2ltYWdlLm9sZHpob3UuY24vRmhjck5KaUNfNW5Bb0VmZVVJZm1tUDZrNUpBaw?x-oss-process=image/format,png)

想要我们注入的`webpackChunkName`生效，就需要在Webpack的配置文件`webpack.base.conf.js`中的`output`选项中，添加如下配置：

```JS
module.exports = {
  output: {
    filename: '[name].js', 
    chunkFilename: '[name].chunk.js', // 新增
  },
}
```
这样设置后，我们利用特殊的注释语法来设置的`webpackChunkName`就会替换`[name]`占位符

```JS
const Foo = () => import(/* webpackChunkName: "group-foo" */ './Foo.vue')
const Bar = () => import(/* webpackChunkName: "group-foo" */ './Bar.vue')
const Baz = () => import(/* webpackChunkName: "group-foo" */ './Baz.vue')
```

这样三个组件会打打包到同一个`chunk`中，不会再被分割，也就不会动态加载。

也可以在`/* webpackChunkName: "group-foo" */`中使用Webpack提供的内置的[模板字符串](https://webpack.docschina.org/configuration/output/#output-filename)

![](https://imgconvert.csdnimg.cn/aHR0cDovL2ltYWdlLm9sZHpob3UuY24vRm15MXRvenVORkxmMWZQVWVlQkZpUWtDWDJDTw?x-oss-process=image/format,png)

一般可以使用`/* webpackChunkName: "view-[request]-[index]" */`，`request`并没有在上面的模板中列出来，但是它可以输出我们请求这个模板的路径：

![](https://imgconvert.csdnimg.cn/aHR0cDovL2ltYWdlLm9sZHpob3UuY24vRmhiTmxzcV81VkFDNGVVQ0VaNDh3WFlwa0pUMA?x-oss-process=image/format,png)

这样设置对于调试显示信息可能更全一点，要注意的事是，这个`webpackChunkName`的设置只能通过Webpack内置的模板字符串传入变量，不支持在业务代码中手动传入值，比如说

```JS
// 懒加载工厂函数
const lazyLoad = (view, str) => () => import(`/* webpackChunkName: "view${str}-[request]-[index]" */` `@/components/demos/${view}.vue`);
```

这样传入`str`是不能生效的

## 优化

当路由增多时，`routes`中每个`cmponent`都需要按照如上的样式实现懒加载，并且在开发环境中使用懒加载，会导致代码更改的热跟新速度变慢，所以需要区分环境来使用路由的懒加载功能

可以对`lazyLoad`函数进行优化：

```JS
const lazyLoad = path => {
  if (process.env.NODE_ENV === 'development') {
    const comp = require(`@/pages/${path}.vue`);
    return comp.default || comp
  }
  return () => import(/* webpackChunkName: "view-[request]-[index]" */ `@/pages/${path}.vue`);
};
```

懒加载完整的Demo在[这里](https://github.com/duola8789/vue-cli-learning/tree/feat-lazyLoad)。

## 参考
- [路由懒加载@Vue Router](https://router.vuejs.org/zh/guide/advanced/lazy-loading.html)
- [使用import()配合webpack动态导入模块时，如何指定chunk name？@github](https://note.youdao.com/)
- [「Vue.js」Vue-Router + Webpack 路由懒加载实现@segmentfault](https://segmentfault.com/a/1190000015904599)
- [output.chunkFilename@webpack](https://webpack.docschina.org/configuration/output/#output-chunkfilename)
- [output.filename@webpack](https://webpack.docschina.org/configuration/output/#output-filename)
