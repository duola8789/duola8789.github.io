---
title: 零散专题34 JavaScript的垃圾回收机制
top: false
date: 2019-07-03 10:47:00
updated: 2019-08-06 15:59:00
tags: 
- Chrome
- 垃圾回收
categories: 零散专题
---

JavaScript的垃圾回收机制学习笔记。

<!-- more -->

## 手动回收？

JavaScript在创建变量时自动进行了内存分配，并且在不使用时自动释放。释放的过程叫做垃圾回收。

JavaScript没有暴露任何垃圾回收期的接口，其内存管理是自动执行且不可见的，所以开发者没有办法手动进行强制的垃圾回收，也没有办法干预内存管理。

在多数情况下，开发者也没有必要手动解除对象的引用，只要简单地把变量放在他们应该的地方（局部变量），垃圾就能正确被回收

## V8的内存分配

当声明变量并赋值时，V8就会在堆内存中分配一部分给这个变量。如果已申请的内存不足以存储这个变量，V8就会继续申请，直到堆的大小到了V8的内存上限为止。

堆内存分为两种分配方式：

1. 静态分配，全局变量、函数之类的分配方式，它们在页面没有关闭之前，是不会被清除的
2. 动态分配，使用`new`创建出来的，主动要求给分配空间的

## 可达性

JavaScript中内存管理关键的一个概念是**可达性**，一个变量的“可达性”值得就是它能够被以某种方式访问到或者引用到，被保证存储到内存中。JavaScript引擎中有一个名为“垃圾回收器”的进程，它监视所有对象，并删除不可访问的对象。

一些常见的具有可达性，会常驻内存的情况：

（1）全局变量，全局变量会常驻内存，除非刷新页面、离开页面时才会被清理

（2）对象引用

```JS
function ob(){
  var bar  = new largeObject(); // 很大一个对象\变量\字符串
  bar.someCall();
  return bar;
}

var a = new ob();
```
现在有一个指向`bar`对象的引用，当`ob`调用结束后，`bar`对象不会被回收，直到变量`a`分配其他引用（或者`a`超出了作用域范围）。

（3）DOM事件，即使DOM元素被移除，其绑定的事件不会被回收，除非使用`removeEventListener`显式的移除事件

（4）定时器，除非显式的清楚定时器

## 例子

假设定义了这样一个变量：

```JS
let user = {
  name: 'John'
}
```

`user`这个变量引用了一个对象`{name: 'John'}`：

![](http://image.oldzhou.cn/Fg0ZsxO9cRo3N0xPB_lR-Z_FME69)

如果将`user`的值覆盖`user = null`，则引用丢失，`{name: 'John'}`会被垃圾回收器清除，分配的内存被回收。

![](http://image.oldzhou.cn/FoT-updtMe1dMVa0MVUCcynn28kO)

如果不将`user`的值覆盖，而是把`user`复制给另一个变量`admin`：

```JS
let admin = user
```

这是两个变量都引用了同一个对象`{name: 'John'}`：

![](http://image.oldzhou.cn/FrpeEj6n8KADkN1ZcxblGYQBu0Q_)

这个时候再将`user`的值覆盖`user = null`，虽然`user`不再引用`{name: 'John'}`，但是`admin`仍然与`{name: 'John'}`保持引用关系，所以`{name: 'John'}`不会被垃圾回收器清除，分配的内存也不会被回收。

## 复杂些的例子

函数`marry`通过给两个对象彼此提供引用来“联姻”它们，并返回一个包含两个对象的新对象

```JS
function marry (man, woman) {
  woman.husban = man;
  man.wife = woman;

  return {
    father: man,
    mother: woman
  }
}

let family = marry({
  name: "John"
}, {
  name: "Ann"
})
```

产生的内存结构:

![](http://image.oldzhou.cn/FjucC_kJRTq0mNA62CSB_sEz3haP)

现在如果想要清除`{name: John}`这个变量，必须要**同时删除外界对它的所有引用**：

```JS
delete family.father;
delete family.mother.husband;
```

![](http://image.oldzhou.cn/FqHC1eNh4qu41wAPOYs83NPupkYD)


它对外的引用`man.wife`是否删除不重要，因为只有外界对变量的引用，才决定了这个变量是否是“可达”的。现在虽然`{name: John}`仍然通过`wife`属性引用了`{name: 'Ann'}`，但是外界对它的引用都删除了，所以它是不可达的，所以会被垃圾回收器清除。

![](http://image.oldzhou.cn/FqHC1eNh4qu41wAPOYs83NPupkYD)

也有可能整个对象变的**不可访问**从而被从内存中删除，例如我们切断`family`的引用`family = null`，那么：

![](http://image.oldzhou.cn/Fke64r2hmdweWxit7WNwWViER5GA)

虽然`{name: John}`和`{name: 'Ann'}`仍然相互引用，但是`family`已经从根上断开了连接，不再有对它们的外界的引用（也就是说无法从局部或者全局变量访问到它们），所以下面整个块都是不可达的，整体都会被删除。

## 垃圾回收的算法

### 标记-清除算法

垃圾回收的基本算法是“**标记-清除**”，具体步骤如下，会被定期执行：

（1）垃圾回收器获取「根」并标记

![](http://image.oldzhou.cn/FrsIkjttxzhTT7okBmvy6UclWszp)

（2）继续访问并标记所有来自「根」的引用，所有标记过引用都不会被重复访问：

![](http://image.oldzhou.cn/FiYEOnhIOeAfMPG7lkH-Sn3c44P0)

（3）继续标记引用的后代引用，直到所有的“可达”（可以从根访问的）的引用全部被标记为止

![](http://image.oldzhou.cn/FiYEOnhIOeAfMPG7lkH-Sn3c44P0)

（4）删除所有未标记的对象，回收它们占用的内存：

![](http://image.oldzhou.cn/FqXP6v-Pfzy6DptZm5q6LrYcCrVm)

这就是JavaScript垃圾回收期的工作原理，JavaScript引擎应用了许多优化，使其运行的更快，并且不影响执行，例如：

（1）分代回收

对象分为两组“老对象”和“新对象”，许多新对象出现，完成任务后迅速被解除应用，它们很快被清理。而那些存活时间足够久的对象就会认为是“老对象”，很少接受检查

（2）增量回收

如果试图一次性遍历并标记整个对象集，那么花费时间会很长，因此引擎会试图将垃圾回收分解为多个部分，然后各个部分分别执行。这需要额外的标记来跟踪变化

（3）空闲时间收集

垃圾回收期只在CPU空闲时运行，以减少对正常任务的影响


除了“标记-清除”算法之外，还有其他的多种算法，比如“引用计数”算法，记录每个对象被引用的次数，每次新建对象、赋值引用和删除引用的同时更新计数器，如果计数器值为0则直接清除回收内存。[这篇文章](https://www.jianshu.com/p/a8a04fd00c3c)介绍了多种算法，并分析了优缺点，写的很好。

### 引用计数算法


引用计数算法是最初级的垃圾收集算法，此算法把“对象是否不再需要”简化定义为“有没有其他对象引用到它”。如果没有引用指向该对象（零引用），对象将被垃圾回收器回收。


```JS
var o = { 
  a: {
    b:2
  }
}; 
// 两个对象被创建，一个作为另一个的属性被引用，另一个被分配给变量o
// 很显然，没有一个可以被垃圾收集

var o2 = o; // o2变量是第二个对“这个对象”的引用

o = 1;      // 现在，“这个对象”的原始引用o被o2替换了

var oa = o2.a; // 引用“这个对象”的a属性
               // 现在，“这个对象”有两个引用了，一个是o2，一个是oa

o2 = "yo"; // 最初的对象现在已经是零引用了
           // 他可以被垃圾回收了
           // 然而它的属性a的对象还在被oa引用，所以还不能回收

oa = null; // a属性的那个对象现在也是零引用了
           // 它可以被垃圾回收了
```

这种算法没有办法处理循环引用的情况，例如下面的例子，两个对象被创建，并相互引用，形成循环。它们被调用后不再被外部引用，没用了，可以被回收了。但是引用计数算法认为它们都还至少有一次引用，所以不会被回收，一直保存在内存中。

```JS
function f(){
  var o = {};
  var o2 = {};
  o.a = o2; // o 引用 o2
  o2.a = o; // o2 引用 o

  return "azerty";
}

f();
```

## 经验法则

为了使Chrome的垃圾回收器不保留不再需要的对象，有几点需要牢记：

1. 在恰当的作用域中使用变量，尽量在函数作用域中声明变量，尽量声明局部变量，尽量避免全局变量
2. 确保移除不再需要的事件监听器，比如即将被移除的DOM对象所绑定的事件
3. 避免缓存大量不会被重用的数据
4. 少用闭包
5. 不要再生产环境使用`console.log()`、`console.error()`、`console.dir()`等方法打印任何复杂对象，因为这些对象不会被垃圾回收器回收。


## 参考

- [前端面试：谈谈 JS 垃圾回收机制@segmentfault](https://segmentfault.com/a/1190000018605776)
- [chrome v8引擎以及垃圾回收机制（原创）@掘金](https://juejin.im/post/5abb637f5188255c620f23ac)
- [几种垃圾回收算法@简书](https://www.jianshu.com/p/a8a04fd00c3c)
- [解读生产环境为何避免使用console.log@segmentfault](https://segmentfault.com/a/1190000012295395)
