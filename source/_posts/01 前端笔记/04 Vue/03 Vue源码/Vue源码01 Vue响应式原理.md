---
title: Vue源码01 Vue响应式原理
top: false
date: 2018-12-04 14:52:54
updated: 2019-08-20 10:20:55
tags:
- 响应式
categories: Vue
---

Vue响应式原理

<!-- more -->

## 响应式过程

![](http://image.oldzhou.cn/18-12-4/69856904.jpg)

简单来说，依赖收集的过程是：

1. 在组件`init`的过程中，为`data`中的属性添加`getter/setter`方法
2. 在组件渲染过程中（`render`函数执行时），每个组件实例内部会实例化一个`Watcher`对象，`data`中的属性会被`touch`，触发`getter`方法，记录组件和属性的对应关系
3. 当属性更新时，访问`setter`方法，会调用对应的`wachter`重新计算，调用`render`函数，导致关联组件更新

## `data`的`getter/setter`

在`init`阶段，`data`中的属性会被添加`getter`和`setter`方法，手段就是调用`Object.defineProperty`方法

```JS
function defineReactive(obj: Object, key: string, ...) {
  const dep = new Dep()
  Object.defineProperty(obj, key, {
    enumerable: true,
    configurable: true,
    get: function reactiveGetter() {....
      dep.depend()
      return value....
    },
    set: function reactiveSetter(newVal) {...
      val = newVal
      dep.notify()...
    }
  })
}
```
`data`中的每个属性都有一个`dep`对象，`getter`中会调用`dep.depend()`方法

## `Watcher`的创建

Vue对象在`init`之后进入`mount`阶段，关键函数是`mountComponent`：

```JS
mountComponent(vm: Component, el: ? Element, ...) {
  vm.$el = el

  ...

  updateComponent = () = > {
    vm._update(vm._render(), ...)
  }

  new Watcher(vm, updateComponent, ...)
  ...
}
```
在组件渲染过程中，实例化了一个`Watcher`对象，它有两个参数，第一个参数`vm`就是当前组件实例，第二个参数`updateComponent`会调用`vm._render`和`vm._update`方法，主要目的就是更新页面

> `vm._render`目的是将Vue对象渲染为虚拟DOM  
> `vm._update`目的是将虚拟DOM创建或更新为真实DOM

在组件需要更新的时候，`Watcher`就会被调用，更新页面

## 收集依赖

```JS
class Watcher {
  getter: Function;

  // 代码经过简化
  constructor(vm: Component, expOrFn: string | Function, ...) {
      ...
    this.getter = expOrFn
    Dep.target = this // 暂且不管
    this.value = this.getter.call(vm, vm) // 调用组件的更新函数
    ...
  }
}
```

在`Watcher`的构造函数中，会调用`expOrFn`方法，这个`expOrFn`就是上面的`updateComponent`方法，进而调用`vm._render`方法

在这个过程中，会访问`data`中的属性的`getter`（也就是`touch`的过程），这是会调用上面提到的`dep.depend()`方法

为了弄清`dep.depend()`方法究竟做了什么，在`dep`对象的构造函数中看：

```JS
class Dep {
  static target: ? Watcher;
  subs : Array < Watcher > ;

  depend() {
    if (Dep.target) {
      Dep.target.addDep(this)
    }
  }

  notify() {
    const subs = this.subs.slice()
    for (let i = 0, l = subs.length; i < l; i++) {
      subs[i].update()
    }
  }
}
```
`dep.depend()`实际上调用了`Dep.target.addDep(this)`，而在`Watcher`的构造函数中有这样的代码`Dep.target = this`，所以`Dep.target.addDep(this)`也就是调用了`Watcher`的`addDep`方法

而`Watcher`中的`addDep`方法简化后：

```JS
class Watcher {
  addDep(dep: Dep) {
      ...
    this.newDeps.push(dep)
    dep.addSub(this)
      ...
  }
}

class Dep {
  addSub(sub: Watcher) {
    this.subs.push(sub)
  }
}
```
`Watcher`将这个`Dep`保存了下来，然后调用了Dep的`addSub`方法，将`Watcher`存了进去

我这样理解，`Dep`记录了所有依赖这个属性的组件和组件的Watcher实例，同样，在Watcher中也记录了当前组件实例都使用了哪些属性

经过这些步骤，`Dep.depend`的导致`addSub`方法被调用，将当前的`Watcher`记录到了组件的`Dep`的`subs`中

## 派发更新

修改`data`中已经双向绑定后的属性，会调用`setter`方法，调用了`dep.notify`方法

```JS
class Dep {
  static target: ? Watcher;
  subs : Array < Watcher > ;

  depend() {
    if (Dep.target) {
      Dep.target.addDep(this)
    }
  }

  notify() {
    const subs = this.subs.slice()
    for (let i = 0, l = subs.length; i < l; i++) {
      subs[i].update()
    }
  }
}
```
`dep.notify`方法调用所有依赖这个属性的组件的Watcher实例，运行`render`方法，更新页面

![](http://image.oldzhou.cn/18-12-4/66385003.jpg)

`render`-`Watche`r-`组件`是一一对应的，data中的每个属性都有对应的`dep`实例

![](http://image.oldzhou.cn/18-12-4/38001406.jpg)

更详细的细节：

![](http://image.oldzhou.cn/18-12-4/84733927.jpg)

## 问题

由于在Vue的`init`过程中，对`data`中的属性执行了`getter`和`setter`转化过程，如果是未初始化话添加的`data`根属性，则无法被追踪：


```JS
var vm = new Vue({
  data: {
    a: 1
  }
})

// `vm.a` 是响应的

vm.b = 2
// `vm.b` 是非响应的
```
还有一个问题：

```JS
var vm = new Vue({
  data: {
    items: ['a', 'b', 'c']
  }
})

vm.items[1] = 'x' // 不是响应性的
vm.items.length = 2 // 不是响应性的
```

`items`是数组，数组的索引并不是属性，很难用`Dep`去绑定，长度`length`也没有被处理，解决方法：

```JS
vm.items.splice(indexOfItem, 1, newValue)
vm.items.splice(newLength)
```

## 参考
- [Vue.js - 深入响应式原理](https://cn.vuejs.org/v2/guide/reactivity.html#%E5%A6%82%E4%BD%95%E8%BF%BD%E8%B8%AA%E5%8F%98%E5%8C%96)
- [从零开始 - Vue 响应式原理白话版](https://www.njleonzhang.com/2018/09/26/vue-reactive.html)
