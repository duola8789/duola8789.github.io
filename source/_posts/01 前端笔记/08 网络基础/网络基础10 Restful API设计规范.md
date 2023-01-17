---
title: 网络基础10 Restful API设计规范
top: false
date: 2017-11-13 19:01:30
updated: 2020-06-16 14:20:47
tags:
- Restful
- API
- HTTP
categories: 网络基础
---

REST API 设计规范知识总结。

<!-- more -->

# 概念

Restful API用来规范应用如何在HTTP层与API提供方进行数据交互 。

Restful API描述了HTTP层里客户端和服务器端的数据交互规则：客户端通过向服务器端发送HTTP(S)请求，接收服务器的响应，完成一次HTTP交互。这个交互过程中，REST架构约定两个重要方面就是HTTP请求的所采用方法，以及请求的链接。

在请求层面，Restful API规范可以简单粗暴抽象成以下两个规则：

1. 请求API的URL表示用来定位资源
2. 请求的METHOD表示对这个资源进行的操作

# API的URL

## 版本号

在Restful API中，API应当尽量兼容之前的版本。Web端很容易为了适配服务端的新的API接口进行版本升级，而Android、IOS等客户端必须通过用户主动升级产品到新版本，才能适配新接。

为了解决这个问题，在设计Restful API时一般情况下会在URL中保留版本号，并同时兼容多个版本：

```TEXT
【GET】  /v1/users/{user_id}  // 版本 v1 的查询用户列表的 API 接口
【GET】  /v2/users/{user_id}  // 版本 v2 的查询用户列表的 API 接口
```

现在可以在不改变V1版本的接口情况下，新增V2版本的接口满足新的业务需求。服务端会同时兼容多个版本，但是同时维护版本过多也会成为不小的负担。

常见的做法是，不维护全部的兼容版本，而是只维护最新的几个兼容版本，例如维护最新的三个兼容版本。在一段时间后，大部分的用户升级到新的版本后，废弃一些使用量较少的服务端老版本的API接口，并要求使用产品老旧版本的用户墙纸升级。

## 不合理的URL

URL用来定位资源，跟要进行的操作区分开，这就意味这URL**不该有任何动词**

下面示例中的`get`、`create`、`search`等动词，都不应该出现在REST架构的后端接口路径中。在以前，这些接口中的动名词通常对应后台的某个函数。比如：

```TEXT
/api/getUser
/api/createApp
/api/searchResult
/api/deleteAllUsers
```

当我们需要对单个用户进行操作时，根据操作的方式不同可能需要下面的这些接口：

```TEXT
/api/getUser
//  用来获取某个用户的信息，还需要以参数方式传入用户 id 信息）

/api/updateUser
// 用来更新用户信息

/api/deleteUser
// 用来删除单个用户

/api/resetUser
// 重置用户的信息
```

这样的弊端在于：

1. URL更长了
2. 对一个资源实体进行不同的操作就是一个不同URL，造成URL过多难以管理。

其实当你回过头看「URL」这个术语的定义时，更能理解这一点。URL的意思是统一资源定位符，这个术语已经清晰的表明，一个URL应该用来定位资源，而不应该掺入对操作行为的描述。

## Restful的URL

在REST架构的URL应该是这个样子：

1. URL中不应该出现任何表示操作的动词，链接只用于**对应资源**；
2. URL中应该单复数区分，推荐的实践是永远只用复数；比如`GET /api/users`表示获取用户的列表；如果获取单个资源，传入ID，比如`/api/users/123`表示获取单个用户的信息；
3. 按照资源的逻辑层级，对URL进行嵌套，比如一个用户属于某个团队，而这个团队也是众多团队之一；那么获取这个用户的接口可能是这样：

```TEXT
GET /api/teams/123/members/234
// 表示获取 id 为 123 的小组下，id 为234 的成员信息
```

按照类似的规则，可以写出如下的接口:

```TEXT
/api/teams
// 对应团队列表

/api/teams/123
// 对应 ID 为 123 的团队

/api/teams/123/members
// 对应 ID 为 123 的团队下的成员列表

/api/teams/123/members/456
// 对应 ID 为 123 的团队下 ID 为 456 的成员
```

## 特殊情况

有的时候一个资源变化难以使用标准的Restful API来命名，可以考虑使用一些特殊的Actions命名。比如，“密码修改”这个接口的命名很难完全使用名词来构建路径，此时可以引入Action：

```TEXT
PUT  /v1/users/{user_id}/password/actions/modify
// 密码修改
```

## 大小写

根据RFC3986定义，URL是大小写明暗的，所以为了避免歧义，尽量使用小写字母。

# API的请求方法

在很多系统中，几乎只用GET和POST方法来完成了所有的接口操作。这个行为类似于全用`<div>`来布局。实际上，我们不只有GET和POST可用，在REST架构中，有以下几个重要的请求方法：GET，POST，PUT，PATCH，DELETE。这几个方法都可以与对数据的 CRUD 操作对应起来。

> CRUD 是指在做计算处理时的增加(Create)、读取查询(Retrieve)、更新(Update)和删除(Delete)几个单词的首字母简写。即增删改查

简单来说，GET用于查询资源，POST用于创建资源，PUT用于更新服务端的资源的全部信息，PATCH 用于更新服务端的资源的部分信息，DELETE 用于删除服务端的资源。

```TEXT
GET          /users                # 查询用户信息列表
GET          /users/1001           # 查看某个用户信息
POST         /users                # 新建用户信息
PUT          /users/1001           # 更新用户信息(全部字段)
PATCH        /users/1001           # 更新用户信息(部分字段)
DELETE       /users/1001           # 删除用户信息
```

## GET

资源的读取，用GET请求，比如：

```TEXT
GET /api/users
// 表示读取用户列表
```

GET应当实现为一个安全幂等的方法。用于获取数据而不应该产生副作用。

## POST

资源的创建，用POST方法；

POST 是一个**非幂等**的方法，多次调用会造成不同效果；

> 幂等（Idempotent）：如果对服务器资源的多次请求与一次请求造成的副作用是一样的的话，那这个请求方法可以被认为是幂等。

比如下面的请求会在服务器上创建一个`name`属性为`John`的用户，多次请求就会创建多个这样的用户。

```TEXT
POST /api/users

{
  "name": "John"
}
```

## PUT和PATCH

用于更新的HTTP方法有两个，PUT和PATCH。

他们都应当被实现为**幂等方法**，即多次同样的更新请求应当对服务器产生同样的副作用。

PUT和PATCH有各自不同的使用场景：

- PUT用于更新资源的**全部信息**，在请求的`body`中需要传入修改后的全部资源主体；
- PATCH用于**局部更新**，在`body`中只需要传入需要改动的资源字段。

设想服务器中有以下用户资源`/api/users/123`

```TEXT
{
 "id": 123,
 "name": "Original",
 "age": 20
}
```

当我们往后台发送更新请求时，PATCH 和 PUT 造成的效果是不一样。

```TEXT
PUT /api/users/123

{
 "name": "PUT Update"
}
```

上述 PUT 请求操作后的内容是：

```TEXT
{
 "id": 123,
 "name": "PUT Update"
}
```
可以观察到，资源原有的 age 字段被清除掉了。

而如果改用 PATCH 的话，

```TEXT
PATCH /api/users/123

{
 "name": "PATCH Update"
}
```
更新后的内容是：

```TEXT
{
 "id": 123,
 "name": "PATCH Update",
 "age": 20
}
```

请求中指定的`name`属性被更新了，而原有的`age`属性则保持不变。

PATCH的作用在于如果一个资源有很多字段，在进行局部更新时，只需要传入需要修改的字段即可。否则在用PUT的情况下，你不得不将整个资源模型全都发送回服务器，造成网络资源的极大浪费。

## DELETE

资源的删除，相应的请求HTTP方法就是DELETE。这个也应当被实现为一个幂等的方法。如:

```TEXT
DELETE /api/users/123
```

用于删除服务器上`ID`为`123`的资源，多次请求产生副作用都是，是服务器上`ID`为`123`的资源不存在。

## HEAD和OPTIONS

HEAD和OPTIONS不太常用

- HEAD：获取资源的头部信息，比如只想了解某个文件的大小、某个资源的修改日期等
- OPTIONS：获取信息，关于资源的哪些属性是客户端可以改变的。针对非简单请求的CORS请求，会在正式通信之前增加一次HTTP查询请求，称为“预检”请求，对应的请求方法就是OPTION

## 不符合CRUD的情况

在实际资源操作中，总会有一些不符合CRUD的情况，一般会添加控制参数或者把动作转换成资源，Github采用的后者，比如『喜欢』一个gist，就增加一个`/gists/:id/star`子资源，然后对齐进行操作，『喜欢』使用`PUT /gists/:id/star`，『取消喜欢』使用`DELETE /gists/:id/star`

# 查询参数

REST风格的接口地址，表示的可能是单个资源，也可能是资源的集合；当我们需要访问资源集合时，设计良好的接口应当接受参数，允许只返回满足某些特定条件的资源列表。

## 公共参数

常规的公共查询参数有：

参数名 | 作用
---|---
`offset` | 返回记录的开始位置
`limit`  | 返回记录的数量
`keyword`| 提供关键词进行搜索
`sort`   | 指定排序的字段
`orderby`| 指定排序方式

具体来看：

（1）以`offset`和`limit`参数来进行分页：

```TEXT
GET /api/users?offset=0&limit=20
```

（2）使用`keyword`提供关键词进行搜索：

```TEXT
GET /api/users?keyword=john
```

（3）使用`sort`参数和`orderby`参数进行排序

```TEXT
GET /api/users?sort=age&orderby=asc    // 按年龄升序
GET /api/users?sort=age&orderby=desc   // 按年龄降序
```

有的时候也可以只用`orderby`来进行排序：

```TEXT
GET /api/users?se&orderby=age_asc      // 按年龄升序
GET /api/users?se&orderby=age_desc     // 按年龄降序
```

## 个性参数

上面介绍的`offset`、`limit`、 `orderby`是一些公共参数。此外，业务场景中还存在许多个性化的参数：


```TEXT
【GET】  /v1/categorys/{category_id}/enable=[1|0]&os_type={field}&device_ids={field,field,…}
```

注意不要过度设计，只返回用户需要的查询参数，此外，需要考虑是否对查询参数创建数据库索引以提高查询性能。

# 语义化

设计合适的API URL，以及选择合适的请求方法，可以语义化的描述一个HTTP请求的操作。

当我们都熟悉且遵循这样的规范后，基本可以看到一个REST风格的接口就知道如何使用这个接口进行CRUD操作了。

比如下面这面这个接口就表示搜索`ID`为`123`的图书馆的书，并且书的信息里包含关键字`game`，返回前十条满足条件的结果。

```TEXT
GET /api/libraries/123/books?keyword=game&sort=price&limit=10&offset=0
```

同样，下面这个请求的意思也就很明显了吧。

```TEXT
PATCH /api/companies/123/employees/234

{
    "salary": 2300
}
```

# 状态码

服务端会在响应头的`status code`中向用户返回的状态码，它说明了请求的大致情况，是否正常完成、需要进一步处理、出现的错误。大改分为几个区间：

- `2xx`，请求正常处理并返回
- `3xx`，重定向，请求的资源位置发生变化
- `4xx`，客户端发送请求错误
- `5xx`，服务端错误

常见的有以下一些：

状态码 | 状态信息 | 说明
---|---|---
200 | OK | 请求成功
201 | Created | 创建成功
204 | No Content  | 删除数据成功
301 | Moved Permanently  | 请求的资源已经永久性地移动到另外一个地方，<br>后续所有的请求都应该直接访问新地址。<br>服务端会把新地址写在Location头部字段，方便客户端使用
304 | Not Modified  | 请求的资源和之前的版本一样，没有发生改变。用来缓存资源
400 | Bad Request | 请求语法错误，body数据格式有误，body缺少必须的字段等，<br>导致服务端无法处理
401 | Unauthorized | 未授权
403 | Forbidden | 有授权（与401相对），但是被拒绝
404 | Not Found | 客户端要访问的资源不存在
405 | Method Not Allowed | 服务端接收到了请求，资源存在，但是不支持对应的方法。<br>服务端必须返回Allow头部，告诉客户端哪些方法是允许的
406 | Not Acceptable  |用户请求的格式不可得<br>（比如用户请求JSON格式，但是只有XML格式）
500 | Internal Server Error | 服务器发生错误
503 | Service Unavailable | 服务器因为负载过高或者维护，暂时无法提供服务。

# 错误处理

当RESTful API接口出现非2xx的HTTP错误码响应时，采用全局的异常结构响应信息。

一般来说，返回的信息中将`error`作为键名，出错信息作为键值即可。

```JS
{
  error: "Invalid API key"
}
```

也可以采取下面的结构：

```TEXT
HTTP/1.1 400 Bad Request
Content-Type: application/json

{
    "code": "INVALID_ARGUMENT",
    "message": "{error message}",
    "cause": "{cause message}",
    "request_id": "01234567-89ab-cdef-0123-456789abcdef",
    "host_id": "{server identity}",
    "server_time": "2014-01-01T12:00:00Z"
}
```

# 返回结果

## 返回内容

最好采用JSON作为返回内容的格式。如果用户需要其他格式，比如XML，应该在请求头的`Accept`字段中指定。

对于不支持的格式，服务端需要返回正确的状态码并给出详细说明。

## 返回规范

针对不同操作，服务器向用户返回的结果应该符合以下规范。

```TEXT
【GET】     /{version}/{resources}                    // 返回资源对象的列表（数组）
【GET】     /{version}/{resources}/{resource_id}      // 返回单个资源对象
【POST】    /{version}/{resources}                    // 返回新生成的资源对象
【PUT】     /{version}/{resources}/{resource_id}      // 返回完整的资源对象
【PATCH】   /{version}/{resources}/{resource_id}      // 返回完整的资源对象
【DELETE】  /{version}/{resources}/{resource_id}      // 状态码 200，返回完整的资源对象。
【DELETE】  /{version}/{resources}/{resource_id}      // 状态码 204，返回一个空文档
```

## Hypermedia API

RESTful API最好做到Hypermedia，即返回结果中提供链接，连向其他API方法，使得用户不查文档，也知道下一步应该做什么。

比如，当用户向`api.example.com`的根目录发出请求，会得到这样一个文档。

```JS
{
  "link": {
    "rel": "collection https://www.example.com/zoos",
    "href": "https://api.example.com/zoos",
    "title": "List of zoos",
    "type": "application/vnd.yourformat+json"
  }
}
```
上面代码表示，文档中有一个`link`属性，用户读取这个属性就知道下一步该调用什么API了。`rel`表示这个API与当前网址的关系（`collection`关系，并给出该`collection`的网址），`href`表示API的路径，`title`表示API的标题，`type`表示返回类型。

Hypermedia API的设计被称为HATEOAS。Github的API就是这种设计，访问`api.github.com`会得到一个所有可用API的网址列表。

```JS
{
  "current_user_url": "https://api.github.com/user",
  "authorizations_url": "https://api.github.com/authorizations",
  // ...
}
```

从上面可以看到，如果想获取当前用户的信息，应该去访问`api.github.com/user`， 然后就得到了下面结果。

```JS
{
  "message": "Requires authentication",
  "documentation_url": "https://developer.github.com/v3"
}
```

# 其他

1. API的身份认证应该使用[OAuth 2.0](http://www.ruanyifeng.com/blog/2014/05/oauth_2_0.html)框架。
2. 服务器返回的数据格式，应该尽量使用JSON，避免使用XML。

关于REST的更多详细规范，可以参考[这个仓库](https://github.com/aisuhua/restful-api-design-references)。

# 参考

- [RESTful API 设计指南@阮一峰的网络日志](http://www.ruanyifeng.com/blog/2014/05/restful_api.html)
- [RESTful 接口实现简明指南@知乎](https://zhuanlan.zhihu.com/p/28674721?group_id=886181549958119424)
- [服务端指南 | 良好的 API 设计指南@掘金](https://juejin.im/post/59826a4e518825359c5e72d1#heading-7)
- [跟着 Github 学习 Restful HTTP API 设计@cizixs.com](https://cizixs.com/2016/12/12/restful-api-design-guide/)
