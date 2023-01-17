---
title: 零散专题35 AST抽象语法树
top: false
date: 2019-08-01 17:53:56
updated: 2019-08-08 18:27:34
tags: 
- Babel
- AST
categories: 零散专题
---

AST抽象语法树学习笔记。

<!-- more -->

## 什么是抽象语法树

抽象语法树（abstract syntax tree，AST，或者简称语法树）是源代码的抽象语法结构的树状表现形式，它以树状的形式表现编程语言的语法结构，树上的每个节点都表示源代码中的一种结构。

之所以说是抽象的，是因为这里的语法并不会表示出真实语法中出现的每个细节，比如嵌套括号被隐含在树结构中，并不会以节点的形式呈现出来。而类似于`if...else`这样的条件语句，可以使用两个分支的节点来表示。

比如下面这段代码：

```JS
while (b !== 0) {
  if (a > b) {
      a = a - b
  } else {
      b = b - a
  }
}
return a;
```

它的对应的抽象语法树如下：

![](http://image.oldzhou.cn/FghfGt6-abAekpIk3IZQeCdRiPae)

可以通过[astexplorer](https://astexplorer.net)这个网站，将我们输入的代码转换为AST展示出来，可以展示为书结构，也可以直接以JSON格式展示，比如在左侧输入`let a = 123;`，对应的语法树：

![](http://image.oldzhou.cn/Ft_l0slhgR-hd7KdFjWqAbxgnnwZ)

也可以直接输入JSON：

```JS
{
  "type": "Program",
  "start": 0,
  "end": 12,
  "body": [
    {
      "type": "VariableDeclaration",
      "start": 0,
      "end": 12,
      "declarations": [
        {
          "type": "VariableDeclarator",
          "start": 4,
          "end": 11,
          "id": {
            "type": "Identifier",
            "start": 4,
            "end": 5,
            "name": "a"
          },
          "init": {
            "type": "Literal",
            "start": 8,
            "end": 11,
            "value": 123,
            "raw": "123"
          }
        }
      ],
      "kind": "let"
    }
  ],
  "sourceType": "module"
}
```

语法树中每个元素可以成为一个节点（node），类型包括：

```
(parameter) node: 
Identifier | SimpleLiteral | RegExpLiteral | Program | FunctionDeclaration | FunctionExpression |
ArrowFunctionExpression | SwitchCase | CatchClause | VariableDeclarator | ExpressionStatement |
BlockStatement | EmptyStatement | DebuggerStatement | WithStatement | ReturnStatement |
LabeledStatement | BreakStatement | ContinueStatement | IfStatement | SwitchStatement | ThrowStatement | 
TryStatement | WhileStatement | DoWhileStatement | ForStatement | ForInStatement | ForOfStatement |
VariableDeclaration | ClassDeclaration | ThisExpression | ArrayExpression | ObjectExpression |
YieldExpression | UnaryExpression | UpdateExpression | BinaryExpression | AssignmentExpression |
LogicalExpression | MemberExpression | ConditionalExpression | SimpleCallExpression | NewExpression |
SequenceExpression | TemplateLiteral | TaggedTemplateExpression | ClassExpression | MetaProperty |
AwaitExpression | Property | AssignmentProperty | Super | TemplateElement | SpreadElement |
ObjectPattern | ArrayPattern | RestElement | AssignmentPattern | ClassBody | MethodDefinition |
ImportDeclaration | ExportNamedDeclaration | ExportDefaultDeclaration | ExportAllDeclaration |
ImportSpecifier | ImportDefaultSpecifier | ImportNamespaceSpecifier | ExportSpecifier
```




## 使用场景

平时我们好像从来没有接触过AST，但是实际上它的适用场景一直相伴左右，比如：

- JS反编译，语法解析
- Babel编译ES6语法
- 代码高亮
- 关键字匹配
- 作用域判断
- 代码压缩

## AST分析

使用刚才提到的[astexplorer](https://astexplorer.net/)这个网站，解析`1 + 2 * 3`，得到的AST语法树

![](http://image.oldzhou.cn/FlH29OFwifeYWhw62_PkenyIHObl)

从语法树我们开出来，`body`里面的`exporession`（表达式）是`BinaryExpress`（二院表达式），`start`和`end`标识了我们这个表达式字符的起止位置，`operator`是`+`，然后`left`属性的值是一个`Literal`字面量，`value`是`2`，`right`属性的值又是另外一个`BinaryExpress`，`left`的值是`2`，`operator`是`*`，`right`右侧值是`3`

这样，通过一个树状结构，将我们输入的表达式明明白白展示出来了。

将表达式添加一个括号，变成`(1 + 2) * 3`，得到的新的语法树：

![](http://image.oldzhou.cn/FnTKobs0WWza3GzqHS2np0f0j6DH)

可以看出来，`left`值变为了一个`BinaryExpress`，`operator`是`*`，`right`值是`3`。

比较两个表达式的语法树，可以发现：

1. 在确定类型为`ExpressionSatatement`后，会按照代码执行的先后顺序，将表达式`BinaryExpression`分解为`left`、`operator`、`right`三部分
2. 每部分都标明了类型、起止位置、值等信息
3. 标明了操作符类型

再来看一下常用的箭头函数：

```JS
const fn = (a, b) => {
  return a + b
};
```
AST的JSON结果：

```JS
{
  "type": "Program",
  "start": 0,
  "end": 40,
  "body": [
    {
      "type": "VariableDeclaration",
      "start": 0,
      "end": 40,
      "declarations": [
        {
          "type": "VariableDeclarator",
          "start": 6,
          "end": 39,
          "id": {
            "type": "Identifier",
            "start": 6,
            "end": 8,
            "name": "fn"
          },
          "init": {
            "type": "ArrowFunctionExpression",
            "start": 11,
            "end": 39,
            "id": null,
            "expression": false,
            "generator": false,
            "params": [
              {
                "type": "Identifier",
                "start": 12,
                "end": 13,
                "name": "a"
              },
              {
                "type": "Identifier",
                "start": 15,
                "end": 16,
                "name": "b"
              }
            ],
            "body": {
              "type": "BlockStatement",
              "start": 21,
              "end": 39,
              "body": [
                {
                  "type": "ReturnStatement",
                  "start": 25,
                  "end": 37,
                  "argument": {
                    "type": "BinaryExpression",
                    "start": 32,
                    "end": 37,
                    "left": {
                      "type": "Identifier",
                      "start": 32,
                      "end": 33,
                      "name": "a"
                    },
                    "operator": "+",
                    "right": {
                      "type": "Identifier",
                      "start": 36,
                      "end": 37,
                      "name": "b"
                    }
                  }
                }
              ]
            }
          }
        }
      ],
      "kind": "const"
    }
  ],
  "sourceType": "module"
}
```

耐下性子，仔细读一下语法树，其实也不难理解，首先`body`的`type`变为了`VariableDeclaration`，`declarations`是一个数组，首先声明了`fn`这个变量，然后在`init`里面是一个`type`为`ArrowFunctionExpression`的节点，里面有`params`、`body`、`ReturnStatement`等各种用来标识箭头函数的属性值，

## 实现

从上面可以看出来，抽象语法树其实就是将一类标签转化成通用标识符，从而归纳出的一个类似于树形结构的语法树。

那么这个过程是如何实现的呢？

简单学习了一下，首先需要几个工具包用来解析JS语法、遍历树结构以及生成新的树结构：

```JS
const esprima = require('esprima'); // 解析js的语法的包
const estraverse = require('estraverse'); // 遍历树的包
const escodegen = require('escodegen'); // 生成新的树的包

let code = `function getAST(){}`;

//解析js的语法
let tree = esprima.parseScript(code);
//遍历树
estraverse.traverse(tree, {
  enter(node) {
    console.log('enter: ' + node.type);
  },
  leave(node) {
    console.log('leave: ' + node.type);
  }
});

//生成新的树
let r = escodegen.generate(tree);
console.log(r);
```

通过遍历、修改抽象语法树，我们可以去改变任何输出结果，也就是说，在编译的过程中，借助抽象语法树，我们可以去改变原来的输入，得到想要的结果。

这也就是Babel转义JavaScript代码的原理。

## 关于Babel

Babel使用一个基于ESTree并修改过的AST，它的内核说明文档可以在[这里](https://github.com/babel/babel/blob/master/packages/babel-parser/ast/spec.md)找到。

Babel的三个主要处理步骤分别是： 解析（parse），转换（transform），生成（generate）。

（1）解析

接受代码并输出AST，这个步骤又分为两个阶段：词法分析（Lexical Aanalysis）和语法分析（Syntactic Analysis）

词法分析阶段把字符串形式的代码转换为令牌流，可以将令牌看做一个扁平的语法片段数组

```JS
n * n;

↓

[
  { type: { ... }, value: "n", start: 0, end: 1, loc: { ... } },
  { type: { ... }, value: "*", start: 2, end: 3, loc: { ... } },
  { type: { ... }, value: "n", start: 4, end: 5, loc: { ... } },
  ...
]
```

其中每一个`type`都有一组数形来描述该令牌，和AST节点一样也有`start`、`end`等属性：

```JS
{
  type: {
    label: 'name',
    keyword: undefined,
    beforeExpr: false,
    startsExpr: true,
    rightAssociative: false,
    isLoop: false,
    isAssign: false,
    prefix: false,
    postfix: false,
    binop: null,
    updateContext: null
  },
  ...
}
```

语法分析阶段会吧一个令牌转换成AST的形式，这个阶段会使用令牌中的信息把他们转换成一个AST的表述结构，便于后续操作

（2）转换

接受AST并对其进行遍历，此过程中对节点进行添加、更新和移除等操作。这是Babel最复杂的过程，也是Babel插件接入工作的部分。

（3）生成

深度优先遍历将经过一系列转换后的AST，并将其转换成字符串形式的代码，同时还会创建源码映射。

## 单元测试覆盖率

JavaScript单元测试覆盖率统计方法的核心思想，是在源代码响应的位置注入设定的统计代码，当执行测试代码的时候，代码运行到注入的地方，就会执行对应的统计代码，生成覆盖率统计报告。大概步骤如下：

（1）对源代码进行语法分析、解析，然后生成抽象语法树

（2）在语法树相应的位置注入统计代码。

在程序执行到这个位置的时候对相应的全局变量赋值，确保执行之后能够根据全局变量知道代码的执行流程。

（3）通过注入统计代码的抽象语法树，生成对应的JavaScript代码

（4）将生成好的JavaScript代码交给执行环境（Node或者浏览器）运行

（5）执行单元测试，产生的统计信息，放到全局变量里面。

（6）根据全局变量中的覆盖率信息生成特定格式的报告。

## 总结

大致学习了一下AST的知识，并没有很深入，只是了解到AST是对源代码的一种树状结构的表示形式，可以用来在编译阶段对代码进行处理。Babel之所以能够实现JavaScript的转换，就是利用了AST，经历了解析、转换、生成三个步骤，其中转换是最复杂的步骤，各种Babel的插件也是在这个步骤中发挥作用。

## 参考

- [抽象语法树@维基百科](https://zh.wikipedia.org/zh-hans/%E6%8A%BD%E8%B1%A1%E8%AA%9E%E6%B3%95%E6%A8%B9)
- [AST 抽象语法树@jartto](http://jartto.wang/2018/11/17/about-ast/)
- [Babel 插件有啥用？@知乎](https://zhuanlan.zhihu.com/p/61780633)
- [代码测试覆盖率分析@segmentfault](https://segmentfault.com/a/1190000010838845)
