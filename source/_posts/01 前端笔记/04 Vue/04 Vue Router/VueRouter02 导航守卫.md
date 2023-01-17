---
title: VueRouter02 导航守卫
top: false
date: 2019-09-17 17:46:54
updated: 2019-09-18 19:27:14
tags:
- Vue Router
- 导航守卫
categories: Vue
---

Vue Router导航守卫学习笔记。

<!-- more -->

vue-router提供了导航守卫，在路由跳转过程中来对导航进行处理，导航氛围三个级别：全局的、单个路由独享的、组件级的。

动态参数或者参数参数的改变并不会触发进入/离开的守卫，可以`watch`监听`$route`对象或者在`beforeRouteUpdate`函数中进行处理


##  全局守卫

### （1）全局前置守卫

`router.beforeEach`注册全局前置守卫：

```JS
const router = new VueRouter({ ... })

router.beforeEach((to, from, next) => {
  // ...
})
```

守卫是异步执行，导航跳转过程在所有守卫resolve之前一直处于等待中。

`beforeEach`接受三个参数，第一个参数`to`是将要进入的目标的路由对象，第二个参数`from`是当前导航要离开的路由对象，第三个参数`next`是一个函数，必须通过执行这个函数来确定导航的状态

```JS
// 继续向下执行（执行钩子，如果钩子执行完毕确认导航）
next();

// 中断导航
next(false);

// 跳转到不同的地址
next('/');
next({ path: '/'});

// 抛出 Error 实例给 router.onError 回调，终止导航，
next(new Error('error'))
```

### （2）全局解析守卫（V2.5.0）

`router.beforeResolve`也可以注册全局守卫，与`router.beforeEach`类似，区别是在导航被确认前，且组件内所有守卫和异步组件被解析后执行

```JS
router.beforeResolve((to, from, next) => {});
```

### （3）全局后置钩子

`router.afterEach`是全局后置钩子，它不会接受`next`函数，也不会改变导航本身


## 路由独享守卫

路由配置上定义`beforeEnter`守卫：

```JS
const router = new VueRouter({
  routes: [
    {
      path: '/foo',
      component: Foo,
      beforeEnter: (to, from, next) => {
        // ...
      }
    }
  ]
})
```
它与全局守卫的参数一致，只是会在特定的路由起作用

## 组件内的守卫

组件内可以定义进入、离开、更新三中守卫

### （1）进入

在组件内定义`beforeRouterEnter`，会在导航被确认前调用，这时候组件实例还没有创建，所以不能获取到组件实例`this`

```JS
const Foo = {
  template: `...`,
  beforeRouteEnter (to, from, next) {},
}
```

可以为`next`传一个回调来访问组件实例，在导航被确认时执行，这个回调的参数就是组件实例：

```JS
beforeRouteEnter (to, from, next) {
  next(vm => {
    // 通过 `vm` 访问组件实例
  })
}
```

注意，`beforeRouterEnter`是支持给`next`传递回调的唯一的守卫。


### （2）离开

在组件内定义`beforeRouterLeave`，会在导航离开组件的对应路由前调用，可以访问组件实例`this`

```JS
const Foo = {
  template: `...`,
  beforeRouteLeave (to, from, next) {},
}
```

这个守卫一般用`next(false)`来禁止在用户在还未保存修改前突然离开

### （3）更新

在组件内定义`beforeRouterUpdate`，在路由发生改变，组件被复用时会调用

例如动态路由`/user/:id`的参数`id`改变时，从`/user/1`跳转到`/user/2`时，会访问同一个组件，组件实例会被复用，此时其他导航守卫都不会被调用，只会触发`beforeRouterUpdate`这个守卫

```JS
const Foo = {
  template: `...`,
  beforeRouteUpdate (to, from, next) {},
}
```

## 导航解析流程

### （1）路由间跳转

各种导航解析时，会先从组件离开导航开始执行，然后执行**前置→解析→后置**，而各种前置导航是从全局到局部的，**全局→路由→组件**

> 路由执行`next`后的顺序与Koa的中间件的原理类似，都是洋葱圈模型，一层层进入后直到`afterEach`在按照原来的进入的相反顺序离开。


```
// 组件离开
beforeRouteLeave start
// 全局前置
beforeEach start
// 路由独享前置
beforeEnter start
// 组件前置
beforeRouteEnter start
// 全局解析
beforeResolve start

// 全局后置
afterEach

beforeResolve end
beforeRouteEnter end
beforeEnter end
beforeEach end
beforeRouteLeave end
```

### （2）路由动态参数变化，组件复用

当组件复用时，只会执行全局的前置、解析，以及组件本身的更新

```
beforeEach start
beforeRouteUpdate start
beforeResolve start

afterEach

beforeResolve end
beforeRouteUpdate end
beforeEach end
```

### （3）总结

1. 导航被触发
2. 失活的组件里调用离开守卫`beforeRouteLeave`
3. 调用全局的前置守卫`beforeEach`
4. 在重用的组件里调用`beforeRouteUpdate`守卫（注意，只有重用组件才有这一步）
5. 在路由配置里调用路由独享守卫`beforeEnter`（注意，重用组件不会执行这一步）
6. 解析异步路由组件
7. 在即将要进入的路由组件里调用`beforeRouteEnter`（注意，重用组件不会执行这一步）
8. 调用全局的解析守卫`beforeResolve`
9. 导航被确认
10. 调用全局后置钩子`afterEach`
11. 触发DOM更新
12. 用创建好的实例调用`beforeRouteEnter`守卫中传给`next`的回调函数。

## 参考

- [导航守卫@Vue Router](https://router.vuejs.org/zh/guide/advanced/navigation-guards.html)
- [Vue的钩子函数（路由导航守卫、keep-alive、生命周期钩子）@掘金](https://juejin.im/post/5b41bdef6fb9a04fe63765f1)
