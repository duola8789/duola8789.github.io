---
title: Webstorm10 快速插入当前时间
top: false
date: 2019-02-01 15:30:35
updated: 2019-02-01 15:30:35
tags:
- IDE
- WebStorm
categories: IDEp配置
---

Webstorm快速插入当前时间的小技巧

<!-- more -->

在Webstorm中通过自定义`Live Template`可以快速插入当前时间

首先在设置中，`Editor` → `Live Template`，点击添加模板

![](http://image.oldzhou.cn/1.png)

填写如下信息：

- `Abbreviation`，在编辑器中输入的可以出发插入模板的缩写字符，这里填写`datetime`
- `Description`，对这个模板的介绍，可以不填写
- `Template`，模版的样式，格式为`$datetime`，其中的`datetime`就是上面定义的模板字符，需要用`$`包裹
- `Expand with`，出发插入模板的操作，默认点击`Tab`插入
- `No applicable context yet`，定义这个模板应用的文件类型，这里选择全部

填写完成后：

![](http://image.oldzhou.cn/2.png)

然后点击`EDIT VARIABLES`，编写模板内容，选择`datetime`那一项，在`Expression`中输入`date("yyyy-MM-dd HH:mm:ss")`

![](http://image.oldzhou.cn/3.png)

现在在编辑器界面就可以快速插入当前时间了。

输入`da`时，编辑器就会提示，按下`Tab`补全，插入成功。

![](http://image.oldzhou.cn/4.png)
