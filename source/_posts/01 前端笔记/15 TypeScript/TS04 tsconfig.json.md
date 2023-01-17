---
title: TS04 tsconfig.json
top: false
date: 2019-12-17 16:51:00
updated: 2019-12-17 16:51:00
tags:
- TypeScript
- tsconfig.json
categories: TypeScript
---

tsconfig.json配置简介。

<!-- more -->

## 概述

在进行少量编译时，可以直接在命令行使用`tsc`命令：

```BASH
tsc --outFile file.js --target es3 --module commonjs file.ts
```

在根目录下新建`tsconfig.json`文件，用来指定编译这个项目的选项，在命令行使用`tsc`命令时，如果没有任何输入文件情况下调用，那么编译器会从当前目录开始查找`tsconfig.json`文件，并逐级向上搜索，直至根目录。

## 示例

```JS
{
  "compilerOptions": {
    "module": "commonjs",
    "noImplicitAny": true,
    "removeComments": true,
    "preserveConstEnums": true,
    "sourceMap": true
  },
  "files": [
    "app.ts",
    "foo.ts",
  ]
}
```

`compilerOptions`用来配置编译选项，`files`用来指定带编译的入口文件，任何被入口文件依赖的文件，都会被编译器自动纳入为编译对象。

也可以使用`include`和`exclude`来指定和排除带编译文件：

```JS
{
  "compilerOptions": {
    "module": "commonjs",
    "noImplicitAny": true,
    "removeComments": true,
    "preserveConstEnums": true,
    "sourceMap": true
  },
  "include": [
    "src/**/*"
  ],
  "exclude": [
    "node_modules",
    "**/*.spec.ts"
  ]
}
```

## 指定编译目录

可以使用`files`属性，它接受一个数组，数组元素可以是相对路径或者绝对路径

也可以使用`include`和`exclude`属性，它们都接受一个数组，支持通配符`*`（匹配任意字符）、`?`（匹配任意单个字符）、`**/`（递归匹配任何子路径）

对于编译器来说，TS文件默认指的是`.ts`、`.tsx`和`.d.ts`后缀的文件，如果开启了`allowJs`选项，那么`.js`和`.jsx`文件也会被认为是TS文件。

如果`files`和`include`都未设置，那么除了`exclude`排除的文件，编译器会默认包含路径下所有TS文件。

如果同时设置`files`和`include`，那么二者指定的文件都会被编译

`exclude`的默认值是`node_modules`、`bower_components`、`jspm_packages`和编译选项`outDir`指定的路径。

`exclude`对`include`有效，但是对于`files`无效，如果文件在`files`中引入，那么`exclude`不生效。

## 编译选项

[完整的选项列表](http://www.typescriptlang.org/docs/handbook/compiler-options.html)。

字段 | 类型 | 默认值 | 说明
---|---|---|---
`allowJs` | boolean | false | 允许编译JS文件
`checkJs` | boolean | false | 报告JS文件中存在的类型错误（需打开`allowJS`）
`declaration` | boolean | false | 生成对应的`.d.ts`声明文件
`declarationDir` | string | - | 生成的`.d.ts`文件路径（默认与`.ts`相同）
`experimentalDecorators` | boolean | false | 启用装饰器功能
`jsx` | string | Preserve | 在`.tsx`中支持JSX，[详细说明](http://www.typescriptlang.org/docs/handbook/jsx.html#basic-usage)
`lib` | string[] | - | 编译对应的ES版本（`es5`/`es6`/`es7`/`dom`等），根据`target`不同取值不同
`module` | string | `commonjs`/`es6` | 生成模块格式，与`target`有关
`moduleResolution` | string | `classic`/`node` | 模块解析方式，与`module`有关，[详细说明](http://www.typescriptlang.org/docs/handbook/module-resolution.html/)
`noImplicitAny` | boolean | false | 存在隐式`any`时报错
`noImplicitReturns` | boolean | false | 函数无`return`时报错
`noImplicitThis` | boolean | false | `this`可能为`any`时报错
`outDir` | string | - | 编译输出目录，默认与`.ts`相同
`sourceMap` | boolean | false | 生成`.map`文件
`target` | string | es3 | 编译的JS文件版本


### 类型相关

`typeRoots`和`types`，指定默认的类型声明文件的查找路径，默认为`node_modules/@types`，只对NPM安装的模块有效

```JS
{
  "compilerOptions": {
    "typeRoots": ["./typings"], // 引入`./typings`下所有TS类型声明模块
    "types": ["node", "lodash"], // 只引入两个声明模块，其他的不会被引入
  }
}
```

自动引入只对包含全局声明的模块有效，比如`jQuery`，不用再手动`import`就可以在任何文件使用`$`的类型，对于`import 'foo'`，编译器会在`node_modules`和`node_modules/@types`文件下查找`foo`模块和声明模块

所以，如果想让自定义声明的类型不需要手动引入就可以在任何地方引入，可以将其声明为`global`：

```JS
declare global {
  const graphql: (query: TemplateStringsArray) => void;
  namespace Gatsby {
    interface ComponentProps {
      children: () => React.ReactNode,
      data: RootQueryType
    }
  }
}
```

这样就可以在任何地方直接使用`graphql`和`Gatsby`对应的类型了

## 配置复用

可以使用`extends`来实现配置复用，即一个配置文件可以继承另外一个文件的配置属性：

```JS
{
  "extends": "./configs/base",
  "files": [
    "main.ts",
    "supplemental.ts"
  ]
}
```

要注意，子配置的同名配置会覆盖父配置文件的配置，同时相对路径都会根据子配置文件所在路径进行解析

## 参考

- [理解 Typescript 配置文件@segmentfault](https://segmentfault.com/a/1190000013514680)
- [tsconfig.json@TypeScript](http://www.typescriptlang.org/docs/handbook/tsconfig-json.html/)
