---
title: Egg05 Session
top: false
date: 2019-06-11 19:39:28
updated: 2019-06-11 19:39:30
tags:
- Cookie
- Session
categories: Egg
---

Session是Web应用在Cookie基础上的封装，主要目的就是进行用户身份识别。

<!-- more -->

## 关于Session

### 为什么要有Session

HTTP是无状态的协议，但是在很多情况下服务端需要记录用户的状态，典型的场景比如用户登录、购物车等。所以服务端需要某种机制来识别具体的用户，这种机制就是Session。

当用户购物时，服务端并不知道是那个用户操作的，所以需要为特定的用户创建特定的Session，来标识、跟踪这个用户，才能弄清楚用户购物车内的东西并且不与其他用户的购物车混淆。

### Session的保存

Session是保存在服务端的，有一个唯一的标识。服务端保存Session的方法很多，内存、数据库、文件都有。大型的网站一般会有专门的Session服务器集群，用来保存用户回话，这个时候Session是保存在内存的。

### Session如何是识别用户身份

Session是使用Cookie来识别用户身份的。每次HTTP请求时，客户端会发送响应的Cookie到服务器。

第一次创建Session的时候，服务端会在HTTP响应中告诉客户端，需要在Cookie里面记录一个Session ID，以后每次请求都需要把这个ID发送到服务器，服务端就知道是哪个用户发送的请求了。

**这个Session ID是维持Session回话的客户端的唯一标识，是Session实现的核心。**

> 如果浏览器禁用了Cookie，Session ID需要通过url来传递。

### 与cookie的区别

Session是存储在服务端的数据，用来跟踪是被用户的状态，可以保存在集群、数据库、内存、文件中

Cookie是客户端保存信息的一种机制，用来记录用户的一些信息，会随着网络请求发送给服务端，也是实现Session的一种方式

### 其他的实现方式

除了使用Session实现用户身份的识别，也可以使用JWT的机制实现，详情可以参考之前的笔记《JS42 JWT实现单点登录》


## Egg中对Session的处理

Egg内置了[egg-session](https://github.com/eggjs/egg-session)插件，可以直接使用`ctx.session`访问或者修改当前用户的Session

```JS
class HomeController extends Controller {
  async fetchPosts() {
    const ctx = this.ctx;
    
    // 获取 Session 上的内容
    const userId = ctx.session.userId;
    const posts = await ctx.service.post.fetch(userId);
    
    // 修改 Session 的值
    ctx.session.visited = ctx.session.visited ? (ctx.session.visited + 1) : 1;
    
    ctx.body = {
      success: true,
      posts,
    };
  }
}
```

Session可以直接读取或者修改，删除的话直接置为`null`就可以了：

```JS
ctx.session = null;
```

需要**特别注意**的是，设置Session属性时要避免以`_`开头，并且不能为`isNew`这个属性，会造成字段丢失。


Session的实现是基于Cookie的，默认配置下，用户Session的内容加密后直接存储在Cookie的一个字段中：

![](http://image.oldzhou.cn/Fo8Tuy79xzewFaoG6C5QseSu1PHs)

用户每次发送请求时都会带上这个Cookie，在服务端解密后使用。

Session的默认配置如下：

```JS
exports.session = {
  key: 'EGG_SESS',
  maxAge: 24 * 3600 * 1000, // 1 天
  httpOnly: true,
  encrypt: true,
};
```
这些参数中，`key`代表存储Session的Cookie的键值对的`key`是什么，其他的参数都与Cookie的设置参数相同。在默认的配置下，存放Session的Cookie将会加密存储、不能被前端JavaScript访问，这样保证用户的Session是安全的。

## 扩展存储

Session默认放在cookie中，当Session过大时，会带来两个问题：

1. cookie存储空间有限制，可能无法存储过大的Session
2. 每次请求都要发送庞大的Cookie信息

Egg提供了将Session存储到除了Cookie之外的其他存储的扩展方案，只需要设置`app.seesionStore`就可以将Session存储到指定的位置

```JS
// app.js
module.exports = app => {
  app.sessionStore = {
    // support promise / async
    async get (key) {
      // return value;
    },
    async set (key, value, maxAge) {
      // set key to store
    },
    async destroy (key) {
      // destroy key
    },
  };
};
```

可以使用[egg-session-redis](https://github.com/eggjs/egg-session-redis)配合[egg-redis](https://github.com/eggjs/egg-redis)，将Session存储到redis中。

**注意，一旦选择了将Session存储到外部存储中，就意味着系统强依赖于这个外部存储，当它挂了后，我们就完全无法使用Session的相关功能了。因此Egg推荐大家只将必要的信息存储到Session中，保持Session的精简并使用默认的Cookie存储。用户级别的缓存不要存到Session中。**

## Session实践

### 修改用户Session失效时间

Session的配置中的`maxAge`只能设置全局Session的有效期，有一些登陆页面的“记住我”的功能会让登陆用户的Session有效期更长，这种针对特定用户的Session有效时间社会可以通过`ctx.session.max`来实现

```JS
const ms = require('ms');

class UserController extends Controller {
  async login() {
    const ctx = this.ctx;
    const { username, password, rememberMe } = ctx.request.body;
    const user = await ctx.loginAndGetUser(username, password);

    // 设置 Session
    ctx.session.user = user;
    // 如果用户勾选了 `记住我`，设置 30 天的过期时间
    if (rememberMe) ctx.session.maxAge = ms('30d');
  }
}
```
### 延长用户Session有效期

默认情况下，当用户请求没有导致Session被修改时，框架不会延长Session的有效期。有些时候希望用户长时间都在访问我们的站点，要延长其Session有效期，不让用户退出登录状态。

框架提供了`renew`配置项来实现上述功能，它会在发现当用户Session的有效期仅剩下最大有效期的一般时，重置Session的有效期。

```JS
// config/config.default.js
module.exports = {
  session: {
    renew: true,
  },
};
```

## 参考

- [Session@Egg](https://eggjs.org/zh-cn/core/cookie-and-session.html#session)
- [详解 Cookie 和 Session 关系和区别@Ruby China](https://ruby-china.org/topics/33313)



