---
title: Vue3-09 Vue Router
top: false
date: 2021-05-19 17:29:24
updated: 2021-05-19 17:29:26
tags:
- Vue
- Vue3
categories: Vue
---

Vue3学习笔记-09 Vue Router

<!-- more -->

> For Vue3

# 安装

```BASH
yarn add vue-router@4
```

# 创建路由实例

创建一个Hash模式的最简单路由并应用：

```JS
import {createApp} from 'vue';
import VueRouter from 'vue-router';

const router = VueRouter.createRouter({
  history: VueRouter.createWebHashHistory(),
  routes,
})


const app = createApp(App);

app
  .use(router)
  .use(store)
  .mount('#app');
```

## 历史记录模式

使用`createWebHashHistory()`创建Hash模式，使用`createWebHistory()`创建HTML5模式，并应用在`createRouter`的`history`选项

推荐使用HTML5模式，但是需要在服务器上添加回退路由，[配置实例参考官网](https://next.router.vuejs.org/zh/guide/essentials/history-mode.html#%E6%9C%8D%E5%8A%A1%E5%99%A8%E9%85%8D%E7%BD%AE%E7%A4%BA%E4%BE%8B)。

要注意，如果使用HTML5模式，`vue.config.js`中的`publicPath`需要配置为绝对路径`'/'`，不应该配置为相对路径，否则会出现资源找不到的情况！

# 路由匹配

## 在参数中自定义正则

在动态匹配路由时，如果有两个路径相同的动态路由，可以再括号中为参数指定自定义正则，例如为`orderId`指定数字匹配：

```JS
const routes = [
  // /:orderId -> 仅匹配数字
  { path: '/:orderId(\\d+)' },
  // /:productName -> 匹配其他任何内容
  { path: '/:productName' },
]
```

这样，`/25`就会匹配第`/:orderId`，其他情况会匹配`/:productName`，路由的定义顺序不重要，因为自定义正则有着更高的优先级

要主要确保转义`\`

## 可重负参数

如果有需要匹配多个部分的路由，例如`/first/second/third`，使用`*`（0个或多个）和`+`（一个或多个）将参数标记为可重复

```JS
const routes = [
  // /:chapters ->  匹配 /one, /one/two, /one/two/three, 等
  { path: '/:chapters+' },
  // /:chapters -> 匹配 /, /one, /one/two, /one/two/three, 等
  { path: '/:chapters*' },
]
```

这将提供一个参数数组，使用命名路由时也需要传递数组。同时也可以通过在右括号天津嘉它们与自定义正则结合使用

```JS
const routes = [
  // 仅匹配数字
  // 匹配 /1, /1/2, 等
  { path: '/:chapters(\\d+)+' },
  // 匹配 /, /1, /1/2, 等
  { path: '/:chapters(\\d+)*' },
]
```

## 可选参数

使用`?`来将一个参数标记为可选：

```JS
const routes = [
  // 匹配 /users 和 /users/posva
  { path: '/users/:userId?' },
  // 匹配 /users 和 /users/42
  { path: '/users/:userId(\\d+)?' },
]
```

## 命名视图

如果希望同级展示多个视图，在同一个组件中存在多个`<router-view>`，这个时候就可以使用命名视图，例如：

```HTML
<router-view class="view left-sidebar" name="LeftSidebar"></router-view>
<router-view class="view main-content"></router-view>
<router-view class="view right-sidebar" name="RightSidebar"></router-view>
```

在配置时，多个视图需要配置多个组件在`components`选项上，`key`值为`<router-view>`的`name`：

```JS
const router = createRouter({
  history: createWebHashHistory(),
  routes: [
    {
      path: '/',
      components: {
        default: Home,
        // LeftSidebar: LeftSidebar 的缩写
        LeftSidebar,
        // 它们与 `<router-view>` 上的 `name` 属性匹配
        RightSidebar,
      },
    },
  ],
})
```

利用命名视图也可以创建嵌套视图的复杂布局，例如下面的例子：

```
<!-- UserSettings.vue -->
<div>
  <h1>User Settings</h1>
  <NavBar />
  <router-view />
  <router-view name="helper" />
</div>
```

`Nav`只是一个常规组件。
`UserSettings`是一个视图组件。
`UserEmailsSubscriptions`、`UserProfile`、`UserProfilePreview`是嵌套的视图组件。

配置时：

```JS
{
  path: '/settings',
  // 你也可以在顶级路由就配置命名视图
  component: UserSettings,
  children: [{
    path: 'emails',
    component: UserEmailsSubscriptions
  }, {
    path: 'profile',
    components: {
      default: UserProfile,
      helper: UserProfilePreview
    }
  }]
}
```

# 重定向和别名

## 重定向

重定向的目标除了路径和命名路由外，还可以是一个方法，这个方法接受目标路由作为参数，动态返回重定向目标

```JS
const routes = [
  {
    // /search/screens -> /search?q=screens
    path: '/search/:searchText',
    redirect: to => {
      // 方法接收当前路由作为参数
      // return 重定向的字符串路径/路径对象
      return { path: '/search', query: { q: to.params.searchText } }
    },
  },
  {
    path: '/search',
    // ...
  },
]
```

注意，导航守卫并不会应用在跳转路由上，只会应用在其跳转的目标路由上

## 别名

`alias`可以接收数组，可以接受绝对路径从而避免嵌套结构的限制，如果路由有参数，确保在别命中包含参数

```JS
const routes = [
  {
    path: '/users',
    component: UsersLayout,
    children: [
      // 为这 3 个 URL 呈现 UserList
      // - /users
      // - /users/list
      // - /people
      // - /book/123
      { path: '', component: UserList, alias: ['/people', 'list', '/book/:id'] },
    ],
  },
]
```

# 路由组件传参

把路由参数作为组件的Props传递以组件，避免组件与Route耦合，有三种模式：

（1）布尔模式

将`props`设置为`true`，`route.params`将被设置为组件的Props

```JS
const User = {
  props: ['id'],
  template: '<div>User {{ id }}</div>'
}
const routes = [{ path: '/user/:id', component: User, props: true }]
```

对于命名视图，需要为每个视图定义Props配置：

```JS
const routes = [
  {
    path: '/user/:id',
    components: { default: User, sidebar: Sidebar },
    props: { default: true, sidebar: false }
  }
]
```

（2）对象模式

当`props`是一个对象，会将对象中的属性设置为组件的Props，适合于Props为静态的情况

```JS
const routes = [
  {
    path: '/promotion/from-newsletter',
    component: Promotion,
    props: { newsletterPopup: false }
  }
]
```

（3）函数模式

`props`可以是一个函数，接受当前路由作为参数，返回一个对象作为组件的Props

```JS
const routes = [
  {
    path: '/search',
    component: SearchUser,
    props: route => ({ query: route.query.q })
  }
]
```

# 导航守卫

导航守卫可以返回值：

- `false`，取消当前导航
- 一个路由地址，就像调用`router.push`一样
- 抛出`Error`，取消导航，并调用`router.onError`注册的回调
- `undefined`或`true`，导航有效，调用下一个导航守卫

```JS
router.beforeEach(async (to, from) => {
  // canUserAccess() 返回 `true` 或 `false`
  return await canUserAccess(to)
})
```

`next`是第三个参数，仍然被支持，但是根据上面的描述，完全可以不通过`next`完成导航，使用`next`经常会出现调用多次或者不被调用的问题，而通过返回值来确定导航是否有效会确保戴航始终有效

# 路由元信息

通过扩展`RouteMeta`接口来输入`meta`字段：

```JS
// typings.d.ts or router.ts
import 'vue-router'

declare module 'vue-router' {
  interface RouteMeta {
    // 是可选的
    isAdmin?: boolean
    // 每个路由都必须声明
    requiresAuth: boolean
  }
}
```

# 组合式API

## 在`setup`中访问路由和当前路由

在`setup`中不能使用`this`，所以需要引入`useRouter`和`useRoute`函数代替`this.$router`和`this.$route`

`route`对象是一个响应式对象，属性都可以监听，但是应该避免监听整个`route`对象以提升性能：

```JS
import { useRoute } from 'vue-router'

export default {
  setup() {
    const route = useRoute()
    const userData = ref()

    // 当参数更改时获取用户信息
    watch(
      () => route.params,
      async newParams => {
        userData.value = await fetchUser(newParams.id)
      }
    )
  },
}
```

## 导航守卫

可以通过导入`onBeforeRouteLeave`和`onBeforeRouteUpdate`两个函数，来使用组件内的导航守卫。

可以用在任何由`<router-view>`渲染的组件中，不必像组件内守卫那样直接用在路由组件上

# 过渡动效

需要使用`v-slot` API

## 单个路由的过渡

可以将路由的元信息和动态`name`结合在一起，放在`<transition>`上

```JS
const routes = [
  {
    path: '/custom-transition',
    component: PanelLeft,
    meta: { transition: 'slide-left' },
  },
  {
    path: '/other-transition',
    component: PanelRight,
    meta: { transition: 'slide-right' },
  },
]
```

```HTML
<router-view v-slot="{ Component, route }">
  <!-- 使用任何自定义过渡和回退到 `fade` -->
  <transition :name="route.meta.transition || 'fade'">
    <component :is="Component" />
  </transition>
</router-view>
```

## 基于路由的动态过渡

可以根据目标路由和当前路由的关系，动态的确定使用过渡，需要利用`afterEach`导航守卫，下面的例子是根据路径的深度动态添加`transitionName`

```JS
router.afterEach((to, from) => {
  const toDepth = to.path.split('/').length
  const fromDepth = from.path.split('/').length
  to.meta.transitionName = toDepth < fromDepth ? 'slide-right' : 'slide-left'
})
```

# 导航故障

## 检测导航故障

`router.push`是一个异步方法，它会返回一个Promise，如果导航被阻止，用户停留在同一个页面上，`router.push`返回的`Promise`的解析值将是Navigation Failure，否则导航成功是，`router.push`的返回结果是一个falsy值（通常是`undefined`），这样来区分导航是否成功：

```JS
const navigationResult = await router.push('/my-profile')

if (navigationResult) {
  // 导航被阻止
} else {
  // 导航成功 (包括重新导航的情况)
  this.isMenuOpen = false
}
```

Navigation Failure是一个带有额外属性的`Error`实例，通过这个实例来判断哪些导航被阻止了及其原因，检查导航结果需要使用`isNavigationFailure`和`NavigationFailureType`


```JS
import { NavigationFailureType, isNavigationFailure } from 'vue-router'

// 试图离开未保存的编辑文本界面
const failure = await router.push('/articles/2')

if (isNavigationFailure(failure, NavigationFailureType.aborted)) {
  // 给用户显示一个小通知
  showToast('You have unsaved changes, discard and leave anyway?')
}
```

导航故障的`fail`实例会暴露`to`和`from`属性，反应当前失败导航的当前位置和目标位置


## 鉴别导航故障

有不同的情况会导致导航终止，可以使用`isNavigationFailure`和`NavigationFailureType`来区分类型：

- `aborted`：在导航守卫中返回`false`或者调用`next(false)`中断了本次导航。
- `cancelled`： 在当前导航还没有完成之前又有了一个新的导航。比如，在等待导航守卫的过程中又调用了`router.push`。
- `duplicated`：导航被阻止，因为我们已经在目标位置了。

> `isNavigationFailure`接受两个参数，第一个参数是Navigation Failure实例，第二个参数是决具体的导航失败的枚举值（通过`NavigationFailureType`获取），如果不传入第二个参数，则只判断当前Error是不是Navigation Failure
>
> `NavigationFailureType`包含所有可能导航失败类型的枚举值，不要使用数组，永远只使用枚举值
>
> ![](http://image.oldzhou.cn/Fo1_DE9IvTHotWYAsCMyT5W3fp42)

## 检测重定向

重定向不会阻止导航，而是创建一个新的导航，可以读取路由地址的`redirectedFrom`属性，判断重定向：

```JS
await router.push('/my-profile')
if (router.currentRoute.value.redirectedFrom) {
  // redirectedFrom 是解析出的路由地址，就像导航守卫中的 to和 from
}
```

# 动态路由

通过`addRoute`和`removeRoute`来完成动态路由功能，但是这两个函数，只注册路由，如果新增路由与当前位置匹配，还需要调用`router.push`或`router.replace`来手动导航，才能显示该新路由

## 添加路由

只有一个路由的配置情况下：

```JS
const router = createRouter({
  history: createWebHistory(),
  routes: [{ path: '/:articleName', component: Article }],
})
```

进入任何页面都会显示`Article`组件，在组件上为`/about`添加一个新路由：

```JS
router.addRoute({ path: '/about', component: About })
```

这时页面不会有任何改变，需要手动调用`replace`方法来改变当前位置：

```JS
router.addRoute({ path: '/about', component: About })
// 我们也可以使用 this.$route 或 route = useRoute() （在 setup 中）
router.replace(router.currentRoute.value.fullPath)
```

如果要等待新路由的完成，可以使用`await router.replace()`

## 在导航守卫中添加路由

在导航守卫中添加或删除路由，需要通过返回新的位置来触发重定向：

```JS
router.beforeEach(to => {
  if (!hasNecessaryRoute(to)) {
    router.addRoute(generateRoute(to))
    // 触发重定向
    return to.fullPath
  }
})
```

上面的例子实际上你是在替换要跳转的导航，在实际场景中，添加路由的欣慰更有可能是发生在导航之外，那么就不需要替换当前的导航了

## 添加嵌套路由

`router.addRoute`除了添加单个路由之外，还可以将路由的`name`作为第一个参数传递，这可以添加嵌套的路由，就像通过`children`添加的一样

```JS
router.addRoute({ name: 'admin', path: '/admin', component: Admin })
router.addRoute('admin', { path: 'settings', component: AdminSettings })
```

这等效于

```JS
router.addRoute({
  name: 'admin',
  path: '/admin',
  component: Admin,
  children: [{ path: 'settings', component: AdminSettings }],
})
```

## 删除路由

当路由被删除时，所有的别名和子路由都会被删除。有几种方法来删除当前路由：

（1）添加一个名称相同的路由，那么就会删除之前的路由“

```JS
router.addRoute({ path: '/about', name: 'about', component: About })
// 这将会删除之前已经添加的路由，因为他们具有相同的名字且名字必须是唯一的
router.addRoute({ path: '/other', name: 'about', component: Other })
```

（2）如果是动态添加的路由，那么可以调用`router.addRoute`

```JS
const removeRoute = router.addRoute(routeRecord)
removeRoute() // 删除路由如果存在的话
```

（3）使用`router.removeRoute`按名称删除路由

```JS
router.addRoute({ path: '/about', name: 'about', component: About })
// 删除路由
router.removeRoute('about')
```

为了避免名字冲突，可以使用`Symbol`作为路由的名称

## 查看现有路由

- `router.hasRoute()`：确认是否存在指定名称的路由，接受类型为`string`或者`symbol`的参数
- `router.getRoutes`：获取所有路由记录的完整列表

# 从Vue2迁移

## 创建路由实例的改变

### `new Router`变为`createRouter`

Vue Router不再是一个类，而是一组函数：

```JS
// 以前是
// import Router from 'vue-router'
import { createRouter } from 'vue-router'

const router = createRouter({
  // ...
})
```

### 路由模式改变

`mode: 'history'`被`history`配置替换：

- `"history"`: `createWebHistory()`
- `"hash"`: `createWebHashHistory()`
- `"abstract"`: `createMemoryHistory()`，用于SSR的情况


```JS
import { createRouter, createWebHistory } from 'vue-router'
// 还有 createWebHashHistory 和 createMemoryHistory

createRouter({
  history: createWebHistory(),
  routes: [],
})
```

### `base`配置改变

`base`作为`createWebHistory`的第一个参数传递

## 路由配置

### 删除了通配符路由`*`

必须使用自定义的正则参数来定义全部路由

```JS
{path: '/:pathMatch(.*)', component: NotFound}
```

### `<keep-alive>`和`<transition>`

`<keep-alive>`和`<transition>`需要通过`v-slot`API在`<router-view>`内部使用

```HTML
<router-view v-slot="{ Component }">
  <transition>
    <keep-alive>
      <component :is="Component" />
    </keep-alive>
  </transition>
</router-view>
```

细节还是挺多的，更详细的还是看[文档](https://next.router.vuejs.org/zh/guide/migration/index.html#%E7%A0%B4%E5%9D%8F%E6%80%A7%E5%8F%98%E5%8C%96)吧。
