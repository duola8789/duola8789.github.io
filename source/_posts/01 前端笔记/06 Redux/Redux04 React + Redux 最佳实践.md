---
title: Redux04 React + Redux 最佳实践
top: false
date: 2019-12-18 10:42:02
updated: 2019-12-18 10:42:02
tags:
- React
- Redux
categories: Redux
---

看到一篇文章，总结了React和Redux的最佳实践，可以作为参考。

<!-- more -->

> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 https://github.com/sorrycc/blog/issues/1

> 更新：我们基于此最佳实践做了一个封装方案：[dva](https://github.com/sorrycc/dva)，可以简化使用 redux 和 redux-saga 时很多繁杂的操作。

![](http://image.oldzhou.cn/FvIVdTBSmmHShVjwdX0t78I7HZb_)

前端变化虽快，但其实一直都围绕这几个概念在转：

*   URL - 访问什么页面
*   Data - 显示什么信息
*   View - 页面长成什么样
*   Action - 对页面做了什么操作
*   API Server - Data 数据的来源

在 Redux 的生态圈内，每个环节有多种方案，比如 Data 可以是 `immutable` 或者 `plain object`，在你选了 `immutable` 之后，用 [immutable.js](https://facebook.github.io/immutable-js/) 还是 [seamless-immutable](https://github.com/rtfeldman/seamless-immutable)，以及是否用 [redux-immutable](https://github.com/gajus/redux-immutable) 来辅助数据修改，都需要选择。

本文总结目前 react + redux 的最佳实践，解释原因，并提供可选方案。

心急的朋友可以直接看代码：[https://github.com/sorrycc/github-stars](https://github.com/sorrycc/github-stars)

一、URL > Data
------------

### 需求

routing

### 选择

[react-router](https://github.com/rackt/react-router) + [react-router-redux](https://github.com/rackt/react-router-redux): 前者是业界标准，后者可以同步 route 信息到 state，这样你可以在 view 根据 route 信息调整展现，以及通过 action 来修改 route 。

### 可选

无

二、Data
------

### 需求

为 redux 提供数据源，修改容易。

### 方案

`plain object`: 配合 combineReducer 已经可以满足需求。

同时在组织 Store 的时候，层次不要太深，尽量保持在 2 - 3 层。如果层次深，可以考虑用 [updeep](https://github.com/substantial/updeep) 来辅助修改数据。

### 可选

[immutable.js](https://facebook.github.io/immutable-js/): 通过自定义的 api 来操作数据，需要额外的学习成本。不熟悉 immutable.js 的可以先尝试用 [seamless-immutable](https://github.com/rtfeldman/seamless-immutable)，JavaScript 原生接口，无学习门槛。

另外，不推荐用 [redux-immutable](https://github.com/gajus/redux-immutable) 以及 redux-immutablejs，一是没啥必要，具体看他们的实现就知道了，都比较简单；更重要的是他们都改写了 `combineReducer`，会带来潜在的一些兼容问题。

三、Data > View
-------------

### 需求

数据的过滤和筛选。

### 方案

[reselect](https://github.com/reactjs/reselect): store 的 select 方案，用于提取数据的筛选逻辑，让 Component 保持简单。选 reselct 看重的是 `可组合特性` 和 `缓存机制` 。

### 可选

无

四、View 之 CSS 方案
---------------

### 需求

合理的 CSS 方案，考虑团队协作。

### 方案

[css-modules](https://github.com/css-modules/css-modules): 配合 webpack 的 css-loader 进行打包，会为所有的 class name 和 animation name 加 local scope，避免潜在冲突。

直接看代码：

Header.jsx

```
import style from './Header.less';
export default () => <div className={style.normal} />;
```

Header.less

```CSS
.normal { color: red; }
```

编译后，文件中的 `style.normal` 和 `.normal` 在会被重命名为类似 `Header__normal___VI1de` 。

### 可选

[bem](https://en.bem.info/), [rscss](http://rscss.io/) ，这两个都是基于约定的方案。但基于约定会带来额外的学习成本和不遍，比如 rscss 要求所有的 Component 都是两个词的连接，比如 `Header` 就必须换成类似 `HeaderBox` 这样。

[radium](https://github.com/FormidableLabs/radium)，inline css 方案，没研究。

五、Action <> Store，业务逻辑处理
------------------------

### 需求

统一处理业务逻辑，尤其是异步的处理。

### 方案

[redux-saga](https://github.com/yelouafi/redux-saga): 用于管理 action，处理异步逻辑。可测试、可 mock、声明式的指令。

### 可选

[redux-loop](https://github.com/raisemarketplace/redux-loop): 适用于相对简单点的场景，可以组合异步和同步的 action 。但他有个问题是改写了 `combineReducer`，会导致一些意想不到的兼容问题，比如我在特定场景下用不了 redux-devtool 。

[redux-thunk](https://github.com/gaearon/redux-thunk), [redux-promise](https://github.com/acdlite/redux-promise) 等: 相对原始的异步方案，适用于更简单的场景。在 action 需要组合、取消等操作时，会不好处理。

### saga 入门

在 saga 之前，你可能会在 action creator 里处理业务逻辑，虽然能跑通，但是难以测试。比如：

```JS
// action creator with thunking
function createRequest () {
  return (dispatch, getState) => {
    dispatch({ type: 'REQUEST_STUFF' });
    someApiCall(function(response) {
      // some processing
      dispatch({ type: 'RECEIVE_STUFF' });
    });
  };
}
```

然后组件里可能这样：

```JS
function onHandlePress () {
  this.props.dispatch({ type: 'SHOW_WAITING_MODAL' });
  this.props.dispatch(createRequest());
}
```

这样通过 redux state 和 reducer 把所有的事情串联到起来。

但问题是：

> Code is everywhere.

通过 saga，你只需要触发一个 action 。

```JS
function onHandlePress () {
  // createRequest 触发 action `BEGIN_REQUEST`
  this.props.dispatch(createRequest());
}
```

然后所有后续的操作都通过 saga 来管理。

```JS
function *hello() {
  // 等待 action `BEGIN_REQUEST`
  yield take('BEGIN_REQUEST');
  // dispatch action `SHOW_WAITING_MODAL`
  yield put({ type: 'SHOW_WAITING_MODAL' });
  // 发布异步请求
  const response = yield call(myApiFunctionThatWrapsFetch);
  // dispatch action `PRELOAD_IMAGES`, 附上 response 信息
  yield put({ type: 'PRELOAD_IMAGES', response.images });
  // dispatch action `HIDE_WAITING_MODAL`
  yield put({ type: 'HIDE_WAITING_MODAL' });
}
```

可以看出，调整之后的代码有几个优点：

*   所有业务代码都存于 saga 中，不再散落在各处
*   全同步执行，就算逻辑再复杂，看起来也不会乱

六、Data <-> API Server
--------------------

### 需求

异步请求。

### 方案

[isomorphic-fetch](https://github.com/matthew-andrews/isomorphic-fetch): 便于在同构应用中使用，另外同时要写 node 和 web 的同学可以用一个库，学一套 api 。

然后通过 `async` + `await` 组织代码。

示例代码：

```JS
import fetch from 'isomorphic-fetch';
export async function fetchUser(uid) {
  return await fetch(`/users/${uid}`).then(res => res.json());
};
```

### 可选

[reqwest](https://github.com/ded/reqwest)

最终
--

![](http://image.oldzhou.cn/FrHsrbNprsOjThBqCyF0NVhSm84T)

（完）
