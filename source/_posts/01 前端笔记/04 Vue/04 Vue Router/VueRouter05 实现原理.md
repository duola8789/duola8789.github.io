---
title: VueRouter05 实现原理
top: false
date: 2018-11-20 19:30:15
updated: 2019-09-18 19:31:15
tags:
- Vue Router
categories: Vue
---

Vue router 实现原理学习笔记。

<!-- more -->

## 概要

在VueRouter中提供了两种模式：

1. Hash模式
2. History模式

Hash模式的基础是当URL的`#`后的参数改变时浏览器不会发送请求

History模式的基础是`pushState`和`replaceState`修改浏览器的历史栈后浏览器不会立即发送请求

## 前端路由

前端路由是通过改变URL，在不重新刷新整体页面的情况下，更新页面视图。

目前浏览器实现这一功能主要通过两种方式：

1. 利用URL中的hash值
2. 利用H5中的`history`对象

## Hash模式

vue-router默认使用的是hash模式，使用URL的hash来模拟一个完整的URL，当URL改变时，页面不会重新加载。

### `#`的含义

`#`代表网页中的位置，其莜面的字符，就是该位置的标志符号

```
http://www.example.com/index.html#print
```
浏览器读取这个URL周，会自动将`print`位置滚动至可视区域，对应`print`位置可以通过两种方法实现：使用锚点`<a name="print"></a>`或者使用id属性`<div id="print"></div>`

### `#`的特性

`#`是用来指导浏览器动作的，对服务器端完全无用，==HTTP请求中不包括`#`及后面的内容字符==

单单改变`#`后面的部分，只会可能触发浏览器的滚动，==不会重新加载网页==。

改变了`#`后面的部分，==都会改变浏览器的访问历史==，使用后退按钮，就可以回到上一个位置,这对于Ajax应用程序特别有用，可以用不同的`#`值，表示不同的访问状态，然后向用户给出可以访问某个状态的链接。

### 使用

通过`window.location.hash`可以读取、写入页面的hash值，读取时，可以用来判断网页状态是否改变；写入时，则会在不重载网页的前提下，创造一条访问历史记录。

HTML5中增加的`onhashchange`事件，当`#`发生变化时，就会触发这个事件

### VueRouter中的实现

首先构建了`HashHistory`构造函数，获取页面的hash值

然后定义了`HashHistory.push()`方法，当页面的hash值发生变化时，会替换`window.location.hash`，hash的改变会自动添加到浏览器的访问历史记录中，视图的更新是首先通过`Vue.mixin()`方法，全局注册一个混合，定义了响应式的`_route`属性，当`_route`改变时会触发Vue实例的`render`方法，更新视图

```
graph TB
$router.push-->HashHistory.push
HashHistory.push-->History.transitionTo
History.transitionTo-->History.updateRoute
History.updateRoute-->app._route=route
app._route=route-->vm.render
```

`HashHistory.replace()`与`push()`方法不同之处在于，它并不是将新路由添加到浏览器访问历史栈顶，而是替换掉当前的路由

上面的`VueRouter.push()`和`VueRouter.replace()`是可以在Vue组件的逻辑代码中直接调用的，除此之外在浏览器中，用户还可以直接在浏览器地址栏中输入改变路由，因此还需要监听浏览器地址栏中路由的变化，并具有与通过代码调用相同的响应行为，在HashHistory中这一功能通过`setupListeners`监听`hashchange`实现

详细的源码解读看[这篇文章](https://juejin.im/post/5b08c9ccf265da0dd527d98d)。

## HTML History模式

History interface是浏览器历史记录栈提供的接口，通过`back()`，`forward()`，`go()`等方法，我们可以读取浏览器历史记录栈的信息，进行各种跳转操作。

`history.pushState()`和`history.replaceState()`方法，分别可以添加和修改历史记录条目。这些方法通常与`window.onpopstate`配合使用。

```JS
window.history.pushState(stateObject,title,url)
window.history.replaceState(stateObject,title,url)
```
- `stateObject`：当浏览器跳转到新的状态时，将触发`popState`事件，该事件将携带这个`stateObject`参数的副本
- `title`：所添加记录的标题
- `url`：所添加记录的url(可选的)

`pushState`和`replaceState`两种方法的共同特点：==当调用修改浏览器历史栈后，虽然当前url改变了，但浏览器不会立即发送请求该url==，这就为单页应用前端路由，更新视图但不重新请求页面提供了基础。

### VueRouter中使用History模式

VueRouter也提供了Hsitory模式，利用的是H5中的`history`对象和事件，只需要在新建VueRouter的实例时传入`mode`参数：

```JS
const router = new VueRouter({
  mode: 'history',
  routes: [...]
})
```
这种模式下页面的URL就像正常的url，例如`http://yoursite.com/user/id`

这种模式需要后台配置支持，如果后台没有正确的配置，当用户在浏览器直接访问`http://oursite.com/user/id`就会返回404，所以需要在服务端增加一个覆盖所有情况的候选资源：如果URL匹配不到任何静态资源，则应该返回同一个`index.html`页面，这个页面就是app依赖的页面。

注意：

这么做以后，服务器就不再返回404错误页面，因为对于所有路径都会返回`index.html`文件。为了避免这种情况，应该在Vue应用里面覆盖所有的路由情况，然后在给出一个404页面。

```JS
const router = new VueRouter({
  mode: 'history',
  routes: [
    { path: '*', component: NotFoundComponent }
  ]
})
```

### VueRouter中的实现

代码结构以及更新视图的逻辑与hash模式基本类似，只不过将对`window.location.hash`直接进行赋值或者调用`window.location.replace()`改为了调用`history.pushState()`和`history.replaceState()`方法。

## 两种模式的比较

一般的需求场景中，Hash模式与History模式是差不多的，根据MDN的介绍，调用`history.pushState()`相比于直接修改Hash主要有以下优势：

1. `pushState`设置的新url可以是与当前url同源的任意url,而hash只可修改`#`后面的部分，故只可设置与当前同文档的url
2. `pushState`设置的新url可以与当前url一模一样，这样也会把记录添加到栈中，而Hash设置的新值必须与原来不一样才会触发记录添加到栈中
3. `pushState`通过`stateObject`可以添加任意类型的数据记录中，而Hash只可添加短字符串
4. `pushState`可额外设置`title`属性供后续使用

## 参考

- [vue-router原理剖析@掘金](https://juejin.im/post/5b08c9ccf265da0dd527d98d)
- [History模式@Vue Router](https://router.vuejs.org/zh/guide/essentials/history-mode.html#html5-history-%E6%A8%A1%E5%BC%8F)
- [URL的井号@阮一峰的网络日志](http://www.ruanyifeng.com/blog/2011/03/url_hash.html)
