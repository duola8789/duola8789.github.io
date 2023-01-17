---
title: React-Router02 静态路由和动态路由
top: false
date: 2019-05-13 11:12:03
updated: 2019-05-13 11:12:03
tags:
- React
- React-Router
categories: React
---

React-RouterV3/V4两个版本理念上的区别。

<!-- more -->

## 静态路由

传统的路由一般都是静态路由，像Express等框架，使用的都是静态路由：

```
React.render((
  <Router>
    <Route path="/" component={App}>
      <Route path="about" component={About} />
      <Route path="inbox" component={Inbox} />
    </Route>
  </Router>
), document.body)
```

路由集中配置，UI与路由强绑定。

React-Router V3版本采用的就是静态路由，本质就是`path`到模块的映射，这种映射关系是静态的。只要程序已启动，映射关系就不能改变了。

从V4版本开始变为了动态路由。这是因为静态路由存在着天生的问题：

（1）路就有写法需要满足约定的格式，比如不能将`<Route>`脱离`<Router>`使用，这与React倡导的“可以声明式灵活性进行组件组装”的理念。

```
// 静态路由不支持
const CoolRoute = (props) => <Route {...props} cool={true}/>
```

（2）因为`<Router>`接管了组件，内部实现了`createElement`、`render`等方法，同时提供了组件生命周期回调`onEnter`、`onLeave`、·onChange`等，而这些生命周期React本身是为组件提供了的。

（3）为了适应代码拆分，引入了`getComponent`和`getChildRoutes`

## 动态路由

为了解决以上问题，V4版本中引入了动态路由的概念。

既然React组件渲染时动态发生的，在V4中可以将路由看成普通的React组件，传递`props`来正常使用，借助它来控制组件的展现。展示的逻辑及权利回归到了组件本身。这样没有了静态配置的路由规则，取而代之的是程序在运行渲染过程中动态控制的展现。

这边是V4中的动态路由的概念。

当`<Router>`作为普通组件后，React-Router提供的一些特殊的API就不存在了，因为React会为组件提供对应的生命周期函数等API。

通过代码看一下静态路由和动态路由的区别：

```
// V3
import { Router, Route, IndexRoute } from 'react-router'

const PrimaryLayout = props => (
  <div className="primary-layout">
    <header>
      Our React Router 3 App
    </header>
    <main>
      {props.children}
    </main>
  </div>
)

const HomePage =() => <div>Home Page</div>
const UsersPage = () => <div>Users Page</div>

const App = () => (
  <Router history={browserHistory}>
    <Route path="/" component={PrimaryLayout}>
      <IndexRoute component={HomePage} />
      <Route path="/users" component={UsersPage} />
    </Route>
  </Router>
)

render(<App />, document.getElementById('root'))
```

在上面的路由中，配置集中在`App`中，`<Route>`直接嵌套与`<Router>`，页面组件作为路由的一分部分与路由机密耦合。这便是整个路由的配置。

V4版本的写法：

```
import { BrowserRouter, Route } from 'react-router-dom'

const PrimaryLayout = () => (
  <div className="primary-layout">
    <header>
      Our React Router 4 App
    </header>
    <main>
      <Route path="/" exact component={HomePage} />
      <Route path="/users" component={UsersPage} />
    </main>
  </div>
)

const HomePage =() => <div>Home Page</div>
const UsersPage = () => <div>Users Page</div>

const App = () => (
  <BrowserRouter>
    <PrimaryLayout />
  </BrowserRouter>
)

render(<App />, document.getElementById('root'))
```

V4中React-Router仓库拆分成了多个包进行发布：

- `react-router`，路由基础库
- `react-router-dom`，浏览器中使用的封装
- `react-router-native`，React Native的封装

## 嵌套路由

相比于V3中的集中式的静态路由，V4中似乎看不到路由的影子了，它穿插在了各组件中。尤其是在需要路由嵌套的情况下更加明显，路由嵌套到了组件中，而不再是像V3中那样，`<Route>`中嵌套`<Route>`，V4中`<Route>`中不允许再嵌套`<Route>`

```
// V3
const App = () => (
  <BrowserRouter>
    <div>
      {/* 外层路由 */}
      <Route path="/tacos" component={Tacos}/>
    </div>
  </BrowserRouter>
)

// 当URL匹配`/tacos`时渲染Tacos组件
const Tacos  = ({ match }) => (
  <div>
    {/* 嵌套路由，match.url可以更方便的使用相对地址 */}
    <Route
      path={match.url + '/carnitas'}
      component={Carnitas}
    />
  </div>
)
```

上面的代码中，`<Tacos>`组件是否展示，取决于当前路由是否与`/tacos`匹配，而`<Route>`可以理解为一个容器，它做的事情很简单，将传入的`path`与当前的`location`进行比较，撇皮则渲染`component`属性传入的组件，否则`return null`

在看一个嵌套路由的例子，V3中`<Route>`中嵌套`<Route>`：

```
 <Router history={history} createElement={createElement}>
   <Route path="/" component={App}>
     <IndexRoute component={Home} />
     <Route path="home" component={Home} />
     <Route path="about" component={About} />
     <Route path="*" component={NotFound} />
   </Route>
 </Router>
```

最顶层的`<Route>`对应的是`<App>`这个组件，这个组件的事情就是渲染`<TopMenu>`，并且把传入的子组件全部渲染出来：

```
const App = ({children}) => {
  return (
    <div>
      <TopMenu />
      <div>{children}</div>
    </div>
  );
};
```
因为`<App>`对应的URL是`/`，所以所有的路由都会鲜明中`<App>`，然后`<App>`会渲染子组件，而它的子组件除了`<TopMenu>`之外就是其他的路由`<Route>`，于是`/home`就会命中对应的路由，渲染结果中既包含了`<TopMenu>`也包含了`<Home>`

在V4中，`<Route>`不能再包含`<Route>`，代码需要改为：

```
<Router history={history}>
  <div>
    <TopMenu />
    <Switch>
      <Route exact path="/" component={Home} />
      <Route exact path="/about" component={About} />
      <Route path="*" component={NotFound} />
    </Switch>
  </div>
</Router>
```
`<Route>`变成了普通的React组件，可以认为它是在渲染时动态决定路由对应什么组件的。`<Switch>`的左右就是只渲染子组件中`<Route>`第一个匹配的。


## 动态路由的问题

动态路由也有着自己的问题：

1. 不够直观，由于路由分散，无法一下子了解程序中所有的路由
2. 测试难度增加，组件中掺杂了路由逻辑，原本针对组件的单元测试需要考虑路由的逻辑

实际上，React开发者对V4版本的改变也是喜忧参半，React官方也并没有强制开发者升级，因为V2/3版本会持续维护。


## 参考
- [心中无路由，处处皆自由/react-router v4 动态路由@github](https://github.com/wayou/wayou.github.io/issues/16)
- [React Router v4 几乎误我一生@知乎](https://zhuanlan.zhihu.com/p/27433116)
