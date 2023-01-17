---
title: TS09 unknow与any
top: false
date: 2021-05-19 17:30:43
updated: 2021-05-19 17:30:45
tags:
- TypeScript
categories: TypeScript
---

TypeScript中`unknow`和`any`类型的区别。

<!-- more -->

# `any`

`any`是代表所有可能的JavaScript值，对象、数组、函数、Error，以及任何你可能定义的值。

TypeScript中任何类型都可以归为`any`类型，折让`any`成为了类型系统的顶级类型，例如：

```JS
let value: any;

value = true;             // OK
value = 42;               // OK
value = "Hello World";    // OK
value = [];               // OK
value = {};               // OK
value = Math.random;      // OK
value = null;             // OK
value = undefined;        // OK
value = new TypeError();  // OK
value = Symbol("type");   // OK
```

本质上`any`是类型系统的逃逸仓，TypeScript允许我们对`any`类型的值执行任何操作，而无需执行任何形式的检查，这也就导致了我们很容易编写类型正确但是执行异常的代码，也就是说，使用了`any`就无法享受TypeScript大量的保护机制

所以TypeScript在3.0版本时推出了`unknow`类型

# `unknow`

就像所有类型都可以归为`any`一样，所有类型都可以归为`unknow`，这使得`unknow`成为了除了`any`之外，TypeScript类型系统的另一种顶级类型

```JS
let value: unknown;

value = true;             // OK
value = 42;               // OK
value = "Hello World";    // OK
value = [];               // OK
value = {};               // OK
value = Math.random;      // OK
value = null;             // OK
value = undefined;        // OK
value = new TypeError();  // OK
value = Symbol("type");   // OK
```

对`unknow`类型的变量赋值都被认为是类型正确的

但是当把`unknow`类型的值赋给其他类型的变量时，则不会像`any`一样，不受到任何限制：

```JS
let value: unknown;

let value1: unknown = value;   // OK
let value2: any = value;       // OK

let value3: boolean = value;   // Error
let value4: number = value;    // Error
let value5: string = value;    // Error
let value6: object = value;    // Error
let value7: any[] = value;     // Error
let value8: Function = value;  // Error
```

`unknow`类型的变量只能赋值给`unknow`和`any`类型的变量。此外，对`unknow`类型的变量执行操作的话，都会被认为是异常的：

```JS
let value: unknown;

value.foo.bar;  // Error
value.trim();   // Error
value();        // Error
new value();    // Error
value[0][1];    // Error
```

这就是`unknow`类型的主要价值主张：TypeScript不允许我们对类型为`unknow`的值进行任意操作，我们需要首先执行某种类型检查，以缩小`unknow`的类型范围

# 区别

总结一下`unknow`和`any`的区别：

1. `unknow`的值只能赋值给`unknow`和`any`类型的变量，`any`类型无限制
2. `unknow`的值无法进行任意操作，需要先进行类型检查以缩小类型，`any`类型无限制

# 缩小`unknow`类型范围

可以使用`typeof`、`instance`运算符（一般应用在逻辑判断分支中）来缩小类型

```JS
const dogName = getDogName();
if (typeof dogName === 'string') {
  console.log(dogName.toLowerCase());
}


type getAnimal = () => unknown;
const dog = getAnimal();
if (dog instanceof Dog) {
  console.log(dog.name.toLowerCase());
}

```


还可以通过`is`类型守卫来缩小类型范围：


```JS
function isA(x): x is number {
  return true
}

function foo(x: unknown) {
  if(isA(x)) {
    return x
  }
  return null;
}
```


# 使用断言`as`

也可以使用`as`类型断言来让TypeScript信任`unknow`的类型为给定类型：

```JS
const value: unknown = "Hello World";
const someString: string = value as string;
const otherString = someString.toUpperCase();  // "HELLO WORLD"
```

要注意的是，TypeScript并未执行任何特殊检查以去报类型断言的正确性，类型检查器假定你使用的断言是正确的，如果我们指定了错误的类型断言，就会在运行时抛出错误

```JS
const value: unknown = 42;
const someString: string = value as string;
const otherString = someString.toUpperCase();  // BOOM
```

# 联合类型中的`unknow`

联合类型中`unknow`会吸收其他类型，也就是说联合类型中的某一个成员是`unknow`，那么联合类型也就相当于`unknow`

```JS
type UnionType1 = unknown | null;       // unknown
type UnionType2 = unknown | undefined;  // unknown
type UnionType3 = unknown | string;     // unknown
type UnionType4 = unknown | number[];   // unknown
```

例外之处就是`any`，联合类型中的某一个成员是`any`，那么联合类型也就相当于`any`（可以理解为`any`的权重更高）

```JS
type UnionType5 = unknown | any;  // any
```

# 交叉类型中的`unknow`

交叉类型的`unknow`也会吸收其他类型，也就是说，如果交叉类型的某一个成员是`unknow`，那么不会改变交叉类型的结果

```JS
type IntersectionType1 = unknown & null;       // null
type IntersectionType2 = unknown & undefined;  // undefined
type IntersectionType3 = unknown & string;     // string
type IntersectionType4 = unknown & number[];   // number[]
type IntersectionType5 = unknown & any;        // any
```

# 实例：在`localStorage`中读取数据

假设我们需要编写一个函数，完成下述功能：从`localStorage`中读取字符串，并将读取结果反序列化为JSON，如果该项不存在或者是无效JSON，则函数返回错误结果

因为我们不知道反序列化的JSON字符串的结果是什么类型，所以需要使用`unknow`作为反序列化值的类型，也就是说函数调用者着必须在对返回值执行操作之前对函数的返回结果进行某种类型的检查

```JS
type Result =
  | { success: true, value: unknown }
  | { success: false, error: Error };

function tryDeserializeLocalStorageItem(key: string): Result {
  const item = localStorage.getItem(key);

  if (item === null) {
    // The item does not exist, thus return an error result
    return {
      success: false,
      error: new Error(`Item with key "${key}" does not exist`)
    };
  }

  let value: unknown;

  try {
    value = JSON.parse(item);
  } catch (error) {
    // The item is not valid JSON, thus return an error result
    return {
      success: false,
      error
    };
  }

  // Everything's fine, thus return a success result
  return {
    success: true,
    value
  };
}
```

函数的调用者需要在使用结果之前对类型进行收窄：

```JS
const result = tryDeserializeLocalStorageItem("dark_mode");

if (result.success) {
  // We've narrowed the `success` property to `true`,
  // so we can access the `value` property
  const darkModeEnabled: unknown = result.value;

  if (typeof darkModeEnabled === "boolean") {
    // We've narrowed the `unknown` type to `boolean`,
    // so we can safely use `darkModeEnabled` as a boolean
    console.log("Dark mode enabled: " + darkModeEnabled);
  }
} else {
  // We've narrowed the `success` property to `false`,
  // so we can access the `error` property
  console.error(result.error);
}
```

# 参考

- [TypeScript 3.0: unknown 类型@掘金](https://juejin.cn/post/6844903866073350151)
