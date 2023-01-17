---
title: JS语言理解07 this的理解
top: false
date: 2017-08-24 09:14:18
updated: 2019-09-23 11:57:26
tags: 
- bind
- this
categories: JavaScript
---

JavaScript里`this`非常重要，又很让我迷惑，需要反复的学习、体会。

<!-- more -->

## 什么是`this`

`this`是在运行的时候绑定的，并不是在编写的时候绑定，它的上下文取决于函数调用时候的各种条件。所以`this`的绑定和函数声明的位置没有关系，只与函数的调用方式有关系。`this`是上下文的一个属性。

> 上下文： 当一个函数调用的时候，会创建一个活动记录（上下文），这个记录会包含函数在哪里被调用，调用的方法，传入参数的信息。this就是这个记录（上下文）的一个属性

在全局函数中，`this`等于`window`，而当函数被视为某个对象的方法调用时，`this`等于那个对象。

**`this`实际上是在函数被调用时发生的绑定，它指向什么完全取决于函数在哪里被调用**

也就是说，**只有函数才有`this`属性，其他的对象并没有这个属性**。

## `this`的绑定规则

上面提到了，`this`是在函数被调用时发生的绑定，它指向什么完全取决于在哪里调用。所以说，我们的第一步就是判断函数的调用位置。

分析出了函数的调用位置之后，接下来就是要判断它是应用于哪种绑定规则，`this`有4种绑定规则：

1. 默认绑定
2. 隐式绑定
3. 显示绑定
4. `new`绑定

四种绑定规则的优先级从上到下依次递增，默认绑定的优先级最低，`new`绑定的优先级最高，而`bind`方法就是显示绑定的一种


（1） 默认绑定

独立函数调用时，`this`默认指向全局

```JS
var a = 2;
function foo() {
  console.log(this.a);
}

foo(); 
// 2
```
这个例子中，函数的调用位置是全局环境，所以其指向全局即等于`window`（注意这是在非严格模式下），如果在严格模式下会输出`undefined`

（2）隐式绑定 

调用位置是否有上下文对象，或者说是否被某个对象拥有或者包含。

```JS
function foo() {
  console.log(this.a);
}

var obj = {
  a: 2,
  foo: foo
};

obj.foo(); 
// 2
```

当函数引用有上下文对象时，隐式绑定规则会把函数调用中的`this`绑定到这个上下文对象中。当然，也有一些隐式绑定的函数丢失绑定对象的问题，例如：

```JS
var a = '全局中的a';

function foo() {
  console.log(this.a);
}

var obj = {
  a: 2,
  foo: foo
}

var s = obj.foo;
s(); 
// 全局中的a;
```

`s`只是`obj.foo`的一个引用，而在当`s`调用的时候已经没有了上下文对象，因此将会默认绑定，从而输出`全局中的a`

（3）显示绑定

用`call`/`apply`/`bind`方法，我们可以在调用时强制把函数的`this`绑定到某个对象上。

（4）`new`绑定

使用`new`关键字调用一个构造函数时，也会发生`this`的绑定，具体过程：

1. 创建一个新对象
2. 这个对象会被执行`__proto__`连接
3. 这个新对象会绑定到函数调用的`this`.
4. 如果函数没有返回其他对象，那么`new`表达式中的函数会自动返回这个新对象。

```JS
function foo(a) {
  this.a = a;
}

var bar = new foo(2);
console.log(bar.a); 
//2
```

## 一种对`this`的理解方式

来理解一道题：

```JS
var obj = {
  foo: function(){
    console.log(this)
  }
}

var bar = obj.foo

obj.foo() 
// obj

bar() 
// window
```

为什么是这样的结果呢？

先看 `bar()`，相当于是这样的调用：

```JS
bar.call(undefined)
```
在非严格模式下，如果传入的 context 是 `context` 或者 `undefined`，那么就默认是 `window`

而对象的调用 `obj.foo()`，相当于是这样的调用：

```JS
obj.foo.call(obj)
```
结果自然就是 `obj`

所以可以得出这样的结论：

1. `this` 就是 `call` 一个函数时，传入的第一个参数
2. 如果函数调用形式不是 `call` 形式，请转换为 `call` 形式再来看 `this` 指向谁

## `call`/`apply`和函数执行的本质

当我们执行一个函数，一下集中调用方式等价：

```JS
"use strict"
function fn(a, b) {
  console.log(this)
}

fn(1, 2)

// 等价于
fn.call(undefined, 1, 2)
fn.apply(undefined, [1, 2])
```

在严格模式下， `fn`里的`this`就是`call`的第一个参数，也就是`undefined`。

在非严格模式下， `call`传递的第一个参数如果是`undefined`或者`null`，那`this`会自动替换为`Window`对象

```JS
var obj = {
  fn: function(a, b) {
    console.log(this)
  },
  child: {
    fn2: function() {
      console.log(this)
    }
  }
}
obj.fn(1, 2)
// 等价于
obj.fn.call(obj, 1, 2)
obj.fn.apply(obj, [1, 2])
// 所以 this 是 obj

obj.child.fn2()
// 等价于
obj.child.fn2.call(obj.chid) 
// 所以 this 是 obj.child
```

## 练习

以上就是原理，我们根据原理来发散做几道测试题

```JS
var name = '饥人谷';

var people = {
  name: '若愚',
  sayName: function() {
    console.log(this.name)
  }
};

var sayAgain = people.sayName;

function sayName() {
  console.log(this.name)
};

sayName()
//解析：相当于 sayName.call(undefined) 
//因为是非严格模式，所以 this 被替换成 Window，所以这里输出全局的 name 即 "饥人谷"

people.sayName()
//解析: 相当于 `people.sayName.call(people)` 
//所以这里输出 `people.name` 即 "若愚"

sayAgain()
//解析: 相当于 `sayAgain.call(undefined)` ，
//因为是非严格模式，所以 this 被替换成 Window，所以这里输出全局的 name 即 "饥人谷"
```

第二道：（这道题有点意思）

```JS
var arr = []
for (var i = 0; i < 3 ; i++) {
  arr[i] = function () { 
    console.log(this) 
  }
}
var fn = arr[0]

arr[0]()
// 解析: 因为函数是个特殊的对象，所以 arr 相当于 
// { 
//   '0': function () {}, 
//   '1': function () {}, 
//   '2': function () {}, 
//   length: 3
// }
// arr[0] 相当于 arr.0 （当然这种写法不符合规范）
// 所以 arr[0] 等价于 arr.0.call(arr), this 就是 arr

fn()
// 解析: 相当于 `fn.call(undefined)`， 所以 fn 里面的 this 是 Window
```

## `bind`

`bind`的作用和`call`与`apply`类似，区别在于使用上。`bind`的执行的结果返回的是绑定了一个对象的新函数

先看一个使用例子：

```JS
var obj = { name: '饥人谷' };

function sayName(){
  console.log(this.name);
}

var fn = sayName.bind(obj) ;
// 注意 这里 fn 还是一个函数，功能和 sayName 一模一样，区别只在于它里面的 this 是 obj

fn() 
// 输出： '饥人谷'
```

`bind()`方法创建了一个新的函数，在`bind()`被调用时，这个新函数的`this`会被设定为`bind`方法传入的第一个参数，其余的参数将作为新函数的参数供调用时使用。

从上面的定义来看，`bind`函数的功能包括：

1. 改变原函数的`this`指向，绑定`this`
2. 返回原函数的拷贝，并预传参数

要注意的，**当使用`new`调用`bind`返回的参数时，`bind`绑定`this`失效**。这是由于上面提到过的`this`绑定的优先级，`new`的优先级高于`bind`

## 箭头函数

箭头函数没有自己的`this`，它的`this`是继承而来，默认指向在定义它时,它所处的对象（宿主对象），而不是执行时的对象,


```JS
let app = {
  fn1: function(a) {
    console.log(this) 
  }
  fn2 (a) {
    consoel.log(this)
  },
  fn3: (a) = > {
    console.log(this)
  }
};

app.fn1();
// app

app.fn2();
// app

app.fn3();
// window
```

粗略一看，`fn1`、`fn2`、`fn3`貌似都一样，实际上`fn1`和`fn2`完全等价，但`fn3`是有区别的

以上代码等同于

```JS
app.fn2.call(app)
app.fn2.call(app)
app.fn3.call(/* 它的上一级的 this */ window)
```

再看一道题目：

```JS
const app = {
  init() {
    const menu = {
      fn1: () => {
        console.log(this);
        // 相当于  menu.fn1.call(/* menu 所在的环境下的 this */)  
        // 所以 fn1 里面的 this 也就是 app。
      },
      fn2 () {
        console.log(this);
        // 相当于  menu.fn2.call(/* menu */)  
        // 所以 fn1 里面的 this 也就是 app。
      }
    };
    menu.fn1();

    menu.fn2()
  }
};
app.init()
```

再看一道题目：

```JS
const app = {
  fn1() {
    setTimeout(function () {
      console.log(this)
    }, 10)
  },
  fn2() {
    setTimeout(() => {
      console.log(this)
    }, 20)
  },
  fn3() {
    setTimeout((function () {
      console.log(this)
    }).bind(this), 30)
  },
  fn4: () => {
    setTimeout(() => {
      console.log(this)
    }, 40)
  }
};

app.fn1();
// fn.call(undefined) ，所以输出 window
// window

app.fn2();
// 箭头函数没有自己的 this，借用 setTimeout 外面的 this，也就是 app
// app

app.fn3();
// 创建了一个新函数，这个新函数里面绑定了外面的 this，也就是 app
// app

app.fn4();
// 箭头函数没有 this，用 setTimeout 外面的 this
// setTimeout 所在的 fn4 也是箭头函数，没有自己的 this，借用外面的 this ，也就是 window     
// window
```

## 易犯错误

（1）`this`对象指向自身

```JS
function foo(num) {
  this.count++;
}

var count = 0;
foo.count = 0;

for (var i = 0; i < 5; i++) {
  foo(i)
}
console.log(foo.count);
// 0

console.log(count);
// 5
```

`this`并不是指向`foo`自身的，因此当输出`foo.count`值的时候,其依然是`0`。此时`foo()`作为一个全局函数被调用，`this`指向的`window`

这种错误是对`this`根本没有理解，`this`代表的是函数调用时的上下文环境，而不是从字面意义的理解，表示函数自己

（2）`this`指向自身的作用域

```JS
var a = 1;
function foo() {
  var a = 2;
  console.log(this.a);
}

foo();
// 1
```

仍然是比较初级的错误，`foo`里面定义的`var a`是函数的局部作用于。



## 参考
- [javascript中关于this的理解@博客园](http://www.cnblogs.com/qqqiangqiang/p/5316973.html)
- [别再为了this发愁了------JS中的this机制@front-Thinking](http://www.cnblogs.com/front-Thinking/p/4364337.html)
- [看过这篇文章以后不要再提this@知乎](https://zhuanlan.zhihu.com/p/31823164?group_id=922446377062383616)
- [this 的值到底是什么？一次说清楚@知乎](https://zhuanlan.zhihu.com/p/23804247)
- [Function.prototype.bind()@MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Function/bind)
