---
title: Vue3-03 从Vue2迁移
top: false
date: 2021-05-19 17:19:03
updated: 2021-05-19 17:19:03
tags:
- Vue
- Vue3
categories: Vue
---

Vue3学习笔记-03 从Vue2迁移

<!-- more -->

# Teleport

一个新增的内置组件，用来提供一种干净的方法，允许我们控制在DOM哪个父节点下渲染HTML，不必求助于全局状态或者将其拆分为两个组件

```HTML
<teleport to="body">
  <div v-if="modalOpen" class="model-wrapper">
    <div class="modal">
      <i class="el-icon-close" @click="modalOpen = false"></i>
      <p>Hello</p>
    </div>
  </div>
</teleport>
```

Vue会将`<teleport>`标签内的内容渲染为`<body>`标签的子级

它接受两个参数，第一个参数`to`是有效的查询选择器，指定要挂载的目标元素，第二个参数`disabled`用来禁用`<teleport>`的功能，元素会在原组件位置渲染

注意，`disabled`状态变化后，将移动实际的DOM节点，而不是被销毁和重新创建，并且它还将保持任何组件实例的活动状态（例如播放的视频等）

在同一个目标上使用多个`<teleport>`组件，会将内容挂载到同一个目标元素，顺序就是简单的追加，稍后挂载的将位于目标元素中较早的挂载之后

# `v-for`中的Ref数组

从单个绑定获取多个Ref时，需要将`ref`绑定到一个更灵活的函数上，在函数中，将函数的参数`el`推书预置的数组中：

```HTML
<div v-for="item in list" :ref="setItemRef"></div>
```

选项式API：

```JS
export default {
  data() {
    return {
      itemRefs: []
    }
  },
  methods: {
    setItemRef(el) {
      if (el) {
        this.itemRefs.push(el)
      }
    }
  },
  beforeUpdate() {
    this.itemRefs = []
  },
  updated() {
    console.log(this.itemRefs)
  }
}
```

组合式API：

```JS
import { onBeforeUpdate, onUpdated } from 'vue'

export default {
  setup() {
    let itemRefs = []
    const setItemRef = el => {
      if (el) {
        itemRefs.push(el)
      }
    }
    onBeforeUpdate(() => {
      itemRefs = []
    })
    onUpdated(() => {
      console.log(itemRefs)
    })
    return {
      setItemRef
    }
  }
}
```

注意：

- `itemRefs`不必是数组，可以是一个对象
- 如果需要，`itemRef`也可以是响应式的，可以被监听
- 需要在`onBeforeUpdate`中对`itemRefs`重置，否则会出问题

# 异步组件（新增）

Vue3中函数式组件被定义为纯函数，异步组件的定义需要通过`defineAsyncComponent`方法显示定义：

```JS
import { defineAsyncComponent } from 'vue'
import ErrorComponent from './components/ErrorComponent.vue'
import LoadingComponent from './components/LoadingComponent.vue'

// 不带选项的异步组件
const asyncPage = defineAsyncComponent(() => import('./NextPage.vue'))

// 带选项的异步组件
const asyncPageWithOptions = defineAsyncComponent({
  loader: () => import('./NextPage.vue'),
  delay: 200,
  timeout: 3000,
  errorComponent: ErrorComponent,
  loadingComponent: LoadingComponent
})
```

相比于Vue2，`component`选项被重命名为`loader`，`loader`函数不接受`resolve`和`reject`参数，必须始终返回Promise

```JS
const { createApp, defineAsyncComponent } = Vue

const app = createApp({})

const AsyncComp = defineAsyncComponent(
  () =>
    new Promise((resolve, reject) => {
      resolve({
        template: '<div>I am async!</div>'
      })
    })
)

app.component('async-example', AsyncComp)
```

如果利用Webpack和ES2015的能力，可以使用`import`直接导入：

```JS
import { defineAsyncComponent } from 'vue'

const AsyncComp = defineAsyncComponent(() =>
  import('./components/AsyncComponent.vue')
)

app.component('async-component', AsyncComp)
```

`defineAsyncComponent`的完整用法：

```JS
import { defineAsyncComponent } from 'vue'

const AsyncComp = defineAsyncComponent({
  // 工厂函数
  loader: () => import('./Foo.vue')
  // 加载异步组件时要使用的组件
  loadingComponent: LoadingComponent,
  // 加载失败时要使用的组件
  errorComponent: ErrorComponent,
  // 在显示 loadingComponent 之前的延迟 | 默认值：200（单位 ms）
  delay: 200,
  // 如果提供了 timeout ，并且加载组件的时间超过了设定值，将显示错误组件
  // 默认值：Infinity（即永不超时，单位 ms）
  timeout: 3000,
  // 定义组件是否可挂起 | 默认值：true
  suspensible: false,
  /**
   *
   * @param {*} error 错误信息对象
   * @param {*} retry 一个函数，用于指示当 promise 加载器 reject 时，加载器是否应该重试
   * @param {*} fail  一个函数，指示加载程序结束退出
   * @param {*} attempts 允许的最大重试次数
   */
  onError(error, retry, fail, attempts) {
    if (error.message.match(/fetch/) && attempts <= 3) {
      // 请求发生错误时重试，最多可尝试 3 次
      retry()
    } else {
      // 注意，retry/fail 就像 promise 的 resolve/reject 一样：
      // 必须调用其中一个才能继续错误处理。
      fail()
    }
  }
})
```


# `$attrs`包含`class`和`style`

Vue2中`v-bind="$attrs"`会把除了`class`和`style`的属性应用到元素上，而Vue3中的`$attrs`则会包含`class`和`style`

> 组件的`inheritAttrs`默认为`true`，这样加载组件的不被认作Props的Attribute会应用到子组件的根元素上，通过设置`inheritAttrs`为`false`，这个默认行为会被取消，通过配合`v-bind="$attrs"`会把这些Attribute显性的绑定到非根元素上

# `$children`

Vue2中可以通过`$children`访问当前组件实例的子组件，这个API在Vue3中取消了，需要使用`ref`来访问子组件

# 自定义指令

Vue3中自定义指令的钩子有了很大变化：

- `created`（new）：在元素的Attribute或事件侦听器创建前调用
- `bind` → `beforeMount`
- `inserted` → `mounted`
- `beforeUpdated`（new）：在元素本身更新前调用
- `update`移除，改为使用`updated`
- `componentUpdated` → `updated`
- `beforeUnmount`（new），在卸载元素前调用
- `unbind` → `unmounted`

最终API如下：

```JS
const MyDirective = {
  beforeMount(el, binding, vnode, prevVnode) {},
  mounted() {},
  beforeUpdate() {}, // 新
  updated() {},
  beforeUnmount() {}, // 新
  unmounted() {}
}
```

Vue中通过`binding.vnode`访问组件实例：

```JS
mounted(el, binding, vnode) {
  const vm = binding.instance
}
```


# Data选项

Vue3的Data选项只接受返回`object`的`function`，不再接受`object`本身

# Data的Mixin合并

在Vue2中，来自组件的Data与来自Mixin或者Extends的Data合并时，会执行深层次的合并，在Vue3中只会执行浅层次的合并：

```JS
const Mixin = {
  data() {
    return {
      user: {
        name: 'Jack',
        id: 1
      }
    }
  }
}
const CompA = {
  mixins: [Mixin],
  data() {
    return {
      user: {
        id: 2
      }
    }
  }
}
```

上面的Mixin，在Vue2中得到的`data`是：

```JS
{
  user: {
    id: 2,
    name: 'Jack'
  }
}
```

在Vue3中，得到的`data`是：

```JS
{
  user: {
    id: 2
  }
}
```

对于依赖Mixin深度合并的行为，建议重构代码以完全避免这种依赖，因为Mixin的深度合并非常隐式，这让代码逻辑难以理解和调试

# `emits`选项

Vue3中新增了`emits`选项，与`props`选项很类似。这个选项用来定义和校验（包括类型定义）组件能想父组件触发什么事件

```JS
const app = Vue.createApp({})

// 数组语法
app.component('todo-item', {
  emits: ['check'],
  created() {
    this.$emit('check')
  }
})

// 对象语法
app.component('reply-form', {
  emits: {
    // 没有验证函数
    click: null,

    // 带有验证函数
    submit: payload => {
      if (payload.email && payload.password) {
        return true
      } else {
        console.warn(`Invalid submit event payload!`)
        return false
      }
    }
  }
})
```

`emits`与`props`一样，可以接受对象作为验证参数

# 移除了`v-on.native`修饰符

Vue中，传递给带有`v-on`的组件的事件监听器，只有通过`this.$emit`触发，要将原生DOM监听器添加到子组件的根元素上，需要使用`.native`修饰符

```HTML
<my-component
  v-on:close="handleComponentEvent"
  v-on:click.native="handleNativeClickEvent"
/>
```

在Vue3中，`.native`已经被移除，同时，上面提到的`emits`选项允许组件定义真正会被触发的事件，而为包含在`emits`选项中的事件监听器，Vue会把它们作为原生事件监听器添加到子组件的根元素中（除非在子组件中设置了`inheritAtts: false`）

```HTML
<script>
  export default {
    emits: ['close']
  }
</script>
```

这样`click`事件就会添加到`my-component`的根元素上

要特别注意，需要将要出发的事件添加到`emits`中，否则就会出现事件被触发两次的问题，例如：

```HTML
<template>
  <button v-on:click="$emit('click', $event)">OK</button>
</template>
<script>
export default {
  emits: [] // without declared event
}
</script>
```

父组件中为`mu-button`添加`click`监听器：

```HTML
<my-button v-on:click="handleClick"></my-button>
```

这时`click`会被触发两次：

- `emit`触发一次
- 因为`emits`选项中未定义，所以`my-button`上定义的`handleClick`会被添加到子组件的根元素上，再次被触发

# 事件API

Vue2中的`$on`、`$off`、`$once`方法被移除了，也就是说在Vue3中，无法在通过下面的形式创建全EventHub全局事件监听器：

```JS
// eventHub.js

const eventHub = new Vue()
export default eventHub

// ChildComponent.vue
import eventHub from './eventHub'

export default {
  mounted() {
    // 添加 eventHub 监听器
    eventHub.$on('custom-event', () => {
      console.log('Custom event triggered!')
    })
  },
  beforeDestroy() {
    // 移除 eventHub 监听器
    eventHub.$off('custom-event')
  }
}

// ParentComponent.vue
import eventHub from './eventHub'

export default {
  methods: {
    callGlobalCustomEvent() {
      eventHub.$emit('custom-event') // 当 ChildComponent 被挂载，控制台中将显示一条消息
    }
  }
}
```

Vue3移除了上述API，推荐使用[mitt](https://github.com/developit/mitt)或者[tiny-emitter](https://github.com/scottcorgan/tiny-emitter)来实现全局事件监听器

# 过滤器

Vue2中的过滤器被移除了，建议使用方法调用或者计算属性替换

如果在应用中定义了全局过滤器，可以通过全局属性在所有组件中使用：

```JS
// main.js
const app = createApp(App)

app.config.globalProperties.$filters = {
  currencyUSD(value) {
    return '$' + value
  }
}
```

然后通过`$filters`对象修改所有模板：

```HTML
<template>
  <h1>Bank Account Balance</h1>
  <p>{{ $filters.currencyUSD(accountBalance) }}</p>
</template>
```

注意此方式只能应用在方法中，不能在计算属性中使用，因为计算属性只有在单个组件的上下文中定义才有意义

# 多根节点组件支持

Vue中不支持多根节点组件，所以需要使用`<div>`包裹多个组件：

```HTML
<template>
  <div>
    <header>...</header>
    <main>...</main>
    <footer>...</footer>
  </div>
</template>
```

Vue3中组件支持包含多个根节点，但是需要开发者显示定义Attribute应该分布在哪个元素或者组件上：“

```HTML
<!-- Layout.vue -->
<template>
  <header>...</header>
  <main v-bind="$attrs">...</main>
  <footer>...</footer>
</template>
```

# 全局API

## `createApp`

Vue中通过`new Vue()`创建根Vue实例，从同一个Vue构造函数创建的每个根实例共享相同的全局配置，导致了两个问题：

1.  测试期间，全局配置很容易污染测试用例
2.  同一个页面上的多个Vue副本无法使用不同的全局配置

```JS
// 这会影响两个根实例
Vue.mixin({
  /* ... */
})

const app1 = new Vue({ el: '#app-1' })
const app2 = new Vue({ el: '#app-2' })
```

为了避免这个问题，引入了新的全局API`createApp`，调用它返回一个应用实例：

```JS
import { createApp } from 'vue'

const app = createApp({})
```

在Vue中任何全局改变Vue的行为的API都移动到了应用实例上：

![](http://image.oldzhou.cn/FsJfdztDLk76HJU8LKRtauYOjCx-)

## `config.ingoredElements`替换为`config.isCustomElement`

使用`config.isCustomElement`可以支持原生自定义元素，取值是一个函数，符合这个函数返回值的标签名都不会编译为Vue组件，而是作为原生的自定义元素出现

```JS
// Vue2.x
Vue.config.ignoredElements = ['my-el', /^ion-/]

// Vue3.x
const app = createApp({})
app.config.isCustomElement = tag => tag.startsWith('ion-')
```

Vue3中元素是否是Vue组件的检查是在模板编译阶段完成的，因此只有在使用运行时编译器才考虑配置此选项，回事在Vue CLI的配置中新的顶层选项

## `Vue.prototype`替换为`config.globalProperties`

Vue2中通过为Vue的原型`Vue.prototype`添加属性，让所有组件都可以访问， 例如：

```JS
// 之前 - Vue 2
Vue.prototype.$http = () => {};

// 组件中
this.$http();
```

在Vue中被替换为`config.globalProperties`：

```JS
// 之后 - Vue 3
const app = createApp({})
app.config.globalProperties.$http = () => {}
```

如果在选项是API中，仍然使用`this.$http`访问

要注意，如果使用了TypeScript，直接访问`this.$http`会报错，需要在`shims-vue.d.ts`添加如下的定义：

```JS
// Vue 原型上添加的东西，需要在此定义
declare module '@vue/runtime-core' {
  interface ComponentCustomProperties {
    $http: () => {};
  }
}
```

具体可以参考[这里](https://github.com/vuejs/vue-next/blob/1abcb2cf61ec16807cae11cfe56acefab19487a1/packages/runtime-core/src/componentPublicInstance.ts#L41-L66)。

使用`provide`在编写插件时非常有用，可以替代`globalProperties`。

# 全局Tree Shaking

在Vue2中，例如`Vue.nextTick`等AAPI，无法通过Webpack的tree-shaking移除，在Vue3.0中对全局和内部API进行了冲过，全局API只能通过ES模块构建的命名导出进行文旦，例如使用`nextTick`时：

```JS
import { nextTick } from 'vue'

nextTick(() => {
  // 一些和DOM有关的东西
})
```

这样可以更好的支持Webpack的tree-shaking，Vue应用程序中未使用的全局API会从最终打包的产出中消除，减小打包体积

# `key`属性

`key`不能相同，并且在Vue2中，`<template>`不能拥有`key`属性，需要为每个子节点设置`key`：

```HTML
<!-- Vue 2.x -->
<template v-for="item in list">
  <div :key="item.id">...</div>
  <span :key="item.id">...</span>
</template>
```

在Vue3中则可以设置在`<template>`上了
```HTML
<!-- Vue 3.x -->
<template v-for="item in list"  :key="item.id">
  <div>...</div>
  <span>...</span>
</template>
```

# 按键修饰符

Vue2中支持`keyCodes`作为修改`v-on`方法的途径：

```HTML
<!-- 键码版本 -->
<input v-on:keyup.13="submit" />

<!-- 别名版本 -->
<input v-on:keyup.enter="submit" />
```

由于[keyboradEvent.keyCode不再被推荐使用](https://developer.mozilla.org/zh-CN/docs/Web/API/KeyboardEvent/keyCode/)，所以建议对任何要用作修饰符的键使用`kebab-cased`大小写名称

# 移除`$listeners`

Vue2中的事件监听器通过`$listeners`访问，在Vue3中`$listeners`被移除了，事件监听器是`$attrs`的一部分，事件监听器现在只是以`on`为前缀的Attribute

# Props默认函数中不能访问`this`

生成Prop默认值的工厂函数不能再访问`this`（虽然我以前也没这么用过），可以将组件接收到的原始Prop作为参数传递给默认函数，也可以在默认函数中使用Inject API

```JS
import { inject } from 'vue'

export default {
  props: {
    theme: {
      default (props) {
        // `props` 是传递给组件的原始值。
        // 在任何类型/默认强制转换之前
        // 也可以使用 `inject` 来访问注入的 property
        return inject('theme', 'default-theme')
      }
    }
  }
}
```

# 渲染函数

- 函数`h`从Vue中导入，不再作为参数传递给`render`函数
- 渲染函数参数更改，有状态组件和函数组件之间更一致
- VNode的Prop结构更扁平
- 注册组件时不能在使用字符串ID隐式查找已注册组件，需要使用导入的`resolveComponent`方法

```JS
// 2.x
Vue.component('button-counter', {
  data() {
    return {
      count: 0
    }
  }
  template: `
    <button @click="count++">
      Clicked {{ count }} times.
    </button>
  `
})

export default {
  render(h) {
    return h('button-counter')
  }
}

// 3.x
import { h, resolveComponent } from 'vue'

export default {
  setup() {
    const ButtonCounter = resolveComponent('button-counter')
    return () => h(ButtonCounter)
  }
}
```

详细的参考[渲染函数的介绍](https://v3.cn.vuejs.org/guide/render-function.html)。


# 插槽统一

移除了`$scopedSlots`，同时插槽定义为当前节点的子对象：

```JS
// 3.x Syntax
h(LayoutComponent, {}, {
  header: () => h('div', this.header),
  content: () => h('div', this.content)
})
```

需要以编程方式引用时，都被统一到了`$slots`中：

```JS
// 2.x 语法
this.$scopedSlots.header

// 3.x 语法
this.$slots.header()
```

# 过渡CSS类名更改

`v-enter`更改为`v-enter-from`，`v-leave`更改为`v-leave-from`

# `<transition-group>`过渡组件

`<transition-group>`不再默认渲染根元素，但仍然可以用`tag`prop创建根元素。

# `v-model`和`.sync`

## Vue2中

Vue2中在组件上使用`v-model`相当于绑定了`value`的Prop和`input`事件：

```HTML
<ChildComponent v-model="pageTitle" />

<!-- 是以下的简写: -->
<ChildComponent :value="pageTitle" @input="pageTitle = $event" />
```

如果需要将属性或事件名更改为其他名称，需要在子组件中添加`model`选项进行修改：

```HTML
<!-- ParentComponent.vue -->
<ChildComponent v-model="pageTitle" />

<!-- 是以下的简写: -->
<ChildComponent :title="pageTitle" @change="pageTitle = $event" />
```

```JS
// ChildComponent.vue

export default {
  model: {
    prop: 'title',
    event: 'change'
  },
  props: {
    // 这将允许 `value` 属性用于其他用途
    value: String,
    // 使用 `title` 代替 `value` 作为 model 的 prop
    title: {
      type: String,
      default: 'Default title'
    }
  }
}
```

某些情况下需要对一个Prop进行『双向绑定』，在Vue中可以使用`update:propName`来抛出事件，例如对于上面的`ChildComponent`来说，可以通过下面的方法更新父元素中的`title`：

```JS
this.$emit('update:title', newValue)
```

父组件中可以监听该事件并更新本地数据：

```HTML
<ChildComponent :title="pageTitle" @update:title="pageTitle = $event" />
```

而`.sync`修饰符就是上面的代码的语法糖：

```HTML
<ChildComponent :title.sync="pageTitle" />
```

## Vue3中

Vue3里面，自定义组件的`v-model`相当于传递了`modelValue`的Prop，并接受抛出的`update:modelValue`事件：

```HTML
<ChildComponent v-model="pageTitle" />

<!-- 是以下的简写: -->

<ChildComponent
  :modelValue="pageTitle"
  @update:modelValue="pageTitle = $event"
/>
```

如果要更改`v-model`传递的Prop名称，不再需要去子组件中配置`model`选项，而是给`v-model`传递一个参数即可：

```HTML
<ChildComponent v-model:title="pageTitle" />

<!-- 是以下的简写: -->
<ChildComponent :title="pageTitle" @update:title="pageTitle = $event" />
```

这也可以作为`.sync`的替代，所以`.sync`就没有用武之地了，所以在Vue2中的`sync`直接替换为`v-model`即可

```HTML
<ChildComponent :title.sync="pageTitle" />

<!-- 替换为 -->
<ChildComponent v-model:title="pageTitle" />
```


同时在同一个组件上，可以同时使用多个`v-model`：

```HTML
<ChildComponent v-model:title="pageTitle" v-model:content="pageContent" />

<!-- 是以下的简写： -->
<ChildComponent
  :title="pageTitle"
  @update:title="pageTitle = $event"
  :content="pageContent"
  @update:content="pageContent = $event"
/>
```

# `v-model`自定义修饰符

Vue2中`v-model`提供了`.trim`、`lazy`、`.number`内置修饰符，Vue3中还提供了自定义修饰符的能力。

添加到`v-model`的修饰符通过`modelModifiers`Prop 提供给组件，它接受一个`default`的对象，取值是一个函数，这个函数会返回一个对象，当父组件没有提供`v-model`的修饰符时，会从这个函数返回的对象里面取值

> 注意，`modelModifiers`是一个Prop，不是Vue实例的选项，它提供的并不是修饰符具体逻辑的实现，而是提供组件默认包含哪些修饰符，取值是布尔值

请注意，当组件的`created`生命周期钩子触发时，`modelModifiers`Prop会包含`capitalize`，且其值为`true`——因为`capitalize`被设置在了写为`v-model.capitalize="myText"`的`v-model`绑定上。

建议把组件中定义的修饰符显式的写在`modelModifiers`Prop中，这样可以为TypeScript版本的组件提供正确的类型推导，也便于一眼看出组件中使用了哪些修饰符

另外，`v-model`自定义修饰符也支持多级串联

```HTML
<v-model-child v-model.capitalize.test="msg" />
```

在`v-model-child`组件中，我们会检查`props.modelModifiers`是否包含`capitalize`和`test`，并且编写一个处理器来更改`emit`发出的值：

```HTML
<template>
  <label>
    请输入：
    <input :value="modelValue" @input="onInput" />
  </label>
</template>

<script lang="ts">
import {defineComponent} from 'vue';

export default defineComponent({
  name: 'v-model-child',
  props: {
    modelValue: String,
    modelModifiers: {
      default: () => ({
        capitalize: true
      })
    }
  },
  emits: ['update:modelValue'],
  setup(props, {emit}) {
    const capitalize = (val: string) => val.charAt(0).toUpperCase() + val.slice(1);
    const isCapitalize = props.modelModifiers && props.modelModifiers.capitalize;

    const onInput = (e: {target: HTMLInputElement}) => {
      const originValue = e.target.value;
      const value = isCapitalize ? capitalize(originValue) : originValue;
      emit('update:modelValue', value);
    };

    return {
      onInput
    };
  }
});
</script>
```

对于带有参数的（即自定义`model`使用的Prop名的`v-model`）`v-model`绑定，生成的自定义修饰符的Prop名称为`参数名 + "Modifiers"`：

```HTML
<my-component v-model:description.capitalize="myText"></my-component>
```

```JS
app.component('my-component', {
  props: ['description', 'descriptionModifiers'],
  emits: ['update:description'],
  template: `
    <input type="text"
      :value="description"
      @input="$emit('update:description', $event.target.value)">
  `,
  created() {
    console.log(this.descriptionModifiers) // { capitalize: true }
  }
})
```

# `v-for`与`v-if`的优先级

Vue2中，在同一个元素上应用`v-if`和`v-for`，`v-for`优先级更高，而Vue3中`v-if`的优先级更高，也就是说`v-if`将没有权限访问`v-for`里面的变量，下面的代码会报错：

```HTML
<!-- This will throw an error because property "todo" is not defined on instance. -->

<li v-for="todo in todos" v-if="!todo.isComplete">
  {{ todo.name }}
</li>
```

所以要尽量避免在同一个元素上使用`v-for`和`v-if`，可以把`v-for`移动到`<template>`中来修正：

```HTML
<template v-for="todo in todos" :key="todo.name">
  <li v-if="!todo.isComplete">
    {{ todo.name }}
  </li>
</template>
```

# `v-bind`合并行为

Vue2中，如果一个元素同时定义了`v-bind="object"`和一个相同的单独的Property，那么单独的Property会覆盖`object`中的绑定：

```HTML
<!-- template -->
<div id="red" v-bind="{ id: 'blue' }"></div>

<!-- result -->
<div id="red"></div>
```

在Vue3中覆盖结果取决于声明绑定的书序，后面绑定的结果会覆盖前面的绑定结果：

```HTML
<!-- template -->
<div id="red" v-bind="{ id: 'blue' }"></div>
<!-- result -->
<div id="blue"></div>

<!-- template -->
<div v-bind="{ id: 'blue' }" id="red"></div>
<!-- result -->
<div id="red"></div>
```

# 对数组的监听

在Vue3中，使用`watch`监听一个数组时，默认只有数组被替换时才会触发回调，数组成员改变时不会触发回调，如果希望在数组改变时触发需要添加`deep`参数

```JS
watch: {
  bookList: {
    handler(val, oldVal) {
      console.log('book list changed')
    },
    deep: true
  },
}
```

