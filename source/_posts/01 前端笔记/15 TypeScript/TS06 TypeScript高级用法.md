---
title: TS06 TypeScript高级用法
top: false
date: 2020-06-22 18:23:03
updated: 2020-06-22 18:23:05
tags:
- TypeScript
categories: TypeScript
---

TypeScript中的一些高级技巧。

<!-- more -->

# 使用泛型+`type`定义类型函数

使用`type`+泛型，可以定义类型函数：

```TS
type foo<T> = T;
type bar = foo<string>
```

可以对泛型入参进行约束、设置默认值

```TS
// 对入参进行约束
type foo<T extends string> = T;
type bar = foo<string>

// 设定默认值
type foo<T extends string = '123'> = T;
type bar = foo<string>
```

# 条件判断

TypeScript中使用`extends`进行类型判断，与三元运算符很相似：


```TS
T extends U ? X : Y;
```

如果`T`的类型能够`extends U`，结果返回`X`，否则返回`Y`。

结合上面的类型函数，可以进行扩展：

```
type num = 1;
type str = 'hello world';

type IsNumber<N> = N extends number ? 'yes is a number' : 'no not a number';

type result1 = IsNumber<num>; // 类型为 'yes is a number'
type result2 = IsNumber<str> // 类型为  'no not a number'
```

# 遍历联合类型

使用`in`关键在来遍历`type`的联合类型

> 联合类型就是使用`|`来定义的类型的合集

```TS
type Key = 'vue' | 'react';

type Mapped = {[k in Key]: string}; // {vue: string; react: string}

const bar: Mapped = {
  vue: 's',
  react: 's'
}
```

如果联合类型不是我们显式的定义出来的，那么想要动态的推导出联合类型的类型，需要使用`keyof`方法

```TS
interface Person {
  name: string;
  age: number;
}

type Foo = keyof Person; // 'name' | 'age'
```

# 对联合类型进行`map`操作

可以使用`extends`+泛型，实现将一组联合类型批量映射为另一种类型：


```TS
type Foo = 'a' | 'b';
type UnionTypesMap<T, U> = U;

type Bar = UnionTypesMap<Foo, 5>
```

# 全局作用域

使用`declare`关键字来声明全局作用域：

```TS
declare module '*.png';
declare module '*.svg';
declare module '*.jpg';

declare type str = string;
declare interface Foo {
  propA: string;
  propB: number;
}
```

要注意，如果模块使用了`export`关键字导出了内容，上述方式可能会失效，需要显式的声明到全局：

```TS
declare global {
  const ModuleGlobalFoo: string;
}
```

注意，上述方式之恩能够用在模块声明内，也就是说代码中必须包含`export`，否则就会报错：

```TEXT
TS2669: Augmentations for the global scope can only be directly nested in external modules or ambient module declarations.
```

# 模块作用域

模块作用域的触发条件之一就是使用`export`关键字导出内容，在其他模块获取时需要`import`导入

# `never`类型的作用

`never`类型代表空集，常用语校验“类型收窄”是否符合预期，常被用来做“类型收底”。

例如，有一个联合类型时：

```TS
interface Foo {
  type: 'foo'
}

interface Bar {
  type: 'bar'
}

type All = Foo | Bar
```

在`switch`判断`type`，TS是可以收窄类型的（discriminated union）：

```JS
funcfunction handleValue(val: All) {
  switch (val.type) {
    case 'foo':
      // 这里 val 被收窄为 Foo
      break
    case 'bar':
      // val 在这里是 Bar
      break
    default:
      // val 在这里是 never
      const exhaustiveCheck: never = val
      break
  }
}
```

在`default`里面把收窄的`never`的`val`赋给了显示声明为`never`的变量。如果有一天`type`类型被修改了：

```TS
type All = Foo | Bar | Baz
```

如果没有在`handleValue`添加针对`Baz`的处理逻辑，这时候在`default`的分支中就会编译错误。所以通过这个办法可以确保`switch`总是穷尽了所有`All`的可能性

# 用`never`进行类型过滤联合类型

当`never`参与运算时`T | never `的结果是`T`，根据这个规则就可以过滤联合类型中不符合期望的类型，TS内置的`Exclude`泛型操作符就是根据这个原理实现的：

```TS
/**
 * Exclude from T those types that are assignable to U
 */
type Exclude<T, U> = T extends U ? never : T;

type A = 1 | 2;
type B = Exclude<A, 1>; // 2
```

# 自定义类型守卫

类型守卫（Type Guard）的目的就是为了对类型进行分流，将联合类型分发到不同管道。可以出发类型守卫的常见方式有：`typeof`、`instancof`、`in`、`==`、`===`、`!=`、`!==`等

```JS
function foo(x: A | B) {
  if (x instanceof A) {
    // x is A
  } else {
    // x is B
  }
}
```

当以上的方式不满足需求时，可以通过`is`关键字自定义类型守卫：

```TS
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

# 参考

- [【万字长文】深入理解 Typescript 高级用法@知乎](https://zhuanlan.zhihu.com/p/136254808)
- [TypeScript中的never类型具体有什么用？ - 尤雨溪的回答 @知乎？](https://www.zhihu.com/question/354601204/answer/888551021)
