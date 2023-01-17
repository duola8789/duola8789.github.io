---
title: Vuex01 简单状态管理
top: false
date: 2018-01-31 17:46:54
updated: 2019-09-24 14:46:48
tags:
- Vue
- Vuex
categories: Vue
---

Vue中的简单状态管理方式（不全，新的API和方法都没有添加）

<!-- more -->

Vue中对原始数据对象的访问，只是简单的代理访问（也就是引用了同一个地址的数据），所以当一份数据被多个实例共享，不必维护多份数据，只需要维护一份即可。

但是这会带来调试上无法辨别何人何时改变过数据的问题，为了解决这个问题，可以引入**store模式**

```JS
var store = {
  debug: true,
  state: {
    message: 'Hello!'
  },
  setMessageAction (newValue) {
    if (this.debug){
      console.log('setMessageAction triggered with', newValue)
    }
    this.state.message = newValue
  },
  clearMessageAction () {
    if (this.debug){
      console.log('clearMessageAction triggered') 
    }
    this.state.message = ''
  }
}
```

**注意，改变`stroe`中`state`的行为方法，都应放在`store`中的`action`进行统一管理。**

这时，每个实例、组件都可以同时拥有来自组件的共享状态和来自自身的私有状态

```JS
var vmA = new Vue({
  data: {
    privateState: {},
    sharedState: store.state
  }
})

var vmB = new Vue({
  data: {
    privateState: {},
    sharedState: store.state
  }
})
```

![](http://image.oldzhou.cn/Fo2AMwJ8Sf7h95XNygc2xDFe_FT6)

> 重要的是，注意你不应该在`action`中替换原始的状态对象 - 组件和`store`需要引用同一个共享对象，`mutation`才能够被观察

所以一个重要的约定就是：

**组件不允许直接修改属于`store`实例的`state`，而应该执行`action`来分发（`dispatch`）事件通知`store`去改变**

> 回想起在Exam项目中，使用MOBX，完全违反了这个规定。什么都不懂，乱七八糟的代码就上线了

## 参考

- [状态管理@Vue](https://cn.vuejs.org/v2/guide/state-management.html#%E7%AE%80%E5%8D%95%E7%8A%B6%E6%80%81%E7%AE%A1%E7%90%86%E8%B5%B7%E6%AD%A5%E4%BD%BF%E7%94%A8)
