---
title: Vuex03 Vuex其他
top: false
date: 2018-02-21 17:56:54
updated: 2019-09-24 14:46:59
tags:
- Vue
- Vuex
categories: Vue
---

Vuex其他知识点。

<!-- more -->

## 插件

Vuex的插件就是一个函数，接收`store`作为唯一参数，通过`subscribe`对`store`每次的`mutation`进行监听：

```JS
const myPlugin = store => {
  // 当store初始化后调用
  store.subscribe((mutation, state) => {
    // 每次 mutation 之后调用
    // mutation 的格式为 { type, payload }
  })
}
```
插件需要使用`plugins`选项引入：

```JS
const store = new Vuex.Store({
  // ...
  plugins: [myPlugin]
})
```

插件内同样不允许直接修改`state`，只能通过只能通过提交`mutation`来触发变化

## 严格模式

开启严格模式，当不通过`mutation`而直接修改`state`时，Vuex都会抛出错误。

**不要再发布环境下启用严格模式**，严格模式会深度检测状态树来检测不合规的状态变更，造成性能上的损失：

```JS
const store = new Vuex.Store({
  // ...
  strict: process.env.NODE_ENV !== 'production'
})
```

## 表单处理

在开启了严格模式后，把Vuex的`state`使用到`v-model`会报错：

```HTML
<input v-model="subTitle.message" type="text" />
```

上面的代码中的`subTitle`是属于Vuex的store的对象，用户输入时相当于没有通过`mutation`直接修改了`state`，Vuex会抛出错误：

```TEXT
Error: [vuex] do not mutate vuex store state outside mutation handlers.
```

解决方法有两个，一种是利用了`v-model`的语法糖的本质，将`obj.message`作为`value`，再`input`方法中手动触发`commit`方法，然后在`mutation`中修改`state`值


```HTML
<input :value="subTitle.message" type="text" @input="inputHandler"/>
```

```JS
export default {
  methods: {
    inputHandler(e) {
      this.$store.commit('changeSubTitle', { message: e.target.value })
    },
  },
  computed: {
    ...mapState(['subTitle']),
  }
}
```
在Store中：

```JS
export default new Vuex.Store({
  mutations: {
    changeSubTitle(state, { message }) {
      state.subTitle.message = message;
    }
  },
})
```
另一种方法就是使用带有setter的双向绑定计算属性：

```HTML
<input v-model="title" type="text" />
```

```JS
computed: {
  // 带有 setter 的双向绑定的计算书行
  title: {
    get() {
      return this.$store.state.title
    },
    set(value) {
      this.$store.commit('changeTitle', {
        message: value
      })
    }
  },
}
```

## 测试

Mutation和Getter测试时思路相同，将Mutation或者Getter单独导出来，在测试文件中模拟一个`state`，来进行断言：

```JS
const state = {
  count: 0,
}

// mutations 作为命名输出对象
export const mutations = {
  increment: state => state.count++
}

export default new Vuex.Store({
  state,
  mutations
})
```

```JS
// mutations.spec.js
import { expect } from 'chai'
import { mutations } from './store'

const { increment } = mutations

describe('mutations', () => {
  it('INCREMENT', () => {
    // 模拟状态
    const state = { count: 0 }
    // 应用 mutation
    increment(state)
    // 断言结果
    expect(state.count).to.equal(1)
  })
})
```

测试Action比较麻烦，因为它们可能会调用外部的API，需要将外部的API调用进行Mock，可以使用webpack和[inject-loader](https://github.com/plasticine/inject-loader)打包测试文件：

```JS
// actions.js
import shop from '../api/shop'

export const getAllProducts = ({ commit }) => {
  commit('REQUEST_PRODUCTS')
  shop.getProducts(products => {
    commit('RECEIVE_PRODUCTS', products)
  })
}
```

```JS
// actions.spec.js

// 使用 require 语法处理内联 loaders。
// inject-loader 返回一个允许我们注入 mock 依赖的模块工厂
import { expect } from 'chai'
const actionsInjector = require('inject-loader!./actions')

// 使用 mocks 创建模块
const actions = actionsInjector({
  '../api/shop': {
    getProducts (cb) {
      setTimeout(() => {
        cb([ /* mocked response */ ])
      }, 100)
    }
  }
})

// 用指定的 mutations 测试 action 的辅助函数
const testAction = (action, args, state, expectedMutations, done) => {
  let count = 0

  // 模拟提交
  const commit = (type, payload) => {
    const mutation = expectedMutations[count]

    try {
      expect(mutation.type).to.equal(type)
      if (payload) {
        expect(mutation.payload).to.deep.equal(payload)
      }
    } catch (error) {
      done(error)
    }

    count++
    if (count >= expectedMutations.length) {
      done()
    }
  }

  // 用模拟的 store 和参数调用 action
  action({ commit, state }, ...args)

  // 检查是否没有 mutation 被 dispatch
  if (expectedMutations.length === 0) {
    expect(count).to.equal(0)
    done()
  }
}

describe('actions', () => {
  it('getAllProducts', done => {
    testAction(actions.getAllProducts, [], {}, [
      { type: 'REQUEST_PRODUCTS' },
      { type: 'RECEIVE_PRODUCTS', payload: { /* mocked response */ } }
    ], done)
  })
})
```

还是挺复杂的，实际上`actionsInjector`模块mock的仅仅是API部分（`../api/shop`），而`actions.getAllProducts`执行的还是原来的`action`，但是`commit`和`state`已经都被我们替换了。

如果可以使用[Sinon.JS](https://sinonjs.org/)，那么可以使用它来替换上面的辅助函数`testAction`：

```JS
describe('actions', () => {
  it('getAllProducts', () => {
   
    // 一步直接模拟 commit
    const commit = sinon.spy()
    const state = {}
    
    actions.getAllProducts({ commit, state })
    
    expect(commit.args).to.deep.equal([
      ['REQUEST_PRODUCTS'],
      ['RECEIVE_PRODUCTS', { /* mocked response */ }]
    ])
  })
})
```

如果需要给Vuex写单元测试的时候，还是需要到这里对照着例子来实现一下。

执行测试可以在Node环境下，也可以在浏览器环境下，在Node环境下执行的时候需要创建以下webpack配置（需要配置好`.babelrc`）：

```JS
// webpack.config.js
module.exports = {
  entry: './test.js',
  output: {
    path: __dirname,
    filename: 'test-bundle.js'
  },
  module: {
    loaders: [
      {
        test: /\.js$/,
        loader: 'babel-loader',
        exclude: /node_modules/
      }
    ]
  }
}
```

执行的时候：

```BASH
$ webpack 
$ mocha test-bundle.js
```

在浏览器中测试可以[参考文档](https://vuex.vuejs.org/zh/guide/testing.html#%E5%9C%A8%E6%B5%8F%E8%A7%88%E5%99%A8%E4%B8%AD%E6%B5%8B%E8%AF%95)。

## 热重载

Vue-cli脚手架针对Vuex提供了热刷新的功能，当更改Store的数据，页面会自动刷新，但是相比于Vue组件的热重载功能，体验还是略逊一筹。

Vuex想要实现热重载，也是借助了webpack的[Hot Module Replacement API](https://webpack.js.org/guides/hot-module-replacement/)，以前曾经学习过它的[实现原理](https://duola8789.github.io/2018/04/24/01%20%E5%89%8D%E7%AB%AF%E7%AC%94%E8%AE%B0/11%20%E5%89%8D%E7%AB%AF%E5%B7%A5%E7%A8%8B/01%20%E6%9E%84%E5%BB%BA%E5%B7%A5%E5%85%B7/%E6%9E%84%E5%BB%BA%E5%B7%A5%E5%85%B703%20Webpack%E6%A8%A1%E5%9D%97%E7%83%AD%E9%87%8D%E8%BD%BD(HMR)/)（注意，面试的时候的高频题目）

实现热重载的前提就是，必须将代码模块化，所以Store中的Mutation/Module/Action/Getter必须导出为单独的JS文件，才可以实现热重载

```JS
if (module.hot) {
  module.hot.accept(['./modules/todo-list'], () => {
    // 获取更新后的模块
    // 因为 babel 6 的模块编译格式问题，下面需要加上 .default
    const newTodoList = require('./modules/todo-list').default;

    console.log(newTodoList);

    // 加载新模块
    store.hotUpdate({
      modules: {
        store_todoList: newTodoList,
      }
    })
  })
}
```


注意：热重载的目标只能是Mutation/Module/Action/Getter，**手动对`state`的修改不能触发HMR**，可以[参考这个issue](https://github.com/vuejs/vuex/issues/967)。所以这就导致了一个问题：

如果配置了热重载，那么如果改动state时就必须手动刷新，热刷新也没有了；如果不配置热重载，修改任何文件都是热刷新。这样的话热重载我感觉意义不大
