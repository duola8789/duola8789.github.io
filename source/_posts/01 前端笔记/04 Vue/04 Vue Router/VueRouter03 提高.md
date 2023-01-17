---
title: VueRouter03 提高
top: false
date: 2019-09-18 11:46:54
updated: 2019-09-18 19:27:14
tags:
- Vue Router
categories: Vue
---

Vue router 提高技巧

<!-- more -->

## 路由元信息

`routes`中配置的每个路由对象是一条路由记录，路由记录中有一个`meta`字段，可以向这个字段中添加一些自定义的属性，可以在定义路由的时候配置`meat`字段：

```JS
const router = new VueRouter({
  routes: [
    {
      path: '/foo',
      component: Foo,
      children: [
        {
          path: 'bar',
          component: Bar,
          // a meta field
          meta: { requiresAuth: true }
        }
      ]
    }
  ]
})
```

访问这个字段可以在各种守卫中通过`to`（或者组件中的`$route`对象）中获取到这个字段`$route.meat`

有时路由匹配会匹配父级路由及子路有，一个路由匹配到的所有路由记录都会暴露为路由对象的`matched`数组，数组的成员时匹配到的所有父级、子级路由的全部路由对象

下面这个例子，就是通过`to.matched`访问所有记录，检查`meat`中设定字段是否符合要求：

```JS
router.beforeEach((to, from, next) => {
  if (to.matched.some(record => record.meta.requiresAuth)) {
    // this route requires auth, check if logged in
    // if not, redirect to login page.
    if (!auth.loggedIn()) {
      next({
        path: '/login',
        query: { redirect: to.fullPath }
      })
    } else {
      next()
    }
  } else {
    next() // 确保一定要调用 next()
  }
})
```

## 过渡特效

可以用`<transition>`包围`<router-view>`，为组件添加[过渡特效](https://cn.vuejs.org/v2/guide/transitions.html)，这样会给所有路由设置一样的过渡效果

也可以基于当前路由与目标路由的变化关系，动态设置过渡特效：

```HTML
<!-- 使用动态的 transition name -->
<transition :name="transitionName">
  <router-view></router-view>
</transition>
```

在父组件内，`watch`组件的`$route`路由对象，动态改变`transitionName`

```JS
// 在父组件内
// watch $route 决定使用哪种过渡
watch: {
  '$route' (to, from) {
    const toDepth = to.path.split('/').length
    const fromDepth = from.path.split('/').length
    this.transitionName = toDepth < fromDepth ? 'slide-right' : 'slide-left'
  }
}
```

## 数据获取

有时候进入某个路由后，需要从服务器获取数据，有两种方式：

（1）导航完成后获取

先完成导航，渲染组件，在接下来的组件的生命周期钩子（比如`created`）中获取数据，这也是比较常用的方式

```JS
export default {
  data () {
    return {
      data: null
    }
  },
  created () {
    // 组件创建完后获取数据，
    // 此时 data 已经被 observed 了
    this.fetchData()
  },
  watch: {
    // 如果路由有变化，会再次执行该方法
    '$route': 'fetchData'
  },
  methods: {
    fetchData () {
      // 获取数据
    }
  }
}
```

在组件中，通过`$route.params`获取网络请求需要的参数，再`watch`路由对象`$route`（或者在`beforeRouteUpdate`中）重新获取数据即可

（2）导航完成前获取

在导航转入新的路由之前获取数据，可以在目标组件的`beforeRouterEnter`守卫中获取数据，数据获取成功后调用`next`方法

```JS
export default {
  data () {
    return {
      post: null,
      error: null
    }
  },
  beforeRouteEnter (to, from, next) {
    getPost(to.params.id, (err, post) => {
      next(vm => vm.setData(err, post))
    })
  },
  // 路由改变前，组件就已经渲染完了
  // 逻辑稍稍不同
  beforeRouteUpdate (to, from, next) {
    this.post = null
    getPost(to.params.id, (err, post) => {
      this.setData(err, post)
      next()
    })
  },
  methods: {
    setData (err, post) {
      if (err) {
        this.error = err.toString()
      } else {
        this.post = post
      }
    }
  }
}
```
这种方式，相当于在前一个路由组件中获取下一个路由组件的数据，用户会停留在当前界面。应该在数据获取期间显示进度条用来提示用户，如果数据获取失败，也应该展示全局的错误提醒。

## 页面滚动

创建Router实例时，提供`scrollBehavior`方法，来定义页面的滚动行为：

```JS
const router = new VueRouter({
  routes: [...],
  scrollBehavior (to, from, savedPosition) {
    // return 期望滚动到哪个的位置
  }
})
```

`scrollBehavior`方法需要返回滚动位置的对象信息：

```
{ x: number, y: number }
{ selector: string, offset? : { x: number, y: number }} // offset 只在 2.6.0+ 支持
```

`scrollBehavior`方法的第三个参数`savedPosition`仅当通过浏览器的『前进』『后退』按钮触发才可用，这时候返回`savedPosition`，就会像浏览器原生表现的一样：

```JS
scrollBehavior (to, from, savedPosition) {
  if (savedPosition) {
    return savedPosition
  } else {
    return { x: 0, y: 0 }
  }
}
```

也可以模拟滚动到锚点的行为：

```JS
scrollBehavior (to, from, savedPosition) {
  if (to.hash) {
    return {
      selector: to.hash
    }
  }
}
```

也可以返回一个Promise来进行异步的滚动：

```JS
scrollBehavior (to, from, savedPosition) {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      resolve({ x: 0, y: 0 })
    }, 500)
  })
}
```
