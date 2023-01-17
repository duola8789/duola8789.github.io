---
title: Vue提高20 日期选择器插件
top: false
date: 2019-06-18 17:01:06
updated: 2019-06-18 17:01:09
tags:
- 日历
- blur
- focus
categories: Vue
---

以前收藏了一篇自己动手实现日期选择器的插件，最近没什么事，就想着仿照ElementUI的DatePicker，自己也写了一个简易的日期选择器，本以为不会很麻烦，实际动手才发现有很多问题需要解决。并且在写完之后，才发现可扩展性很差，距离ElementUI的水平差的很远，下一步就是把ElementUI的源码学习一下，看清楚自己的差距。

<!-- more -->

以前收藏了一篇自己动手实现日期选择器的插件，最近没什么事，就想着仿照ElementUI的DatePicker，自己也写了一个简易的日期选择器，本以为不会很麻烦，实际动手才发现有很多问题需要解决。并且在写完之后，才发现可扩展性很差，距离ElementUI的水平差的很远，下一步就是把ElementUI的源码学习一下，看清楚自己的差距。

## 结构

我将这个日期选择做成了Vue的插件的形式，有三个文件：

```
- index.js
- MyDate.js
- MyDatePicker.vue
```

`index.js`很简单，只有一个方法`intstall`，在`install`里面注册了全局组件：

```JS
import MyDatePicker from './MyDatePicker'

export default {
  install(Vue) {
    Vue.component('MyDatePicker', MyDatePicker)
  }
}
```

`MyDatePicker.vue`是UI部分，在这里定义样式和交互事件，我将数据单独放到了`MyDate.js`中，以Class形式导出

## 数据部分

我的顺序是先完成`MyDate.js`的数据部分，一个日期选择器基本结构如下：

![](http://image.oldzhou.cn/FuoucVvkQhblrJ08jomCI3g-Vk8A)

最后导出一个二维数组，二维数组外层包含6个数组，对应日期选择器中的每一行的数据，内层数组又包含7个对象元素，对应周一到周日，这样相当于总共有42个元素，正好对应面板中的42个日期。

当选择一个日期后，首先通过`new Date`构造函数获得当前日期所在月的第一天及这一天是星期几

```JS
// 当前选择日期所在月的第一天及这一天是星期几
const firstDayOfCurrentMonth = new Date(this.current.year, this.current.month - 1);
const firstDayOfWeek = firstDayOfCurrentMonth.getDay();
```

要注意的是，`new Date().getMonth()`的范围是`[0, 11]`，和我们日常使用的月份是少`1`的。而我在`current`里面存的日期是为了显示所用的，已经加过`1`了，所以需要在上面减`1`

接下来，通过两层的遍历来生成我们所需要的二维数组，外层遍历是对应的是行数据：

```JS
for (let row = 0; row < 6; row++) {
}
```

关键是内层数据，假设我们选择的就是2019年6月，6月1日是星期六，在二维数组的内部数组里面的七个元素，应该吻别对应`['日', '一', '二', '三', '四', '五', '六']`，现在6月1日星期六，它位于数组的第七项，补齐这个数组的结果应该是：

```JS
[5.26, 5.27, 5.28, 5.29, 5.30, 5.31, 6.1]
```

JS的`Date`构造函数会自动对超出当前月份的日期进行转换，意思就是，当我们构造一个日期`new Date(2019, 5, 0)`，它会自动往前倒一天，生成的日期就是`2019-05-31`

所以上面的数组转为对应的以6月为天数就是：

```JS
[-5, -4, -3, -2, -1, 0, 1]
```

所以当前遍历的范围就是`[-5, 1]`，起始点与6月1日的星期几存在这样的关系：

```JS
// 内层遍历起始点
let weekLoopStart = -firstDayOfWeek + 1; // -5
```
结束点是`7 + weekLoopStart - 1`

这样当内层遍历结束一次时，将`weekLoopStart`加`7`，就可以生成新的一行数据了：

```JS
[2, 3 4, 5, 6, 7, 8]
```

所以两层遍历的基本形式就是：

```JS
// 行数据
for (let row = 0; row < 6; row++) {
  const rowDate = [];
  // 列数据
  for (let weekDay = weekLoopStart; weekDay <= 7 + weekLoopStart - 1; weekDay++) {
    // 生成需要的对象
  }
  weekLoopStart += 7;
  this.dates.push(rowDate)
}
```

> 有点绕，而且可扩展性也不是很好，还是太笨了。

在内层遍历生成的对象有这样几个属性：

```JS
const targetDate = new Date(this.current.year, this.current.month - 1, weekDay);
const day = targetDate.getDate();
const month = targetDate.getMonth() + 1;
rowDate.push({
  date: targetDate,
  value: format(targetDate),
  label: day,
  key: weekDay,
  isCurrentMonth: month === this.current.month,
  isToday: +targetDate === +this.today
})
```
`date`是标准的日期对象，`value`是选择后用于展示的格式化的日期，`label`是在日历中选择的日期，`key`是整个遍历过程中它实际的标号，`isCurrent`用来判断这个日期是否属于当前月份，还是以`-5`这样的格式转换为前一个月的日期（这样的日期在面板上是有不同的样式），`isToday`用来标识今天的日期：

这样就生成了一个二维数组放到了`this.dates`这个实例属性中

当改变选择的月份，面板的日期也会随着变化，对应的实例方法就是`changeDate`，因为刚才的生成数据的`getDateArray`方法都是依赖于`this.current`来进行的，所以只需要改变`this.current`的值，然后重新执行`getDateArray`方法就行了

```JS
// 改变日期
changeDate(date = new Date()) {
  this.current = {
    year: date.getFullYear(),
    month: date.getMonth() + 1
  };
  this.getDateArray()
}
```

这样基本的数据就完成了。

## UI部分

UI部分是在`.vue`的单文件组件完成的，面板使用了`<table>`标签，在`date`里面引入`MyDate`的实例，其余都声明为计算属性，与`MaDate`的实例相关联，这样形成了这样的变化过程：

```
graph LR
用户点击-->改变实例属性
改变实例属性-->计算属性改
计算属性改-->UI界面改变
```

<!--![](http://image.oldzhou.cn/FilGniQyaW5gPHwif42QN_n9Xzcr)-->


有三个事情需要记录一下

（1）设定单元格样式

```HTML
<tr v-for="(row, rowIndex) in tbody" :key="'row-'  + rowIndex">
  <td v-for="cell in row" :key="cell.key" @mousedown="selectDate(cell)">
    <span :class="tableCellClass(cell)">{{cell.label}}</span>
  </td>
</tr>
```
因为单元格的原始和遍历的数据`cell`有关系，如果卸载模板中会有一大堆的代码，不太直观，用计算属性生成一个对象有没有办法传入参数，所以可以用一个`method`，返回一个对象传给`:class`

```JS
// 设定日期单元格样式
tableCellClass(cell) {
  return {
    'not-current-month': !cell.isCurrentMonth,
    today: cell.isToday,
    selected: cell.value === this.selectedDate
  }
},
```

（2）动画效果

ElementUI的动画效果是向上滑出

![](http://image.oldzhou.cn/FnOKLVk4IDcKGj2UVJUlOAG4GOq4)

它是通过Vue的`<transition>`组件实现的，而`<transition>`是用JS实现的动画，使用了`requestAnimationFrame`的API，很流畅，而且便于复用。找个时间还是要好好看一些Vue的源码，学习一下。

我使用了CSS动画来实现，当选择框出现时，添加一个类`container-visible`，将原本的`height`由`0`改为`320px`，同时将`opacity`由`0`改为`1`，同时添加了`will-change`和`transform: translateZ(0)`来提升性能：


```CSS
.date-container {
  position: absolute;
  left: 0;
  top: 46px;
  color: #606266;
  box-shadow: 0 2px 12px 0 rgba(0, 0, 0, .1);
  background: #fff;
  border-radius: 4px;
  line-height: 30px;
  margin: 5px 0;
  transition: all 0.5s ease;
  border: 1px solid #e4e7ed;
  height: 0;
  overflow: hidden;
  will-change: height;
  opacity: 0;
  transform: translateZ(0);
}
.container-visible {
  height: 320px;
  opacity: 1;
}
```
实现的效果还可以，但是有两个问题，一是不太好复用，而是需要改为固定的高度，如果面板高度变化，效果就有可能有偏差

![](http://image.oldzhou.cn/FrKnNSWuQ9sDA6dqz4MBp18bjlZN)

（3）第三个问题是日期选择框的出现和隐藏，它具体的逻辑如下：

1. 点击输入框，出现选择框
2. 点击输入框和选择框之外的部分，选择框消失
3. 点击输入框和选择框之内的部分，选择框不消失
4. 点击选择框的快速选择月份（那几个小箭头），选择框发生相应改变，不消失
5. 点击选择框的具体日期，选择框消失，选择成功

我选用的方案是使用`<input>`的`focus`和`blur`事件，发生两个事件时，改变控制选择框是否显示的变量`containerVisible`

`focus`没有问题，但是`blur`有着比较大的问题，首先遇到的问题时，当点击选择框的按钮功能和时期时，没有触发对应的功能，选择框就消失了（以前在开发业务的时候遇到过类似的问题），这主要是因为`blur`事件发生的时机：

```
graph LR
mousedown-->blur
blur-->mouseup
mouseup-->${click}
```

<!--![](http://image.oldzhou.cn/FnYyZc4DreR9UnprWxPp4zEkQbl8)-->

在`click`事件发生之前`blur`事件就发生了，导致`click`事件没有发生时，元素就隐藏了，`click`事件无效。

所以像以前一样，将选择框绑定的`click`事件改为了`mousedown`事件，这样做的效果是，点击日期能够选择了，并且选择事件执行了，并且之后选择框失效了，这时候上面的五条逻辑满足了1/2/5，但是3/4又出问题了，点击选择框的小按钮，选择框意外消失了。

之所以这样，是因为`mousedown`事件之后，`blur`事件执行，导致选择框小事，我们要做的是在`mousedown`之后，不触发`blur`事件，所以应该使用`peventDefault`方法（注意不是`stopImmdeiation`，因为不是冒泡导致的），Vue中提供的修饰符是`prevent`，所以在所有的`mousedown`事件后面添加上修饰符`prevnet`：

```HTML
<button type="button" class="btn next-month-btn" @mousedown="changeMonth(1)">
 <span class="iconfont icon-el-icon-arrow-right"></span>
</button>
```

这样条件4满足了，但是3不行，所以需要在整个选择框的容器上添加一个`mousedonw`事件，并且使用`prevnet`修饰符，里面的点击事件只需要使用`mousedown`就可以了

```HTML
<div class="date-container" :class="{'container-visible': containerVisible}" @mousedown.prevent>
</div>
```
这样基本上就成功了，但是还是有一些小瑕疵，一个问题就是`blur`事件发生的过于容易，比如我点击浏览器之外的桌面部分，`blur`事件也会发生，选择框会消失，而ElementUI的并不会消失，还有就是绑定了没有必要的点击事件，不好复用，并且不知道如果同时有多个弹出框的时候还不会有其他的问题。

ElementUI是把这块单独提出了一个方法，位于[element/src/utils/clickoutside.js](https://github.com/ElemeFE/element/blob/dev/src/utils/clickoutside.js)，它对这种情况的点击事件做了统一的处理，主要的思路就是在`document`绑定了统一的点击事件，通过收集此刻的弹窗元素到一个队列中，隐藏这个队列中的元素，它没有使用`blur`事件，更可控，也更适合更多的元素。

## 优化

这个日期选择器插件的基本功能能够满足，但是如果作为ElementUI那样的轮子，还差的很多，扩展非常困难（快速选择月、年的面板我就没有做）

下一步的计划就是首先学习`clickoutside`的实现，然后学习ElementUI的源码，这个计划也好久了，要尽快执行啊~

最后，[代码都在这里](https://github.com/duola8789/vue-cli-learning/tree/master/src/plugin/myDatePicker)。
