---
title: Vue3-10 Vuex
top: false
date: 2021-05-19 17:29:42
updated: 2021-05-19 17:29:43
tags:
- Vue
- Vue3
categories: Vue
---

Vue3学习笔记-10 Vuex

<!-- more -->

> For Vue3

# 安装

```BASH
yarn add vuex@next --save
```

# 创建

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


const app = createApp({ /* 根组件 */ })

// 将 store 实例作为插件安装
app.use(store)
```

# 组合式API

通过调用`useStore`函数，在`setup`中访问Store，这与在选项是API中使用`this.$store`是等效的

## 访问State和Getter

需要创建`computed`引用并保留响应性

```JS
import { computed } from 'vue'
import { useStore } from 'vuex'

export default {
  setup () {
    const store = useStore()

    return {
      // 在 computed 函数中访问 state
      count: computed(() => store.state.count),

      // 在 computed 函数中访问 getter
      double: computed(() => store.getters.double)
    }
  }
}
```

## 访问Mutation和Action

访问Mutation和Action，只需要在`setup`钩子函数中调用`commit`和`dispatch`即可

```JS
import { useStore } from 'vuex'

export default {
  setup () {
    const store = useStore()

    return {
      // 使用 mutation
      increment: () => store.commit('increment'),

      // 使用 action
      asyncIncrement: () => store.dispatch('asyncIncrement')
    }
  }
}
#
```

# TypeScript支持

看了官方文档，也查了一些资料，目前（2021.05.18），使用Vue3 + Vuex4 + TypeScript的解决方案，想要获得比较完美的TypeScript支持是比较困难的，尤其是在使用了Vuex的`modules`的情况下，Vuex目前对TypeScript的支持实在太差了

而之前[Vue2 + TypeScript + Vuex + Vuex-class的解决方案](https://blog.csdn.net/duola8789/article/details/103979022)，虽然书写比较麻烦，但是好歹能够获得比较好的类型支持，但是由于Vuex-class迟迟没有支持Vue3，所以这套解决方案也是不可行的

所以目前使用Vuex4要慎重，可以等等Vuex5，Vuex5带来了很多令人兴奋的特性，比如取消了Mutation，用组合代替了Modules的嵌套等等，或者自己实现一个简单版本的Vuex5也是可行的，具体可以参考[这篇文章](https://juejin.cn/post/6920118166224666632#heading-7)，介绍的很详细，我很赞同

基于现在的方案，使用Vuex鼓捣出了一个勉强在组合式API中可以用的方案，但是有两个缺点：

1. Module内部再嵌套Module的话类型判断会报错
2. `commit`和`dispatch`都无法获得类型提示

具体的看[代码](https://github.com/duola8789/vue3-learning/tree/master/src/store)吧

# 简易Store

仿照上面的文章，实现了一个简易的Store，好像也能凑合着用，感觉在Vuex5出来之前，也许在业务代码中再完善一下，采用这种方案可能更优雅一点？

![](http://image.oldzhou.cn/FgIrasNz-lCe2J4SB2kRY53PUCeF)

两个Store，不存在嵌套关系了，`root-store`中还是定义全局变量，`types`定义类型：

```JS
// types.ts
export interface RootStates {
  count: number;
}

export interface RootActions {
  changeCountAct(payload: {newVal: number}): Promise<void>;
}
```

在`index`中定义Store的具体实现，借助`reactive`这个API，返回值是`readonly`防止被意外更改，Action负责更改State，也可以在里面完成异步请求（不考虑快照和调试的问题了），然后导出一个`useRootStore`

```JS
// root-store/index.ts
import {reactive, readonly} from 'vue';
import {RootStates, RootActions} from './types';
import {mock} from '@/utils';

const createStore = () =>
  reactive<RootStates>({
    count: 0
  });

const createAction = (state: RootStates): RootActions => ({
  async changeCountAct({newVal}) {
    state.count = await mock(newVal);
  }
});

const state = createStore();
const action = createAction(state);

export const useRootStore = () => ({
  state: readonly(state),
  dispatch: readonly(action)
});
```

`example-store`中类似，导出的`useExampleStore`，这两个Store在外层的`index`中组装，并且可以相互调用

```JS
// index.ts
import {useRootStore} from './root-store';
import {useExampleStore} from './exmaple-store';

const allStore = {
  root: useRootStore(),
  example: useExampleStore()
};

export const useStore = (storeName: keyof typeof allStore = 'root') => allStore[storeName];
```

在组件中调用`root-store`时，可以获得类型提示和校验：
```JS
import {useStore} from '@/data';

export default defineComponent({
  name: 'Vuex',
  setup() {
    const store = useStore();

    const rootCount = computed(() => store.state.count);

    const changeRootCount = (isAdd: boolean) => {
      const newVal = isAdd ? store.state.count + 2 : store.state.count - 2;
      store.dispatch.changeCountAct({newVal});
    };

    return {rootCount, changeRootCount};
  }
});
```

同样调用`example-store`时，为`useStore`传入`example`字符串即可，[全部代码在这里](https://github.com/duola8789/vue3-learning/tree/master/src/data)。
