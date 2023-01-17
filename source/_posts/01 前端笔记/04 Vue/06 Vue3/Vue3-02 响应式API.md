---
title: Vue3-02 响应式API
top: false
date: 2021-05-19 17:17:31
updated: 2021-05-19 17:17:31
tags:
- Vue
- Vue3
categories: Vue
---

Vue3学习笔记-02 响应式API

<!-- more -->

# 升级到Vue3

## 升级VueCLI

VueCLI需要在4.3.1以上才可以支持Vue3

```BASH
npm update -g @vue/cli

vue -V
@vue/cli 4.4.6
```

## 创建项目

```BASH
vue create vue3-learning
vue add vue-next # 添加 vue3 插件升级为 vue3
```

在创建项目时选择手动添加配置，选择vue-router和Vuex，这样创建完的项目各个插件也都会升级为支持Vue3的版本

```JS
{
  "dependencies": {
    "core-js": "^3.6.5",
    "vue": "^3.0.0-beta.1",
    "vue-router": "^4.0.0-alpha.6",
    "vuex": "^4.0.0-alpha.1"
  }
}
```

## 创建Vue实例

```JS
import {createApp} from 'vue';
import App from './App.vue';
import router from './router';
import store from './store';

createApp(App)
  .use(router)
  .use(store)
  .mount('#app');
```

## 创建Router

```JS
import {createRouter, createWebHistory} from 'vue-router';

const router = createRouter({
  history: createWebHistory(process.env.BASE_URL),
  routes
});

export default router;
```

创建路由的方式与以前有所变化，路由模式除了Hash模式（`createWebHashHistory`）和History模式（`createWebHistory`），还多了带缓存的History路由（`createMemoryHistory`）

使用路由跳转的API也有所变化：

```JS
import {useRouter} from 'vue-router';

export default {
  setup() {
    const router = useRouter();
    const goHome = () => router.push('/');
    return {goHome}
  }
}
```

关于router的具体变化，后面再单独学习

## 创建Store

```JS
import {createStore} from 'vuex';

export default createStore({
  state: {
    count: 0
  },
  mutations: {
    changeCount(state, isAdd) {
      state.count = isAdd ? state.count + 1 : state.count - 1;
    }
  },
  actions: {},
  modules: {}
});
```

使用：

```JS
import {useStore} from 'vuex';

export default {
  setup() {
    const store = useStore();
    const storeState = store.state;
    return {storeState}
  }
}
```

可以看出来，Vuex和vue-router，API都有了一些变化，与React Hooks的API很类似，但是基本原理没有太大变化

# `setup`

`setup`函数是新的Composition API的入口点

## 调用时机

`setup`在创建组件之前调用（在`beforeCreate`之前），Props初始化后就会调用`setup`函数，在`beforeCreate`钩子前被调用

## 返回值

`setup`返回的对象的属性将被合并到组件模板的渲染上下文，也可以返回一个渲染函数

## 参数

接受两个参数：

- `props`
- `context`

### `props`

接受`props`作为第一个参数，使用的时候，需要首先声明`props`：

```JS
export default {
  props: {
    name: String,
  },
  setup(props) {
    console.log(props.name)
  },
}
```

==`props`是响应式的，前提是不对`props`进行解构，解构后会失去响应性==

如果需要结构`props`，可以通过使用`roRefs`来完成解构操作，并且保持响应式：

```JS
import { toRefs } from 'vue'

setup(props) {
  const { title } = toRefs(props)

  console.log(title.value)
}
```

如果`title`是一个可选的Prop，此时`toRefs`不会为`title`创建一个`ref`，需要使用`toRef`来创建响应式变量：

```JS
import { toRef } from 'vue'

setup(props) {
  const title = toRef(props, 'title')

  console.log(title.value)
}
```

### `context`

`setup`第二个参数是上下文对象`context`，从2.x中的`this`选择性地暴露出三个组件的属性`attrs`、`slots`、`emit`。`context`就是一个普通的JavaScript对象，它不是响应式的，所以可以对`context`进行解构，不需要担心失去响应性

```JS
export default {
  setup(props, context) {
    // Attribute (非响应式对象)
    console.log(context.attrs)

    // 插槽 (非响应式对象)
    console.log(context.slots)

    // 触发事件 (方法)
    console.log(context.emit)
  }
}

// 或者
export default {
  setup(props, { attrs, slots, emit }) {
    ...
  }
}
```

## `this`的用法

`this`在`setup`中不可用，它并不会指向该组件实例

## 访问组件的Property

执行`setup`时，==组件实例尚未被创建==，因此只能访问`slots`/`props`/`attrs`/`emit`这些组件属性，无法访问`computed`、`data`、`methods`组件选项

## 结合模板使用

如果`setup`返回一个对象，那么可以在组件模板中直接使用，就如同使用Props一样

```HTML
<template>
  <div>{{ readersNumber }} {{ book.title }}</div>
</template>

<script>
  import { ref, reactive } from 'vue'

  export default {
    setup() {
      const readersNumber = ref(0)
      const book = reactive({ title: 'Vue 3 Guide' })

      // expose to template
      return {
        readersNumber,
        book
      }
    }
  }
</script>
```

`setup`返回的`refs`在模板中访问是被自动解开的，不需要在模板中使用`.value`

# 生命周期

生命周期钩子函数只能在`setup`期间同步使用，在组件卸载时，生命周期内部创建的侦听器和计算状态也会被自动删除

与Vue2.x相比，`beforeCreated`和`created`被删除了，对应的逻辑在`setup`内部完成，其他的生命周期钩子都改为了`onXxx`的形式（`beforeDestoryed`改为了`onBeforeUnmount`，`destroyed`改为了`onUnmounted`）

![](http://image.oldzhou.cn/FkSOv8v0no7J-Hp3VxfrL-GF-qoj)

两个新增的调试钩子函数`onRenderTracked`和`onRenderTriggered`：

```JS
export default {
  onRenderTriggered(e) {
    debugger
    // 检查哪个依赖性导致组件重新渲染
  },
}
```

函数接受一个回调函数，当钩子被组件调用时将会被执行

```JS
export default {
  setup() {
    // mounted
    onMounted(() => {
      console.log('Component is mounted!')
    })
  }
}
```

# 依赖注入

## 基本使用

使用`provide`和`inject`实现依赖注入，与2.x版本中基本一致，只能在`setup`中使用

```JS
import { provide, inject } from 'vue'

const ThemeSymbol = Symbol()

const Ancestor = {
  setup() {
    provide(ThemeSymbol, ref('dark'))
  },
}

const Descendent = {
  setup() {
    const theme = inject(ThemeSymbol, ref('light') /* optional default value */)
    return {
      theme,
    }
  },
}
```

使用`ref`传值可以保证`provided`和`injected`之间值的响应性

## 响应式修改

在使用响应式`provide`/`inject`时，建议尽可能，==在提供者内保持响应式Property的任何更改==

如果需要在注入数据的组件内部更新`inject`的数据，在这种情况下，建议`provide`一个方法来负责改变响应式Property的方法，让注入方调用

如果要确保通过`provide`传递的数据不会被`inject`的组件更改，可以对提供者的Property使用`readonly`

```JS
import { provide, reactive, readonly, ref } from 'vue'

export default {
  setup() {
    const location = ref('North Pole')
    const geolocation = reactive({
      longitude: 90,
      latitude: 135
    })

    const updateLocation = () => {
      location.value = 'South Pole'
    }

    provide('location', readonly(location))
    provide('geolocation', readonly(geolocation))
    provide('updateLocation', updateLocation)
  }
}
```

# `getCurrentInstance`

如果在`setup`中要访问组件实例，需要使用这个方法来访问，例如在`app.config.globalProperties`定义了全局变量，如果希望在`setup`中访问，就要用到这个方法：

```JS
// main.js
app.config.globalProperties.$http = () => {
  console.log(123);
};

// 组件中
import { getCurrentInstance } from 'vue'

const MyComponent = {
  setup() {
    const internalInstance = getCurrentInstance()

    internalInstance.appContext.config.globalProperties.$http(); // 访问 globalProperties
  }
}
```


注意，它只能在`setup`和生命周期钩子中调用，如果在其外调用（例如Event Listener中）需要在`setup`中调用`getCurrentInstance()`获取实例后传入对应的函数中使用

```JS
const MyComponent = {
  setup() {
    const internalInstance = getCurrentInstance(); // works

    // 在组合式函数中调用也可以正常执行
    const id = useComponentId();// works

    const handleClick = () => {
      getCurrentInstance(); // doesn't work
      useComponentId(); // doesn't work

      internalInstance; // works
    }

    onMounted(() => {
      getCurrentInstance() // works
    });

    return () =>
      h(
        'button',
        {
          onClick: handleClick
        },
        `uid: ${id}`
      )
  }
}


function useComponentId() {
  return getCurrentInstance().uid
}
```




# 模板Refs

## 常规使用

Vue2.x中的`ref`原本是用于获取DOM的， Vue3中`ref`不仅可以响应化数据，也可以实现获取DOM的功能

```HTML
<template>
  <div ref="root"></div>
</template>

<script>
  import { ref, onMounted } from 'vue'

  export default {
    setup() {
      const root = ref(null)

      onMounted(() => {
        // 在渲染完成后, 这个 div DOM 会被赋值给 root ref 对象
        console.log(root.value) // <div/>
      })

      return {
        root,
      }
    },
  }
</script>
```

在`setup`中声明一个`ref`并返回，在模板中声明`ref`并且值与返回的`ref`相同，这时在渲染初始化后（`onMounted`）就可以获取分配的DOM或组件实例

## 在`v-for`中使用

在`v-for`中使用时，需要使用3.0新增的函数形的`ref`，为`ref`赋值：

```HTML
<template>
  <div v-for="(item, i) in list" :ref="el => { divs[i] = el }">
    {{ item }}
  </div>
</template>

<script>
  import { ref, reactive, onBeforeUpdate } from 'vue'

  export default {
    setup() {
      const list = reactive([1, 2, 3])
      const divs = ref([])

      // 确保在每次变更之前重置引用
      onBeforeUpdate(() => {
        divs.value = []
      })

      return {
        list,
        divs,
      }
    },
  }
</script>
```

## 侦听模板引用

在`watch`或者`watchEffect`中引用Dom Ref，需要使用`flush: 'post'`选项，因为默认情况下`watch`和`watchEffect`的副作用函数是在DOM被挂载或者更新之前运行的，此时模板引用还没有被更新

使用了`flush: 'post'` ，保证监听器在DOM更新后执行副作用，确保模板引用于DOM保持同步，引用正确元素


# 组合式API实现更灵活的复用

# Mixin的缺点

- 容易发生冲突
- 可复用性优先，不能向Mixin传递参数来改变逻辑，降低了在抽象逻辑方面的灵活行

Vue3的组合式API就是为了解决这些问题而存在的

## 更多的灵活性来自更多的自我克制

组合式API的初衷就是为了实现更有组织的代码，实现更灵活的逻辑提取与复用，在代码中会出现更多的、零碎的函数模块，在不同的位置、不同的组件间进行重复调用

它可以避免Vue2.x时代逻辑复用的几种主要形式（Mixin/HOS/SLOT）的弊端，带来了比较明显的好处：

![](http://image.oldzhou.cn/Fk7zo_E1Y0zYdzdz9xVr-RHH2NYy)

但是它在提到了代码质量的上限的同时，降低了下线，`setup`中会出现大量面条式的代码，避免这种糟糕情况的关键就是，将逻辑更合理的划分为单独的函数，将`setup`作为一个入口，在其中进行不同组合函数的调用。

## 与React Hooks比较

Vue3的基于函数的组合式API受到了React Hooks的启发，在很多思维模型方面与React Hooks很类似，提供了同等级别的逻辑组合能力，但是也有着比较明显的不同，组合式API的`setup`函数只会被调用一次，也就意味着使用组合式API时：

1. 不需要考虑调用顺序，可以用在条件语句中（React Hooks不可以）
2. 不会再每次渲染时重复执行，降低垃圾回收的压力（React Hooks每次渲染都会重复执行）
3. 不存在内联处理函数导致子组件永远更新的问题，也不需要`useCallback`（React Hooks需要用`useCallback`进行性能优化）
4. 不存在忘记记录依赖的问题，也不需要`useEffecr`和`useMemo`并传入依赖数组以捕获过时的变量，Vue的自动以来可以确保侦听器和计算值总是准确无误的（React Hooks需要手动记录依赖）
