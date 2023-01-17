---
title: Vue基础19 Vue2.6更新
top: false
date: 2019-02-21 15:58:04
updated: 2019-02-21 15:58:04
tags:
- Vue
- slot
- nextTick
categories: Vue
---

2019.2月，大年三十，Vue在时隔一段时间后发布了新的版本Vue2.6，版本号是Macross（超时空要塞）

真是，不让人过好年啊。学不动了。

吐槽归吐槽，该学还是得学习。相对来说Vue的文档和教程是最平易近人、最易理解的了，而且尤雨溪还亲自发表文章介绍了Vue2.6的更新情况，更没有不跟进的理由了。

<!-- more -->

## Slots：新语法，性能优化，准备接轨3.0

### 语法

最重要的更新之一就是对Slots的语法的更新，Slots对于Vue的组件解耦和分发复用有很重要的意义。旧的语法在2.x版本将获得支持，但是在3.0版本后将被废弃。

这次更新引入了`v-slot`来代替原来的`slot`和`slot-scope`语法

有这样一个组件`Comp`：

```HTML
<template>
  <div class="slot-inner">
    <slot>默认Slot</slot>
    <slot name="slot1">具名Slot</slot>
    <slot name="slot2" :innerUser="user">Hello {{user.firstName}}</slot>
  </div>
</template>

<script>
export default {
  data() {
    return {
      user: {
        firstName: 'jay',
        lastName: 'chow'
      },
    }
  }
}
</script>
```

原来的语法：

```HTML
<Comp>
  <!-- 默认插槽 -->
  <p>向默认Slot插入的内容</p>
  <!-- 具名插槽 -->
  <p slot="slot1">向具名Slot插入的内容</p>
  <!-- 作用域插槽 -->
  <p slot="slot2" slot-scope="{innerUser}">Hi, {{innerUser.lastName}}</p>
</Comp>

<!--默认插槽-->
<foo>
  <div slot-scope="{ msg }">
    {{ msg }}
  </div>
</foo>

<!--命名插槽-->
<foo>
  <template slot="one" slot-scope="{ msg }">
    text slot: {{ msg }}
  </template>

  <div slot="two" slot-scope="{ msg }">
    element slot: {{ msg }}
  </div>
</foo>
```

更新后的语法：

```HTML
<Comp>
  <!-- 默认插槽 -->
  <template><p>向默认Slot插入的内容</p></template>
  <!-- 具名插槽 -->
  <template v-slot:slot1><p>向具名Slot插入的内容</p></template>
  <!-- 作用域插槽 -->
  <template v-slot:slot2="{innerUser}"><p>Hi, {{innerUser.lastName}}</p></template>
</Comp>

<!--默认插槽-->
<foo v-slot="{ msg }">
  {{ msg }}
</foo>


<!--命名插槽-->
<foo>
  <template v-slot:one="{ msg }">
    text slot: {{ msg }}
  </template>

  <template v-slot:two="{ msg }">
    <div>
      element slot: {{ msg }}
    </div>
  </template>
</foo>
```

要注意，`v-slot`只能用在`<template>`元素上，除了一种情况，那就是独占插槽的情况。

当被提供的内容只有默认插槽时，组件的标签才可以被当做插槽的模板来使用。

```HTML
<Comp v-slot:slot1>
  <template><p>向默认Slot插入的内容</p></template>
</Comp>
```
也可用于作用域插槽：
```HTML
<Comp v-slot="{ msg }">
  <template><p>{{ msg }}</p></template>
</Comp>
```

注意，独占插槽下不能出现具名插槽，会导致作用域不明确

```HTML
<!-- 无效，会导致警告 -->
<Comp v-slot:slot1>
  <template><p>向默认Slot插入的内容</p></template>
  <template v-slot:slot2><p>向默认Slot插入的内容</p></template>
</Comp>
```
只要出现多个插槽，应该始终为所有的插槽使用完整的基于`<template>`语法

### 为什么使用新指令而不修改旧语法的语义？

尤雨溪给出了三点原因：

1. 会导致断裂性的变化，这个特性就不能在2.x版本发布了
2. 即使在3.x版本中修改了`slot-scope`的语义，但是对于大量的学习者而言会在网络上搜索到大量的基于旧的语义的资料，让人困惑，而引入新的区别于`slot-scope`的新指令就可以避免这个问题。
3. 在3.x版本后不同类型Slot的概念将会统一，所以也就没有必要去区分普通插槽和作用域插槽。插槽无论是否接受参数都是插槽。在统一插槽的概念后，在用`slot`和`slot-scope`代表两种特性就没必要了。

### 新语法嵌套更清晰

原有的语法在多个组件嵌套时会有一个问题，就是不能清晰的判断变量分别是由哪个组件提供的：

```HTML
<foo>
  <bar slot-scope="foo">
    <baz slot-scope="bar">
      <div slot-scope="baz">
        {{ foo }} {{ bar }} {{ baz }}
      </div>
    </baz>
  </bar>
</foo>
```
而且，有`<foor>`提供的`foo`却声明在了`<bar>`上，而是用了新的语法后：

```HTML
<foo v-slot="foo">
  <bar v-slot="bar">
    <baz v-slot="baz">
      {{ foo }} {{ bar }} {{ baz }}
    </baz>
  </bar>
</foo>
```
这是，作用域变量的提供者和声明者是同一个组件，新的语法能够清晰的显示多个作用域变量的关系及其提供者。

### 性能优化

普通的`slot`是在父组件的渲染函数中生成的，因为此当一个普通的`slot`依赖的数据变化时，会首先触发父组件的更新，然后新的`slot`内容被传到子组件，触发子组件更新。

而`scoped slot`在编译时生成的一个函数，这个函数背传入子组件后会在子组件的渲染函数中被低啊用。这意味着`scoped slot`的依赖会被子组件手机，那么当依赖变动时就只会触发子组件的更新了。

在2.6中还引入了一个优化，如果子组件只是用了`scoped slot`，那么父组件自身依赖变动时，不会再强制子组件更新。这个优化是的父子组件之间的依赖意识在存在slot的情况下依然完全解耦。

此外，所有使用新的`v-slot`的语法的`slot`都会编译为`scoped slot`，这以为这所有使用新语法的`slot`代码都会获得上述的性能优化。

所有的非`scoped slot`现在也被以函数的形式暴露在`this.$scopedSlots`上。如果是直接用`render`函数的用户，现在可以完全抛弃`this.$slots`而全部使用`this.$scopedSlots`来处理所有的`slot`了

### 总结

这次更新不仅用新的语法`v-slot`同一了普通的`slot`和`scoped slot`的语法，并且针对新的语法在编译性能上进行了优化提升。

虽然在3.0中才会废弃旧的语法，不再有普通的`slot`和`scoped slot`的区分，并且3.0中`this.$slots`将会直接暴露函数，取代 `this.$scopedSlots`，但是我认为从2.6起，就应该在项目中使用新的语法代替旧的语法，不仅能更好地迎接3.0的到来，还会获得性能上的提升。

## 异步错误处理

Vue 的内置错误处理机制（组件中的`errorCaptured`钩子和全局的`errorHandler`配置项）现在也会处理`v-on`侦听函数中抛出的错误了。

另外，如果组件的生命周期钩子或者侦听函数中有异步操作现在也可以捕获了，只需要返回一个Promise，来让Vue处理可能存在的异步错误。

例如，在子组件中的`button`点击事件：

```JS
click() {
  return new Promise(resolve => {
    throw Error('click')
  })
}
```
父组件中的`errorCaptured`钩子函数和全局的`errorHandler`配置项中就可以捕获到这个错误：

```JS
// 父组件
errorCaptured(e, vm, msg) {
  console.log(msg, 999);
}

// 全局错误处理
Vue.config.errorHandler = function(err, vm ,info) {
  console.log(info, 'config')
};
```

如果使用了`async`/`await`就更加简单，因为`async`函数默认返回Promise：

```JS
export default {
  async mounted() {
    // 这里抛出的异步错误会被 errorCaptured 或是
    // Vue.config.errorHandler 钩子捕获到
    this.posts = await api.getPosts()
  }
}
```

## 动态指令参数

指令的参数现在可以接受动态的JavaScript表达式：

```HTML
<div v-bind:[attr]="value"></div>
<div :[attr]="value"></div>

<button v-on:[event]="handler"></button>
<button @[event]="handler"></button>

<my-component>
  <template v-slot:[slotName]>
    Dynamic slot name
  </template>
</my-component>
```

通过这种语法，当表达式的值为`null`时，绑定/侦听器会被移除：

```HTML
<button @[event]="click">click</button>

<script>
export default {
  name: 'demo30',
  data() {
    return {
      event: 'click'
    }
  },
  methods: {
    async click() {
      console.log(123);
      this.event = null;
    }
  }
}
</script>
```
## 编译警告位置信息

2.6开始，所有的编译器都包含了源码的位置信息：

![微信截图_20190221141938.png](http://image.oldzhou.cn/微信截图_20190221141938.png)



https://cn.vuejs.org/v2/guide/components-slots.html

https://github.com/vuejs/rfcs/blob/master/active-rfcs/0001-new-slot-syntax.md

## 显示地创建响应式对象

2.6引入了一个新的全局API，可以用来显示地创建响应式对象：

```JS
const reactiveState = Vue.observable({
  count: 0
})
```
Vue内部就是使用这个方法来处理`data`函数返回的对象

返回的对象可以直接用于==渲染函数==和==计算属性==内，并且在发生改变时触发相应的更新。也可以作为最小化的跨组件状态存储器，用于简单的场景：

```JS
const state = Vue.observable({ count: 0 })

const Demo = {
  render(h) {
    return h('button', {
      on: { click: () => { state.count++ }}
    }, `count is: ${state.count}`)
  }
}
```
用在计算属性内：

```HTML
<template>
  <div class="slot-inner">
    <button @click="click">click {{count}}</button>
  </div>
</template>

<script>

import Vue from 'vue';

let count = { count: 0 };
const state = Vue.observable(count);

export default {
  methods: {
    click() {
      state.count++
    }
  },

  computed: {
    count() {
      return state.count
    }
  }
}
```
在Vue2.x中，被传入的对象会直接被`Vue.observable`改变，它和被返回的对象是同一个对象。在Vue3.x中，由于用`Proxy`代替了`Object.defineProperty`，所以会返回一个可响应的代理，而对原对象直接进行修改仍然是不可响应的。

因此为了向前兼容，推荐式中操作使用`Vue.observable`返回的对象，而不是传入源对象：

```JS
click() {
  // 推荐的做法
  state.count++;
  
  // 虽然目前有效，但在Vue3.x中无效，所以不推荐
  // count.count++;
}
```

## 重要的内部改动

### `nextTick`重新调整为全部使用Microtask

在2.5中引入了一个改动，当一个`v-on`的DOM时间侦听器触发更新时，会使用Macrotask而不是Microtask类进行移步缓冲（具体的检测降级方案是`setImmediate`→`MessageChannel`→`setTimeout`）。

这原本是为了修正一类浏览器的特殊边际情况导致的bug才引入的（这种bug就是Edge浏览器处理Microtask的优先级比冒泡的优先级更高，会导致使用了Microtask的事件处理程序在冒泡之间就被触发），更多的细节看[这里](https://gist.github.com/yyx990803/d1a0eaac052654f93a1ccaab072076dd)。

但这个改动本身却导致了更多其他的问题。在2.6里面对原本的边际情况找到了更简单的[fix](https://github.com/vuejs/vue/blob/7721febcc3c0594af0abc25fa32d72d63a52942f/src/platforms/web/runtime/modules/events.js#L48-L62)。

## 其他的改动

（1）`this.$scopedSlots`函数统一返回数组，只影响`render`函数用户

（2）SSR数据预抓取，新增`serverPrefetch`钩子使得任意组件都可以在服务端渲染时请求异步的数据（不再限制于路由组件）

（3）可直接在浏览器中引入的ES Modules构建文件，2.6包含了一个可以直接在浏览器导入的版本：


```HTML
<script type="module">
import Vue from 'https://unpkg.com/vue/dist/vue.esm.browser.js'
  
new Vue({
  // ...
})
</script>
```

## 参考

- [Vue 2.6 发布了@知乎](https://zhuanlan.zhihu.com/p/56260917)
- [reverting-to-microtast@github](https://gist.github.com/yyx990803/d1a0eaac052654f93a1ccaab072076dd)
- [插槽@Vue](https://cn.vuejs.org/v2/guide/components-slots.html)
