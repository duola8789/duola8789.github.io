---
title: 零散专题38 Structuring Frontend Code
top: false
date: 2019-11-03 19:27:24
updated: 2019-11-03 19:27:26
tags: 
- 异步竞态
- 错误
- 逻辑
categories: 零散专题
---

2019.10.3参加了我张立理厂[（知乎：张立理）](https://www.zhihu.com/people/otakustay/activities)大神的分享《Structuring Frontend Code》，感觉很有收获，结合他的PPT，把自己的收获整理成为笔记，日常温习，与大家分享。

<!-- more -->

## 逻辑&组织

到底要表达什么样的逻辑，将逻辑到函数一一映射。

`if`里面的判断的条件是什么？可以使用函数来表达。函数比注释更有用。

**编码是从自然逻辑到程序逻辑的映射过程，其本质是对问题的分解再组合，非复杂度改善的性能往往并没有那么重要。**

也就是说，有些时候对于可维护性、可读性的提高，带来的遍历次数、复杂度的提升，对于前端来说，是可以忽略的。因为前端的数据量往往不会很大，从DOM操作、渲染的回流与重绘角度对前端性能的提升更加显著。

所以，可以牺牲一部分性能，换取可读性和可维护性。

## 以简为道，简以组合

下面的代码，是我现在的水平，实现的目的是做了一个缓存，如果有缓存的时候就走缓存，没有的时候会重新计算，并存到缓存中：

```JS
const evaluationCache = {};
const evaluate = expression => {
  if (!evaluationCache[expression]) {
    evaluationCache[expression] = // 几十行代码;
  }
  return evaluationCache[expression];
};
```

可以进行优化，使用高阶函数，将`几十行代码`抽离出来，便于测试：

```JS
const evaluationCache = {};
const cacheEvaluationResult = evaluate => expression => {
  if (!evaluationCache[expression]) {
    evaluationCache[expression] = evaluate(expression);
  };
}
return evaluationCache[expression];
const evaluate = expression => ...;
const evaluateWithCache = cacheEvaluationResult(evaluate);
```

在此基础上，可以发现，为函数添加缓存，实际上是与业务无关的功能，完全可以抽离成为公共函数`memoize`（一个很多公共库中都有的函数，比如`lodash.memorize`，有时间可以学习一下它的源码）

```JS
const memoize = fn => {
  const cache = new Cache({algorithm: 'lfu', maxSize: 20});
  return (...args) => {
    const cachedResult = cache.find(previous = > shallowEquals(previous, args));
    if (cachedResult.exists) {
      return cachedResult.value;
    }
    const result = fn(...args);
    cache.add(args, result);
  };
  return result;
};
const evaluateWithCache = memoize(evaluate);
const getContainerWithCache = memoize(getContainer);
```

## 组合以传播

有下面这样一个例子，代码的作用是，获得生日在指定时间段的用户的集合，（以我现在的水平）代码如下：

```JS
// Without cache
const getDateRange = months => ...;

const filterBirthdays = (dateRange, users) => users.map(...);

const getRecentBirthday = (months, users) => {
  const dateRange = getDateRange(months);
  return filterBirthdays(dateRange, users);
};
```

如果想加缓存，我可能要在`getRecentBirthday`里面对`getDateRange`和`filterBirthdays`进行修改，但是更好的做法是结合桑上面的`memoize`函数，创建一个`createRecentBirthFilter`工厂函数，这个工厂函数的两个参数就是获得日期范围和筛选用户的两个函数，这样的话，可以随意更改传入的两个函数（是否添加缓存的版本）

```js
// With cache
const createRecentBirthFilter = (getDateRange, filterBirthdays) => (months, users) => {
  const dateRange = getDateRange(months);
  return filterBirthdays(dateRange, users);
};
const getRecentBirthdaysWithCache = createRecentBirthFilter(
  memoize(getDateRange), 
  memoize(filterBirthdays)
);
```

上面的例子说明：**始终将逻辑拆解到最简，将会获得更多的可能性**。将业务无关的逻辑提取出来，争取复用的最大化。

## 优先使用组合

使用面向对象的继承，看起来很美好，对URL添加了前缀：

```JS
class AjaxBase {
  // @protected
  mapURL(requestURL) {
    return '/api' + requestURL;
  },
  // @protected
  request(method, apiURL, data) {
    const url = this.mapURL(apiURL);
    // ...
  };
}

class PostRepository extends AjaxBase {
  savePost(post) {
    return this.request('POST', '/posts', post);
  }
}
```

但是当API分版本控制，不同的请求使用不同的版本，那么会导致不同的子类都需要维护自己的`mapURL`方法：

```JS
class PostRepository extends AjaxBase {
  mapURL(requestURL) {
    return '/api/v2' + requestURL;
  }
}

class CommentRepository extends AjaxBase {
  mapURL(requestURL) {
    if (requestURL.includes('draft')) {
      return '/api/v3' + requestURL;
    }
    return '/api/v2' + requestURL;
  }
}
```

可以使用**组合来代替继承**，虽然本身的实现有些复杂，但是对于代码的可维护性提高确实非常有意的：

```JS
class URLMapperV2 {
  map(url) {
    return '/api/v2' + url;
  }
}

class URLMapperV3 {
  // ...
}

class URLMapperComposite {
  constructor(keywordMapping) {
    this.keywordMapping = keywordMapping;
  }
  map(url) {
    const actualMapper = keywordMapping.find(([prefix]) = > url.includes(prefix));
    return actualMapper.map(url);
  }
}

const commentURLMapper = new URLMapperComposite(
  [
    ['drafts', new URLMapperV3()],
    ['', new URLMapperV2()]
  ]
);
```

实际上使用函数式变成配合组合，是现在更加主流和受推崇的方法，比如React的Hooks API，使用函数进行组合的话，代码会更精简一些：

```JS
const mapV2 = url => '/api/v2' + url;

const mapV3 = url => '/api/v3' + url;

const mapComposite = keywordMapping => url => {
  const map = keywordMapping.find(([prefix]) => url.includes(prefix));
  return map[1](url);
};

const mapCommentURL = mapComposite(
  [
    ['drafts', mapV2()], 
    ['', mapV3()],
  ]
)
```


还有这样的例子，如果使用继承，创建了一个`TextBox`基类，可以通过继承它得到而来带前缀的`TextBoxWithPrefix`、可缩放的`ResizableTextBox`、带后缀的`TextBoxWithSuffix`，但是随着业务发展，如果需要一个带前缀且可缩放的输入框，继承就有些无能为力了，必然会带来冗余的代码

但是如果换用一种思路，使用组合的方式，每个函数实现为当前的`TextBox`添加一种功能，也是通过高阶函数完成：

```JS
class TextBox {
  // ...
}

const withPrefix = TextBox => class extends TextBox {
  // ...
};

const withSuffix = TextBox => class extends TextBox {
  // ...
};

const draggable = TextBox => class extends TextBox {
  // ...
};

withPrefix(withSuffix(TextBox));

draggable(withPrefix(TextBox));

draggable(withSuffix(TextBox));

draggable(withPrefix(withSuffix(TextBox)));
```

可以看出来，**组合提供更小的复用粒度和更灵活的按需调整**。

## 从编程到编织

有一些基础的功能，通常是与业务无关的功能，比如日期时间的处理，完全没有必要自己从零开始造轮子，社区里有现成的、更加健壮的工具可以使用，从而让我们把主要的关注点放到业务逻辑上。

这一点也是最近开始有感触，以前实现功能，什么都想自己写，现在也开始逐渐学着使用现有的工具，但是有一些同事还没有这样的概念，处理事件 ，自己写了一堆逻辑，负责且脆弱，依赖的某些方法出现了兼容性问题，导致了Bug。

尝试使用编织而不是编写的方式看待代码，逐渐培养对代码拆解的直觉。

我们工作的目的是什么，我们工作不是为了写代码，而是为了通过写代码来实现需求、解决问题，所以不要纠结于自己写了多少代码，而是解决了什么样的问题，更高效的实现了需求。一切都要从这一点出发。

##  关于纯函数

什么是纯函数：

- 不依赖外部作用域内可变化的状态
- 不对任何环境进行修改操作
- 相同的输入始终得到相同的输出

什么情况会到导致函数不纯？

- 使用`Math.random()`
- 使用`new Date()`
- 使用除了空函数意外的没有返回值的函数
- 异步函数
- 修改全局/作用于内变量的函数
- 修改参数


纯函数具备稳定性，易于调试、测试，纯函数调用纯函数，结果还是纯函数，纯函数调用非纯函数，结果是非纯函数

由于前端必然要操作DOM，所以不不可能完全是纯函数，所以尽量增多纯函数，减少或者标记非纯函数。在正常的函数中引入副作用容易隐藏BUG的源头。例如下面的函数：

```JS
const branches = data.branches.map(ref => {
  if (ref.isDefaultBranch) {
    localStorage.set(`baseBranch: ${repo}`, encodeURIComponent(ref.name));
    localStorage.set(repo, encodeURIComponent(ref.name));
  }
  ref.isBranch = true;
  ref.commit = ref.commitId;
  return ref;
});
```

这显然不是一个纯函数，因为：

- 对`localStorage`进行了写操作
- 在`map`方法中对原数组进行了修改

如何优化这样的函数呢？

- 在`map`函数中不引入副作用（包括`some`/`reduce`等方法)，不应该修改原数组
- 最小化需要副作用的内容
- 明确标记副作用（下面的代码使用了`for...of`来代替`forEach`进行遍历，用来标记有副作用的内容）

**副作用是BUG的种子，使用良好的压缩与隔离让更多的代码稳定可测**。

## 异步与乱序

网络的存在注定前端与异步不可分离，而异步会引入静态，错误的静态处理容易导致状态的错误。最简单的例子就是，分别点击`3`/`2`/`1/`三个按钮，发送请求获取数据，将结果渲染在DOM上，但是由于网速的不稳定，返回的结果顺序是`1`/`2`/`3`，这样就会导致了错乱，按钮`1`最后被点击，而现实的结果是按钮`3`的请求结果：

```HTML
<template>
  <div class="main">
    <p>结果：{{message}}</p>
    <button @click="clickHandler(1)">1</button>
    <button @click="clickHandler(2)">2</button>
    <button @click="clickHandler(3)">3</button>
  </div>
</template>

<script>
export default {
  data() {
    return {
      message: '',
    }
  },

  methods: {
    async clickHandler(type) {
      this.message = await this.fetch(type);;
    },

    fetch(time) {
      return new Promise(resolve => {
        setTimeout(resolve, time * 1000, time)
      })
    }
  },
}
</script>
```

![](http://image.oldzhou.cn/FqQmCAJD7eA_OD3_Uc-8XXscNsXE)

以前面试也会遇到过这种问题，最简单的做法就是在请求内部、结果返回后进行状态校对，其实也是利用了闭包，当时头条的面试官考察的时候我还一知半解

```JS
async clickHandler(type) {
  this.selected = type;
  const res = await this.fetch(type);
  if (type !== this.selected) {
    return;
  }
  this.message = res;
},
```

![](http://image.oldzhou.cn/FqPmvTl6letDX_RiJhyVLQUU92gL)

这种方法的好处就是简单直接，但是缺点也很明显，每一处都需要如此处理，重复编码量大，而且造成了资源浪费，例如上面的例子中，最先点击的按钮`3`返回的结果被舍弃了，但如果我马上再点击`3`，还需要重复发送请求。

所以可以使用资源池以及发布订阅的模式。在面试快手的时候被问到这个问题，想到使用资源池解决这个问题，但是当时被问到，如果连续两个相同的请求如何处理时，我想到的是增加一个`pending`状态，但是依然无法解决问题。

一种解决方法就是使用发布订阅的模式，当我们请求资源时，不直接依赖于请求的结果或者回调函数，而是先进行订阅对应的事件，通过监听这个事件的返回值来获取响应结果。

本来想自己按照PPT上的源码，自己实现一个能够全局缓存、同时能够解决竞态的公共函数，结果发现比自己预想的要复杂。现在项目中最欠缺的是解决缓存的问题，再设置缓存失效时间，且如果对同一个URL的连续两次访问如何避免连续发送两次请求，这些是当前最需要解决的。关于竞态的问题目前还是在组件中单独解决吧。

下面的代码没有经过生产验证，也没有单元测试验证，所以可靠性没有保证。并且设计水平和设计思路都欠佳，希望看到的同学见谅并且予以指正。

```JS
// import axios from 'axios';

// mock axios
const axios = {
  request({url: time}) {
    console.log('HTTP!!!!');
    return new Promise(resolve => {
      setTimeout(resolve, time * 1000, time)
    })
  }
};

const requestFactory = () => {
  // TODO: Vuex
  const cache = {};
  let current = null;

  const flush = key => {
    if (cache[key] && Array.isArray(cache[key].listeners)) {
      cache[key].listeners.forEach(listener => {
        listener(cache[key].value)
      });
      cache[key].listeners = [];
    }
  };

  const add = (key, value) => {
    cache[key].value = value;
    cache[key].pending = false;
    return value;
  };

  return {
    visit({method = 'get', url, params = {}}) {
      const key = [method, url, JSON.stringify(params)].join('_');
      current = key;

      // 缓存中已有值，从缓存取值
      if (cache[key] && cache[key].value !== null) {
        Promise.resolve(cache[key].value).then(() => flush(key));
        return this;
      }

      // 无缓存，但已处于 pending 状态
      if (cache[key] && cache[key].pending) {
        return this;
      }

      // 无缓存，且未 pending
      cache[key] = {value: null, listeners: [], pending: false};
      cache[key].pending = true;

      // 发送请求
      axios.request({method, url, params}).then(value => add(key, value)).then(() => flush(key));

      return this;
    },

    listen(cb) {
      if (!cache[current]) {
        throw new Error('Listen must be called after visit');
      }
      cache[current].listeners.push(cb);
    }
  }
};

const requestWithCache = requestFactory();

export default requestWithCache;
```

**前端的挑战在于状态的持久性，持久的状态加上乱序的异步逐渐引入混乱**。良好的设计有助于解决异步乱序。

## 避免数据的过早加工

对于加工数据，每个加工步骤都存在信息的丢失，大部分错误仅能在末端发现，过长的加工链导致发现的错误难以追溯具体的引入环节。

所以，应该复用加工的工具（函数），而不是加工的结果。

但是这也会带来性能的降低，可以引入缓存解决多次加工的性能冲击。（但是实际上我并不是太理解，引入了缓存，那么也就意味着复用了加工的结果，和前面不是冲突吗？）

## 重构代码

明确重构的边界，也就是明确我们重构的目标是什么，边界之外的代码坚决一行不改动，否则重构就会无边际的蔓延开来，不受控制。

明确重构的边界是目的，但是还需要足够的能力和技巧来完成，那就是明确具备传染性的技术（例如CSS Modules），使用丑陋的方式（例如允许一定的冗余的代码），拒绝重构传染到边界外部。

在重构过程中，使用`git add`进行阶段性归档（比使用撤销或者IDE的历史记录更可靠）

## 编写安全的代码

前端代码的安全性一个重要的隐患就是XSS漏洞，最主要的引入方式之一就是直接使用了`innerHTML`，解决方法：

1. 使用自带转义的模板引擎；
2. 使用白名单语法（例如Plain Text、BBCode、Markdown）
3. 使用第三方的转义库（XSS）
4. 使用CSP

详细的可以参考我的学习笔记[网络基础07 HTTP安全](https://duola8789.github.io/2017/11/06/01%20%E5%89%8D%E7%AB%AF%E7%AC%94%E8%AE%B0/08%20%E7%BD%91%E7%BB%9C%E5%9F%BA%E7%A1%80/%E7%BD%91%E7%BB%9C%E5%9F%BA%E7%A1%8007%20HTTP%E5%AE%89%E5%85%A8/)

## 编写健壮的JavaScript

尽量早地抛出异常，尽量晚的捕获异常，但是不能不捕获，不处理无法处理的异常。对异常进行分类，实现自己业务相关的异常，封装时保留异常原始信息


```JS
const createError = (type, message, extraData, innerException) => {
  const error = new Error(message);
  return Object.assign(error, extraData, {type, innerException});
};

const toErrorString = error => {
  const segments = [
    error.message,
    error.stack,
    ...[error.innerException ? ['Caused by:', toErrorString(error)] : []],
  ];
  return segments.join('\n');
};
```

更多基础知识也可以参考我总结的笔记[零散专题37 前端代码异常监控](https://duola8789.github.io/2019/08/19/01%20%E5%89%8D%E7%AB%AF%E7%AC%94%E8%AE%B0/07%20%E9%9B%B6%E6%95%A3%E4%B8%93%E9%A2%98/%E9%9B%B6%E6%95%A3%E4%B8%93%E9%A2%9837%20%E5%89%8D%E7%AB%AF%E4%BB%A3%E7%A0%81%E5%BC%82%E5%B8%B8%E7%9B%91%E6%8E%A7/)

## 代码评审

我一直希望有高水平的人能认真的评审我的代码，但是很可惜，到现在都没有这样的运气。

现在的工作有这样的机制和工具，但是同组的同学这方面的意识淡薄，只能我抽时间单方面的评审他们的代码，虽然自己也能从他们的代码中学到东西，但是还是觉得很遗憾。

在评审的时候，还不需要再去评审与ESLint重复的工作（所以提交之前的ESLint检查很重要），例如代码格式等，要评审的重点在于：

- 代码阅读的顺畅感
- 代码的重复性、复用性
- 业务逻辑的理解
- 关键设计的遵循度
