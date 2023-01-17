---
title: TS08 TypeScript中的泛型工具
top: false
date: 2020-12-02 17:19:00
updated: 2020-12-02 17:19:01
tags:
- TypeScript
- 泛型
categories: TypeScript
---

TypeScript中内置的泛型工具。

<!-- more -->

TS提供了很多泛型工具类型，可以帮助我们实现很多复杂的类型。之前对这块没有了解，很多复杂的类型都是自己来实现，其实借助这些工具可以很方便的实现

# 使用`is`进行类型收窄

```TS
function isUser(person: Person): person is User {
  return person.type === 'user'
}
```

# 使用`&`进行属性的合并

使用`&`操作符可以实现属性合并

```TS
type User = {
  name: string
}

type Admin = {
  age: number
}

type Person = User | Admin;
// 此时 Person 可以包含 name 或者可以包含 age

type Staff = User & Admin;
// 此时 Staff 必须同时包含 name 和age
```

# `Partial<Type>`

使用`Partial<Type>`将所有类型属性变成可选


```TS
interface Todo {
  title: string;
  description: string;
}

function updateTodo(todo: Todo, fieldsToUpdate: Partial<Todo>) {
  return { ...todo, ...fieldsToUpdate };
}

const todo1 = {
  title: "organize desk",
  description: "clear clutter",
};

const todo2 = updateTodo(todo1, {
  description: "throw out trash",
});
```

`Partial`的实现：

```TS
type Partial<T> = { [P in keyof T]?: T[P] };
```

# `Required<Type>`

与`Partial`相反，将可选属性变为必选

```TS
interface Props {
  a?: number;
  b?: string;
}

const obj: Props = { a: 5 };

const obj2: Required<Props> = { a: 5 };
// 报错
// Property 'b' is missing in type '{ a: number; }' but required in type 'Required<Props>'.
```


# `Readonly<Type>`

使用`Readonly<Type>`将类型设置为只读

```TS
interface Todo {
  title: string;
}

const todo: Readonly<Todo> = {
  title: "Delete inactive users",
};

todo.title = "Hello";
// 报错
// Cannot assign to 'title' because it is a read-only property.
```

# `Record<keys, Type>`

使用`Record`生成的类型，包含`keys`对应的属性键名，键值为`Type`，常用来实现一种类型对另外一种类型的映射关系：

```TS
interface PageInfo {
  title: string;
}

type Page = 'home' | 'about' | 'concat'';

const nav: Record<Page, PageInfo> = {
  about: {title: 'about'},
  contact: {title: 'contact'},
  home: {title: 'home'},
}
```

# `Pick<Type, Keys>`

使用`Pick`来在`Type`中拾取`keys`对应的属性

```TS
interface Todo {
  title: string;
  description: string;
  completed: boolean;
}

type TodoPreview = Pick<Todo, "title" | "completed">;

const todo: TodoPreview = {
  title: 'Clean Room',
  completed: false
}
```

# `Omit<Type, keys>`

从`Type`中生成所有属性，并且移除`key`对应的属性

```TS
interface Todo {
  title: string;
  description: string;
  completed: boolean;
}

type TodoPrview = Omit<Todo, 'description'>;

const todo: TodoPreview = {
  title: "Clean room",
  completed: false,
};
```

# `Exclude<Type, ExcludedUnion>`

在`Type`中移除`ExcludedUnion`中的所有类型（差集）

```TS
type T0 = Exclude<"a" | "b" | "c", "a">;
// type T0 = "b" | "c"

type T1 = Exclude<"a" | "b" | "c", "a" | "b">;
// type T1 = "c"

type T2 = Exclude<string | number | (() => void), Function>;
// type T2 = string | number
```

# `Extract<Type, Union>`

生成`Type`与`Union`的交集

```TS
type T0 = Extract<"a" | "b" | "c", "a" | "f">;
// type T0 = "a"

type T1 = Extract<string | number | (() => void), Function>;
// type T1 = () => void
```

# `NonNullable<Type>`

生成不为`null`和`undefined`的类型

```TS
type T0 = NonNullable<string | number | undefined>;
// type T0 = string | number

type T1 = NonNullable<string[] | null | undefined>;
// type T1 = string[]
```

# `Parameters<Type>`

生成对应的函数类型的Type的参数元组类型

```TS
declare function f1(arg: { a: number; b: string }): void;

type T0 = Parameters<() => string>;
// type T0 = []

type T1 = Parameters<(s: string) => void>;
// type T1 = [s: string]

type T2 = Parameters<<T>(arg: T) => T>;
// type T2 = [arg: unknown]

type T3 = Parameters<typeof f1>;
// type T3 = [arg: {
//   a: number;
//   b: string;
// }]
```

# `ReturnType<Type>`

生成函数返回值对应的类型

```TS
declare function f1(): { a: number; b: string };

type T0 = ReturnType<() => string>;
// type T0 = string

type T1 = ReturnType<(s: string) => void>;
// type T1 = void

type T2 = ReturnType<<T>() => T>;
// type T2 = unknown

type T4 = ReturnType<typeof f1>;
// type T4 = {
//   a: number;
//   b: string;
// }
```

# `ThisParameterType<Type>`

对于一个函数的`this`的类型，可以通过在参数中对进行声明：

```TS
function toHex(this: Number) {
  return this.toString(16);
}
```

这时可以使用`ThisParameterType<Type>`返回`this`类型，如果函数没有声明`this`类型，那么会返回`unknown`

```TS
function numberToString(n: ThisParameterType<typeof toHex>) {
  return toHex.apply(n);
}
```

# 参考

- [TS](https://www.typescriptlang.org/docs/handbook/utility-types.html)

