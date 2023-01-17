---
title: React-Router03 按需加载
top: false
date: 2019-05-13 11:15:03
updated: 2019-05-13 11:15:03
tags:
- React
- React-Router
- 按需加载
categories: React
---

为了提高页面加载性能，加载首屏的加载速度，有的时候我们需要对路由进行懒按需加载，减少首屏需要加载的代码包的体积。

<!-- more -->

按需加载就是不加载当前路由匹配的组件代码，而不加载其他组件的代码

## V3中的实现

在V4版本之前，需要利用`getComponet`这个API来实现按需加载：

```
const about = (location, cb) => {
  require.ensure([], require => {
    cb(null, require('../Component/about').default)
  }, 'about')
}

//配置route
<Route path="helpCenter" getComponent={about} />
```
其中`require.ensuire`是Webpack的语法：

```JS
require.ensure(
  dependencies: String[],
  callback: function(require),
  errorCallback: function(error),
  chunkName: String
)
```

给定`dependencies`参数，将对应的文件拆分到单独的bundle中，此bundle会被异步加载。可以实现动态加载依赖（使用CommonJS的模块语法时这是动态加载依赖的唯一方法），这样能够做到在模块执行时才运行代码，只有在满足某些条件时才加载依赖项。

## V4中的实现

由于V4版本的动态路由的机制，以及废除了`getComponent`这个API，实现按需加载的方法比较复杂，首先需要介绍`bundle-loader`这个Webpack插件，它可以将模块改成异步方式引用，需要单独安装并且在Webpack中进行配置，具体参考[Webpack的文档](https://webpack.docschina.org/loaders/bundle-loader/)。

准备好`bundle-loader`之后，正式开始：

（1）第一步，创建包装组件模型`bundle.js`，用来处理经过`bundle-loader`处理后的组件

```
import React from 'react';

class Bundle extends React.Component {
  constructor(arg){
    super(arg)
    this.state = {
      mod: null,
    }
  }

  componentWillMount() {
    this.load(this.props);
  }
  componentWillReceiveProps(nextProps) {
    if (nextProps.load !== this.props.load) {
      this.load(nextProps);
    }
  }
  // load 方法，用于更新 mod 状态
  load(props) {
    // 初始化
    this.setState({
      mod: null
    });
    /*
     调用传入的 load 方法，并传入一个回调函数
     这个回调函数接收 在 load 方法内部异步获取到的组件，并将其更新为 mod 
     */
    props.load(mod => {
      this.setState({
        mod: mod.default ? mod.default : mod
      });
    });
  }

  render() {
    /*
     将存在状态中的 mod 组件作为参数传递给当前包装组件的'子'
     */
    return this.state.mod ? this.props.children(this.state.mod) : null;
  }
}

export default Bundle ;
```

（2）第二步，创建包装组件的方法

```
import React from 'react';
import Bundle from './Bundle';

// 默认加载组件，可以直接返回 null
const Loading = () => <div>Loading...</div>;

/*
 包装方法，第一次调用后会返回一个组件（函数式组件）
 由于要将其作为路由下的组件，所以需要将 props 传入
 */

const lazyLoad = loadComponent => props => (
  <Bundle load={loadComponent}>
    {Comp => (Comp ? <Comp {...props} /> : <Loading />)}
  </Bundle>
);

//实际上lazyLoad就是一个函数,组件调用即可
export default lazyLoad;   
```
在上面的代码中回到出一个函数供组件调用，传入的参数`loadComponent`就是要加载的组件，在路由中调用，而`bundle.js`作为按需加载的核心，传入的参数`load`就是要加载的组件，其中的内容是动态的子组件

（3）在路由中使用

```
import React from 'react';
import { NavLink, Route, Switch, BrowserRouter as Router } from 'react-router-dom';
import './style/style.css';
import 'bundle-loader';

// bundle模型用来异步加载组件
import Bundle from '../routes/Bundle.js';
import lazyLoad from '../routes/lazyLoad';

import Page1 from 'bundle-loader?lazy&name=page1!../components/page1/index';
import Page2 from 'bundle-loader?lazy&name=page2!../components/page2/index';
import Page3 from 'bundle-loader?lazy&name=page3!../components/page3/index';

class AppPage extends React.Component {
  constructor(arg) {
    super(arg);
    this.state = {};
  }

  render() {
    return (
      <Router basename="/">
        <div className="appWried">
          <div className="appBtn">
            <NavLink to="/page1" className="button" activeClassName="active">
              PAGE1
            </NavLink>
            <NavLink to="/page2" className="button" activeClassName="active">
              PAGE2
            </NavLink>
            <NavLink to="/page3" className="button" activeClassName="active">
              PAGE3
            </NavLink>
          </div>
          <Switch>
            <Route path="/page1" component={lazyLoad(Page1)} />
            <Route path="/page2" component={lazyLoad(Page2)} />
            <Route path="/page3" component={lazyLoad(Page3)} />
          </Switch>
        </div>
      </Router>
    );
  }
}

export default AppPage;
```

在上面的代码中，使用`bundle-loader`加载模块时语法为

```JS
import Page1 from 'bundle-loader?lazy&name=page1!../components/page1/index';
```

其中`lazy`表示懒加载，`name`表示要异生成的文件的名字。

（4）最后需要在`webpack.config.js`文件中进行配置

```JS
output: {
  path: path.resolve(__dirname, './output'),
  filename: '[name].[chunkhash:8].bundle.js',
  chunkFilename: '[name]-[id].[chunkhash:8].bundle.js',
},
```

## 参考

- [模块方法@weback](https://webpack.docschina.org/api/module-methods/)
- [前端性能优化之按需加载(React-router+webpack)@xiaobe](https://www.cnblogs.com/soyxiaobi/p/9535292.html)
