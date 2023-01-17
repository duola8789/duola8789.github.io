---
title: TS03 TypeScript进阶
top: false
date: 2019-12-13 11:20:00
updated: 2020-06-22 18:22:16
tags:
- TypeScript
categories: TypeScript
---

TypeScript进阶知识学习笔记。

<!-- more -->

# 类型别名

## 概念

可以使用`type`关键字来创建一个类型的别名：

```JS
type num = number;
const a:num = 123;
```

类型别名常用于联合类型，例如字符串字面量类型，用来约束取值只能是几个字符串的一个

```JS
type EventNames = 'click' | 'scroll' | 'mousemove';
function handleEvent(ele: Element, event: EventNames) {
  // do something
}

handleEvent(document.getElementById('hello'), 'scroll');  // 没问题
handleEvent(document.getElementById('world'), 'dbclick'); // 报错，event 不能为 'dbclick'

// index.ts(7,47): error TS2345: Argument of type '"dbclick"' is not assignable to parameter of type 'EventNames'.
```

## 与`interface`的异同

二者的共同点是：

（1）都可以描述对象或者函数

（2）都允许扩展（`extends`）

```TS
type Name = {
  name: string;
}
type User = Name & { age: number  };
```

`type`的扩展使用了`&`

二者的不同点是：

（1）`type`可以声明基本类型的别名，`interface`不可以

（2）`type`语句可以使用`typeof`获取实例的类型赋值

```TS
// 当你想获取一个变量的类型时，使用 typeof
let div = document.createElement('div');
type B = typeof div
```

（3）`interface`能够声明合并，而`type`不同

```TS
interface User {
  name: string
  age: number
}

interface User {
  sex: string
}

/*
User 接口为 {
  name: string
  age: number
  sex: string
}
*/
```


# 元组

元组（Tuple）就是包含不同类型元素的数组：

```JS
const person: [string, number] = ['Tom', 24];
person[0] = 'Jerry';

person[0] = 1;
// Error:(12, 1) TS2322: Type '1' is not assignable to type 'string'.
```

可以赋值其中一项，但是如果对元组整体进行定义或者赋值的时候，必须提供所有元组类型中指定的项目：

```JS
let person1: [string, number];
tom[0] = 'Tom';

let person2: [string, number];
person2 = ['Tom'];
// Property '1' is missing in type '[string]' but required in type '[string, number]'.
```

当添加元组规定的类型数量之外的元素时，元素的类型会被限定为元组包含类型的联合类型：

```JS
const person: [string, number] = ['tom', 24];

person.push('ok');
person.push(true);
// Error:(11, 13) TS2345: Argument of type 'true' is not assignable to parameter of type 'string | number'.
```

# 枚举

## 定义枚举值

枚举类型用于取值被限定在一定范围的场景

```JS
enum Days {Sun, Mon, Tue, Wed, Thu, Fri, Sat}

const sun = Days.Sun;
const sat = Days.Sat;
```

枚举成员会被赋值为从`0`开始递增的数字，同时也会对枚举值到枚举名进行反向映射，上面的编译结果是：

```JS
var Days;
(function (Days) {
    Days[Days["Sun"] = 0] = "Sun";
    Days[Days["Mon"] = 1] = "Mon";
    Days[Days["Tue"] = 2] = "Tue";
    Days[Days["Wed"] = 3] = "Wed";
    Days[Days["Thu"] = 4] = "Thu";
    Days[Days["Fri"] = 5] = "Fri";
    Days[Days["Sat"] = 6] = "Sat";
})(Days || (Days = {}));

var sun = Days.Sun;
var sat = Days.Sat;
```

## 手动赋值

可以为枚举项手动赋值：

```JS
enum Days {Sun = 'test', Mon = 1, Tue, Wed, Thu, Fri, Sat};

console.log(Days["Sun"] === 'test'); // true
console.log(Days["Mon"] === 1); // true
console.log(Days["Tue"] === 2); // true
console.log(Days["Sat"] === 6); // true
```

未手动赋值的枚举项会接着上一个枚举项递增，要注意的是，如果未手动赋值的枚举项和手动赋值的枚举项重复了，TypeScript是不会报错的。

## 计算所得项

枚举项有两种类型，常数项和计算所得项，上面的列子都是常数项，计算所得项如下：

```JS
enum Color {Red, Green, Blue = "blue".length};
```

要注意，**计算所得项后面不等接未手动赋值的项**。

## 常数枚举项

常数枚举是使用`const enum`定义的枚举类型：

```JS
const enum Directions {
  Up,
  Down,
  Left,
  Right
}

let directions = [Directions.Up, Directions.Down, Directions.Left, Directions.Right];
```

常数枚举项会在编译阶段被删除，并且不能包含计算成员，上面的编译结果是：

```JS
var directions = [0 /* Up */, 1 /* Down */, 2 /* Left */, 3 /* Right */];
```

## 外部枚举

使用`declare enum`定义的枚举类象，之前提到的声明文件中的枚举类型就是这种：

```JS
declare enum Directions {
  Up,
  Down,
  Left,
  Right
}

let directions = [Directions.Up, Directions.Down, Directions.Left, Directions.Right];
```

外部枚举定义的类型只会用于编译时的检查，编译结果中会被删除，上面的编译结果：

```JS
var directions = [Directions.Up, Directions.Down, Directions.Left, Directions.Right
```

外部枚举一般都会出现在声明文件中

# 类

## 概念复习

（1）面向对象（OOP）有三大特性：

- 封装
    将对数据的操作细节隐藏起来，只暴露对外的接口，调用者不需要知道细节，通过接口来访问对象，同时保证了外界无法任意更改对象内部的数据
- 继承
    子类继承父类，子类拥有父类所有特性，还有属于自己的特性
- 多态
    由继承产生了不同的类，对同一个方法有不同的响应（例如`Cat`和`Dog`都继承自`Animal`，分别实现了自己的`eat`方法）

（2）存取器

类的属性也有存取器即`setter`和`getter`，来改变属性的读取和赋值行为

（3）抽象类

抽象类是其他类继承的基类，抽象类不允许被实例化，抽象类中的抽象方法必须在子类中被实现。

（4）接口

不同类之间共有的属性和方法，可以抽象为一个接口，接口可以被类实现。一个类智能继承自另一个类，但是可以实现多个接口。

## TypeScript中的访问修饰符

TypeScript可以使用三种访问修饰符，分别是：`public`、`private`和`protected`

（1）`public`

`public`修饰的属性或方法是共有的，可以在任何地方被访问到，默认所有的属性和方法都是`public`的

（2）`private`

`private`修饰的属性或方法是私有的，不能在声明它的类的外部访问，子类中也是不允许的。当构造函数（`constructor`）是`private`时，这个类不允许被继承或者实例化

（3）`protected`

`protected`修饰的属性或方法是受保护的，它与`private`类似，区别是它在子类中是允许被访问的。当构造函数（`constructor`）是`protected`时，这个类不允许被实例化，只允许被继承。


```JS
class Animal {
  public name;
  public constructor(name) {
    this.name = name;
  }
}

let a = new Animal('Jack');
console.log(a.name); // Jack

a.name = 'Tom';
console.log(a.name); // Tom
```

修饰符还可以用在构造函数参数中，等同于在类中定义该属性，使代码更简洁：

```JS
class Animal {
  // public name: string;
  public constructor (public name) {
    this.name = name;
  }
}
```

## `readonly`

表明一个属性是只读属性，只允许出现在属性声明或者索引前面中，如果`readonly`和其他访问修饰符同时存在，`readonly`要写在后面

```JS
class Animal {
  public readonly name;
  public constructor(name) {
    this.name = name;
  }
}

let a = new Animal('Jack');
console.log(a.name); // Jack
a.name = 'Tom';

// index.ts(10,3): TS2540: Cannot assign to 'name' because it is a read-only property.
```

## 抽象类

用`abstract`来定义抽象类和其中的方法。

抽象类有一下几个特征：

（1）抽象类不允许被实例化

（2）抽象类中的抽象方法必须被子类实现，如果下面`Man`中没有`sayHi`方法，编译器就会报错

```JS
abstract class Person {
  public name: string;
  public constructor(name) {
    this.name = name;
  }

  public abstract sayHi();
}

class Man extends Person {
  sayHi() {
    console.log(this.name)
  }
}

const p1 = new Man('Jay');
p1.sayHi();
```

# 类与接口

接口可以用于对对象进行描述，同时，接口也可以对类的一部分行为进行抽象。

## 类实现接口

实现（implements）是面向对象中的一个重要概念。一般来说，一个类只能继承自另一个类，有时候不同的类之间有一些共有的特性，可以把这些特性提取成接口，用`implements`关键字实现。

举例来说，`门`是一个类，`防盗门`是`门`的子类。如果防盗门有一个报警器的功能，我们可以为`防盗门`添加`报警`的方法。同时，有另外一个类`车`，它也有报警器的功能，就可以将`报警`这个方法提取为一个接口，`防盗门`和`车`都去实现它

```JS
interface Alarm {
  alert()
}

class Door {
}

class SecurityDoor extends Door implements Alarm {
  alert() {
    console.log('SecurityDoor alert')
  }
}

class Car implements Alarm {
  alert() {
    console.log('Car alert')
  }
}
```

一个类可以实现多个接口：

```JS
interface Alarm {
  alert();
}

interface Light {
  lightOn();
  lightOff();
}

class Car implements Alarm, Light {
  alert() {
    console.log('Car alert');
  }
  lightOn() {
    console.log('Car light on');
  }
  lightOff() {
    console.log('Car light off');
  }
}
```

## 接口继承

接口是可以继承接口的:

```JS
interface Alarm {
  alert();
}

interface Light extends Alarm {
  lightOn();
  lightOff();
}

class Car implements Light {
  alert() {
    console.log('Car alert');
  }
  lightOn() {
    console.log('Car light on');
  }
  lightOff() {
    console.log('Car light off');
  }
}
```

接口也可以继承类：

```JS
class Alarm {
  alert() {}
}

interface Light extends Alarm {
  lightOn();
  lightOff();
}

class Car implements Light {
  alert() {
    console.log('Car alert');
  }
  lightOn() {
    console.log('Car light on');
  }
  lightOff() {
    console.log('Car light off');
  }
}
```

## 混合类型

可以用接口来定义函数：

```JS
interface SearchFn {
  (message: string): string,
}
const search: SearchFn = message => {
  return message;
};
```

有时候函数也可以有自己的属性和方法：

```JS
interface SearchFn {
  (message: string): string,
  reset(): void,
}
const search: SearchFn = message => {
  return message;
};
search.reset = function () {
  console.log('reset')
};
```

# 泛型

## 定义泛型

泛型（Generics）是指在定义函数、接口或者类的时候，不预先指定具体的类型，而是在使用的时候再指定类型的一种特性。

例如下面的函数`identity`，使用了类型变量`T`：

```JS
function identity<T>(arg: T): Array<T> {
  return [arg];
}
```

`T`帮助我们捕获用户传入的类型，然后在后面就可以使用这个类型了。我们将`T`作为返回值类型，这样保证了参数类型与返回值类型是相同的。

我们把上面的这种可以使用多个类型的函数叫做泛型，它可以适用于多个类型。不同于使用`any`，它不会丢失信息。


定义泛型的时候，可以一次定义多个类型变量:

```JS
function swap<T, U>(arr: [T, U]): [U, T] {
  return [arr[1], arr[0]];
}

swap([1, '1']);
```

## 使用泛型

定义了泛型后，有两种方法可以使用，第一中学是传入所有的参数，包括类型变量：

```JS
let output = identity<string>("myString");  // type of output will be 'string'
```

注意泛型变量的传递使用的是`<>`而不是`()`

第二种方法更为普遍，利用类型推论，让编译器自动确定`T`的类型

```JS
let output = identity("myString");  // type of output will be 'string'
```

## 泛型约束

在函数内部使用泛型变量的时候，由于事先不知道它是哪种类型，**必须把泛型变量当做任意类型**，所以不能随意的操作它的属性或方法：

```JS
function loggingIdentity<T>(arg: T): T {
  console.log(arg.length);
  return arg;
}

// index.ts(2,19): error TS2339: Property 'length' does not exist on type 'T'.
```

所以我们需要对泛型变量进行约束，使用`extends`来继承一个接口，约束传入的`T`必须符合接口的定义：

```JS
interface LengthWise {
  length: number
}

function identity<T extends LengthWise>(arg: T): Array<T> {
  console.log(arg.length);
  return [arg]
}
```

注意，我们约束的是泛型变量，而不是参数，如果用下面的形式，实际不是进行泛型约束，而是约束的函数参数：

```JS
// 或者用下面的形式，效果相同
function loggingIdentity<T>(arg: Array<T>): Array<T>) {
  console.log(arg.length);  // Array has a .length, so no more error
  return arg;
}
```

多个类型参数之间也可以相互约束。

## 泛型接口

可以使用含有泛型的接口来定义函数的形状：

```JS
interface CreateArrayFn {
  <T>(length: number, value: T): Array<T>
}

const createArray: CreateArrayFn = (length, value) => Array(length).fill(value);
```

可以将泛型接口提前到接口名上：

```JS
interface CreateArrayFn<T> {
  (length: number, value: T): Array<T>
}
```

这时候再使用的时候，必须在接口后定义泛型的类型：

```JS
const createArray: CreateArrayFn<any> = (length, value) => Array(length).fill(value)
```

## 泛型类

与泛型接口相似，泛型也可以用于类的类型定义中：

```JS
class Person<T> {
  key: T;
  constructor(key: T) {
    this.key = key;
  }
  say: (key: T) => T;
  go(key: T): T {
    return key;
  }
}

let  p = new Person<string>('Tom');
```

## 类型参数的默认类型

在TypeScript2.3版本后，可以为泛型中的类型参数指定默认类型。当使用泛型时没有在代码中直接指定类型参数，从实际参数中也无法推测出时，这个默认类型就会起作用：

```JS
class Person<T = string> {
  name: T;
  constructor(key: T) {
    this.name = key;
  }
  say: (name: T) => T;
  go(name: T): T {
    return name;
  }
}

let  p = new Person('Tom');
```

# 声明合并

如果定义了两个相同名字的函数、接口或者类，他们会合并为同一个类型

## 函数的合并

函数的合并实际上就是函数的重载：

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

## 接口的合并

接口中的属性会简单的合并到一个接口中，但是前提是合并的属性的类型必须是唯一的：

```JS
interface Alarm {
  price: number;
}
interface Alarm {
  price: number;  // 虽然重复了，但是类型都是 `number`，所以不会报错
  weight: number;
}
```

接口中方法的合并，与函数的合并相同：

```JS
interface Alarm {
  price: number;
  alert(s: string): string;
}
interface Alarm {
  weight: number;
  alert(s: string, n: number): string;
}
```

相当于：

```JS
interface Alarm {
  price: number;
  weight: number;
  alert(s: string): string;
  alert(s: string, n: number): string;
}
```

## 类的合并

类的合并与接口的合并规则相同。

# 代码检查

## ESLint

目前以及将来的TypeScript的代码检查方案是[`typeScript-eslint`](https://github.com/typescript-eslint/typescript-eslint)


TypeScript关注的类型的检查，而不是代码风格，代码风格的检查还是需要依靠ESLint。

## 在TypeScript中使用

安装：

```BASH
npm install --save-dev eslint typescript
```

由于ESLint默认使用Espree进行语法解析，无法识别TypeScript的某些语法，需要安装[`@typescript-eslint/parser`](https://github.com/typescript-eslint/typescript-eslint/tree/master/packages/parser)替代默认的解析器

```BASH
npm install --save-dev @typescript-eslint/parser
```

最后安装对应的规则插件[`@typescript-eslint/eslint-plugin`](https://github.com/typescript-eslint/typescript-eslint/tree/master/packages/eslint-plugin)

```BASH
npm install --save-dev @typescript-eslint/eslint-plugin
```

对ESLint进行配置，在根目录下创建`.eslintrc.js`：

```JS
module.exports = {
  parser: '@typescript-eslint/parser',
  plugins: ['@typescript-eslint'],
};
```

## 使用AlloyTeam的ESLint配置

推荐使用[AloyTeam的ESLint规则中的TypeScript版本](https://github.com/AlloyTeam/eslint-config-alloy#typescript)，安装和配置时直接按照下面的进行即可，已经包含了上面的安装和配置步骤

安装：

```BASH
npm install --save-dev eslint typescript @typescript-eslint/parser @typescript-eslint/eslint-plugin eslint-config-alloy
```


配置文件：

```JS
module.exports = {
  extends: [
    'alloy',
    'alloy/typescript',
  ],
  env: {
    // 您的环境变量（包含多个预定义的全局变量）
    // Your environments (which contains several predefined global variables)
    //
    // browser: true,
    // node: true,
    // mocha: true,
    // jest: true,
    // jquery: true
  },
  globals: {
    // 您的全局变量（设置为 false 表示它不允许被重新赋值）
    // Your global variables (setting to false means it's not allowed to be reassigned)
    //
    // myGlobal: false
  },
  rules: {
    // 自定义您的规则
    // Customize your rules
  }
};
```

## 检查`.tsx`文件

要检查React的`.tsc`文件，首先需要安装`eslint-plugin-react`：

```BASH
npm install --save-dev eslint-plugin-react
```

然后在`lint`命令后面需要添加`.tsx`后缀：

```JS
{
  "scripts": {
    "eslint": "eslint src --ext .ts,.tsx"
  }
}
```

也可以使用[AlloyTeam的ESLint规则的TypeScript React版本](https://github.com/AlloyTeam/eslint-config-alloy#typescript-react)。

安装：

```BASH
npm install --save-dev eslint typescript @typescript-eslint/parser @typescript-eslint/eslint-plugin eslint-plugin-react eslint-config-alloy
```

配置文件：

```JS
module.exports = {
  extends: [
    'alloy',
    'alloy/react',
    'alloy/typescript',
  ],
  env: {
    // Your environments (which contains several predefined global variables)
    //
    // browser: true,
    // node: true,
    // mocha: true,
    // jest: true,
    // jquery: true
  },
  globals: {
    // Your global variables (setting to false means it's not allowed to be reassigned)
    //
    // myGlobal: false
  },
  rules: {
    // Customize your rules
  }
};
```

# 参考

- [TypeScript入门教程](https://ts.xcatliu.com/advanced)
- [Typescript 中的 interface 和 type 到底有什么区别@掘金](https://juejin.im/post/5c2723635188252d1d34dc7d)

