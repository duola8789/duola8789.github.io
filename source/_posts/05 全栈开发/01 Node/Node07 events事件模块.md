---
title: Node07 events事件模块
top: false
date: 2017-04-03 20:37:17
updated: 2019-04-30 11:05:32
tags:
- Events
categories: Node
---

重新学习Node，整理以前的日志。

Nodejs的大部分核心API都是基于异步事件驱动设计的，所有可以分发事件的对象都是EventEmitter类的实例。

<!-- more -->

## 如何使用

实例化events.EventEmitter这个类，然后就可以使用手册上的一些方法了。

```JS
var event = require("events")//引入事件模块
var emitter = new event.EventEmitter();//实例化事件触发器
```
实例化之后就可以添加EventEmitter的一些方法了。

```JS
emitter.on("事件名称",监听函数)；
emitter.emit("事件名称"，[参数1], [参数2]...)
```
on和emit是最基本的方法，on用来建立普通的事件监听（也就是监听器），当事件触发则执行监听函数；

emit用来触发某个事件，然后将参数按顺序传入监听函数。

下面是一个基本的例子，当emiiter的event1事件被触发后就执行了func1，如果定义了多个evnet1，则event1也会在触发后执行多次。

```JS
emitter.on("event1", function func1(msg1, msg2){
    console.log(msg1+"+"+msg2)
});
emitter.emit("event1","this is event1", "this is event2");
```


## 其他监听方法

```JS
emitter.addListener(event, listener) 
// 与on相同，都是建立监听器

emitter.once(event, listener)
// 建立一次性监听器，执行listener后删除本监听器

emitter.removeListener(event, listener)
// 从指定事件中除去某一个监听函数，也就是再触发这个事件，指定的监听函数不执行了

emitter.removeAllListeners(event)
// 删除所有对这个事件的监听函数，也就就是再触发这个事件，没函数执行了

emitter.setMaxListeners(n) 
// 设置这个emitter实例的最大事件监听数，默认是10个，设置0为不限制
// 如果设置了最大监听数量，则同一事件的监听最好不要超过该最大值，否则很可能发送内存泄漏。

emitter.listeners(event) 
// 返回这个事件的监听函数的数组，例如返回：[function1, function2, function3]
// 修改会改变此事件的监听函数，例如emitter.listeners(event).length = 2;就相当于执行了：emitter.removeListener(event, function3) ;
```
 看一个例子：
 
```JS
var event = require("events"),
    emitter = new event.EventEmitter();
var func1 = function(msg1, msg2){
    console.log(msg1+msg2)};
var func2 = function(){console.log("this is event2")};
var func3 = function(){console.log("this is event3")};

emitter.on("event1",func1);
emitter.addListener("event1", func2);
emitter.on("event1",func3);
console.log(emitter.listeners("event1"));
// [ [Function], [Function], [Function] ]

emitter.emit("event1","this is ", "event1");
// this is event1
// this is event2
// this is event3

emitter.removeListener("event1", func1);
console.log(emitter.listeners("event1"));
// [ [Function], [Function] ]

emitter.emit("event1","this is event1", "this is event2");
// this is event2
// this is event3

emitter.removeAllListeners("event1");
console.log(emitter.listeners("event1"));
// []
emitter.emit("event1","this is event1", "this is event2");
// nothing here
```

## 应用实例

比如有一个秒杀活动，只有第一个点击按钮的人才有机会获取奖品，网站会记录本次活动有多少人点击这个按钮，同时又要根据用户ID记录此用户点过多少次这个按钮。

不使用事件监听代码可能这么写：

```JS
var prize = false;                  //false表示奖品未被领取
var isactive = true;                //活动是否开启
if(有人点击抽奖按钮){
    if(!isactive ) 
    return false;                   //先判断活动是否已经关掉了
    if(!prize){
        getprize(此人id, 按钮id);   //getprize是领奖的方法
        prize = true;
    }
    addbutton(此人id, 按钮id);      //此按钮点击统计+1
    addclick(此人id, 按钮id);       //此人点击此按钮统计+1
}
setTimeout(function(){
    isactive = false; 
}, 1000*60*60*24);                  //活动时间1天，1天以后活动结束
```
上面这段代码对我自己也是有启发的：

奖品是否被领取、活动是否开启在这个代码的开始部分都用了布尔值作为标识符，这样在后面根据条件、状态的变化改变对应的标识符即可，然后将这些标识符作为if的判断条件。

如果用事件监听改写上面代码，看上去更清晰，而且可能效率更高一些。

```JS
var events =  require("events");
var btclick = new events.EventEmitter();
btclick.once('bt0click',getprize)
//getprize是领奖的方法，触发一次后，自动删除本触发器
btclick.on('bt0click',  addbutton);
//此按钮点击统计+1
btclick.on('bt0click', addclick);
//此人点击此按钮统计+1
if(有人点击抽奖按钮){
    btclick.emit('bt0click', 此人ID, 按钮ID);
    //等于触发了bt0click事件的所有监听器
}
setTimeout(function(){
    btclick.removeAllListeners('bt0click'); 
    //移除所有监听器
}, 1000*60*60*24); 
//活动时间1天，1天以后活动结束
```

## 参考
- http://snoopyxdy.blog.163.com/blog/static/6011744020118212553691/
