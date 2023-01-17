---
title: JS63 IntersectionObserver API
top: false
date: 2019-10-31 17:38:00
updated: 2019-10-31 17:38:02
tags:
- Intersection Observer
categories: JavaScript
---

Intersection Observer API 学习笔记。

<!-- more -->

## 元素可见性

页面的可见性可以用`document.visibilityState`或者`document.hidden`获得，通过`document.visibilitychange`来监听页面可见性的变化，但是对于页面的元素的可见性却只能手动通过位置判断。

例如下面这个例子

![](http://image.oldzhou.cn/FrGN3X6jNkU2feE0YHO7jG2LNl0m)

需要监听`scroll`事件，根据`target`元素的是否出现，来更改顶部的文字和样式。由于`target`内嵌了几层，所以判断时需要通过`getBoundingClientRect`方法获得其位置，与视口位置比较，比较的标的还需要根据其父元素的位置变化，实现起来还是有一点麻烦的：

```HTML
<div class="container">
  <header class="head" id="head"></header>
  <main class="main" id="main">
    <div class="content">
      <div class="target-container" id="targetContainer">
        <div class="target" id="target">
        </div>
      </div>
    </div>
  </main>
  <footer class="foot" id="foot"></footer>
</div>

<script>
window.onload = () => {
  const head = document.querySelector('#head'),
    foot = document.querySelector('#foot'),
    main = document.querySelector('#main'),
    targetContainer = document.querySelector('#targetContainer'),
    target = document.querySelector('#target');

  const footTop = foot.getBoundingClientRect().top;

  // 改变 head 的文字和样式
  const changeHeadState = isVisible => {
    head.textContent = isVisible ? 'Visible' : 'Invisible';
    head.setAttribute('style', `background: ${isVisible ? 'lightgreen' : 'red'}`)
  };

  const watchPosition = () => {
    const targetContainerRect = targetContainer.getBoundingClientRect(),
      targetContainerBottom = targetContainerRect.top + targetContainerRect.height;

    const targetTop = target.getBoundingClientRect().top;

    let isVisible = false;

    // targetContainer 完全可见
    if (targetContainerBottom < footTop) {
      isVisible = targetTop < targetContainerBottom;
    } else {
      isVisible = targetTop < footTop;
    }

    return isVisible
  };

  changeHeadState(watchPosition());

  main.addEventListener('scroll', () => {
    changeHeadState(watchPosition());
  });

  targetContainer.addEventListener('scroll', () => {
    changeHeadState(watchPosition());
  });
}
</script>
```

## IntersectionObserver API

现在有了一个新的API来帮助我们简化这个工作，那就是IntersectionObserver API，它的兼容性如下：

![](http://image.oldzhou.cn/FvkBv02JO0aOnAk4QEsxarjMBHmd)


使用的时候首先需要新建一个实例：

```JS
const io = new IntersectionObserver(callback, option);
```

其中`callback`是监听元素的可见性发生变化时的回调函数，它会在元素进入视口（开始可见）或者完全离开视口（开始不可见）时被触发，它的参数是一个数组，每一项都是一个监听对象对应的`IntersectionObserverEntry`对象，它主要包含以下属性：

- `time`，可见性发生变化的时间，是一个高精度时间戳，单位为毫秒
- `target`，被观察的目标元素，DOM节点
- `rootBounds`，根元素的`getBoundingClientRect()`的返回值
- `boundingClientRect`，目标元素的`getBoundingClientRect()`的返回值
- `intersectionRect`，目标元素与视口（或根元素）交叉区域`getBoundingClientRect()`的返回值
- `intersectionRatio`，目标元素的可见比例，即`intersectionRect`占`boundingClientRect`的比例，完全可见时为1，完全不可见时小于等于0

在Chrome 78版本中会返回`isVisible`属性，但是不知道是不是Bug，无论元素是否可见，都为`false`，但是`isTntersecting`的表现是正常的，所以判断是否可见，可以根据`intersectionRatio`或者`isTntersecting`来进行判断

构造函数的第二个参数`option`是一个配置对象，可以配置的属性包括`root`、`rootMargin`、`threshold`，具体配置方法[参考文档](https://developer.mozilla.org/zh-CN/docs/Web/API/IntersectionObserver/IntersectionObserver)。


通过构造函数新建了一个观察器实例，实例的`observe`方法可以观察传入的DOM节点：

```JS
// 开始观察
io.observe(document.getElementById('example'));

// 停止观察
io.unobserve(element);

// 关闭观察器
io.disconnect();
```

如果要观察多个对象，就需要多次调用`observe`方法。

注意，IntersectionObserver API是异步的，不随着目标元素的滚动同步触发。IntersectionObserver的实现，应该采用`requestIdleCallback()`，即只有线程空闲下来，才会执行观察器。这意味着，这个观察器的优先级非常低，只在其他任务执行完，浏览器有了空闲才会执行。

## Demo

可以使用IntersectionObserver API重写上面的例子，简化判断元素可见的逻辑：

```JS
window.onload = () => {
  const head = document.querySelector('#head'),
    target = document.querySelector('#target');

  // 改变 head 的文字和样式
  const changeHeadState = isVisible => {
    head.textContent = isVisible ? 'Visible' : 'Invisible';
    head.setAttribute('style', `background: ${isVisible ? 'lightgreen' : 'red'}`)
  };

  const io = new IntersectionObserver(([obj]) => {
    console.log(obj);
    const isVisible = obj.intersectionRatio > 0;
    changeHeadState(isVisible)
  });

  io.observe(target);
}
```

## 简单的无线滚动

可以使用这个API，实现一个简单的无线滚动的Demo

![](http://image.oldzhou.cn/FnADJXQlOUqNtv47AJWwuSssHf7k)

```HTML
<div class="container">
  <header class="head" id="head"></header>
  <main class="main" id="main">
    <ul class="content" id="content">
    </ul>
    <div class="target" id="target"></div>
  </main>
  <footer class="foot" id="foot"></footer>
</div>
<script>
window.onload = () => {
  const content = document.querySelector('#content'),
    head = document.querySelector('#head'),
    target = document.querySelector('#target');

  // 模拟网络请求
  const fetch = () => {
    return new Promise(resolve => {
      setTimeout(resolve, 2000, 10)
    })
  };

  // 改变 head 的文字和样式
  const getNewContent = (() => {
    // 标志服，避免重复请求
    let pending = false;
    const frag = document.createDocumentFragment();

    return isVisible => {
      if (!isVisible) {
        return;
      }

      if (pending) {
        return;
      }

      pending = true;
      head.textContent = 'Loading...';

      fetch().then(num => {
        for (let i = 0; i < num; i++) {
          const li = document.createElement('li');
          li.classList.add('item');
          frag.appendChild(li)
        }
        content.appendChild(frag);

        pending = false;
        head.textContent = 'Done!';
      })
    };
  })();

  const io = new IntersectionObserver(([obj]) => {
    const isVisible = obj.intersectionRatio > 0;
    getNewContent(isVisible)
  });

  io.observe(target);
}
</script>
```

在列表的末尾放置一个标志元素，当它可见的时候，就去触发网络请求，获取新的数据。

## 参考

- [IntersectionObserver API 使用教程@阮一峰的网络日志](http://www.ruanyifeng.com/blog/2016/11/intersectionobserver_api.html)
- [Intersection Observer@MDN](https://developer.mozilla.org/zh-CN/docs/Web/API/IntersectionObserver)
- [IntersectionObserverEntry@MDN](https://developer.mozilla.org/zh-CN/docs/Web/API/IntersectionObserverEntry)
