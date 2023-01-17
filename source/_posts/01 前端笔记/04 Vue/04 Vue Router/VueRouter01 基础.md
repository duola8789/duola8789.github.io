---
title: VueRouter01 基础
top: false
date: 2018-01-11 17:46:54
updated: 2019-09-18 19:24:48
tags:
- Vue
- Vue Router
categories: Vue
---

Vue Router基础学习笔记。

<!-- more -->

## 安装

```BASH
npm install vue-router
```

## 加载

需要使用`Vue.use()`明确的安装路由

```JS
// main.js
import Vue from 'vue'
import VueRouter from 'vue-router'

Vue.use(VueRouter)
```

##  基本使用

```HTML
<div id="app">
  <h3>Hello Vue Router</h3>
  <template>
    <router-link to="/foo">Go to foo</router-link>
    <router-link to="/bar">Go to bar</router-link>
  </template>
  <router-view></router-view>
</div>
<script>
  'use strict';
  const Foo = {template: '<div> Hello Foo </div>'},
    Bar = {template: '<div> Hello Bar </div>'};

  const routes = [
    {path: '/foo', component: Foo},
    {path: '/bar', component: Bar}
  ];

  const router = new VueRouter({
    routes
  });

  let app = new Vue({
    router
  }).$mount('#app')
</script>
```

（1）用`<router-link>`实现导航标签，在DOM中会被渲染为`<a>`标签，`to`属性指定链接

（2）用`<router-view>`作为路由出口，路由对应的组件内容会被渲染到其中

（3）定义路由`routes`，由对象组成的数组，对象至少包含的属性是`path`和`component`，其中`path`和`<router-link>`中的`to`属性对应，`to`属性如果不加`/`则是以当前地址为基准的相对路径

（4）创建路由实例`new VueRouter({routes})`

（5）创建Vue实例时可以直接添加`router`属性挂载路由

## 动态路由

在`routes`数组的`path`属性中，以`:`开始后面的参数就是动态路径参数：

```JS
const router = new VueRouter({
  routes: [
    // 动态路径参数 以冒号开头
    { path: '/user/:id', component: User }
  ]
})
```

当访问`/user/a`时，可以在组件中通过`$route.params.id`获取到动态路径参数`a`

可以通过`$route.query`获取URL中的查询参数，通过`$route.hash`获取路由的hash值

动态路由间的跳转，比如从`/user/a`到`/user/b`，原组件实例会被复用，所以Vue实例的`mounted`等生命周期钩子函数不会执行。如果想要响应参数变化有两种方法：

（1）`watch`组件的`$route`对象:

```JS
const User = {
  watch: {
    '$route' (to, from) {
      // 对路由变化作出响应...
    }
  }
}
```

（2）使用`beforeRouteUpdate()`方法（2.2+）

```JS
const User = {
  template: '...',
  beforeRouteUpdate (to, from, next) {
    // 对路由变化作出响应...
    // 需要调用 next 方法进行跳转
  }
}
```

## 路由匹配

有时候，一个路径可以匹配多个多路，此时匹配的优先级就是按照路由的定义顺序，先定义的路由优先级最后，后面定义的路由就不会再匹配

可以使用通配符`*`来匹配任意路径，由于上面提到的优先级，所以含有通配符的路由应该放在最后，一般用来匹配`404`路由

```JS
const router = new VueRouter({
  routes: [
    // 会匹配所有路径
    { path: '*', component: NotFound },
    // 匹配以 `/user-` 开头的任意路径
    { path: '/user-'', component: User },
  ]
})
```

使用了通配符后，可以通过`$route.params`的`pathMatch`属性获取URL通过通配符被匹配的部分。

除了通配符之外，vue-router支持很多高级的匹配模式，可以看[这个例子](https://github.com/vuejs/vue-router/blob/dev/examples/route-matching/app.js)来学习。

## 嵌套路由

`<router-view>`是渲染路由内容的组件，最高级路由匹配到的组件会渲染到顶层的`<router-view>`中。组件可以在内部包含自己的嵌套的`<router-view>`，将路由选项中的`children`属性中的组件渲染在其中

```JS
const router = new VueRouter({
  routes: [
    { path: '/user/:id', component: User,
      children: [
        // 当 /user/:id 匹配成功，
        // UserHome 会被渲染在 User 的 <router-view> 中
        { 
          path: '', 
          component: UserHome 
        }, {
          // 当 /user/:id/profile 匹配成功，
          // UserProfile 会被渲染在 User 的 <router-view> 中
          path: 'profile',
          component: UserProfile
        },
      ]
    }
  ]
})
```

以`/`开头的嵌套路径也会被当做根路径进行匹配。

嵌套路由如果要设置默认的子路由，子路有的`path`可以设置为`/`或者`''`，但是父路由不能有的`name`属性

 （想要嵌套路由的`<router-link>`的`active-class`起作用，可以为自由路的默认路由设置`redicet`:
 
```JS
routes.find(v => v.path === '/demo38').children = [
    {path: '/', redirect: 'demo38-1'},
    {path: 'demo38-1', component: HelloWorld},
    {path: 'demo38-2', component: NotFound},
];
```

## 编程式导航

除了使用`<router-link>`来进行路由跳转之外，还可以通过`$router`的实例方法实现编程式的导航

路由跳转可以使用`$router.push`方法，它会向History栈添加新的记录，为用户点击后腿按钮提供后退功能

```JS
router.push(location, onComplete?, onAbort?)
```

第一个参数可以是字符串路径，也可以是一个描述地址的对象：

```JS
// 字符串
router.push('home')

// 对象
router.push({ path: 'home' })

// 命名的路由
router.push({ name: 'user', params: { userId: '123' }})

// 带查询参数，变成 /register?plan=private
router.push({ path: 'register', query: { plan: 'private' }})
```

如果提供了`path`，`param`虚选项会被忽略：

```JS
const userId = '123'
router.push({ name: 'user', params: { userId }}) // -> /user/123
router.push({ path: `/user/${userId}` }) // -> /user/123

// 这里的 params 不生效
router.push({ path: '/user', params: { userId }}) // -> /user
```

后两个参数是导航成功或终止后的回调函数，在3.1.0+版本后如果省略这两个参数，将会返回一个Promise

如果跳转时路由相同，只是参数发生变化（比如`/user/:id`跳转），需要使用`beforeRouteUpdate`来响应变化

除了`router.push`，还可以使用`router.replace`来进行跳转，不同的是`router.replace`不会为History添加新纪录

还可以使用`router.go(n)`，在History记录中向前或者后退多少步。

## 命名路由

在创建Router的实例时，在`routes`数组内部对象增加`name`属性，为路由设置名称

```JS
const router = new VueRouter({
  routes: [
    {
      path: '/user/:userId',
      name: 'user',
      component: User
    }
  ]
})
```
这样在`<router-link>`的`to`属性中，就可以使用`name`属性来进行路由的匹配

```HTML
<router-link :to="{ name: 'user', params: { userId: 123 }}">User</router-link>
```

## 命名视图

在一个组件中，可以为`<router-view>`添加`name`属性，同时展示多个视图，没有设置`name`的`<router-view>`，默认为`default`

```HTML
<router-view class="view one"></router-view>
<router-view class="view two" name="a"></router-view>
<router-view class="view three" name="b"></router-view>
```

在路由设置里面，需要使用`components`在当前路由下渲染多个组件，不同组件对应不同`name`的`router-view`

```JS
const router = new VueRouter({
  routes: [
    {
      path: '/',
      components: {
        default: Foo,
        a: Bar,
        b: Baz
      }
    }
  ]
})
```

## 重定向和别名

重定向时在`routes`数组的对象中添加`rediect`属性：

```JS
const router = new VueRouter({
  routes: [
    { path: '/a', redirect: '/b' }
  ]
})
```

`redirect`的值可以是字符串，也可以是一个命名路由`{ name: 'foo' }`，也可以是一个函数，返回值是字符串路径或者路径对象

注意导航守卫没有应用在跳转路由上，仅应用在目标上。

## 别名

别名与重定向不同，重定向会将URL进行替换，而设置了别名的路由，URL并不会改变：

```JS
const router = new VueRouter({
  routes: [
    { path: '/a', component: A, alias: '/b' }
  ]
})
```

当用户访问`/b`时，URL显示为`/b`，但是加载的组件是`A`

## 路由组件传参

在组件中使用`$route`会使组件与对应路由高耦合，可以使用`props`将组件合路由解耦：

```JS
const User = {
  template: '<div>User {{ $route.params.id }}</div>'
}
const router = new VueRouter({
  routes: [
    { path: '/user/:id', component: User }
  ]
})
```

使用`props`解耦，将`props`设置为`true`，那么`$route.params`就会被设置为组件属性

```JS
const User = {
  props: ['id']
  template: '<div>User {{ id }}</div>'
}
const router = new VueRouter({
  routes: [
    { path: '/user/:id', component: User, props: true }
  ]
})
```

针对包含多个命名视图的路由，必须分别为每个命名视图添加`props`选项：

```JS
const router = new VueRouter({
  routes: [
    {
      path: '/user/:id',
      components: { default: User, sidebar: Sidebar },
      props: { default: true, sidebar: false }
    }
  ]
})
```

如果`props`是一个对象，那么会被原样设置为组件的属性，一般用做`props`为静态的情况：

```JS
const router = new VueRouter({
  routes: [
    { path: '/a/b', component: C, props: { popup: false } }
  ]
})
```

`props`是也可以接受一个函数，参数是当前的路由对象`route`：

```JS
const router = new VueRouter({
  routes: [
    { path: '/search', component: SearchUser, props: (route) => ({ query: route.query.q }) }
  ]
})
```

访问`/search?q=123`会将`{query: 123}`作为属性传递给`SearchUser`组件

应该尽可能保持`props`函数为无状态的，如果需要状态来定义`props`，请使用包装组件。

## History模式

vue-router默认使用hash模式，使用URL的hash值来模拟一个完整的URL，当URL改变时，实际上改变的是`#`后面的hash值，页面不会被重新加载

vue-router提供了history模式，它利用的是`history.pushState`API来实现URL跳转而无需重新加载页面

```JS
const router = new VueRouter({
  mode: 'history',
  routes: [...]
})
```

这种情况下URL不会有`#`，但是需要后台配置支持，因为如果后台没有配置，那么用户直接访问URL时，首先会经过后台的真正的路由，但是后台没有匹配的路由会返回404，所以应该在服务端增加一个覆盖所有情况的候选资源，如果URL匹配不到任何静态资源，就返回同一个`index.html`，这个页面就是单页应用的容器页面

要注意，这个时候的后台无法响应404页面了，必须在前端给出404页面。
