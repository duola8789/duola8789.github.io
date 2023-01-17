---
title: React-Router01 入门
top: false
date: 2017-05-07 12:20:20
updated: 2019-05-09 17:16:20
tags:
- React
- React-Router
categories: React
---

React-Router入门学习笔记。

<!-- more -->

## React-Router

React-Router是[官方插件](https://github.com/ReactTraining/react-router)，[文档在这里](https://reacttraining.com/react-router/web/guides/quick-start)。

## 引入

安装：(4.0版本后引入的是`react-router-dom`，而不是原本的`react-router`，使用方式也有了大的变化，和阮一峰的教程以及慕课网的视频教程都不同，这里按照新版本的API写代码）

```BASH
npm install --save react-router-dom
```

新建一个`root.js`，作为入口文件，渲染到文档中，原来的`index.js`作为其中一个组件嵌入，需要在`webpack.config.js`中入口文件的更改：

```JS
module.exports = {
  context: __dirname + '/app',
  entry:  "./js/root.js",
  // ...
 }
```

如果使用了

引入需要的方法：

```JS
import { BrowserRouter, Route, Link } from 'react-router-dom';
```

## `<link>`

连接不使用`a`标签，使用`<link>`标签

```HTML
<ul>
  <li><Link to="/">首页</Link></li>
  <li><Link to="/details">详情页</Link></li>
  <li><Link to="/list">列表页</Link></li>
</ul>
```
## `<BrowserRouter>`

`Hash history`不支持`location.key`和`location.state`。另外由于该技术只是用来支持旧版浏览器，因此更推荐大家使用`BrowserRouter`，此API不再作多余介绍。

`BrowserRouter`，里面的元素是在此基础上跳转的页面

```
import React from 'react';
import ReactDOM from 'react-dom';
import {BrowserRouter, Route, Link} from 'react-router-dom';
import Index from './index'
import ComponentDetails from './components/details';
import ComponentList from './components/list';
import ComponentHeader from './components/header';

export default class Root extends React.Component{
  render(){
    return(
      <BrowserRouter basename="/a/">
        <div>
          <ul>
            <li><Link to="/home">首页</Link></li>
            <li><Link to="/details">详情页</Link></li>
            <li><Link to="/list">列表页</Link></li>
          </ul>
          <Route path="/home" component={Index} />
          <Route path="/details" component={ComponentDetails} />
          <Route path="/list" component={ComponentList} />
        </div>
      </BrowserRouter>
    )
  }
}

ReactDOM.render(<Root />, document.getElementById('example'));
```


## `<Switch>`

当进行地址匹配的时候，如果有多个匹配项值匹配首个匹配的对象（需要增加`exact`属性）


```
export default class Root extends React.Component{
  render(){
    return(
      <BrowserRouter basename="/a/">
        <div>
          <ul>
            <li><Link to="/">首页</Link></li>
            <li><Link to="/details">详情页</Link></li>
            <li><Link to="/list">列表页</Link></li>
          </ul>
          <Route exact path="/" component={Index} />
          <Route path="/details" component={ComponentDetails} />
          <Route path="/list" component={ComponentList} />
        </div>
      </BrowserRouter>
    )
  }
}
```
此时点击首页，会将所有三个`<Route>`全部渲染，但是使用`<Switch>`只会渲染首个匹配对象

可以用这个组件实现前端页面404的功能：

```
import { BrowserRouter as Router, Route, Link, Switch } from 'react-router-dom'

const NoMatch = () => (<div><h1>Sorry, 404</h1></div>);

// 路由设置
export default class RouterContainer extends React.Component {
  render() {
    return (
      <Router>
        <div className="container">
          <nav>
            <div className="head">
              <Link to="/" />
            </div>
          </nav>
          <Switch>
            <Route exact path="/" component={App} />
            <Route path="/demo1/" component={Demo1} />
            <Route path="/demo2/" component={Demo2} />
            <Route path="/demo3/" component={Demo3} />
            <Route component={NoMatch} />
          </Switch>
        </div>
      </Router>
    )
  }
}
```

## 参数传递

传递参数在`<Route>`中的`path`参数后面用`:+paramName`，在组件中使用`this.props.match.params.paramName`来获取传入的值

路由：

```HTML
<BrowserRouter>
  <div>
    <ul>
      <li><Link to="/home">首页</Link></li>
      <li><Link to="/details/123">详情页</Link></li>
      <li><Link to="/list">列表页</Link></li>
    </ul>
    <Route path="/home" component={Index} />
    <Route path="/details/:id" component={ComponentDetails} />
    <Route path="/list" component={ComponentList} />
  </div>
</BrowserRouter>
```

页面：

```HTML
<div>这是详情页details idd {this.props.match.params.id}</div>
```

## Koa-Router

之前对二者的理解存在着偏差

**React-Router是前端路由，而Koa-Router控制的是后端路由**

React是单页面应用，也就是在单一一个页面上实现多个组件、布局的切换，React-Router实现的就是在前端页面上根据url的切换对应不同的组件

实现单页面应用的前提是服务器返回**单页面应用的根地址对应的页面**，也就是需要Koa-Router首先配置好url对应的地址

比如，我想要实现`www.test.com`这个单页面应用，并配置了前端路由：

```HTML
<Router history={ browserHistory } key={ Date.now() }>
  <Route path="/" components={keyPointPage}>
    <IndexRoute component={home} />
    <Route path="page1" components={page1}/>
  </Route>
</Router>
```
当我访问`www.test.com`对应的是`IndexRoute`设置的`home`组件，当我访`www.test.com/page1`加载`page1`组件

但是前提是要服务器首先要对`www.test.com`这个地址的访问请求给出回应，也就是需要通过Koa-Router配置地址，并且对`www.test.co`后面的访问进行配置，都需要访问同一个view模版文件（单页面）

```JS
// 对根地址配置
router.get('/', async(ctx) => {
  await dealPageRequest(ctx, 'keypoint_react');
});

// 对根地址对应的其他地址指向同一个view
router.get('/*', async(ctx) => {
  await dealPageRequest(ctx, 'keypoint_react');
});
```
## 坑

1. 如果要在`<Link>`标签中使用`target="_blank"`属性时只能使用`HashRouter`，不知道为什么`BrowserRouter`不行
2. 根目录如果是`/`，一定要在`<Route>`加上`exact`属性，要不然都会进行匹配。
