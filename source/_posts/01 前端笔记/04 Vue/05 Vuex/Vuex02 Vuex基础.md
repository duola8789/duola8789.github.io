---
title: Vuex02 Vuex基础
top: false
date: 2018-01-31 17:56:54
updated: 2019-09-24 14:50:59
tags:
- Vue
- Vuex
categories: Vue
---

Vuex快速学习笔记。

<!-- more -->

## 如何引入Vuex

首先安装Vuex：

```BASH
npm install vuex --save
```

然后在`src`中新建一个文件夹`store`，在里面新建一个JS文件`index.js`，这就是我们的Vuex的主文件（如果项目木块比较多，可以在`store`中再建立`moudles`文件夹，利用Vuex提供的模块拆分功能，拆分模块）


```TEXT
├── index.html
├── main.js
├── api
│   └── ... # 抽取出API请求
├── components
│   ├── App.vue
│   └── ...
└── store
    ├── index.js          # 我们组装模块并导出 store 的地方
    ├── actions.js        # 根级别的 action
    ├── mutations.js      # 根级别的 mutation
    └── modules
        ├── cart.js       # 购物车模块
        └── products.js   # 产品模块
```

![](http://image.oldzhou.cn/FthBln51oCt768CZVWmYNW1SGlQ0)

然后在`index.js`中，引入Vuex（`Vue.use(Vuex)`），创建`store`和对应的`state`、`getter`、`mutation`、`action`和`module`：

```JS
import Vue from 'vue';
import Vuex from 'vuex';
import cart from './modules/cart.js'
import products from './modules/products.js'

Vue.use(Vuex);

export default new Vuex.Store({
  state: {
    count: 0,
  }
  modules: {
    cart,
    products,
  }
})
```

最后将创建好的`store`导入到Vue实例中，在`main.js`中：

```JS
import Vue from 'vue';
import store from './store';

new Vue({
  el: '#app',
  router,
  store,
  components: { App },
  template: '<App/>'
});
```

## 使用Vuex的准则：

（1）应用层级的状态应该集中到单个store对象中。

（2）提交`mutation`是更改状态的唯一方法，并且这个过程是同步的。

（3）异步逻辑应该封装到`action`里面

（4）组件仍然可以保有局部状态，如果某些状态严格属于单个组件，最好还是作为组件的局部状态。

## Store

Vuex的核心就是`store`， 包含了应用中大部分的状态（`state`）

一个简单的store:

```JS
// 如果在模块化构建系统中，请确保在开头调用了 Vue.use(Vuex)

const store = new Vuex.Store({
  state: {
    count: 0
  },
  mutations: {
    increment (state) {
      state.count++
    }
  }
})
```
`muaction`就是定义在`store`中，用来改变`store`中`state`的方法

通过提交`mutation`的方式，而非直接改变`store.state.count`，可以更明确地追踪到状态的变化。这个简单的约定能够让你的意图更加明显，这样你在阅读代码的时候能更容易地解读应用内部的状态改变。此外，这样也让我们有机会去实现一些能记录每次状态改变，保存状态快照的调试工具。

```HTML
<body>
<div id="app">
  <h3>{{count}}</h3>
  <button @click="add">+</button>
  <button @click="reduce">-</button>
</div>
<script>
  const store = new Vuex.Store({
    state: {
      count: 0
    },
    mutations: {
      add: state => state.count++,
      reduce: state => state.count--
    }
  });
  const vm = new Vue({
    el: '#app',
    computed: {
      count: store.state.count
    },
    methods: {
      add(){
        store.commit('add')
      },
      reduce(){
        store.commit('reduce')
      }
    }
  });
</script>
```

## State

Vuex是单一状态树，用一个对象就包含了全部的应用层级状态，是整个应用的唯一数据源。

首先可以通过根组件使用`store`选项，将状态库从根组件注册到每一个组件中（需要提前调用`Vue.use(Vuex)`）:


```JS
new Vue({
  el: '#app',
  router,
  store,
  components: { App },
  template: '<App/>'
});
```

**在Vue组件中，获取状态的方法是在组件的计算属性中返回某个状态**。有两种方法，一种是直接使用`this.$store.state`来获取：

```JS
export default {
  // ...
  computed: {
    count() {
      return this.$store.state.count;
    }
  }
}
```
另一种是使用`mapState`辅助函数：

```JS
import { mapState } from 'vuex'

export default {
  // ...
  computed: mapState({
    count: 'count', // 等同于state => state.count 
    
    // 使用普通函数（非箭头函数）获取局部状态
    myStr (state) {
        return state.str + this.msg
    }
  })
}
```

映射的计算属性名与`state`子节点名称相同，也可以给`mapState`传一个数组：

```JS
computed: mapState([
  // 映射 this.count 为 store.state.count
  'count'
])
```

利用对象展开运算符，可以让`mapState`与局部的计算属性混合使用：

```JS
computed: {
  count() {
    return this.$store.state.count;
  }, 
  ...mapState({
    myMessage: function(state) {
      return state.str + this.msg
    }
  })
}
```

## Getter

Getter可以认为是`store`中的计算属性，它的返回值会根据它的依赖被缓存起来，且只有它的依赖值发生了改变才会被重新计算

Getter的第一个参数事`state`对象，第二个参数事其他`getters`

```JS
const store = new Vuex.Store({
  state: {
    todos: [
      { id: 1, text: '...', done: true },
      { id: 2, text: '...', done: false }
    ]
  },
  getters: {
    doneTodos: state => {
      return state.todos.filter(todo => todo.done)
    },
    count: (state, getters) => {
      return getters.doneTodos.length
    }
  }
})
```

获取`getter`通过`store.getters`对象，可以直接以属性的方式获取：

```JS
computed: {
  doneTodosCount () {
    return this.$store.getters.doneTodosCount
  }
}
```

也可以通过方法的形式获取`getters`，让`getters`返回一个函数，实现给`getters`传参：

```JS
getters: {
  // ...
  getTodoById: (state) => (id) => {
    return state.todos.find(todo => todo.id === id)
  }
}
```

使用时：

```JS
this.$store.getters.getTodoById(2)
```

和`state`类似，`getters`也有辅助函数，它将`store`中的`getter`映射到局部计算属性：

```JS
import { mapGetters } from 'vuex'

export default {
  // ...
  computed: {
    // 数组形式
    ...mapGetters([
      'doneTodosCount',
      'anotherGetter',
    ])，
    // 对象形式
    ...mapGetters({
      doneCount: 'doneTodosCount'
    })，
  }
}
```

## Mutation

更改Vuex中的`state`的唯一方法就是提交`mutation`，`mutation`在`store`中注册，`key`值就是事件类型，对应的函数就是回调函数，它接受`state`作为第一个参数：

```JS
const store = new Vuex.Store({
  state: {
    count: 1
  },
  mutations: {
    increment (state, payload) {
      // 变更状态
      state.coun += payload.amount;
    }
  }
})
```

`mutation`不能直接调用，只能通过`store.commit`方法来调用，第一个参数就是要出发的`mutation`的事件名，第二个额外的参数是`mutation`的载荷，载荷应该是一个对象：

```JS
this.$store.commit('increment', { amount: 100 }); 

// 或者也可以
this.$store.commit({
  type: 'increment', 
  amount: 100,
}); 
```

也可以使用`mapMutations`辅助函数将`store.commit`映射为组件的`methods`：

```JS
import { mapMutations } from 'vuex'

export default {
  // ...
  methods: {
    ...mapMutations([
      'increment', // 将 `this.increment()` 映射为 `this.$store.commit('increment')`
    ]),
    ...mapMutations({
      add: 'increment' // 将 `this.add()` 映射为 `this.$store.commit('increment')`
    })
  }
}
```

使用`mutation`时也需要遵循与Vue一样的规则：

1. `state`中初始化所有所需属性
2. 不能直接添加新属性，必须使用`Vue.set`方法或者使用新对象替换老对象

要注意的事，**Mutation必须是同步函数**，原因是为了使devtool能够捕捉前后状态的快照，异步函数则让状态改变试无法变更的

## Action

Action是一个架构上的概念，它提交的是`mutation`，不直接改变状态，一般用来在`action`内部执行异步操作：

```JS
const store = new Vuex.Store({
  state: {
    count: 0
  },
  mutations: {
    increment (state) {
      state.count++
    }
  },
  actions: {
    increment (context) {
      context.commit('increment')
    }
    // 或者可以简化为
    decrement({ commit }) {
      setTimeout(() => {
        commit('increment')
      }, 1000)
    }
  }
})
```

它接受一个`store`实例具有相同方法和属性的对象作为参数`context`，所以可以通过`context.commit`来提交`mutation`，也可以通过`context.state`和`contet.getters`来获取`state`和`getters`

可以通过参数解构来简化代码：

```JS
actions: {
  increment ({ commit }) {
    commit('increment')
  }
}
```
Actions通过`store.dispatch`来触发：

```JS
this.$store.dispatch('increment', { amount: 100 }); 

// 或者也可以
this.$store.dispatch({
  type: 'increment', 
  amount: 100,
}); 
```

也可以使用`mapActions`来将`store.dispatch`映射为组件的`methos`：

```JS
import { mapActions } from 'vuex'

export default {
  // ...
  methods: {
    ...mapActions([
      'increment', // 将 `this.increment()` 映射为 `this.$store.dispatch('increment')`
    ]),
    ...mapActions({
      add: 'increment' // 将 `this.add()` 映射为 `this.$store.dispatch('increment')`
    })
  }
}
```

可以在Action中返回Promise，让`store.dispatch.then`继续处理异步流程，也可以将Action进行组合：


```JS
// 假设 getData() 和 getOtherData() 返回的是 Promise

actions: {
  async actionA ({ commit }) {
    commit('gotData', await getData())
  },
  async actionB ({ dispatch, commit }) {
    await dispatch('actionA') // 等待 actionA 完成
    commit('gotOtherData', await getOtherData())
  }
}
```

## Module

可以使用`modules`选项将Store分割成模块，每个模块都有自己的`state`、`getter`、`mutation`、`action`，还可以嵌套子模块

（1）默认情况，未添加命名空间

要注意的是，`state`默认是注册在模块下的，而模块内部的`action`、`mutation`、`getter`都是**注册在全局命名空间**的：

```JS
const moduleA = {
  state: { 
    value: 1
  },
  getters: {
    value: 2
  }，
  mutations: {
    increment() {}
  },
  actions: {
    foo() {}
  }
}

const moduleB = {
  state: { ... },
  mutations: { ... },
  actions: { ... }
}

const store = new Vuex.Store({
  modules: {
    a: moduleA,
    b: moduleB
  }
})

// state
store.state.a.value // -> moduleA 的状态
store.state.b // -> moduleB 的状态

// getters
store.getters.value // -> 2

// mutation
store.commit('increment');

// action
store.dispatch('foo');
```

在未添加命名空间的模块内部的

- `getter`，根节点状态（`rootState`）和根节点Getters（`rootGetters`）会作为第三、四个参数暴露出来，（第一个参数是局部状态对象`state`，第二个参数是局部`getter`对象）
- `mutation`，第一个参数是模块的**局部状态对象**
- `action`，参数仍然是`context`，局部状态通过`context.state`暴露，根节点状态通过`context.rootState`暴露


（2）添加命名空间

由于模块中的`getter`、`mutation`和`action`都是定义在全局空间下的，很有可能在不同模块中出现重名的现象，导致意料之外的情况发生。

为了解决这个问题，并且为了实现更高的封装度和复用性，可以添加`namespaced: true`使其成为带有命名空间的模块，它的所有`getter`、`action`，`mutation`都会自动根据模块注册的路径调整命名：

```JS
// module: user
const state = {
  name: 'Ronald',
};

const getters = {
  fullName(state) {
    return 'C ' + state.name
  }
};

const mutations = {
  changeName(state, payload) {
    state.name = payload.name
  }
};

const actions = {
  changeNameDelay(context, payload) {
    setTimeout(() => {
      context.commit('changeName', payload)
    }, 2000)
  }
};

export default {
  namespaced: true,
  state,
  getters,
  actions,
  mutations,
}
```

组件中使用的时候，除了`state`，`getter`、`action`，`mutation`都会自动根据模块注册的路径调整命名：

```JS
computed: {
  // state
  name() {
    return this.$store.state.user.name
  }, 
  
  // getters
  fullName() {
    return this.$store.getters['user/fullName']
  }, 
  // 或者
  ...mapGetters({
    fullName: 'user/fullName',
  })，
  // 或者
  ..mapGetters('user',['fullName'])，
},

methods: {
  // action
  ...mapActions({
    changeNameDelay: 'user/changeNameDelay'
  }
  
  changeNameHandle() {
    // mutation
    this.$store.commit('user/changeName', { name: 'Messi'});
    
    // action
    this.changeNameDelay({ name: 'Kaka'})
    }
  }
```

在添加了命名空间的模块内部的

- `getter`，根节点状态（`rootState`）和根节点Getters（`rootGetters`）会作为第三、四个参数暴露出来，（第一个参数是局部状态对象`state`，第二个参数是局部`getter`对象）
- `mutation`，第一个参数是模块的**局部状态对象**，触发时`commit`添加`root: true`就可以在全局命名空间内分发`Mutation`
- `action`，参数仍然是`context`，局部状态通过`context.state`暴露，根节点状态通过`context.rootState`暴露，根节点的Getter会通过`context.rootGetters`暴漏，触发时`dispatch`添加`root: true`就可以在全局命名空间内分发`Action`

嵌套模块的情况，如果没有添加`namespaced: true`，则会继承父模块的命名空间。

```JS
const store = new Vuex.Store({
  modules: {
    account: {
      namespaced: true,

      // 模块内容（module assets）
      state: { ... }, // 模块内的状态已经是嵌套的了，使用 `namespaced` 属性不会对其产生影响
      getters: {
        isAdmin () { ... } // -> getters['account/isAdmin']
      },
      actions: {
        login () { ... } // -> dispatch('account/login')
      },
      mutations: {
        login () { ... } // -> commit('account/login')
      },

      // 嵌套模块
      modules: {
        // 继承父模块的命名空间
        myPage: {
          state: { ... },
          getters: {
            profile () { ... } // -> getters['account/profile']
          }
        },

        // 进一步嵌套命名空间
        posts: {
          namespaced: true,

          state: { ... },
          getters: {
            popular () { ... } // -> getters['account/posts/popular']
          }
        }
      }
    }
  }
})
```
在使用`mapState`辅助函数绑定命名空间的模块时，可以将模块的空间名称字符串作为第一个参数传递给函数，这样所有绑定都自动将该模块作为上下文：

```JS
computed: {
  ...mapState({
    a: state => state.some.nested.module.a,
    b: state => state.some.nested.module.b
  }),
  // 优化为
  ...mapState('some.nested.module', {
    a: state => state.a,
    b: state => state.b
  })
},
methods: {
  ...mapActions([
    'some/nested/module/foo', // -> this['some/nested/module/foo']()
    'some/nested/module/bar' // -> this['some/nested/module/bar']()
  ]),
  // 优化为
  ...mapActions('some/nested/module', [
    'foo', // -> this['some/nested/module/foo']()
    'bar',
  ]),
}
```

可以对上面的辅助函数的使用再进一步优化，通过使用`createNamespacedHelpers`创建基于某个命名空间辅助函数。它返回一个对象，对象里有新的绑定在给定命名空间值上的组件绑定辅助函数，就无须再为辅助函数逐个添加命名空间了：

```JS
import { createNamespacedHelpers } from 'vuex'

const { mapState, mapActions } = createNamespacedHelpers('some/nested/module')

export default {
  computed: {
    // 在 `some/nested/module` 中查找
    ...mapState({
      a: state => state.a,
      b: state => state.b
    })
  },
  methods: {
    // 在 `some/nested/module` 中查找
    ...mapActions([
      'foo',
      'bar'
    ])
  }
```

当需要创建一个模块的多个实例时，需要使用一个函数来声明模块状态（与Vue组件内的`data`出于同样的原因和解决方法）

```JS
const MyReusableModule = {
  state () {
    return {
      foo: 'bar'
    }
  },
  // mutation, action 和 getter 等等...
}
```
