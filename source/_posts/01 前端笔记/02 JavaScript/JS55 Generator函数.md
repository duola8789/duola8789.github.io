---
title: JS55 Generator函数
top: false
date: 2019-01-18 14:47:00
updated: 2019-01-18 14:47:00
tags:
- Generator
- ES6
categories: JavaScript
---

Generator函数的基本知识。

<!-- more -->

## 简介

Generator函数有两个特征：

1. `function`关键字后面有一个`*`
2. 函数体内部使用`yield`表达式

```JS
function* helloWorldGenerator() {
  yield 'hello';
  yield 'word';
  return 'ending'
}
const hw = helloWorldGenerator()
```
调用函数后，函数并不执行，返回的也不是函数运行结果，而是指向内部状态的指针对象（即==遍历器对象==）

然后必须调用遍历器对象的`next`方法，让指针移向下一个状态，直到遇到`yidld`（或者`return`）为止。也就是说，`yiled`是暂停执行的标记，而`next`方法可以恢复执行：

```JS
hw.next()
// { value: 'hello', done: false }

hw.next()
// { value: 'world', done: false }

hw.next()
// { value: 'ending', done: true }

hw.next()
// { value: undefined, done: true }
```
每次调用`nexd`方法就会返回一个对象，对象有着`value`和`done`两个属性，`value`是`yield`后面的值，`done`是一个布尔值，表示便利是否结束。

### `yiled`表达式

`yield`是函数内部的暂停标志，执行逻辑：

（1）遇到`yiled`暂停执行，将其后面的值作为`next`返回对象的`value`属性值

（2）下一次调用`next`方法，继续执行，直到遇到下一个`yield`

（3）如果没有新的`yield`则运行到`return`或者函数运行结束

（4）将`return`的值作为返回对象的`value`属性值，如果没有`return`，返回对象的`value`属性值为`undeinfed`

`yield`提供了==惰性求值==的功能。

==`yield`表达式只能用在Generator函数里面，用在其他地方都会报错==。

### 与Iterator接口的关系

可以将Generator函数赋值给对象的`Symbol.iterator`属性，从而使得对象具有Iterator接口

```JS
let obj = {};
[...obj]; // Uncaught TypeError: obj is not iterable

obj[Symbol.iterator] = function* () {
  yield 1;
  yield 2;
  yield 3;
};

[...obj] // [1, 2, 3]
```
## `next`方法

==`yield`表达式本身没有返回值==或者说总是返回`undefined`（这个指的是它在内部对于本身的传递，而非传递给`next`返回对象的`value`的属性值）

```JS
function* f() {
  let a = yield 100;
  console.log(a, 'a');
}
let gen = f();

gen.next();
// { value: 100, done: false }

gen.next();
// undefined "a"
// { value: undefined, done: true }
```
`a`的值是`undefined`

`next`方法可以带一个参数，这个参数会被当做==上一个==`yield`的表达式的返回值。

```JS
function* f() {
  let a = yield 100;
  console.log(a, 'a');
}
let gen = f();

gen.next();
// { value: 100, done: false }

gen.next('hello');
// hello "a"
// { value: undefined, done: true }
```
这个功能，可以在Generator函数开始运行后，==从外部向函数体内部注入值==，从而调整函数行为。

注意，==`next`注入的参数改变的是`yield`表达式的返回值==：

```JS
function* foo(x) {
  var y = 2 * (yield (x + 1));
  var z = yield (y / 3);
  return (x + y + z);
}

var a = foo(5);
a.next() // { value:6, done:false }
a.next() // { value:NaN, done:false }
a.next() // { value:NaN, done:true }

var b = foo(5);
b.next() // { value:6, done:false }
b.next(12) // { value:8, done:false }
b.next(13) // { value:42, done:true }
```

当执行`b.next(12)`时，不是给`y`赋值`12`，而是`yield (x + 1)`为`12`，所以`value`是`8`

上面提到了，`next`的参数是赋值给==上一个==`yield`表达式返回值，所以在首次调用`next`传参是==无效==的。

==第一次执行`next`方法，等同于启动执行Generator函数的内部代码==

## `for...of`循环

`for...of`循环可以自动遍历Generator生成的Iterator对象，不需要再逐步调用`next`方法

```JS
function* foo() {
  yield 1;
  yield 2;
  yield 3;
  yield 4;
  yield 5;
  return 6;
}

for (let v of foo()) {
  console.log(v);
}
// 1 2 3 4 5
```
要注意的是，==当`nextd`方法的返回值的`done`属性为`true`时，`for...of`循环就会终止，并且不包括`return`的值==

除了`for...of`之外，扩展运算符、解构赋值、`Array.from`内部都调用的遍历器接口，都可以将Generator函数返回的Iterator对象作为参数。

```JS
function* numbers () {
  yield 1
  yield 2
  return 3
  yield 4
}

// 扩展运算符
[...numbers()] // [1, 2]

// Array.from 方法
Array.from(numbers()) // [1, 2]

// 解构赋值
let [x, y] = numbers();
x // 1
y // 2
```
## `Generator.prototype.throw()`
 
Generator函数返回的遍历器对象，都有一个`throw`方法，可以在函数体外抛出错误，然后在Generator函数体内捕获
 
 
```JS
function* f() {
  try {
    yield 100;
    yield 200;
  } catch(e) {
    console.log('内部捕获', e)
  }
}
const gen = f();

try {
  console.log(gen.next()); // { value: 100, done: false }
  gen.throw('a'); // '内部捕获, a'
  console.log(gen.next()); // { value: undefined, done: false }
  gen.throw('a'); // '外部捕获, b'
} catch (e) {
  console.log('外部捕获', e)
}
```
遍历器对象抛出的错误被Generator函数体内捕获后，Generator函数`try`语句内的其他语句就==不再继续运行==，并且遍历器对象抛出的其他错误也==不会被Generator函数内捕获==，而是被全局的`catch`捕获
 
> `try`语句中抛出错误，`try`中的其他语句不会在继续执行
 
==不要混淆遍历器对象的`throw`方法和全局的`throw`命令==
 
如果函数内部没有部署`try...catch`代码，遍历器对象抛出的错误会被外部的额`try...catch`代码捕获

要注意的是，`throw`抛出的错误要被内部捕获，前提是==必须执行过一次`next`方法==，因为不执行一次`next`代码，意味着Generator函数没有启动执行，所以错误会被抛出在函数外部。

遍历器对象的`throw`方法被捕获后，自动执行了一次`next`方法，并且只要Generator函数内部部署了`try...catch`代码，`throw`方法也不会影响下一次遍历


```JS
function* f() {
  try {
    yield 100;
    yield 200;
  } catch(e) {
    console.log('内部捕获', e)
  }
  yield 300;
}

const gen = f();

console.log(gen.next()); // { value: 100, done: false }

console.log(gen.throw('a')); // '内部捕获, a', {value: 300, done: false}
```
这种在==函数体内==捕获错误的机制，大大方便了错误的处理，多个`yield`表达式，可以在函数内部==使用一个`try...catch`代码块==来捕获错误就可以了。

Generator函数内部的错误，也可以被函数体外的`catch`捕获，但是==函数内部的代码就不会再继续执行了==，JavaScript认为这个Generator已经==结束运行==了，再调用`next`方法会返回一个`value`属性为`undefined`，`done`属性为`true`的对象
 
## `Generator.prototype.return()`

Generator函数返回的遍历器对象有`return`方法，可以返回给定的值，提前==结束==Generator函数。

```JS
function* f() {
  yield 100;
  yield 200;
  return 300;
}

const g = f();

g.next()
// { value: 100, done: false }

g.return(800)
// { value: 800, done: true }

g.next()
// { value: undefined, done: true }
```
`return`不提供参数，返回值的`value`属性是`undefined`。

如果Generator内部有`try...finally`代码块，且正在执行`try`代码，`return`方法会推迟到`finally`代码块执行完再执行
 
## `next`、`throw`、`return`的共同点

三者都是让Generator函数恢复执行，并且使用不同的语句替换`yield`表达式

`next`是将`yield`表达式替换为一个值，`throw`是将`yield`表达式替换成`throw`语句，`return`是将`yield`表达式替换为`return`语句

```JS
const g = function* (x, y) {
  let result = yield x + y;
  return result;
};

const gen = g(1, 2);
gen.next(); // Object { value: 3, done: false }
```
`next`：

```JS
gen.next(1); // Object { value: 1, done: true }
// 相当于将 let result = yield x + y
// 替换成 let result = 1;
```
`throw`：

```JS
gen.throw(new Error('出错了')); // Uncaught Error: 出错了
// 相当于将 let result = yield x + y
// 替换成 let result = throw(new Error('出错了'));
```

`return`：

```JS
gen.return(2); // Object { value: 2, done: true }
// 相当于将 let result = yield x + y
// 替换成 let result = return 2;
```
 
## `yield*`表达式

如果在Generator函数内部调用另外一个Generator函数，默认情况下是无效的

```JS
function* foo() {
  yield 100;
  yield 200;
}

function* bar() {
  yield 300;
  foo();
  yield 400;
}

for(let i of bar()) {
  console.log(i)
}
// 300 400
```
==在Generator函数内部调用另外一个Generator函数需要用到`yield*`表达式==：

```JS
function* foo() {
  yield 100;
  yield 200;
}

function* bar() {
  yield 300;
  yield* foo();
  
  // 相当于
  // yield 100;
  // yield 200;
  
  // 等同于
  // for(let v of foo()) {
  //   yield v
  // }
  
  yield 400;
}

for(let i of bar()) {
  console.log(i)
}
// 300 100 200 400
```

`yield*`后面的Generator函数（没有`return`语句时）等同于在Generator函数内部部署一个`for...of`循环
 
```JS
function* concat(iter1, iter2) {
  yield* iter1;
  yield* iter2;
}

// 等同于

function* concat(iter1, iter2) {
  for (var value of iter1) {
    yield value;
  }
  for (var value of iter2) {
    yield value;
  }
}
```
有`return`语句时，可以通过赋值`var value = yield* iterator`获取`return`语句的值，==`yield*`后面表达式中的`return`语句作为一个遍历的结果，而不是作为`yield*`的的返回值==

```JS
function* foo() {
  return 1;
}

function* bar() {
  const x = yield* foo();
  return x;
}

const gen = bar();
gen.next();
// { value: 1, done: true }
```


如果`yiled*`后面跟着一个数组，会直接遍历这个数组：

```JS
function* foo() {
  yield 300;
  yield [1, 2, 3];
  yield 400;
}

for(let i of bar()) {
  console.log(i)
}
// 300 [1, 2, 3], 400

function* bar() {
  yield 300;
  yield* [1, 2, 3];
  yield 400;
}

for(let i of bar()) {
  console.log(i)
}
// 300 1, 2, 3, 400
```
实际上，==任何数据结构只要有Iterator接口，就可以被`yield*`遍历==。

```JS
const read = (function* () {
  yield 'hello';
  yield* 'hello'
})();
read.next().value; // 'hello'
read.next().value; // 'h'
```

## 作为对象属性的Generator函数

可以简写为下面的形式：

```JS
let obj = {
  * myGeneratorMethod() {
    ···
  }
};
```
## Generator函数的`this`

Generator函数总是返回一个遍历器，ES6规定这个遍历器是Generator函数的实例：

```JS
function* gen(){}
gen.prototype.say = function() {
  console.log('hello')
}

let obj = gen();

obj instanceof gen; // true
obj.say(); // 'hello'
```
但是==Generator不能作为构造函数使用==，因为它的返回值总是一个遍历器对象，而非`this`对象（即使显示声明`return this`也不可以）

```JS
function* gen(){
  this.test = 123;
}

let obj = gen();

gen.test; // undefined
```
Generator函数也不能和`new`一起使用，会报错。

## Generator与协程

协程有多个线程（函数），可以并行执行，但是只有一个线程（函数）处在正在运行的状态，其他线程（函数）都处于暂停状态（suspended），线程（函数）之间可以交换控制权。

协程以多占用内存为代价，实现多任务的并行。

Generator函数是ES6对协程的==不完全==实现，成为“半协程”，只有Generator函数的调用者才有权将程序的执行权还给Generator函数（完全协程，任何函数都可以将暂停的协程继续执行）

如果将Generator函数当作协程，可以将多个需要相互协作的任务写作Generator函数，之间==使用`yield`表达式交换控制权==。


## 练习 & 应用

### 判断Generator函数输出结果1

```JS
function* dataConsumer() {
  console.log('Started');
  console.log(`1. ${yield}`);
  console.log(`2. ${yield}`);
  return 'result';
}

let genObj = dataConsumer();

console.log(genObj.next());

console.log(genObj.next('a'))

console.log(genObj.next('b'))
```
思路：`next`的参数是对上一次的`yield`表达式返回值赋值，所以拆开来看：

```JS
function* dataConsumer() {
  console.log('Started');
  console.log(`1. ${yield}`); // -- genObj.next() {value: undefined, done: false }，此时第二个console.log被暂停了
  console.log(`2. ${yield}`); // -- genObj.next('a') {value: undefined, done: false } 第二个console.log执行，传入了a
  return 'result'; // -- genObj.next('b') {value: 'result', done: true }，最后一个console.log执行，传入了b
}
```

### 判断Generator函数输出结果2

```JS
function* g() {
  yield 1;
  console.log('throwing an exception');
  throw new Error('generator broke!');
  yield 2;
  yield 3;
}

function log(generator) {
  var v;
  console.log('starting generator');
  try {
    v = generator.next();
    console.log('第一次运行next方法', v);
  } catch (err) {
    console.log('捕捉错误', v);
  }
  try {
    v = generator.next();
    console.log('第二次运行next方法', v);
  } catch (err) {
    console.log('捕捉错误', v);
  }
  try {
    v = generator.next();
    console.log('第三次运行next方法', v);
  } catch (err) {
    console.log('捕捉错误', v);
  }
  console.log('caller done');
}

log(g());
```
Generator函数内部的错误，如果没有被内部捕获，则会被外部捕获，这时候Generator函数执行完毕，不再继续执行

注意，在内部抛出错误后，`next`返回值仍为上一次的返回值，并非抛出的结果。

结果：

```JS
// starting generator
// 第一次运行next方法 { value: 1, done: false }
// throwing an exception
// 捕捉错误 { value: 1, done: false }
// 第三次运行next方法 { value: undefined, done: true }
// caller done
```

### 判断Generator函数输出结果3

```JS
function* foo() {
  yield 2;
  yield 3;
  return "foo";
}

function* bar() {
  yield 1;
  var v = yield* foo();
  console.log("v: " + v);
  yield 4;
}

var it = bar();

console.log(it.next());

console.log(it.next());

console.log(it.next());

console.log(it.next());

console.log(it.next());
```
要注意的是，被代理的Generator函数的`return`语句，不再作为`next`方法的输出结果，而是用来向代理它的`Generator`函数返回数据

```JS
// { value: 1, done: false }

// { value: 2, done: false }

// { value: 3, done: false }

// v: foo
// { value: 4, done: false }

// { value: undefined, done: true }
```

### 判断Generator函数输出结果4

```JS
function* genFuncWithReturn() {
  yield 'a';
  yield 'b';
  return 'The result';
}
function* logReturned(genObj) {
  let result = yield* genObj;
  console.log(result);
}

console.log([...logReturned(genFuncWithReturn())])
```
需要好好判断顺序，首先`genFuncWithReturn()`返回了一个迭代器对象，然后传入了`logReturned`中，按顺序执行，执行了`console.log(result)`之后，才会执行解构操作，所以顺序是：

```JS
// The result
// ["a", "b"]
```
如果换一种形式输出结果就不同了：
```JS
for (let i of logReturned(genFuncWithReturn())) {
  console.log(i)
}
// a
// b
// The result
```
关键点就是解构操作符是等到函数执行后再执行的。


### 让`for...of`可以遍历原生对象

原生的JavaScript对象时候不能使用`for...of`进行遍历的，因为并没有部署遍历接口。

```JS
let obj = {
  name: 'jay',
  age: 31
}

for(let i of obj) {
  console.log(i)
}
// Uncaught TypeError: obj is not iterable
```
使用Generator函数，`for...of`可以遍历原生对象。

思路：由于原生对象没有部署遍历器接口，所以需要为对象的遍历器接口部署一个Generator函数，返回一个遍历器对象

```JS
obj[Symbol.iterator] = function* () {
  let keys = Object.getOwnPropertyNames(this);
  for(let key of keys) {
    yield [key, this[key]]
  }
};
```
可以编写一个更通用的方法：

```JS
function makeIterator (obj) {
  let keys = Object.getOwnPropertyNames(obj);
  obj[Symbol.iterator] = function* () {
    for(let key of keys) {
      yield [key, obj[key]]
    }
  }
}
```


我们之所以能够使用上面的Generator函数，就是因为它的返回结果是一个Iterator对象，这个Iterator对象有`next`方法，每次遍历时都要调用这个方法，返回的记结果就是包含了`value`和`done`两个属性的值

所以，我们不使用Generator函数，自己都构造返回一个具有`next`方法的对象也是可以的，`next`方法返回对象也需要包括了`value`和`done`连个属性，`value`属性是`for...of`的返回值，`done`用来标识遍历何时结束。


```JS
function makeIterator (obj) {
  let keys = Object.getOwnPropertyNames(obj);
  obj[Symbol.iterator] = function () {
    let index = 0;
    return {
      next() {
        const key = keys[index]
        return { value: [key, obj[key]], done: index++ === keys.length }
      }
    }
  }
}
```


### 第一次调用`next`方法就能够传值

`next`的参数是赋值给==上一个==`yield`表达式返回值，所以在首次调用`next`传参是==无效==的。

构造一个`wrapper`函数，返回一个Generator函数，实现在第一次调用`next`方法时就能够输入值。

```JS
const wrapped = wrapper(function* () {
  console.log(`First input: ${yield}`);
  return 'DONE';
});

wrapped().next('hello!')
// First input: hello!
```
思路：既然Generator首个`next`不能传参，那么就在我们的包裹函数中，将首次`next`调用在包裹函数内执行

```JS
const wrapper = function(fn) {
  return function(...args) {
    const gnObject = fn(...args);
    gnObject.next();
    return gnObject
  }
}
```

### 利用Generator函数和`for...of`循环，实现斐波那契数列

```JS
function* fibonacci() {
  let [prev, curr] = [0, 1];
  for(;;) {
    yield curr;
    [prev, curr] = [curr, curr + prev]
  }
}

for(let v of fibonacci()) {
  if(v > 100) {
    break
  }
  console.log(v)
}
```

### 实现一个clock状态机

如果不使用Generator函数：

```JS
const clock = (function () {
  let ticking = true;
  return function () {
    console.log(ticking ? 'tick' : 'tock');
    ticking = !ticking
  }
})()
```
使用Generator函数：

```JS
const clock = (function* () {
  for(;;) {
    console.log('tick');
    yield;
    console.log('tock');
    yield
  }
})()
```

### 输出多维数组中的值

```JS
const numbers = flatten2([1, [[2], 3, 4], 5])
numbers.next().value // => 1
numbers.next().value // => 2
numbers.next().value // => 3
numbers.next().value // => 4
numbers.next().value // => 5
```
思路就是递归调用Generator函数：

```JS
function* flatten(arr) {
  for(let i of arr) {
    if (Array.isArray(i) {
      yield* flatten(i)
    } else {
      yield i
    }
  }
}
```
要注意的就是，在一个Generator函数里面调用另外一个Generator函数，默认是无效的，所以必须使用`yield*`表达式来调用

### 遍历二叉树

> 对二叉树这里有些迷糊，因为基础不牢固，回头好好不玩了数据结构和算法，再来重新看一下这里（2019.01.17）

```JS
// 下面是二叉树的构造函数，
// 三个参数分别是左树、当前节点和右树
function Tree(left, label, right) {
  this.left = left;
  this.label = label;
  this.right = right;
}

// 下面是中序（inorder）遍历函数。
// 由于返回的是一个遍历器，所以要用generator函数。
// 函数体内采用递归算法，所以左树和右树要用yield*遍历
function* inorder(t) {
  if (t) {
    yield* inorder(t.left);
    yield t.label;
    yield* inorder(t.right);
  }
}

// 下面生成二叉树
function make(array) {
  // 判断是否为叶节点
  if (array.length == 1) return new Tree(null, array[0], null);
  return new Tree(make(array[0]), array[1], make(array[2]));
}
let tree = make([[['a'], 'b', ['c']], 'd', [['e'], 'f', ['g']]]);

// 遍历二叉树
var result = [];
for (let node of inorder(tree)) {
  result.push(node);
}

result
// ['a', 'b', 'c', 'd', 'e', 'f', 'g']
```
### 控制流管理

如果一个多步操作非常耗时，采用回调函数，可能会写成下面这样：

```JS
step1(function (value1) {
  step2(value1, function(value2) {
    step3(value2, function(value3) {
      step4(value3, function(value4) {
        // Do something with value4
      });
    });
  });
});
```
改写成Promise格式：

```JS
Promise.resolve(step1)
  .then(step2)
  .then(step3)
  .then(step4)
  .then(value4 => {
    // Do something with value4
  }, error => {
    // catch the error from step1 through step4
  })
```
改写成Generator格式：

```JS
function* gen(value1) {
  try {
    const value2 = yield step1(value1);
    const value3 = yield step2(value2);
    const value4 = yield step3(value3);
    const value5 = yield step4(value4);
    // Do something with value4
  } catch(e) {
    // catch the error from step1 through step4
  }
}
```
需要一个函数按次序调用：

```JS
scheduler(longRunningTask(initialValue));

function scheduler (task) {
  const taskObj = task.next(task.value);
  // 如果Generator函数未结束，就继续调用
  if(!taskObj.done) {
    task.value = taskObj.value;
    scheduler(task)
  }
}
```
上面这种做法，只适合同步操作，即所有的`task`都必须是同步的，不能有异步操作。



### 让Generator函数能够使用`new`

Generator函数不能和`new`命令一起用，会报错：

```JS
function* F() {
  yield this.x = 2;
  yield this.y = 3;
}

new F()
// TypeError: F is not a constructor
```
如何让Generator函数返回一个正常的对象实例，既可以用`next`方法，又可以获得正常的`this`？

```JS
function* gen () {
  this.a = 1;
  yield this.b = 2;
  yield this.c = 3;
}

function F (){
  // do something here
}

let f = new F();

f.next();  // Object {value: 2, done: false}
f.next();  // Object {value: 3, done: false}
f.next();  // Object {value: undefined, done: true}

f.a // 1
f.b // 2
f.c // 3
```
实现：

```JS
function F (){
  return gen.call(gen.prototype)
}
```
实际上这是一个有欺骗性的做法，实际上`new`关键字无效的，我们要的只是执行`F`即可。

```JS
let f = new F();
```
而`gen.call(gen.prototype)`相当于在`gen`原型上添加了属性，当访问`f.a`时实际上访问的就是原型链上的属性。

## 参考

- [Generator 函数的语法@ECMAScript6入门](http://es6.ruanyifeng.com/#docs/generator)
