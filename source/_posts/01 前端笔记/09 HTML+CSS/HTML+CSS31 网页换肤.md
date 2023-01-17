---
title: HTML+CSS31 网页换肤
top: false
date: 2019-05-13 22:27:10
updated: 2019-05-13 22:27:16
tags:
- CSS
- link
categories: HTML+CSS
---

实现网页换肤的三种办法。

<!-- more -->

## 常规做法1

一个全局`class`来控制样式，切换皮肤时通过更改全局样式来实现换肤：

```HTML
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Title</title>
  <link href="../styles/base.css" rel="stylesheet" type="text/css">
  <style>
    /*默认样式*/
    .default  {
      color: #000;
      background: #ddd;
    }
    .default .text {
      border: 1px solid #000;
    }
    /*样式1*/
    .green {
      color: lemonchiffon;
      background: green;
    }
    .green .text {
      border: 1px solid darkblue;
    }
    /*样式2*/
    .red {
      color: navajowhite;
      background: red;
    }
    .red .text {
      border: 1px solid pink;
    }
  </style>
</head>
<body>
  <div class="container">
    <div id="skin-container" class="default">
      <p class="text">Hello!</p>
    </div>
  </div>
  <div class="input-container">
    <label class="input"><input type="radio" value="default" name="skin" checked></label>
    <label class="input"><input type="radio" value="green" name="skin"></label>
    <label class="input"><input type="radio" value="red" name="skin"></label>
  </div>
</body>
<script>
  const skinContainer = document.querySelector('#skin-container');
  const inputContainer = document.querySelector('.input-container');

  const changeSkin = theme => {
    const { classList } = skinContainer;
    const { value } = classList;
    classList.replace(value, theme)
  };

  inputContainer.addEventListener('change', (e) => {
    changeSkin(e.target.value)
  })
</script>
</html>
```

这种方法的缺点是，加入了一个全局的`class`控制样式，提高了样式优先级，如果换肤样式多，代码会非常罗嗦（Less可以简化一部分代码，但是维护还是不方便）

## 常规做法2

第二种做法是新建多个对应皮肤样式的CSS文件，通过改变`<Link>`元素的`href`属性来切换皮肤

```HTML
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Title</title>
  <link href="../styles/base.css" rel="stylesheet" type="text/css">
  <link id="skinLink" href="../styles/default.css" rel="stylesheet" type="text/css">
</head>
<body>
  <div class="container">
    <div id="skin-container" class="theme">
      <p class="text">Hello!</p>
    </div>
  </div>
  <div class="input-container">
    <label class="input"><input type="radio" value="default" name="skin" checked></label>
    <label class="input"><input type="radio" value="green" name="skin"></label>
    <label class="input"><input type="radio" value="red" name="skin"></label>
  </div>
</body>
<script>
  const skinLink = document.querySelector('#skinLink');
  const inputContainer = document.querySelector('.input-container');

  const changeSkin = theme => {
    skinLink.href = `../styles/${theme}.css`
  };

  inputContainer.addEventListener('change', (e) => {
    changeSkin(e.target.value)
  })
</script>
</html>

```
几个不同的CSS文件`default.css`、`red.css`、`green.css`中定义不同皮肤的样式

这种方法的缺点是使用JS改变`href`属性会带来加载延迟，样式切换不流畅（因为新切换的CSS文件原本没有加载，改变了`href`属性后才会加载，会有延迟）

## 第三种方法

可以借助`<link>`标签的`rel`属性的`alternate`属性值来实现：

```HTML
<link href="reset.css" rel="stylesheet" type="text/css">

<link href="default.css" rel="stylesheet" type="text/css" title="默认">

<link href="red.css" rel="alternate stylesheet" type="text/css" title="红色">
<link href="green.css" rel="alternate stylesheet" type="text/css" title="绿色">
```

上面有4个`<link>`元素，出现了三种不同性质的CSS文件加载：

1. 没有`title`属性，`rel`属性值仅仅是`stylesheet`的`<link>`元素（第一种），无论如何都会加载，可以利用它来加载`reset.css`
2. 有`title`属性，`rel`属性值仅仅是`stylesheet`的`<link>`元素（第二种），作为默认的CSS文件加载并渲染，如`default.css`
3. 有`title`属性，`rel`属性值同时包含`alternate stylesheet`的`<link>`元素（第三种和第四种），作为备选的CSS样式文件加载，但不渲染，通过控制其`disabled`属性来确定是否渲染，用来指定皮肤样式


```HTML
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Title</title>
  <link href="../styles/base.css" rel="stylesheet" type="text/css">
  <link id="theme-default" title="默认" href="../styles/default.css" rel="stylesheet" type="text/css">
  <link id="theme-green" title="绿色" href="../styles/green.css" rel="stylesheet alternate" type="text/css">
  <link id="theme-red" title="红色" href="../styles/red.css" rel="stylesheet alternate" type="text/css">
</head>
<body>
  <div class="container">
    <div id="skin-container" class="theme">
      <p class="text">Hello!</p>
    </div>
  </div>
  <div class="input-container">
    <label class="input"><input type="radio" value="default" name="skin" checked></label>
    <label class="input"><input type="radio" value="green" name="skin"></label>
    <label class="input"><input type="radio" value="red" name="skin"></label>
  </div>
</body>
<script>
  const inputContainer = document.querySelector('.input-container');
  const links = document.querySelectorAll('link[title]');

  const changeSkin = theme => {
    [...links].forEach(link => {
      link.disabled = true;
      link.disabled = link.id !==`theme-${theme}`;
    });
    console.log(links)
  };

  inputContainer.addEventListener('change', (e) => {
    changeSkin(e.target.value)
  })
</script>
</html>
```

要注意的是，如果具有alternate的<link>元素有两个及以上的`disbaled`都为`false`，那么哪个都不会加载。

这种方法的优点是：

1. 兼容性好
2. 语义非常好
3. 交互体验好，因为CSS文件被提前加载了，切换皮肤时不会有延迟

文档中的代码在[这里](https://github.com/duola8789/demos/tree/master/demo33-%E7%BD%91%E9%A1%B5%E6%8D%A2%E8%82%A4)。

## 参考
- [link rel=alternate网站换肤功能最佳实现@鑫空间鑫生活](https://www.zhangxinxu.com/wordpress/2019/02/link-rel-alternate-website-skin//)

