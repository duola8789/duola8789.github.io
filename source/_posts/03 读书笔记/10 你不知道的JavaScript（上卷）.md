---
title: 《你不知道的JavaScript（上卷）》读书笔记
top: false
date: 2020-03-27 11:29:45
updated: 2020-03-27 11:29:49
tags:
- JavaScript
categories: 读书笔记
---

[《你不知道的JavaScript（上卷）》](https://book.douban.com/subject/26351021/)读书笔记。

<!-- more -->

# 第一部分 作用域和闭包

## 第一章 作用域是什么

### 编译原理

JavaScript是一门编译语言，但是与传统的编译语言不同，它不是提前编译的。

传统编译语言的编译过程分为三个步骤：

1. 分词/词法分析：由字符组成的字符串分解成为有意义的代码块，即词法单元
2. 解析/语法分析：将词法单元流转换成为一个由元素主机嵌套组成的、代表了程序语言结构的树，即抽象语法树（AST)
3. 代码生成：由AST转换成为可执行代码的过程

与其他语言不同，JavaScript的编译过程不是发生在构建之前，大部分情况是发生在代码执行前的几微秒的时间内。

### 理解作用域

```JS
var a = 1;
```

编译器会进行如下处理：

1. 遇到`var a`，会讯问作用域是否有一个该名称的变量存在于同一个作用域中，**如果是编译器会忽略该声明**，继续进行编译，否则会在当前作用域中声明一个新的变量，并命名为`a`
2. 编译器会为引擎生成运行时所需的代码。引擎运行时会查找作用域集合中是否存在变量`a`，如果找到就会使用这个变量并复制，如果没找到就会一直找，直至找不到时抛出异常

所以，变量的赋值会执行两个动作：**先声明（编译器）再赋值（引擎）**

1. 编译器会在当前作用域声明一个变量（如果之前没有声明过）
2. 运行时引擎会在作用域中查找这个变量，如果能找到就赋值

引擎查找变量有两个方式：LHS和RHS，当变量出现在赋值操作的左侧时进行LHS查询，出现在右侧时进行RHS查询。**LSH是获取赋值操作目标，RHS是取到源值**

```JS
console.log(a); // RHS（因为没有赋值）
a = 2; // LHS

function foo(a) { // LHS
  console.log(a); // RHS
}

foo(2); // RHS，找到foo的值并返回

function foo(a) { // LHS
  var b = a; // LHS + RHS
  return a + b; // RHS * 2
}

var c = foo(2); // LHS + RHS
```

### 作用域嵌套

作用域是根据名称查找变量的一套规则。

在当前作用域中无法找到某个变量时，引擎就会在外层嵌套的作用域中继续查找，直到找到该变量，或者达到最外层的全局作用域为止

### 异常

区分RHS和LHS很重要，因为如果引擎RHS查询（取值）在所有嵌套的作用域中找不到所需要的变量，引擎就会抛出RefereceError

如果引擎执行LHS查询（赋值）时，在非严格模式下，如果在全局作用域中无法找到目标变量，全局作用域就会创建一个具有该名称的变量（即非严格模式下，不使用`var`，直接`a = 1`，会创建变量`a`）

严格模式会禁止自动或者隐式地创建全局变量，所以在严格模式LHS查询失败时，不会创建全局变量，也会抛出ReferenceError异常

## 第二章 词法作用域

作用域u有两种主要的工作模型，一种是词法作用域（JavaScript采用的），另一种叫做动态作用域

### 词法阶段

词法作用域就是定义在词法阶段的作用域，也就是由你在写代码时将变量和块作用域写在哪里决定的。因此当词法分析器处理代码时会保持作用域不变。

无论函数在哪里被调用，也无论它如何被调用，它的词法作用域都只由函数被声明时所处的位置决定。

### 欺骗词法

所谓欺骗词法，就是在运行时『修改』词法作用域，欺骗词法作用域会导致性能下降。有下面几种方法可以修改词法作用域：

（1）`eval`

```JS
function foo(str, a) {
  eval(str); // 欺骗词法
  console.log(a, b);
}

var b = 2;

foo('var b = 3', 1);
```

严格模式中，`eval`在运行时有自己的词法作用域，上面的代码就会报错。

（2）`with`

`with`通常被当做重复引用同一个对象的多个属性的快捷方式，可以不需要重复引用对象本身：

```JS
var obj = {
  a: 1,
  b: 2,
  c: 3,
}

with(obj) {
  a = 3;
  b = 4;
  c = 5;
}

console.log(obj);
```
`with`可以将一个对象处理为词法作用域，这个块内部正常的`var`声明会被添加到`with`所处的函数中。`with`实际上是根据你传递给它的对象凭空创建了一个全新的词法作用域。

```JS
const obj = {};

function foo(a){
  with(a){
    x = 100;
  }
}

function bar(a){
  with(a){
    var y = 100;
  }
}

foo(obj);
console.log(x);

bar(obj);
console.log(y);

console.log(obj);
```

### 性能

JavaScript引擎会在预编译阶段进行多项性能优化，其中某些优化依赖于对代码进行静态词法分析，并预先确顶所有变量和函数的定义位置，才能在执行过程中快速找到标识符。

如果使用了`eval`或者`with`，JavaScript无法在词法分析阶段明确知道它们会接收到什么代码、对作用域进行如何的修改。

所以，如果代码中大量使用`eval`和`with`，那么代码会运行更慢，因为引擎无法再在编译时对作用域查找进行优化。

## 第三章 函数作用域和块作用域

### 函数作用域

 函数作用域的含义是指，属于这个函数的全部变量都可以在整个函数的范围内使用及复用（包括嵌套的作用域内部）。

### 隐藏内部实现

可以利用函数作用域，将变量和函数包裹在一个函数的作用域中，从而实现**隐藏内部实现**的目的。

这种基于作用域的隐藏方法，都是从最小特权原则中引申出来的。最小特权原则是指在软件设计中，应该最小限度地暴露必要内容，将其他内容『隐藏』起来（比如某个模块或对象的API设计）

```JS
function doSomething(a) {
  const b = a + doSomethingElse(a * 2);
  reutrn b * 3;
}
 function doSomethingElse(a) {
  return a - 1;
}
```

例如上面的代码中，`doSomethingElse`应该是`doSomething`的内部具体实现的私有内容，更合理的设计是将这些私有的具体内容隐藏在`doSomething`内部。

```JS
function doSomething(a) {
  function doSomethingElse(a) {
    return a - 1;
  }
  const b = a + doSomethingElse(a * 2);
  reutrn b * 3;
}
```

隐藏作用域中的变量和函数的另一个好处是，避免同名标识符之间的冲突（命名空间）：

```JS
var MyReallyCoolLibrary = {
  awesome: 'stuff',
  doSomething: function () {
    // ...
  }
}
```
### 立即执行函数（IIFE）

函数以`(function)`而不是`function`开始，会当做函数表达式而不是一个标准的函数声明来处理。

> 第一个`()`将函数变成表达式，第二个`()`执行了这个函数

> 一个简单的区分函数声明和函数表达式的方法是，观察`function`在整个声明（不仅仅是一行代码）中出现的位置，如果是第一个单词，那么就是函数声明，否则就是函数表达式

```JS
(function foo(){
  var a = 3;
  console.log(a); // 3
})()
```

立即执行函数（Immediate Invoked Function Expression）的名称标识符，只能在`(function foo(){...})`的`...`中被访问，`foo`变量名被隐藏在自身中意味着不会非必要的污染外部作用域。

IIFE的一个非常普遍的用法是把它们当做函数调用并传递参数进去，例如

```JS
var a = 2;
(function IIFE(global) {
  console.log(global.a); // 2
})(window);
```

IIFE还有一种变化的用途是导致代码的运行顺序，将需要运行的函数放在第二位，在IIFE执行后当做参数传递进去。这种模式在UMD（Universal Module Definition）中被广泛使用：

```JS
var a = 2;
(function IIFE(def) {
  defn(window)
})(function def(global){
  var a = 3;
  console.log(a); // 3
  console.log(global.a); // 2
});
```

### 匿名和具名

函数表达式可以是匿名的，函数声明不可以省略函数名。

匿名函数有几个缺点：

1. 匿名函数在调用堆栈中不会显示出有意义的函数名，增加调试难度
2. 匿名函数无法递归调用自己（严格模式下无法使用`arguments.callee`）
3. 匿名函数造成了代码可读性下降

所以最佳实践是，**始终给函数表达式命名**

```JS
setTimeout(function timeoutHandler() {
  // ...
}, 1000);
```

### 块作用域

JS中能够创建块作用域的方法除了函数作用域之外，还有：

（1）`with`

用`with`从对象中创建出的作用域仅仅在`with`声明中而非外部作用域中生效

（2）`try/catch`

`catch`分句会创建一个块作用域，其中声明的变量仅在`catch`内部有效

（3）`let`/`const`

`let`关键字可以将变量绑定到所在的任意作用域中（通常是`{...}`内部），使用`let`进行的声明不会在块作用域中进行提升

块作用域非常有用的原因是闭包及垃圾收回机制有关，块作用域中的变量会被GC及时回收

## 第四章 提升

### 声明提前

```JS
a = 2;
var a;
console.log(a)
```

包括变量和函数在内的所有声明都会在任何代码执行之前都会被首先处理

`var a = 2`会被看成两个声明，`var a`和`a = 2`，前者在编译阶段进行（编译），后者留在原地等待执行阶段（执行）


```JS
console.log(foo);
function foo(){}

console.log(bar);
var bar = function (){}

console.log(fn);
var baz = function fn (){}
```

要注意第三组代码，叫做函数命名表达式，`fn`是可选的函数名称，**只能存在于函数体内**

### 函数优先

函数首先会被提升，然后才是变量


```JS
console.log(foo);

var foo;

function foo(){}

foo = 123;
```

一个普通块内部的函数声明通常会被提升到**所在作用域**的顶部：

```JS
foo();
var a = true;
if (a) {
  function foo() {
    console.log(1)
  }
} else {
  function foo() {
    console.log(2)
  }
}
```

## 第五章 作用域闭包

### 什么是闭包

**闭包是基于词法作用域书写代码时产生的自然结果**。即，当函数可以记住并访问所在的词法作用域时，就产生了闭包，即使函数是在当前词法作用域之外执行的

一个典型的闭包：

```JS
function foo() {
  var a = 2;
  function bar() {
    console.log(a);
  }
  return bar;
}

var a = 1;
var baz = foo();

baz();
```

`bar`在自己定义的词法作用域之外的地方执行，`foo`执行后，`foo()`的内容不会被GC回收，而闭包阻止了这件事情的发生，因为`bar`本身在使用

`bar`仍然持有对该作用域的引用，这个引用就叫做闭包。这个函数在定义时的词法作用域以外的地方被调用。**闭包使得函数可以继续访问定义时的词法作用域**。

只要使用了回调函数，实际上就是在使用闭包：

```JS
function wait(message) {
  setTimeout(function(){
    console.log(message);
  }, 1000)
}
wait('Hello');
```
IFEE是最常用来创建可以被封闭起来的闭包的工具。

### 循环和闭包

判断下面的输出：

```JS
for (var i = 0; i <= 5; i++) {
  setTimeout(function timer() {
    console.log(i)
  }, i * 1000)
}
```

并不会如我们预期的那样，间隔1秒输出数字`1-5`，而是间隔1秒输出五次`6`

这是因为我们试图假设循环中的每个迭代在运行时都会给自己『捕获』一个`i`的副本，而实际上根据作用域的工作原理，尽管循环中的五个函数实在各个迭代中分别定义的，但是他们都被封闭在一个共享的全局作用域中，因此实际上只有一个`i`

解决的方法就是为每个循环的过程的迭代单独创建一个闭包作用域，在单独的闭包作用域中创建自己的`i`：

```JS
for (var i = 0; i <= 5; i++) {
  (function (){
    var j = i;
    setTimeout(function timer() {
      console.log(j)
    }, j * 1000) // 这里使用 i 还是使用 j 都是一样的
  })()
}
```

另一种解决方法就是为每个迭代创建一个块作用域，在`for`循环声明时的`i`，**如果使用`let`声明，那么每次迭代都会重新声明，随后的每个迭代都会使用上一个迭代结束时的值来初始化这个变量**

```JS
for (let i = 0; i <= 5; i++) {
  setTimeout(function timer() {
    console.log(i)
  }, i * 1000)
}
```

### 模块

可以利用闭包来实现模块：

```JS
function coolModule() {
  var something = 'cool';

  function doSomething() {
    console.log(something)
  }

  return {
    doSomething,
  }
}

var foo = coolModule();
foo.doSomething();
```

上面的模式就被称为模块，最常见的实现模块的方法被称为模块暴露。

返回的对象中含有对内部函数而不是内部数据的引用，这样就保持内部数据变量是隐藏且私有的状态，可以将这个对象类型的返回值看做本质上是模块的公共API。当通过返回一个函数属性引用的对象的方式来将函数传递到词法作用域外部时，就创造了可以观察和实践闭包的条件

> 模块可以返回对象，也可以直接返回一个内部函数，jQuery的`jQuery`和`$`标识符就是jQuery模块的公共API

模块模式需要具备两个必要条件：

1. 必须有外部的封闭函数，改函数必须至少被调用一次（每次调用都会创建一个新的模块实例）
2. 封闭函数必须返回至少一个内部函数，这样内部函数才能在私有作用域中形成闭包，并且可以访问或者修改私有的状态

### 现代的模块机制

实际上介绍的就是AMD的模块机制（终于明白一些了），本质上就是将上面的模块定义封装筋也一个友好的API：

```JS
var MyModules = (function Manager() {
  var modules = {};

  function define(name, deps, impl) {
    for (var i = 0; i < deps.length; i++) {
      deps[i] = modules[deps[i]]
    }
    modules[name] = impl.apply(impl, deps)
  }

  function get(name) {
    return modules[name]
  }

  return {
    get,
    define
  }
})()
```
使用的时候：

```JS
MyModules.define('bar', [], function() {
  function hello(who) {
    return 'HELLO ' + who;
  }
  return {
    hello
  }
});

MyModules.define('foo', ['bar'], function (bar) {
  var name = 'Jay';

  function awesome() {
    console.log(bar.hello(name).toUpperCase());
  }

  return  {
    awesome
  }
});

var bar = MyModules.get('bar');
var foo = MyModules.get('foo');

console.log(bar.hello('ZHOU')); // HELLO ZHOU
foo.awesome(); // HELLO JAY
```

### 未来的模块机制

未来已来。

基于函数的模块并不是一个能被静态识别的模式，他们的API语义只有在运行时才会被考虑进来，因此可以再运行时修改一个模块的API

ES6的模块API是静态的（不会在运行时改变），因此可以在编译期间检查对导入模块的引用是否真实存在。如果API引用不存在，编译器在编译时就抛出错误，而不必等到运行期在动态解析后报错

模块文件中的内容会被当做好像包含在作用域闭包中一样被处理。

## 附录A 动态作用域

JavaScript是静态作用域，判断下面的输出结果：

```JS
function foo() {
  console.log(a);
}

function bar() {
  var a = 3;
  foo();
}

var a = 2;

bar();
```

JavaScript并不具有动态作用域，只有词法作用域，简单明了，但是`this`机制在某种程度上『很像』动态作用域

主要区别：词法作用域是在写代码或者说是定义时确定的，而动态作用域是在运行时确定的（`this`也是），词法作用域关注函数在何处声明，而动态作用域关注函数从何处调用。

# 第二部分 `this`和对象原型

## 第一章 关于`this`

`this`是一个很特别的关键字，被自动定义在所有函数的作用域中。

### `this`到底是什么？

`this`是在运行时绑定的，并不是在编写时绑定的。它的上下文取决于函数调用时的各种条件。`this`的绑定和函数声明的位置没有任何关系，只取决于函数的调用方式。

当一个函数被调用时，会创建一个活动记录（也称为执行上下文），这个记录会包含函数在哪里被调用（调用栈）、函数的调用方式、传入的参数等信息。`this`就是这个记录的一个属性，会在函数执行的过程中运动。

## 第二章 `this`全面解析

### 绑定规则

必须先找到函数的调用位置，然后判断应用下面四条规则的哪一条，确定`this`绑定对象

- 默认绑定
- 隐式绑定
- 显示绑定
- `new`绑定

#### （1）默认绑定

独立函数调用，是无法应用其他规则的默认规则：

```JS
function foo(){
  console.log(this.a)
}

foo();
```

上面的函数调用时应用了`this`的默认绑定，指向全局对象。

只有运行在非`strict mode`下时，默认绑定才能绑定到全局对象。对于默认绑定，决定`this`绑定对象的并不是调用位置是否处于严格模式，而是函数体是否处于严格模式。

#### （2）隐式绑定

隐式绑定指的是调用位置是否有上下文对象，或者说是否被某个对象拥有或者包含：

```JS
function foo(){
  console.log(this.a)
}

var obj = {
  a: 2,
  foo: foo
}

obj.foo();
```

当函数中引用有上下文对象时，隐式绑定贵则会吧函数调用中的`this`绑定到这个上下文对象。

对象属性引用链只有上一层或者说最后一层在调用位置中起作用：

```JS
function foo(){
  console.log(this.a)
}

var obj1 = {
  a: 2,
  foo: foo
}

var obj2 = {
  a: 42,
  obj1: obj1
}


obj2.obj1.foo(); // 2
```

一个最常见的`this`绑定问题就是被隐式绑定的函数会丢失绑定对象，下面的现象就是一个例子：

```JS
function foo() {
  console.log(this.a)
}

var a = 'global';

var obj = {
  a: 2,
  foo
};

var bar = obj. foo;

bar();
```

`bar`实际上引用的是`foo`函数本身，应用的是默认绑定的规则。另外一个例子：

```JS
function foo() {
  console.log(this.a)
}

var a = 'global';

function doFoo(fn) {
  fn();
}

var obj = {
  a: 2,
  foo
}

doFoo(obj.foo)
```

参数传递其实就是一种隐式赋值

#### （3）显示绑定

使用函数的`call`/`apply`/`bind`方法来实现显示绑定。

如果为`call`等方法传入了一个原始值，那么原始值会被转换成为它的对象形式，这被称为『装箱』

如果将`call`等方法隐藏在一个函数中，那么外界就无法在修改内部的`this`，这也称为『硬绑定』，实际上这就是`bind`方法。

那么我们就可以通过`call`来模拟`bind`

```JS
functio bind(fn, obj){
  return function() {
    return fn.apply(obj, arguments)
  }
}
```

#### （4）`new`绑定

JavaScript中，构造函数就是普通函数，只是被`new`调用而已。使用`new`来调用函数，会执行下面的操作：

1. 创建（或者说构造）一个全新的对象
2. 这个新对象会被执行`[[Prototype]]`连接
3. 这个新对象会绑定到函数调用的`this`
4. 如果函数没有返回其他对象，那么`new`表达式中的函数会自动返回这个新对象

```JS
function Foo(a) {
  this.a = a
}

Foo.prototype.age = 333;

let x = new Foo(1);

let y = {};
Object.setPrototypeOf(y, Foo.prototype);
Foo.call(y, 1);
```

### 优先级

优先级由高到低分别是：`new`→显式绑定→隐式绑定→默认绑定

`new`的优先级是高于隐式绑定的：

```JS
function foo(something) {
  this.a = something;
}

var obj1 = {
  foo
};

var obj2 = {};

obj.foo(2);
console.log(obj1.foo);

obj1.foo.call(obj2, 3);
console.log(obj2.a);

var bar = new obj1.foo(4);
console.log(obj1.a);
console.log(bar.z)
```

`new`的优先级也是高于显式绑定的，`new`和`call`和`apply`是无法一起使用的，所以需要使用`bind`来比较显式绑定与`new`的优先级:

```JS
function foo(something) {
  this.a = something;
}

var obj1 = {};

var bar = foo.bind(obj1);
bar(2);
console.log(obj1.a); // 2

var baz = new bar(3);
console.log(obj1.a); // 2
console.log(baz.a); // 3
```

在`new`与`bind`的比较过程中，`new`对象没有修改`bind`绑定的独享，而是将`this`指向了新生成的对象。

> 对于`bind`的polyfill，参考《JS语言理解13 bind函数的实现》这篇笔记吧。

对`this`的判断，就需要按照上面的优先级进行判断。

### 例外情况

#### 1 被忽略的`this`

如果把`null`或者`undefined`作为`this`的绑定对象传入`call`、`apply`或`bind`，这些值在调用时会被忽略，应用的默认的绑定规则

一般我们把`apply`用来展开数组或者用`bind`进行柯里化时，会将`null`作为绑定对象传入

但是这种做法会导致一些问题，例如某个函数确实使用了`this`，那么传入`null`导致的默认绑定规则会把`this`绑定到全局对象：

```JS
function foo(a, b, c) {
  this.a = 123;
  console.log(a, b , c);
}

foo.apply(null, [4, 3, 2]);
console.log(a); // 123
```

上面的做法就导致了意外修改全局作用域中的`a`

更安全的做法是用一个不具备任何属性的空对象来代替`null`，这个对象可以使用`Object.create(null)`来创建，它不具备任何属性，也不具备原型继承

```JS
function foo(a, b, c) {
  this.a = 123;
  console.log(a, b , c);
}

foo.apply(Object.create(null), [4, 3, 2]);
console.log(a); // undefined
```

#### 2 间接引用

```JS
function foo() {
  console.log(this.a)
}

var a = 2;
var o = {
  a: 3,
  foo
};
var p = {
  a: 4
};
o.foo(); // 3
(p.foo = o.foo)(); // 2
```

赋值表达式返回的值是目标函数的引用，所以调用位置是`foo()`而不是`p.foo()`或者`o.foo()`，所以会应用默认绑定

### `this`词法（箭头函数）

箭头函数不使用`this`的四种标准规则，而是根据外层（函数或者全局）的作用域来决定`this`

```JS
function foo() {
  return (a) => {
    console.log(this.a);
  }
}
var obj1 = {a: 1};
var obj2 = {a: 2};

var bar = foo.call(obj1);
bar.call(obj2); // 1
```

箭头函数最常用于回调函数中，例如`setTimeout`或者事件处理函数中

箭头函数可以像`bind`一样保证函数的`this`被绑定到指定对象，它用更常见的词法作用域代替了传统的`this`机制

## 第三章 对象

### 类型

`typeof null`的结果是`object`，是因为不同的对象在底层都表示为二进制，JavaScript中二进制前三位都为`0`的话会被判断为`object`类型，`null`的二进制表示是全`0`，所以会返回`object`，但是实际上`null`是基本类型

字符串字面量并不是一个对象，但是语言会自动将其转换为一个对象，这样才可以访问原型链上的属性：

```JS
let a = '1';
a instanceof String; // false
a.__proto__ === String.prototype; // true
```

> 所以我认为面试过程包括在网上看到的通过判断`__proto__`实现`instanceof`的方法是不准确的，应该判断`typeof`是不是`object`

### 内容

在对象中，属性名都是字符串，如果使用字符串字面量之外的其他值作为属性名，那它首先会被转换为一个字符串，即使是数字也不例外，对象属性名中的数字也会被转换为字符串

ES6中增加了可计算属性名，可以在文字形式中使用`[]`包括表达式来当做属性名

数组的`length`值只针对索引有效，添加命名属性，`lenth`是不会发生变化的

`Object.assign()`方法的复制是浅复制，它会比那里所有可枚举的属性

### 属性描述符

属性描述符也称为数据描述符，包含四个值：

- `value`，描述属性的数据值
- `writable`，决定是否可以修改属性的值
- `enumerable`，可枚举
- `configurable`，可配置，单向操作（改为`false`后无法撤销）

如果`configurable`为`true`时，我们可以使用`Object.defineProperty`新增或修改已有属性

### 不变性

JavaScript所有的方法创建的都是浅不变性，也就是说它们只会影响目标对象和它的直接属性。

JavaScript中很少需要深不可变性，如果发现需要密封或者冻结所有的对象，那么应该重新思考程序的设计，让它更好的应对对象值的改变。

#### 1 对象常量

结合`writable: false`和`configurable: false`就可以创建一个正常的常量属性（不可修改、重定义或者删除）

#### 2 禁止扩展

使用`Object.preventExtensions`来禁止一个对象添加新属性，并且保留已有属性

#### 3 密封

使用`Object.seal`会在现有对象上调用`Object.preventExtensions`，并且将现有属性标记为`configurable: false`。

密封后不能添加新属性，越不能重新配置或者删除已有属性（但是可以修改属性的值）

#### 4 冻结

`Object.freeze`会在现有对象上调用`Object.seal`并且将现有属性标记为`writable: false`，这样就无法修改他们的值

### Getter和Setter

Getter和Setter可以部分改写默认操作，但是只能应用在单个属性上，无法应用在整个对象上。

它们属于访问描述符（与数据描述符相对）

利用Getter和Setter来取值和赋值时，需要用另一个值来存储。可以是外部变量，也可以是对象的另一个属性。

### 存在性

可以使用`in`操作符和`hasOwnProperty`来检查属性是否存，`in`会检查原型链，`hasOwnProperty`只会检查对象本身

由于有的对象并不会连接到`Object.prototype`上（比如`Object.create(null)`），更保险的调用`hasOwnProperty`的方法是`Object.prototype.hasOwnProperty.call(a, 'a')`

注意，`in`操作符检查的是某个『属性名』是否存在，这个区别对于数组来说很重要，比如：

```JS
4 in [1, 2, 4]; // false
```

上面的结果并不是`true`，因为`in`操作符检查的是属性名，数组的属性名是`0`/`1`/`2`，所以是false

#### （1）可枚举性

可枚举型不会影响`in`和`hasOwnProperty`的结果，但是会影响`for...in`的遍历操作

可以使用`ppropertyIsEnumberable`来判断给定的属性名是否直接存在对象中（而非原型链）并且是可枚举的

#### （2）遍历

- `Object.keys`会包含自身的所有可枚举属性
- `Object.getOwnPropertyNames`会返回所有可枚举和不可枚举的属性

对象遍历的顺序在不同的环境中是不确定的

`for...of`循环首先会向被访问对象请求一个迭代器对象，然后通过调用迭代器对象的`next`方法来遍历所有返回值o

```JS
const arr = [1, 2, 3];
const it = arr[Symbol.iterator]();

it.next(); // {value: 1, done: false}
it.next(); // {value: 2, done: false}
it.next(); // {value: 3, done: false}
it.next(); // {value: undefined, done: true}
```

可以人为的为对象的`Symbol.iterator`添加迭代器函数，实现使用`for...of`遍历对象：

```JS
const obj = {
  a: 1,
  b: 2
};

obj[Symbol.iterator] = function () {
  const keys = Object.keys(obj);
  let index = 0;
  return {
    next: function () {
      return {
        value: obj[keys[index]],
        done: index++ === keys.length
      }
    }
  }
};

for(let i of obj) {
  console.log(i);
}
```

## 第四章 混合对象“类”

### 4.1 类理论

类/继承描述了一种代码的组织结构形式，一种软件中对真实世界中问题领域的建模方法。

面向对象编程强调的是数据和操作数据的行为本质是相互关联的，好的设计就是把数据和它相关的行为打包起来（或者说封装起来）。这也被成为数据结构。

类的核心概念是类、继承和实例化以及多态。多态指的是父类的通用行为可以被子类用更特殊的行为重写。

类理论强烈建议父类和子类使用相当的方法名来表示特定的行为，从而让子类重写父类，但是在JavaScript中这样做会降低代码的可读性和健壮性。

JavaScript中的额类与其他语言中的类并不一样，类是一种可选的设计模式。

### 4.2 类的机制

类是由构造函数来进行实例化的，这个函数的任务就是初始化实例需要的所有信息。

### 4.3 类的继承

在面向类的语言中，可以先定义一个类，然后定义一个继承前者的类。这个子类对于父类来说，是一个独立且完全不同的类。子类会包含父类行为的原始副本，但是也可以重写继承的行为。

类的多态包含两个方面：相对和重写

『多态』中的『相对多态』指的是，任何方法都可以引用继承层次中高层的方法（无论高层的方法是否与当前方法名相同）

『重写』指的是在继承链的不同层次中一个方法名可以被多次定义，当调用方法时会自动选择合适的定义

> JavaScript中父类和子类的关系只存在于两者构造函数对应的`prototype`对象之间，他们的构造函数并不存在直接联系，所以无法简单的实现二者的相对有信用（ES6的`class`中可以用`super`来解决这个问题）

方法的多态性取决于是在哪个类的实例中引用它。


在继承过程中，子类得到的是继承自父类行为的一份副本，子类对继承得到的方法不会影响父类中的影响。

多态并不表示父类和子类有关联，子类得到的只是父类的一份副本。

**类的继承就是复制**。

JavaScript本身不提供多重继承的共恩能够。

### 4.4 混入

混入就是用来模拟类的复制行为。混入分为显示和隐式。

#### 显示混入

JS中的继承也就是对象的引用的复制。

```JS
function mixin(source, target) {
  for(const key in source) {
    if(!(key in target)) {
      target[key] = source[key]
    }
  }
  return target
}

var Vehicle = {
  engines: 1,
  ignition() {
    console.log('turn on my engine')
  },
  drive() {
    this.ignition();
    console.log('start moving forward')
  }
};

var Car = mixin(Vehicle, {
  wheels:  4,
  drive() {
    Vehicle.drive.call(this);
    console.log(`${this.wheels} wheels`)
  }
})
```

上面的函数并没有被复制，复制的是函数引用。

JavaScript在ES6之前没有相对多态的机制，由于`Car`和`Vehicle`都有`drive`函数，为了指明调用对象必须使用『绝对』引用，通过名称显示指定`Vehicle`对象并调用它的`drive`函数

如果执行`Vehicle.drive()`，函数中的`this`会被绑定到`Vehicle`，而我们需要将`this`绑定到子类也就是`Car`中，所以需要使用`call`。

在JavaScript中由于屏蔽，使用显式伪多态会在所有需要使用伪多态引用的地方创建一个函数关联（`Vehicle.drive.call(this)`），这会极大地增加维护成本。此外，由于显式伪多态可以模拟多重继承，所以它会进一步增加代码的复杂度和维护难度，所以应该尽量避免使用显式伪多态。

如果你向目标对象中显式混入超过一个对象，就可以部分模仿多重继承行为

## 第五章

### 5.1 `[[Prototype]]`

JavaScript中的对象都有一个`[[Prototype]]`内置属性，其实就是对于其他对象的引用

在为一个对象赋值的时候：

```JS
obj.foo = 'bar';
```

如果`foo`不直接存在于`obj`，而是存在于原型链上，不一定会触发屏蔽，会出现三种情况：

- 原型链上存在`foo`，且没有标记为只读（`writable`不为`false`），那么会在`obj`中添加`foo`属性，是屏蔽属性
- 原型链上存在`foo`，且标记为只读（`writable`不为`false`），那么无法修改已有属性，也不能在`obj`上创建评比属性（严格模式下报错）
- 原型链上存在`foo`，且它是一个`setter`，那么会调用`setter`，`foo`不会被添加到`obj`中，也不会重新定义`setter`

如果希望上面的第二种和第三种情况也屏蔽`foo`，那么久不能使用`=`来赋值，需要使用`Object.defineProperty`来添加`foo`

有些情况下会产生隐式屏蔽：

```JS
let a = {
  val: 1
};

let b = Object.create(a);

console.log(a.val); // 1
console.log(a.hasOwnProperty('val')); // true

console.log(b.val); // 1
console.log(b.hasOwnProperty('val')); // false

b.val++;
// 实际上是 b.val = b.val + 1;

console.log(a.val); // 1
console.log(a.hasOwnProperty('val')); // true

console.log(b.val); // 2
console.log(b.hasOwnProperty('val')); // true
```

### 5.2  类

类的实例化（或者继承）意味着复制操作，JavaScript默认并不会复制对象属性，相反，JavaScript会在两个对象之间创建一个关联，这样一个对象口可以通过『委托』访问另一个对象的属性和函数。

原型继承并不是继承，而是委托。

`Foo.prototype`默认有一个公用且不可枚举的属性`constructor`，实例本身并没有`constructor`属性，实例访问到的`constructor`属性都是访问原型上的：

```JS
function Foo(){}
Foo.prototype = {};

let a = new Foo();
console.log(a); // Object
```

可以看出，`constructor`是一个非常不可靠并且不安全的引用，通常来说要尽量避免使用这些引用。

JavaScript中的函数并不是构造函数，但是当且仅当使用`new`时，函数调用会变成『构造函数』调用

### 5.3 原型继承

`instanceof`操作符的目的是，在实例的『整条』`[[Prototype]]`链条中是否有`Foo.prototype`指向的对象，它只能判断对象实例和函数之间的关系，不能判断两个对象之间的关联关系。

`A.isPrototypeof(x)`方法用来判断对象`x`的整条`[[Prototype]]`链中是否出现过`A`

`__proto__`看起来很像一个属性，但是它更像一个`getter/setter`，实现大概是这样的：

```JS
Object.defineProperty(Object.prototype, '__proto__', {
  get() {
    return Object.getPrototypeOf(this)
  },
  set(o) {
    Object.setPrototypeOf(this, o);
    return o;
  }
})
```

### 5.4 对象关联

原型链机制就是存在于对象中的一个内部链接，它的通是，如果在对象上没有找到需要的属性或者方法引用，引擎就会在原型链关联的对象上进行查找。同理，如果在后者中也没有找到需要的引用就会继续查找它的原型，以此类推。

使用`Object.create`方法可以方便的关联两个对象，`Object.create`的Polyfill代码如下：

```JS
function myCreate (target) {
  function Foo(){}
  Foo.prototype = target;
  return new Foo()
}
```

`Object.create`指定了需要添加到新对象中的属性名以及这些属性的属性描述符


内部委托比起直接委托可以让API接口更清晰：

```JS
var foo = {
  cool(){
    console.log('cool')
  }
};

var bar = Object.create(foo);

bar.doCool = function() {
  // 内部委托
  this.cool();
}
```

内部委托更清晰，因为在`bar`中确实存在了`doCool`的方法，这样比起直接调用`cool`方法，可读性更强

## 第六章 行为委托

JavaScript的原型链机制本质就是对象之间的关联关系。

> 书中认为使用构造函数来模拟类的设计模式是不太好的，更好的是利用委托设计的模式，利用`Object.create()`来实现对象之间的关联和继承，对象关联风格比类风格的代码更加简洁

## 附录A ES6中的Class

Class通过`super`来实现相对多态，任何方法都可以引用原型链。

Class中的类和实例关系仍然不是复制的关系，而是基于原型链的实时委托，不会复制。

Class也存在一些问题，只能定义类成员属性（定义到原型链上），无法定义类成员属性。如果想要在类实例之间共享状态，只能手动为类的`prototype`上添加共享属性

另外`super`不像`this`的绑定，它不会自动绑定到链中的上一层，它是静态绑定的，不会动态修改。
