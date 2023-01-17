---
title: JS语言理解01 Javascript中的封装
top: false
date: 2017-02-24 09:27:00
updated: 2019-07-05 16:31:00
tags: 
- 封装
- 面向对象
categories: JavaScript
---

JavaScript中的封装。

<!-- more -->

### 1 封装
#### 1.1 私有变量和私有函数
JavaScript的函数作用域，在函数内定义的变量和函数如果不对外提供接口，那么外部将无法访问到，也就是变为私有变量和私有函数。

```
function animal(name){
    var type="cat",//私有变量
        say=function(){alert("wawawa")};//私有函数
}
alert(type);//undefined
```
在函数对象`animal`外部无法访问变量`type`和函数`say`，它们就变成私有的，只能在`animal`内部使用，即使是函数`animal`的实例仍然无法访问这些变量和函数.

```
var dog=new animal("seven");
alert(dog.type);//undefined
```
#### 1.2 构造函数模式

为了解决从原型对象生成实例的问题，Javascript提供了一个构造函数（`Constructor`）模式。

所谓"构造函数"，其实就是一个普通函数，但是内部使用了`this`变量。对构造函数使用`new`运算符，就能生成实例，并且`this`变量会绑定在实例对象上。


```
function animal(name,age){
    this.name=name;
    this.age=age;
}
var cat=new animal("tom",23);
var dog=new animal("jerry",24);
console.log(cat.name);//tom
console.log(dog.age);//24
```
cat和dog会自动含有一个`constructor`属性，指向它们的构造函数。

```
console.log(cat.constructor===animal);//true
```

![image](http://note.youdao.com/yws/public/resource/676feab4918c01e107b827b44bd71894/xmlnote/B9D80B44320C42F5A5FBD1C509D9AAC4/15204)

Javascript还提供了一个`instanceof`运算符，验证原型对象与实例对象之间的关系。
```
console.log(cat instanceof animal);//true
```
> 注意，`instanceof`是一个操作符，不是一个方法，所以用法是`a instanceof b`

#### 1.3 构造函数的问题

```
function animal(name,age){
    this.name=name;
    this.age=age;
    this.type="animal";
    this.say=function(){alert("hello")}
}
var cat=new animal("tom",23);
var dog=new animal("jerry",24);
console.log(cat.type);//animal
dog.say();//hello
```
表面上好像没什么问题，但是实际上这样做，有一个很大的弊端。那就是对于每一个实例对象，`type`属性和`say()`方法都是一模一样的内容，每一次生成一个实例，都必须为重复的内容，多占用一些内存。这样既不环保，也缺乏效率。

```
console.log(cat.say===dog.say);//false
```
利用原型（`Prototype`）模式就可以让`type`属性和`say()`方法在内存中只生成一次，然后所有实例都指向那个内存地址呢从而提升性能。

#### 1.4 原型Prototype
无论什么时候，只要创建了一个新函数，该函数就会有一个`prototype`属性，这个属性也是一个对象，默认情况下`prototype`属性会默认获得一个`constructor`(构造函数)属性，这个属性是一个指向`prototype`属性所在函数的指针（即函数自己）。

```
function animal(name,age){
    this.name=name;
    this.age=age;
}
animal.prototype.type=animal;
animal.prototype.say=function(){
    alert("hello")
};
var cat=new animal("tom",23);
var dog=new animal("jerry",24);
console.log(cat.__proto__.constructor.name);//animal
console.log(animal.prototype.constructor.name);//animal

```
构造函数的`prototype`的`constructor`指向构造函数本身，而实例的将包含一个`__proto__`指针，指向构造函数的`prototype`。

![image](http://note.youdao.com/yws/public/resource/7a4cc97141934a9d2f7fae9ccbc1f5d8/xmlnote/E60CB3AC295C45F6A4A49B906EA11A35/23139)

> 注意，实例是没有`prototype`属性的，通过`new`关键字声明的实例获得的是指向其构造函数的`prototype`的`__proto__`指针。

![image](http://note.youdao.com/yws/public/resource/676feab4918c01e107b827b44bd71894/xmlnote/4A11D36FCDAB4829985CFA77FA601B2B/15262)

每一个函数被创建时，都会自动的拥有`prototype`属性。这个属性并无什么特别之处，它和其他的属性一样可以访问，可以赋值。不过当我们用 `new`关键字来创建一个对象的时候，`prototype`就起作用了：它的值（也是一个对象）所包含的所有属性，都会被关联到新创建的实例对象的`__proto__`指针上。

#### 1.5 利用原型Prototype创造实例

每一个构造函数都有一个`prototype`属性，指向另一个对象。这个对象的所有属性和方法，都会被构造函数的实例继承。

这意味着，我们可以把那些**不变的**属性和方法，直接定义在`prototype`对象上。

就像上面的例子一样，此时所有实例的`type`和`say`方法都是指向同一个内存地址，提高了运行效率。


```
console.log(cat.say===dog.say);///true
```
#### 1.6 Prototype模式的验证方法

为了配合prototype属性，Javascript定义了一些辅助方法，帮助我们使用它。

##### 1.6.1 `isPrototypeOf()`

这个方法用来判断，某个`proptotype`对象和某个实例之间的关系。


```
console.log(animal.prototype.isPrototypeOf(cat));//true
console.log(animal.prototype.isPrototypeOf(dog));//true
console.log(animal.prototype.isPrototypeOf(animal));//false
```

##### 1.6.2 `hasOwnProperty()`

每个实例对象都有一个`hasOwnProperty()`方法，用来判断某一个属性到底是本地属性，还是继承自prototype对象的属性

```

console.log(cat.hasOwnProperty("name"));//true
console.log(cat.hasOwnProperty("say"));//false

```
##### 1.6.3 `in`运算符
`in`运算符可以用来判断，某个实例是否含有某个属性，不管是不是本地属性.

```
console.log(cat.hasOwnProperty("say"));//false
console.log("say" in cat);//true
```
`in`运算符还可以用来遍历某个对象的所有属性.

```
for(var i in cat){
    var str="";
    str="cat["+i+"]:"+cat[i];
    console.log(str)
}
/*运行结果
cat[name]:tom
test11.js:29 cat[age]:23
test11.js:29 cat[say]:function (){alert("hello")}*/
```
#### 1.7 注意事项

1 只能把不变的属性和方法，绑在`prototype`对象上，而不能把可变属性绑上去。

2 用`new`的这种方法构造对象，如果忘记加上`new`，“既没有编译时警告，也没有运行时警告”，所以为了避免忘记加 `new`，最好的办法是类名首字母大写。

#### 参考
- http://www.ruanyifeng.com/blog/2010/05/object-oriented_javascript_encapsulation.html
- http://www.jb51.net/article/40964.htm
