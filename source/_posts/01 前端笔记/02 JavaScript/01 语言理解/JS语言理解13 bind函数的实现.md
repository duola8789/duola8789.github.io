---
title: JS语言理解13 bind函数的实现
top: false
date: 2019-09-23 12:57:18
updated: 2019-09-23 12:57:26
tags: 
- bind
- new
categories: JavaScript
---

面试经常遇到的问题，以前的实现考虑还是不太全面，总结了一下。

<!-- more -->

## 定义

`bind`的作用和`call`与`apply`类似，区别在于使用上。`bind`的执行的结果返回的是绑定了一个对象的新函数

先看一个使用例子：

```JS
var obj = { name: 'Jay' };

function sayName(){
  console.log(this.name);
}

var fn = sayName.bind(obj) ;
fn() 
// 'Jay'
```

MDN上的定义：

`bind()`方法创建了一个新的函数，在`bind()`被调用时，这个新函数的`this`会被设定为`bind`方法传入的第一个参数，其余的参数将作为新函数的参数供调用时使用。

从上面的定义来看，`bind`函数的功能包括：

1. 改变原函数的`this`指向，绑定`this`
2. 返回原函数的拷贝，并预传参数

要注意的，**当使用`new`调用`bind`返回的参数时，`bind`绑定`this`失效**。这是由于上面提到过的`this`绑定的优先级，`new`的优先级高于`bind`

## 简易版

在面试过程中，无数次遇到过这个问题，如何自己实现一个`bind`函数，一般情况下，都知道了用`apply`/`call`来实现

```JS
Function.prototype.myBind = function (thisArg) {
  if (typeof this !== 'function') {
    return;
  }
  const self = this;
  const args1 = [].slice.call(arguments, 1);
  return function () {
    const args2 = [].slice.call(arguments, 0);
    self.apply(thisArg, args1.concat(args2))
  }
};

const obj = {
  age: 1,
};

function foo(a, b) {
  console.log(this.age, a, b);
}

foo.bind(obj, 2)(3);
// 1 2 3

foo.myBind(obj, 2)(3);
// 1 2 3
```

貌似没有问题了，实现了`bind`上面提到的绑定`this`和复制函数的功能，还附带实现了传参的实现。

但是有一点，对于绑定优先级的处理还有一些问题，`this`有4种绑定规则：

1. 默认绑定
2. 隐式绑定
3. 显示绑定
4. `new`绑定

四种绑定规则的优先级从上到下依次递增，默认绑定的优先级最低，`new`绑定的优先级最高，而`bind`方法就是显示绑定的一种，优先级应该低于`new`绑定

先复习一下`new`调用构造函数时的过程：

1. 创建一个全新的对象
2. 这个对象被执行`[[Prototype]]`连接
3. 将这个对象绑定到构造函数中的`this`
4. 如果函数没有返回其他对象，则`new`操作符调用的函数则会返回这个对象

先看一下正常的`bind`后的函数遇到`new`操作符，表示是如何的：

```JS
const obj = {};

function foo(name) {
  this.name = name;
}

const Person = foo.bind(obj);
Person('jay');
console.log(obj.name);
// 'jay'

let p2 = new Person('chow');
console.log(obj.name); // jay
console.log(p2.name); // chow
```

首先直接调用`bind`后的参数，这时候`this`指向`obj`，所以`obj.name`变成了`jay`，然后通过`new`操作符调用`Person`，根据上面的优先级，`new`的优先级更高，所以`this`会绑定为创建的新对象`p2`，所以对`name`的更改是对`p2`的`name`的更改，`obj`的`name`保持不变

再来看看我们的`myBind`方法遇到`this`后的表现：

```JS
const obj = {};

function foo(name) {
  this.name = name;
}

const Person = foo.myBind(obj);
Person('jay');
console.log(obj.name);
// 'jay'

let p2 = new Person('chow');
console.log(obj.name); // 'chow'
console.log(p2.name); // undefined
```

而我们的简易版`bind`方法，一旦绑定了`this`后，当遇到`new`操作符后也不会更改，而是固定在`obj`上，所以在`new`的过程中绑定的`this`仍然是我们`myBind`绑定的对象`obj`，所以`p2.name`

所以对简易版的`bind`还需要优化

## 优化版

想要解决上面的问题，关键点就是在于处理`new`的第三步，我们的`myBind`函数识别是`new`调用，那么就不再将`this`绑定到我们传入的`thisArg`对象，而是绑定为函数本身调用的`this`

那么问题就转换为了，如何判断一个函数是普通调用，还是通过`new`操作符调用。在`new`的过程中，会首先建立新建对象的原型继承，然后绑定新建对象到构造函数的`this`，也就是下面的过程


```JS
let p1 = new Person();

// 1 let p1 = {};
// 2 p1.__proto__ === Person.prototype;
// 3 p1 === this;
// 4 return p1;
```

通过第二步和第三步，`this`就成为了`Person`的一个实例，可以通过`this insetanceof Person`来判断，所以可以对上面的`myBind`进行优化：

```JS
Function.prototype.myBind = function (thisArg) {
  if (typeof this !== 'function') {
    return;
  }
  const self = this;
  const args1 = [].slice.call(arguments, 1);
  return function fBound() {
    // 判断是否 new 调用
    const target = this instanceof fBound ? this : thisArg;
    const args2 = [].slice.call(arguments, 0);
    self.apply(target, args1.concat(args2))
  };
};
```

再来验证一下效果：

```JS
const obj = {};

function foo(name) {
  this.name = name;
}

const Person = foo.myBind(obj);
Person('jay');
console.log(obj.name);
// 'jay'

let p2 = new Person('chow');
console.log(obj.name); // 'jay'
console.log(p2.name); // 'chow'
```

`myBind`的效果与原生`bind`的效果一致，`new`操作符改变了`bind`的`this`指向。

[MDN上的`bind`实现](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Function/bind#Compatibility)，增加了维护原型关系的步骤，但是我并不是太理解这样做的必要性，我理解`new`的实现就直接将`this`与`fBound`的原型链进行了关联，可以通过`instanceof`判断

```JS
if (!Function.prototype.bind) {
  Function.prototype.bind = function(oThis) {
    if (typeof this !== 'function') {
      // closest thing possible to the ECMAScript 5
      // internal IsCallable function
      throw new TypeError('Function.prototype.bind - what is trying to be bound is not callable');
    }

    var aArgs   = Array.prototype.slice.call(arguments, 1),
        fToBind = this,
        fNOP    = function() {},
        fBound  = function() {
          // this instanceof fBound === true时,说明返回的fBound被当做new的构造函数调用
          return fToBind.apply(this instanceof fBound
                 ? this
                 : oThis,
                 // 获取调用时(fBound)的传参.bind 返回的函数入参往往是这么传递的
                 aArgs.concat(Array.prototype.slice.call(arguments)));
        };

    // 维护原型关系
    if (this.prototype) {
      // 当执行Function.prototype.bind()时, this为Function.prototype 
      // this.prototype(即Function.prototype.prototype)为undefined
      fNOP.prototype = this.prototype; 
    }
    // 下行的代码使fBound.prototype是fNOP的实例,因此
    // 返回的fBound若作为new的构造函数,new生成的新对象作为this传入fBound,新对象的__proto__就是fNOP的实例
    fBound.prototype = new fNOP();

    return fBound;
  };
}
```

这个疑问先放在这里，看以后有没有机会弄明白，或者哪位大神来指点吧。（2019-09-23）

## 参考

- [Function.prototype.bind()@MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Function/bind)
- [bind 函数的实现原理@掘金](https://juejin.im/post/5b3c3377f265da0f8f20131f)
