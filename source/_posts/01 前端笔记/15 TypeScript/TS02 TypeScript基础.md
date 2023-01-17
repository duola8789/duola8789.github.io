---
title: TS02 TypeScript基础
top: false
date: 2019-12-12 14:37:00
updated: 2020-03-25 14:12:33
tags:
- TypeScript
categories: TypeScript
---

TypeScript基础学习笔记，学习完了可以凑合着用了。

<!-- more -->

# 安装

全局安装TypeScript命令行工具

```BASH
npm install -g typescript
```

安装后就可以在全局使用`tsc`命令，来编译TypeScript文件：

```JS
tsc hello.ts
```

TypeScript编写的文件后缀名是`.ts`，用TypeScript编写React应用时文件后缀名是`.tsx`

# Hello TypeScript

```JS
function sayHello(person: string) {
  return `hello, ${person}`
}
const user = 'Tom';
console.log(sayHello(user));
```

在TS中，使用`:`指定变量的类型，编译后的代码：

```JS
function sayHello(person) {
    return "hello, " + person;
}
var user = 'Tom';
console.log(sayHello(user));
```

TypeScript会对代码进行静态检查，如果传入的参数和我们指定的类型不匹配，IDE就可以给出即时的提示，并且在编译阶段会报错（但是并不会阻止编译的过程）

例如我们将上面的`user`的值改为数值`123`，那么在IDE中会提示：

![](http://image.oldzhou.cn/FgnqsqK-8WMDWOVGBELuReCKVXGr)

编译时也报错：

```BASH
src/hello.ts:5:22 - error TS2345: Argument of type '123' is not assignable to parameter of type 'string'.

5 console.log(sayHello(user));
```

但是仍然会生成编译结果。如果需要在报错时终止JS文件的生成，可以在`tsconfig.json`中配置`noEmitOnError`，关于`tsconfig.json`后面单独学习。

# 原始数据类型

## 字符串 + 布尔值 + 数值

JavaScript中数据分为原始数据类型和对象类型，原始数据类型有六种（布尔值、数值、字符串、`null`、`undfined`和`Symbol`），对于前三者类型的定义如下：

```JS
let isDone: boolean = false;
let count: number = 123;
let msg: string = 'hello'
```

注意，类型的定义都是针对字面量的，使用构造函数（例如`new Boolean()`）创建出的变量类型时对象，而非基本类型

## 空值`void`

### JavaScript中的`void`

JavaScript中的`void`是一个运算符，用于计算它右边的表达式，无论表达式是什么、结果返回什么，`void`总是返回`undefined`

它的作用是，由于一个变量被赋值为`undefined`后，它总是可以被覆盖，所以可以使用`void`来确保可以始终返回`undefined`

借用这个特性，我们可以实现下面几种效果：

（1）调用立即执行函数：

```
void function(){
    console.log(123)
}()
```

（2）在函数中调用一个回调函数，但是不返回这个会带哦函数的值：

```JS
// returning something else than undefined would crash the app
function middleware(nextCallback) {
  if(conditionApplies()) {
    return void nextCallback();
  }
}
```

### TypeScript中的`void`

JavaScript中没有空值的概念，而在TypeScript中使用`void`表示没有任何返回值的函数：

```JS
function alertName(): void {
  lert('My name is Tom');
}
```

如果一个变量声明为`void`类型，那么只能将它赋值为`undefined`或`null`

对于`undefined`和`null`，它们是所有类型的子类型，也就是说，`undefined`和`null`类型的变量，可以赋值其他任何类型：


```JS
let msg: string = undefined;
let count: number = null;
```

而`void`类型的变量不是其他类型的子类型，不能赋值给其他类型的变量

```JS
let u: void;
let num: number = u;

// Type 'void' is not assignable to type 'number'.
```

## 任意值

使用`any`来表示允许赋值为任意类型，除此之外的情况，一旦定义了类型，在赋值过程中是不允许改变的：

```JS
let myFavoriteNumber: string = 'seven';
myFavoriteNumber = 7;

// index.ts(2,1): error TS2322: Type 'number' is not assignable to type 'string'.
```

但是如果是`any`类型，则可以被任意改变：

```JS
let myFavoriteNumber: any = 'seven';
myFavoriteNumber = 7;
```

在任意值上访问任何属性都是允许的，也可以调用任何方法：

```JS
let anyThing: any = 'hello';
console.log(anyThing.myName.firstName);
anyThing.myName.setFirstName('Cat');
```

可以认为，声明一个变量为`any`后，对它的任何操作，返回的内容的类型都是任意值（失去了控制）

变量在声明时，如果**没有指定类型并且没有赋值**，那么就会被认为是任意类型

# 类型推论

如果一个变量声明时，没有指定类型吗，但是**进行了赋值**，那么TypeScript会依照类型推论的规则推导出一个类型

```JS
let msg = 'seven';
msg = 7;

// index.ts(2,1): error TS2322: Type 'number' is not assignable to type 'string'.
```

它等价于

```JS
let msg: string = 'seven';
msg = 7;

// index.ts(2,1): error TS2322: Type 'number' is not assignable to type 'string'.
```

# 联合类型（Union Types）

联合类型表示取值可以为多种类型中的一种，使用`|`来分割每个类型

```JS
let myFavoriteNumber: string | number;
myFavoriteNumber = 'seven';
myFavoriteNumber = 7;
```

当TypeScript不确定一个联合类型的变量具体是哪个类型是时，我们只能访问此联合类型的所有类型中都**共有的属性**和方法

联合类型在赋值时也会根据类型推论被确定类型

# 对象的类型：接口

## 接口定义

在TypeScript中，我们使用接口（Interfaces）来定义对象的类型。

在面向对象语言中，接口是一个很重要的概念，它是对行为的抽象，而具体如何行动需要由类（class）去实现（implement）

TypeScript中的接口既可以对类的一部分行为进行抽象，也可以对对象的形状（shape）进行描述，下面是一个简单的例子：

```JS
interface Person {
  name: string,
  age: number,
}

const tom: Person = {
  name: 'tom',
  age: 100,
};
```

上面我们通过定义接口`Person`，并且指定了`tom`的类型为`Person`，这样就约束了`tom`中的形状（也就是各个成员的类型）必须和接口`Person`一致

接口名首字母大写。

## 可选属性

使用了接口的变量属性数目必须和接口完全一致，不能多也不能少，也就是说，赋值的时候，**变量的形状必须和接口的形状完全保持一致**。

但是有些时候，我们希望不要完全匹配一个形状，那么就可以使用可选属性：

```JS
interface Person {
    name: string;
    age?: number;
}

let tom: Person = {
    name: 'Tom'
};
```

这个时候接口中的可选属性是可以在变量中不存在的，但是仍然不允许添加不存在的属性，并且可选属性的类型（如果存在）仍需要和接口中属性的类型一致。

## 任意属性

如果希望一个接口允许有任意的属性，可以使用`[propName: string]`来定义：

```JS
interface Person {
    name: string;
    age?: number;
    [propName: string]: any;
}

let tom: Person = {
    name: 'Tom',
    gender: 'male'
};
```

使用`[propName: string]`，定义了任意属性（属性名类型为`string`）的类型为`any`，要注意的是，**一旦定义了任意属性，那么确定属性和可选属性的类型都必须是它的类型的子集**

## 只读属性

可以在接口的属性前添加`readonly`，定义此属性是只读的，不能为此属性赋值

```JS
interface Person {
  readonly name: string
}

let tom: Person = {
  name: 'tom',
};

tom.name = 'jerry';
// Cannot assign to 'name' because it is a read-only property.
```

# 数组的类型

数组类型有多重定义方法：

## `类型 + 方括号`表示法

这是最简的表达式方法，适用于值都是基本类型的数组：

```JS
let arr: number[] = [1, 2, 3];
arr.push('8');

// Argument of type '"8"' is not assignable to parameter of type 'number'.
```

使用`any`表示数组中可以出现任意类型：

```JS
let list: any[] = ['xcatliu', 25, { website: 'http://xcatliu.com' }];
```

## 数组泛型

也可以使用数组泛型（Array Generic）`Array<elemType>`来表示数组：

```JS
let arr: Array<number> = [1, 2, 3];
```

## 用接口表示数组

因为数组实际上是特殊的对象，所以可以使用接口来描述数组：

```JS
interface NumberArray {
  [index: number]: number;
}
let fibonacci: NumberArray = [1, 1, 2, 3, 5];
```

`[index: number]`限定的是索引的类型是数字的成员（实际上貌似使用`propName`代替`index`也可以成功）

## 复合数组

前面的几种方法都定义的是数组成员是基本类型的数组，如果数组的成员是对象的话，只需要借助接口来实现就好：

```JS
interface Item {
  name: string,
  age: number
}
let arr: Array<Item> = [
  { name: 'Tom', age: 100 },
  { name: 'Jerry', age: 100 },
];
```

## 类数组

类数组并不是数组，不能用普通的数组的方式来描述，而应该使用接口：

```JS
function sum() {
  let args: {
    [index: number]: number;
    length: number;
    callee: Function;
  } = arguments;
}
```

我们通过自定义接口约束了类数组的类型，但是实际上类数组在TypeScript中都有内置的接口定义，比如`arguments`对应的`IArguments`，还有`NodeList`、`HTMLCollection`等

```JS
function sum() {
  let args: IArguments = arguments;
}
```

`IArguments`的内容实际上就是：

```JS
interface IArguments {
  [index: number]: any;
  length: number;
  callee: Function;
}
```

这些内置对象在后面学习。

# 函数的类型

一个函数有输入也有输出，输入和输出的类型都需要进行限制

## 函数声明

函数参数的个数也会被限定

```JS
function sum(x: number, y: number): number {
  return x + y;
}
```

## 函数表达式

如果将上面的函数声明改写为函数表达式，是这样：

```JS
const sum = (x: number, y: number): number => {
  return x + y;
};
```

但是实际上，这样支队等号右侧的匿名函数进行了类型定义，等号左边的`sum`是通过赋值操作进行类型推论而推断出来的，如果需要手动给`sum`添加类型，应该是这样：

```JS
const sum: (x: number, y: number) => number = (x: number, y: number): number => {
  return x + y;
};
```

上面出现了两个箭头`=>`，右侧箭头是ES6中用来定义函数的箭头，而**左侧的箭头是TypeScript中用来表示函数定义的箭头，这个箭头左侧表示输入类型，需要用括号括起来，右侧是输出类型。**

实际上对`sum`的类型的限制及限制了函数的输入，也限制了函数的输出。

## 用接口定义函数类型

也可以用接口来定义函数类型：

```JS
interface sumFn {
  (x: number, y: number): number
}

const sum: sumFn = (x, y) => x + y;
```

接口的属性名为对应的函数输入参数类型，属性值为函数输出的参数类型。

## 可选参数

和接口的可选属性一样，函数的参数后面添加`?`表示这个参数时可选参数，要注意的是，可选参数必须接在必须参数的后面，也就是说，**可选参数后面不能出必须参数了**。

```JS
function buildName(firstName: string, lastName?: string) {
  if (lastName) {
    return firstName + ' ' + lastName;
  } else {
    return firstName;
  }
}

let tomcat = buildName('Tom', 'Cat');
let tom = buildName('Tom');
```

## 默认参数

和ES6中的函数的默认参数一样，TypeScript中也可以给函数的参数添加默认值。

```JS
const sum = (x: number, y: number = 0): number => x + y;

sum(5, 2);
sum(5);
```

函数参数的默认值只能在函数中定义，不能在接口中定义：

```JS
interface sumFn {
  (x: number, y: number = 0): number,
}

const sum: sumFn = (x, y) => x + y;
// Error:(9, 15) TS2371: A parameter initializer is only allowed in a function or constructor implementation.
```

## 剩余参数

ES6中的剩余参数需要使用数组类型来定义（因为它就是一个数组）

```JS
function push(array: any[], ...items: any[]) {
  items.forEach(function(item) {
    array.push(item);
  });
}

let a = [];
push(a, 1, 2, 3);
```

要注意，剩余参数只能在参数的末尾出现。

## 函数重载

一个函数接受不同数量或者类型的参数，做出不同的处理，这种现象就是函数重载。

例如下面的函数，对输入数字和字符串时处理的过程是不同的：

```JS
function reverse(x: number | string): number | string {
  if (typeof x === 'number') {
    return Number(x.toString().split('').reverse().join(''));
  } else if (typeof x === 'string') {
    return x.split('').reverse().join('');
  }
}
```

但是这样并不够精确，因为输入是数字的时候，也应该返回数字，输入是字符串，也应该返回字符串，这时我们可以使用重载定义多个函数类型：

```JS
function reverse(x: number): number;
function reverse(x: string): string;
function reverse(x: number | string): number | string {
  if (typeof x === 'number') {
    return Number(x.toString().split('').reverse().join(''));
  } else if (typeof x === 'string') {
    return x.split('').reverse().join('');
  }
}
```

前两个都是函数定义，最后一个是函数实现，这样做在IDE就可以看到正确的两个提示

![](http://image.oldzhou.cn/FikHzrwsh5llwjxggibwR39Sqt7X)

注意，TypeScript会从最前面的函数开始匹配，所以需要优先把精确定义写在前面

# 类型断言

类型断言（Type Assertion）用来手动指定一个值的类型，一般用在联合类型中，将一个不确定类型的变量指定为联合类型中的一种

比如，下面的函数，如果要访问`length`会报错，因为`number`是没有`length`属性的：

```JS
function getLength(something: string | number): number {
  return something.length;
}

// index.ts(2,22): error TS2339: Property 'length' does not exist on type 'string | number'.
// Property 'length' does not exist on type 'number
```

这个时候我们就可以使用类型断言，将`something`断言为`string`：

```JS
function getLength(something: string | number): number {
  return (<string>something).length;
}
```

或者：

```JS
function getLength(something: string | number): number {
  return (something as string).length;
}
```

上面用了两种语法类实现断言`(<类型>值)`和`(值 as 类型)`，在React应用的`.tsx`文件中，只能使用后一种。


要注意类型断言不是类型转换，不允许将类型断言为一个联合类型中不存在的类型

# 声明文件

## 声明语句

当使用第三方库时，我们需要引用它的声明文件，才能获得对应的代码补全、接口提示等功能

比如使用jQuery时，我们获取一个元素：

```JS
jQuery('#foo');
// ERROR: Cannot find name 'jQuery'.
```

但是这时编译器并不知道`$`或者`jQuery`是什么，所以我们需要使用`declare`定义它的类型

```JS
declare var jQuery: (selector: string) => any;

jQuery('#foo');
```

上面的`declare var`就是声明语句，它并没有定义一个变量，只是定义了`jQuery`的类型，仅仅用于编译时的检查，编译结果中会删除。

## 声明文件

通常情况，我们会将声明语句放到单独的文件中（`jQuery.d.ts`），这个文件就是声明文件：

```JS
// src/jQuery.d.ts

declare var jQuery: (selector: string) => any;
```

声明文件必须以`.d.ts`为后缀，当我们将`jQuery.d.ts`放入项目中，其他所有的`*.ts`文件就都可以获得`jQuery`的类型定义了

如果无法解析，可以检查一下`tsconfig.json`中的`files`、`include`、`exclude`配置，确保包含了声明文件

## 第三方声明文件

大部分热门的第三方库的声明文件都不需要我们自己定义了，我们可以直接使用`@types`统一管理第三方库的声明文件。

可以在[这个页面](https://microsoft.github.io/TypeSearch/)搜索需要的声明文件，然后使用`npm`安装对应的声明模块即可：

```BASH
npm install @types/jquery --save-dev
```

## 书写声明文件

如果第三方库没有提供声明文件，我们需要自己书写声明了，在不同的场景下，声明文件的内容和使用方式有所区别

## 全部变量

当通过`<script>`标签引入第三方库的时候，会注入全局变量，就像上面的例子一样。建议将声明文件和源码一起放到`scr`目录下

（1）声明变量

可以使用`declare var`和`declare let`和`declare const`来声明变量，一般来说全局变量都是禁止修改的常量，所以大部分情况都应该使用`const`

```JS
declare const jQuery: (selector: string) => any;
```

要注意的是，声明语句中只能定义类型，**不要在声明语句中定义具体的实现**。

（2）声明函数

使用`declare function`来声明全局函数的类型，`jQuery`其实就是一个函数，所以也可以使用`function`来定义。在函数类型的声明语句中，也支持函数重载

```JS
declare function jQuery(cb: () => any)
declare function jQuery(selector: string): any;
```

（3）声明类

使用`declare class`定义一个类：

```JS
declare class Person {
  name: string;
  constructor(name: string) ;
  sayHi(): string;
  sayBye: (msg: string) => string
}
```

同样的，`declare class`也只能定义类型，不能定义具体实现

（4）声明枚举类型

JS中是没有枚举类型的，TypeScript中使用`declare enum`声明的枚举类型也成为外部枚举，定义后的变量不能包含枚举值之外的属性

```JS
declare enum Direction {
  up,
  down,
  left,
  right,
}

const d1 = Direction.down;

const d2 = Direction.south;
// Error:(17, 22) TS2339: Property 'south' does not exist on type 'typeof Direction'.
```

（5）声明命名空间

使用`declare namespace`声明命名空间，用来表示全局变量是一个对象，包含很多子属性。

比如`jQuery`是一个全局变量，它是一个对象，提供了一个`jQuery.ajax`的方法可以调用，那么我们就可以通过`declare namespace`来声明这个拥有很多个子属性的全局变量：

```JS
declare namespace jQuery {
  function ajax(url: string, settings?: any): void;
}
```

在`declare namespace`内部，直接使用`function`来声明函数，而不是使用`declare function`，类似的也可以使用`class`、`enum`、`const`等语句

```JS
declare namespace jQuery {
  function ajax(url: string, settings?: any): void;

  const version: number;

  class Event {
    blur(eventType: EventType): void
  }

  enum EventType {
    CustomClick
  }
}
```

如果对象有深层的层级，则需要使用嵌套的`namespace`来声明深层的属性的类型：

```JS
declare namespace jQuery {
  function ajax(url: string, settings?: any): void;

  namespace fn {
    function extend(object: any): void;
  }
}
```

假如`jQuery`下仅有`fn`这一个属性（没有`ajax`等其他属性或方法），则可以不需要嵌套`namespace`

```JS
declare namespace jQuery.fn {
  function extend(object: any): void;
}
```

（6）声明全局的接口或类型

我们可以将接口和其他的类型放到类型声明文件中，这样声明的接口或者类型就暴露成为全局的接口或类型，可以被其他文件使用

```JS
interface AjaxSettings {
  method?: 'GET' | 'POST'
  data?: any;
}
declare namespace jQuery {
  function ajax(url: string, settings?: AjaxSettings): void;
}
```

要注意，暴露在全局的`interface`或者`type`会作为全局类型作用域整个项目中，存在命名冲突的可能性，所以应该尽量减少全局变量或全局类型的数量，所以应该将他们放到`namespace`下：

```JS
declare namespace jQuery {
  interface AjaxSettings {
    method?: 'GET' | 'POST'
    data?: any;
  }
  function ajax(url: string, settings?: AjaxSettings): void;
}
```

使用这个`interface`的受，应该加上命名空间的前缀：

```JS
let settings: jQuery.AjaxSettings = {
  method: 'POST',
  data: {
    name: 'foo'
  }
};
```

（7）声明合并

如果一个对象即是一个函数，可以直接调用`jquery('#foo')`，又是一个对象，有子属性`jQuery.ajax()`，那么可以组合多了个声明语句，他们会不冲突的合并：

```JS
declare function jQuery(selector: string): any;
declare namespace jQuery {
  function ajax(url: string, settings?: any): void;
}
```

合并规则后面单独学习。

## NPM包

### 找到已存在的声明文件

在给引入的NPM包创建声明文件之前，先看它的声明文件是否存在，一般来说可能存在于：

（1）与包绑定在一起，看`package.json`的`types`字段，或者看包中是否有`index.d.ts`声明文件。

推荐这种模式，因为不需要安装额外的其他包。我们自己创建NPM包的时候，最好也将声明与包绑定在一起

（2）发布到`@types`里，可以去上面提到的[页面](https://microsoft.github.io/TypeSearch/)搜索对应的声明文件，然后安装即可。

这种模式一般是由第三方提供的声明文件，发布到`@types`中。

### 编写声明文件

如果没有找到声明文件，我们可以自己编写。由于一般是通过`import`来引入一个NPM包（假设为`foo`），所以声明文件的存放位置有要求。

最常用的方案是，创建一个`types`目录，专门用来管理自己写的声明文件，将自己编写的声明文件放到`types/foo/index.d.ts`中。

![](http://image.oldzhou.cn/Fn-Dbxv-3DSJCSm6Zd3DVpUeO6oP)

这种方式还需要配置`tsconfig.json`中的`paths`和`baseUrl`字段。

```JS
{
  "compilerOptions": {
    "module": "commonjs",
    "baseUrl": "./",
    "paths": {
      "*": ["types/*"]
    }
  }
}
```

NPM声明文件包含下面几种语法：

（1）`export`导出变量

在NPM包的声明文件中，如果使用`declare`不会再声明全局变量，只会在当前文件中声明局部变量。局部变量需要使用`export`导出，然后由使用方`import`导入后，才会被应用。

同样，声明文件中不能定义具体的实现：

```JS
// types/foo/index.d.ts

export const name: string;

export function getName(): string;

export class Animal {
  constructor(name: string);
  sayHi(): string;
}

export enum Directions {
  Up,
  Down,
  Left,
  Right
}

export interface Options {
  data: any;
}
```

也可以使用`declare`先声明多个变量，然后再用`export`一次性导出：

```JS
// types/foo/index.d.ts

declare const name: string;

declare function getName(): string;

declare class Animal {
  constructor(name: string);

  sayHi(): string;
}

declare enum Directions {
  Up,
  Down,
  Left,
  Right
}

interface Options {
  data: any;
}

export {name, getName, Animal, Directions, Options};
```

（2）`export namesapce`

与`declare namspace`一样，`export namespace`导出一个拥有子属性的对象

```JS
// types/foo/index.d.ts

export namespace foo {
  const name: string;
  namespace bar {
    function baz(): string;
  }
}
```

可以使用`export default`导出默认的`function`、`class`和`interface`，这三者可以直接导出，其他的类型需要先定义，然后在使用`export default`导出，一般会将这种导出放在整个声明文件的最前面：

```JS
// types/foo/index.d.ts

export default Directions;

declare enum Directions {
  Up,
  Down,
  Left,
  Right
}
```

## UMD

针对UMD格式的模块，使用`export as namespace`进行导出，一般使用时，都是先有了NPM包的声明文件，再基于它添加`export as namespace`语句，就可以将声明好的一个变量声明为全局变量：

```JS
// types/foo/index.d.ts

export as namespace foo;
export = foo;

declare function foo(): string;
declare namespace foo {
  const bar: number;
}
```

## 其他

可以使用`declare global`来在已有的声明文件中扩展全局变量的类型：

```JS
// types/foo/index.d.ts

declare global {
  interface String {
    prependHello(): string;
  }
}

export {};
```

要注意，此声明文件不需要导出任何东西，但是仍然导出了一个空对象，**用来告诉编译器这是一个模块的声明文件，而不是全局变量的声明文件**

可以使用`declare module`来扩展模块插件

## 自动生成声明文件

如果库的源码本身就是TypeScript编写的，那么使用`tsc`来将`.ts`编译为JS的过程中，可以添加`declaration`选项（简写`-d`，同时生成`.d.ts`声明文件

也可以在`ts.config`中添加`declaration`选项来实现：

```JS
{
  "compilerOptions": {
    "module": "commonjs",
    "outDir": "lib",
    "declaration": true,
  }
}
```

这样就会由`.ts`文件生成`.d.ts`声明文件，并且输出到`lib`目录下：

![](http://image.oldzhou.cn/Fuu_1rseZAwKLPRMDo1GQn8aFXgV)

这样做的时候，每个`.ts`文件都会对应一个`.d.ts`声明文件，这样使用方就可以再使用`import`导入时获得类型提示

此外，`tsconfig.json`中还有其他选项与自动生成声明文件相关：

- `declarationDir`，设置生成`.d.ts`文件的目录
- `declarationsMap`，对每个`.d.ts`文件都生成对应的`.d.ts.map`（sourcemap）文件
- `emitDeclarationOnly`，仅仅生成`.d.ts`文件，不生成`.js`文件

## 发布声明文件

如果是`tsc`命令自动生成的声明文件，不需要做任何其他配置，直接发布到NPM即可

如果是手动编写的，需要满足下面条件之一，才能被正确识别：

- 给`package.json`的`types`或者`typings`字段指定一个类型声明文件地址
- 在项目根目录下，编写`index.d.ts`文件
- 针对入口文件（`package.json`中的`main`字段指定的入口文件）编写一个同名不同后缀的`.d.ts`文件

#  内置对象

内置对象是指根据标准在全局作用域上存在的对象，主要分为以下几种：

（1）ECMAScript的内置对象

例如`Error`、`Date`、`RegExp`等，可以在TypeScript中直接将变量定义为这些类型：

```JS
let e: Error = new Error('Error occurred');
let d: Date = new Date();
let r: RegExp = /[a-z]/;
```

（2）DOM和Bom的内置对象

例如`Document`、`HTMLElement`、`Event`、`NodeList`等，在TypeScript中也可以直接使用它们来定义变量类型：

```JS
let body: HTMLElement = document.body;
let allDiv: NodeList = document.querySelectorAll('div');
document.addEventListener('click', function(e: MouseEvent) {
  // Do something
});
```

（3）TypeScript核心库的定义文件

TypeScript核心库定义了所有浏览器环境需要用到的类型，预置在TypeScript中。当我们在使用一些常用的方法中，实际上TypeScript已经帮我们进行了类型判断：

```JS
Math.pow(10, '2');

// index.ts(1,14): error TS2345: Argument of type 'string' is not assignable to parameter of type 'number'.
```

在TypeScript的核心库`lib.es5.d.ts`中定义了`Math`的接口类型：

```JS
interface Math {
  /**
   * Returns the value of a base expression taken to a specified power.
   * @param x The base value of the expression.
   * @param y The exponent value of the expression.
   */
  pow(x: number, y: number): number;
}
```

要注意，**TypeScript核心库中不包含Node.js部分内容**。

（4）Node.js

Node.js不是内置对象，如果要使用TypeScript写Node.js，需要引入第三方声明文件：

```JS
npm install @types/node --save-dev
```

# 参考

- [TypeScript入门教程](https://ts.xcatliu.com/)
- [JavaScript和TypeScript中的void@segmentfault](https://segmentfault.com/a/1190000020368426)
