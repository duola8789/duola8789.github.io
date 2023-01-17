---
title: Vue提高06 组件间通信
top: false
date: 2018-04-13 16:53:20
updated: 2019-07-13 18:59:26
tags:
- 组件通信
categories: Vue
---

Vue中组件通信的方法。

<!-- more -->

## 父子组件通信

### 父组件向子组件传递数据

父组件向子组件传递数据，直接使用`prop`即可，父组件中在子组件的实例上通过`v-bind`传入`prop`:

```HTML
<LeftChild :message="myMessage"></LeftChild>
```

在子组件中声明了这个`prop`之后就可以使用：

```HTML
<template>
  <h1>{{message}}</h1>
</template>

<script>
  export default {
    props: [ 'message' ],
  }
</script>
```

### 子组件向父组件传递数据


子组件向父组件可以直接通过`$emit`组件上的事件来进行通信，例如在父组件中，通过`v-on`为子组件传入一个事件：

```HTML
<template>
  <LeftChild @changeValue="setValue"></LeftChild>
</template>

<script>
  export default {
    methods: {
      setValue(newValue) {
        // newValue 就是从子组件传递的数据
      }
    },
  }
</script>
```

子组件中通过`#emit`来触发传入的自定义事件，以参数的形式将数据传递给父组件：


```HTML
<template>
  <div @click="clickHandler"></div>
</template>
<script>
  export default {
    methods: {
      clickHandler() {
        // 第二个参数就是要传递的数据
        this.$emit('changeValue', 'hello')
      }
    },
  }
</script>
```

## 非父子组件通信

非父子组件的通信有几种解决方案，如果是比较复杂的应用，可以直接使用Vuex来实现通信和数据管理，如果不使用Vuex则可以使用下面几种方法来实现组件通信。

### 事件总线

原理就是通过一个空的Vue实例`eventBus`，A组件首先在`eventBus`上通过`$on`订阅一个事件，然后在组件B中通过`$emit`发布一个事件，在事件中将数据进行传递。

首先在一个单独的文件中定义名称为`eventBus`的Vue实例，并导出这个实例

```JS
// eventBus.js
import Vue from 'vue';

export default new Vue({
  name: 'eventBus',
})
```

在组件A引入`eventBus`后，在`eventBus`上订阅一个事件`getValue`，最好在组件销毁前注销监听的事件：

```JS
// 组件A
import eventBus from './eventBus';

export default {
  mounted() {
    eventBus.$on('getValue', value => {
      // value 从 getValue 事件传递的数据
    })
  },
  beforeDestroy() {
    // 组件销毁之前
    eventBus.$off('getValue')
  },
}
```

然后在组件B中也引入`eventBus`后，借由`eventBus`发布`getValue`事件，并将数据作为第二个参数传递给事件的订阅者：

```JS
// 组件B
import eventBus from './eventBus';

export default {
  methods: {
    clickHandler() {
      eventBus.$emit('getValue', 'hello')
    }
  },
}
```

### 事件总线的优化

上面这种方式，在每个需要通信的组件都需要手动引入`eventBus`，很麻烦。所以希望能够做到一次注入，到处使用。

改进的方法就是在`main.js`中定义Vue根实例时，将`eventBus`添加到根实例的`data`中，然后再每个组件中都可以通过`this.#root.eventBus`来访问它：

```JS
// main.js
new Vue({
  el: '#app',
  data: {
    eventBus: new Vue() // 增加总线实例
  },
  components: { App },
  template: '<App/>'
});
```

在组件A使用`$root.eventBus.$on`来订阅事件：

```JS
// 组件A
export default {
  mounted() {
    this.$root.eventBus.$on('getValue', value => {
      // value 从 getValue 事件传递的数据
    })
  },
  beforeDestroy() {
    // 组件销毁之前
    this.$root.eventBus.$off('getValue')
  },
}
```

在组件B中使用`$root.eventBus.$on`来发布事件：

```JS
// 组件B
import eventBus from './eventBus';

export default {
  methods: {
    clickHandler() {
      this.$root.eventBus.$emit('getValue', 'hello')
    }
  },
}
```

### 通过原型

Vue对象本质上就是一个JS对象，想要引入`eventBus`只需要在Vue的原型`prototype`上增加一个属性就可以了。本质上所有的Vue组件都是继承全局的Vue，所以只要在初始化Vue对象之前在`prototype`上定义属性，这样所有的组件都可以访问这个属性了。

所以在`main.js`中，在实例化Vue之前增加代码

```JS
// main.js
// 在实例化Vue实例之前
Vue.prototype.$eventBus = Vue.prototype.$eventBus ||  new Vue();
```

组件A中

```JS
// 组件A
this.$eventBus.$on('getValue', value => {})
```

组件B中

```JS
// 组件B
this.$eventBus.$emit('getValue', 'message')
```

### 使用`Vue.observale`

Vue的2.6版本新增了一个`Vue.observale`，它可以定义一个可响应的对象，实际上Vue内部会用它来处理`data`函数返回的对象。

这个API返回的对象可以直接用于**渲染函数和计算属性**内，并且在发生改变时触发相应的更新，可以作为最小化的跨组件状态存储器，用于简单的场景。

我们新建一个`simpleStore.js`，导出一个使用`Vue.observale`处理后的对象：

```JS
// simpleStore.js
import Vue from 'vue';

export default Vue.observable({
  count: 0
})
```

然后再`main.js`中作为Vue的根实例的`data`的属性导入（也可以根据组件通信的范围，分别在不同的组件导入，比如作为组件的数据处理层时）：

```JS
// main.js
import simpleStore from './simpleStore'

new Vue({
  el: '#app',
  data: {
    simpleStore,
  },
  components: { App },
  template: '<App/>'
});
```

这样在需要通信的组件中就可以直接修改这个变量，在另外一个组件中通过**计算属性**引入这个变量，就可以实现响应式的更新：

组件A中直接修改这个变量：

```JS
// 组件A
export default {
  methods: {
    clickHandler() {
      this.$root.observable.count += 1;
    }
  },
}
```

组件B中通过计算属性引入这个变量：

```JS
// 组件B
export default {
  computed: {
    value() {
      return this.$root.observable.count;
    }
  }
}
```

实际上在`simpleStore.js`中我们可以继续定义`commit`、`action`、`mutation`、`dispatch`等事件，来统一管理变量的修改，实际上就是自己实现了一个简易的Vuex

在使用`Vue.observable`时，有两点需要注意：

（1）`Vue.observable`返回的对象只能用于渲染函数或者计算属性内，才能实现响应式的更新，不能直接用于`data`函数中赋值


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

（2）在Vue 2.x中，被传入的对象会直接被`Vue.observable`改变，它和被返回的对象是同一个对象。但是在Vue 3.x中，则会返回一个**可响应的代理**，而对源对象直接进行修改仍是不可相应的。因此为了兼容性，**应该始终操作`Vue.observable`返回的对象，而不是传入的源对象**。

## 参考

- [组件基础@Vue](https://cn.vuejs.org/v2/guide/components.html)
- [Vue2.0 事件发射与接收@CSDN](https://blog.csdn.net/a5534789/article/details/53415201)
- [vue 中央事件总线](https://blog.csdn.net/wy01272454/article/details/77756079)
- [Vue.observable(object)@Vue](https://cn.vuejs.org/v2/api/#Vue-observable)
