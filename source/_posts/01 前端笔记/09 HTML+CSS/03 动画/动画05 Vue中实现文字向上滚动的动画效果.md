---
title: 动画05 Vue中实现文字向上滚动的动画效果
top: false
date: 2018-06-27 17:54:10
updated: 2019-07-13 19:32:00
tags:
- CSS
- 动画
- 文字滚动
categories: HTML+CSS
---

在Vue中实现文字向上滚动的效果。

<!-- more -->

在Vue中，想要实现文字向上滚动的效果，分成两种情况：

## 1 无缝滚动

无缝滚动如图：

![文字无缝滚动](https://img-blog.csdn.net/20180627180219320?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2R1b2xhODc4OQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

我说的无缝滚动主要是指两点：
1. 滚动中没有停顿
2. 从头至尾再循环播放时没有停顿

实现这种情况可以使用CSS3的动画实现，主要原理就是让多个文字的容器进行移动：

模板：

```HTML
<div class="inner-container">
  <p class="text" v-for="(text, index) in arr" :key="index">{{text}}</p>
</div>
```
CSS部分：

```CSS
.inner-container {
  animation: myMove 5s linear infinite;
  animation-fill-mode: forwards;
}
  /*文字无缝滚动*/
@keyframes myMove {
  0% {
    transform: translateY(0);
  }
  100% {
    transform: translateY(-150px);
  }
}
```
滚动中没有停顿是通过`keyframes`实现的，而循环在播放时没有“跳帧”的效果，只要是通过增加一条多余数据：

```JS
data() {
  return {
    arr: [
    '1 不是被郭德纲发现的，也不是一开始就收为徒弟。',
    '2 现在雅阁这个状态像极了新A4L上市那段日子。',
    '3 低配太寒碜，各种需要加装，中配定价过高，又没啥特色',
    '4 然后各种机油门、经销商造反什么的幺蛾子。',
    '5 看五月销量，建议参考A4，打8折吧。', 
    '1 不是被郭德纲发现的，也不是一开始就收为徒弟。', 
    ],
  }
},
```

当动画结束时通过：

```CSS
animation-fill-mode: forwards;
```

将动画重置为第一帧，这样就能够实现无缝的播放了

## 2 停顿滚动

实现停顿真的是在第一行文字滚动出现后，停顿一下，在进行滚动移出：

![文字停滚动](https://img-blog.csdn.net/20180627181104983?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2R1b2xhODc4OQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

实现这种效果有两个思路，第一个思路是沿用上面的方法，用包裹容器的移动来实现，只不过在`keyframes`总要精细化，留出停顿的时间：

```CSS
/*文字停顿滚动*/
@keyframes myMove2 {
  0% {
    transform: translateY(0);
  }
  10% {
    transform: translateY(-30px);
  }
  20% {
    transform: translateY(-30px);
  }
  30% {
    transform: translateY(-60px);
  }
  40% {
    transform: translateY(-60px);
  }
  50% {
    transform: translateY(-90px);
  }
  60% {
    transform: translateY(-90px);
  }
  70% {
    transform: translateY(-120px);
  }
  80% {
    transform: translateY(-120px);
  }
  90% {
    transform: translateY(-150px);
  }
  100% {
    transform: translateY(-150px);
  }
}
```
不过看起来好麻烦，而且如果文字行数过多的话，岂不是吐血，这时候可以使用LESS或者SASS提供的函数功能帮助你完成`keyframes`，具体就不在这里展开了

第二种思路就是不用列表容器的移动，而是仅保留一个`<p>`元素，不断变化其内容，在变化过程中实现移动的效果。

这种思路要借用到Vue提供的`transition`组件了，它提供了一些类名，比如`v-enter`、`v-enter-active`等来对应CSS的类，同时在对应我们这种情况的多个元素的过度时，还需要使用过渡模式`out-in`，具体可以参考[官方文档](https://cn.vuejs.org/v2/guide/transitions.html#%E6%A6%82%E8%BF%B0)。

代码如下：

模板:
```HTML
<div class="text-container">
  <transition class="inner-container2" name="slide" mode="out-in">
    <p class="text2" :key="text.id">{{text.val}}</p>
  </transition>
</div>
```
多了一个计算属性

```JS
computed: {
  text() {
    return {
      id: this.number,
      val: this.arr[this.number]
    }
  }
},
```
再建立一个定时器，在定时器中不断改变计算属性对应的值：

```JS
startMove() {
  let timer = setTimeout(() = > {
    if (this.number === 5) {
      this.number = 0;
    } else {
      this.number += 1;
    }
    this.startMove();
  }, totalDuration)
},
```
同时在CSS部分添加对应的类：

```CSS
.slide-enter-active, .slide-leave-active {
  transition: all 0.5s linear;
}
.slide-leave-to {
  transform: translateY(-20px);
}
.slide-enter {
  transform: translateY(20px);
}
```
这样就OK了，demo在[这里](https://github.com/duola8789/vue-cli-learning)。

## 使用列表过渡实现

博客下面，有人评论，可以使用列表过渡加定时器实现，所以就试了一下，思路是利用定时器，不断改变列表元素的位置，将最后一个元素查到列表的开始。可以实现，但是效果不如上面的完美。

```HTML
<template>
  <div>
    <h2>列表过渡</h2>
    <div class="text-container">
      <transition-group tg="div" name="list" class="list-container" mode="out-in">
        <p v-for="(text, index) in arr2" :key="text + index" class="list-item">{{text}}</p>
      </transition-group>
    </div>
  </div>
</template>

<script>
  const totalDuration = 2000;
  export default {
    name: 'demo11',
    data() {
      return {
        arr: ['1 不是被郭德纲发现的，也不是一开始就收为徒弟。', '2 现在雅阁这个状态像极了新A4L上市那段日子。', '3 低配太寒碜，各种需要加装，中配定价过高，又没啥特色', '4 然后各种机油门、经销商造反什么的幺蛾子。', '5 看五月销量，建议参考A4，打8折吧。', '1 不是被郭德纲发现的，也不是一开始就收为徒弟。', ],
        number2: -1,
      }
    },
    mounted() {
      this.startMove2()
    },
    methods: {
      startMove2() {
        let timer = setTimeout(() = > {
          this.number2 += 1;
          if (this.number2 > 4) {
            const target = this.arr2.splice(4, 1);
            this.arr2.unshift(target[0])
          } else {
            this.arr2.unshift(this.arr[this.number2]);
          }
          this.startMove2();
        }, totalDuration)
      },
    },
  }
</script>

<style scoped>
.list-container {
  position: relative;
  overflow: hidden;
}

.list-item {
  margin: 0;
  transition: all 1s;
  overflow: hidden;
}


.list-enter {
  transform: translateY(30px);
}

.list-enter-to, .list-leave {
  transform: translateY(0);
}

.list-leave-to {
  transform: translateY(-30px)
}

.list-leave-active {
  position: absolute;
  width: 0;
}
</style>
```

因为列表过渡，新的元素插入时可以平滑度过，但是元素移除时如果要实现平滑过渡需要在`v-leave-active`的样式中将定位设定为绝对定位

```CSS
.list-leave-active {
  position: absolute;
}
```

在我当前的需求下，如果设置了绝对定位，元素在移除时会重叠显示多个元素

![](http://image.oldzhou.cn/FnJcyDuZ_M4GyHI8qTuT08eq-CSA)

所以在这个阶段加了一个属性，让宽度为0，不显示过渡期间重叠的元素：

```CSS
.list-leave-active {
  position: absolute;
  width: 0;
}
```

这样重叠的问题解决了，但是元素在移除时的效果就很生硬：

![](http://image.oldzhou.cn/FubWawIvbfz19Fop3Aheu26QKLqS)

可能是我的设置不太对，也许直接将没有过渡期间的列表项的定位就设为绝对定位，通过钩子改变时直接改变绝对定位的`top`值也许能解决这个问题，如果有人尝试成功还望赐教。

## 参考
- [纯css3实现文字间歇滚动效果@小目标](https://www.cnblogs.com/happypayne/p/7534955.html)
- [纯css实现的无缝滚动@知乎](https://zhuanlan.zhihu.com/p/22968728)
- [列表过渡@Vue](https://cn.vuejs.org/v2/guide/transitions.html#%E5%88%97%E8%A1%A8%E8%BF%87%E6%B8%A1)
