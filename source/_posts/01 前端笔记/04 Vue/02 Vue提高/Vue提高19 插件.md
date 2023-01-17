---
title: Vue提高19 插件
top: false
date: 2019-06-11 15:10:54
updated: 2019-06-11 15:10:58
tags:
- 插件
categories: Vue
---

Vue插件学习笔记。

<!-- more -->

## 插件和组件的区别

（1）组件分为全局注册和局部注册，全局注册使用`Vue.component('componetName', component)`实现，全局注册后可以在Vue系统中任意使用，局部注册的组件每次使用都需要`import`，然后在组件的`componentes`中注册，它的目的是复用模板和逻辑，影响的范围大多数是组件自身范围内

（2）插件的范围和能力比组件更大，插件内可以包含多个组件，可以在插件内注册全局组件，并且可以实现其他功能，比如：

- 添加全局方法或者属性（挂载到`Vue`的静态方法）
- 添加Vue实例方法（挂载到`Vue.prototype`上实现），在组件中通过`this`调用
- 通过全局混入来添加组件选项（通过`Vue.mixin`实现），比如在所有组件`created`的钩子上完成一些功能
- 添加自定义指令（通过`Vue.directive`实现）

## 使用组件

使用组件需要通过全局方法`Vue.use()`实现，需要在调用`new Vue`之前完成


```JS
Vue.use(MyDatePicker);

new Vue({
  el: '#app',
  data: {
	eventBus: new Vue()
  },
  router,
  components: { App },
  template: '<App/>'
});
```

也可以传入一个可选的选项对象：

```JS
Vue.use(MyDatePicker, { someOption: true });
```

`Vue.use`会自动阻止多次注册相同插件，即使多次调用也只会注册一次该插件。

## 开发插件

插件应该暴露一个`install`方法，提供给`Vue.use()`调用。

`install`方法第一个参数是`Vue`构造器，第二个参数是传入的可选的选项对象

```JS
MyPlugin.install = function (Vue, options) {
  // 1. 添加全局方法或属性
  Vue.myGlobalMethod = function () {
    // 逻辑...
  }

  // 2. 添加全局资源
  Vue.directive('my-directive', {
    bind (el, binding, vnode, oldVnode) {
      // 逻辑...
    }
    ...
  })

  // 3. 注入组件选项
  Vue.mixin({
    created: function () {
      // 逻辑...
    }
    ...
  })

  // 4. 添加实例方法
  Vue.prototype.$myMethod = function (methodOptions) {
    // 逻辑...
  }
}
```

通过在`Vue`构造器上添加一些属性、方法，就可以实现添加全局的方法和属性。

在`install`方法也可以通过`Vue.component`注册全局组件：

```JS
import MyDatePicker from './MyDatePicker'

export default {
  install(Vue, options) {
    Vue.component('MyDatePicker', MyDatePicker);
  }
}
```

## 例子：开发`Loading`插件

首先创建一个`MyLoading.vue`，接受一个`msg`参数显示在组件内部，loading的效果通过CSS实现：

```HTML
<template>
  <div class="container my-loading">
    <div class="logo"></div>
    <p class="text">This is My Loading -- {{msg}}</p>
  </div>
</template>

<script>
  export default {
    props: [ 'msg' ],
    name: 'MyLoading'
  }
</script>

<style scoped>
  .container {
    position: fixed;
    left: 0;
    top: 0;
    width: 100%;
    height: 100%;
    background: rgba(0, 0, 0, 0.5);
    overflow: hidden;
    padding-top: 200px;
  }
  .logo {
    margin: 0 auto;
    width: 100px;
    height: 100px;
    border: 10px solid darkcyan;
    border-right-color: darkgray;
    border-radius: 50%;
    animation: myRotate 2s linear infinite
  }
  @keyframes myRotate {
    0% {
      transform: rotate(0);
    }
    100% {
      transform: rotate(360deg);
    }
  }
  .text {
    margin-top: 20px;
    text-align: center;
  }
</style>
```

然后创建插件主体文件，创建`index.js`，这个文件需要导出一个对象，对象中必须有一个`install`方法，在`install`方法中来完成主要的逻辑


```JS
import MyLoading from './MyLoading'

export default {
  install(Vue, options) {
    const comp = Vue.extend(MyLoading);
    Vue.prototype.showMyLoading = (msg) => {
      if (document.querySelector('.my-loading')) {
        return
      }
      const tpl = new comp({ propsData: { msg } }).$mount().$el;
      document.body.appendChild(tpl);
      setTimeout(function () {
        document.body.removeChild(tpl);
      }, 3000)
    }
  }
}
```
在上面的代码中，定义了实例的`showPicker`方法，在这个方法中首先使用`Vue.extend`来创建一个继承自`MyLoading`的Vue子类，然后使用`new`关键字将其实例化。

注意，使用`new`创建的实例，想要传递给组件`props`，必须使用`propsData`参数

```JS
var Comp = Vue.extend({
  props: ['msg'],
  template: '<div>{{ msg }}</div>'
})

var vm = new Comp({
  propsData: {
    msg: 'hello'
  }
})
```
然后使用`$mount`方法创建一个未挂载的实例，这个实例没有关联的DOM元素，它返回实例自身，所以可以链式调用其他实例方法，然后通过`$el`获取实例自身的根DOM元素，通过`appendChild`将这个元素手动的挂载到DOM中

使用插件的时候在`main.js`中：

```JS
import MyLoading from '@/plugin/myLoading/index'
Vue.use(MyLoading);
```
这样在任意Vue中间中就可以调用实例的`this.showPicker()`方法动态的创建一个Loading元素，3秒后自动消失。


```HTML
<template>
  <div>
    <h1>使用MyLoading插件</h1>
    <button @click="showLoading"> Show Loading</button>
  </div>
</template>

<script>
  export default {
    name: 'demo33',
    props: [],
    data() {
      return {
        title: ''
      }
    },
    mounted() {
    },
    methods: {
      showLoading(){
        this.showMyLoading('你好')
      }
    }
  }
</script>
<style scoped>
</style>
```

这是命令式的创建动态实例，适合Loading、对话框等形式的元素。主要的思路和之前总结的笔记《Vue提高09 动态创建实例》类似。

当然也可以在插件中注册全局组件，类似于ElementUI的用法。

## 参考

- [插件@Vue](https://cn.vuejs.org/v2/guide/plugins.html)
- [vm.$el@Vue](https://cn.vuejs.org/v2/api/#vm-el)
- [vm.$mount@Vue](https://cn.vuejs.org/v2/api/#vm-mount)
- [propsData@Vue](https://cn.vuejs.org/v2/api/#propsData)
- [Vue插件开发与实战@H2O](http://liaokeyu.com/%E6%8A%80%E6%9C%AF/2017/05/16/vue-plugin-development.html)
- [Vue.js 插件开发详解@掘金](https://juejin.im/post/58d9aae02f301e007e8ee278)
