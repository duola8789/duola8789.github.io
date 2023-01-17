---
title: Node09 swig模版引擎
top: false
date: 2017-04-04 10:48:17
updated: 2019-04-30 11:06:56
tags:
- Swig
categories: Node
---

重新学习Node，整理以前的日志。Swig模版引擎学习笔记。

<!-- more -->


## 简介

swig 是node端的一个优秀简洁的模板引擎，类似Python模板引擎Jinja，目前不仅在node端较为通用，相对于jade、ejs优秀，而且在浏览器端也可以很好地运行。

这是[官方文档
](http://node-swig.github.io/swig-templates/docs/)。

## 语法

### swig的变量


```
{{ foo.bar }}
{{ foo['bar'] }}
//如果变量未定义，输出空字符。
```

### swig的标签

#### extends

使当前模板继承父模板，必须在文件最前

```
{% exdtends file %}

// 参数file：父模板相对模板root的相对路径

```

#### block

定义一个块，使之可以被继承的模板重写，或者重写父模板的同名块


```
{% block blockName %}something can be entended and modified...{% endblcok %}
// 参数name：块的名字，必须以字母数字下划线开头
```

#### parent

将父模板中同名块的内容注入当前块中

```
{% extends "./foo.html" %}
{% block content %}
    My content.
    {% parent %}
{% endblock %}
```

#### include

包含一个模板到当前位置，这个模板将使用当前上下文

参数file是包含模板相对模板 root 的相对路径

```
{% include "a.html" %}
{% include "template.js" %} 
//将引入的文件内容放到被引用的地方

```
#### raw

停止解析swig标签，其中所有内容都将按照字面意思输出 

参数file是包含模板相对模板 root 的相对路径

```
// foobar = '<p>'
{% raw %}{{ foobar }}{% endraw %}
// => {{ foobar }}
```

#### set

设置一个变量，在当前上下文中复用，设置的值会覆盖已定义值

```
// foods = {};
// food = 'chili';
{% set foods[food] = "con queso" %}
{{ foods.chili }}
// => con queso
```

## 模版继承

Swig 使用 extends 和 block 来实现模板继承

example:


```
//layout.html

<!doctype html>
<html>
<head>
    <meta charset="utf-8">
    <title>{% block title %}My Site{% endblock %}</title>

    {% block head %}

    {% endblock %}
</head>
<body>
    {% block content %}{% endblock %}
</body>
</html>
```

```
//index.html

{% extends './layout.html' %}

{% block title %}My Page{% endblock %}

{% block head %}
{% parent %}

{% endblock %}

{% block content %}
    <p>This is just an awesome page.</p>
    <h1>hello,lego.</h1>
    <script>

        //require('pages/index/main');
    </script>
{% endblock %}
```
swig模板经过编译后：

```
<!doctype html>
<html>
<head>
    <meta charset="utf-8">
    My Page
</head>
<body>
    <p>This is just an awesome page.</p>
    <h1>hello,lego.</h1>
    <script src="pages/index/main">
        //require('pages/index/main');
    </script>
</body>
</html>
```

## 在express中使用swig

在express框架中，默认的模版是jade，可以更改为其他模版引擎。修改app.js

```
var app = express(); 
app.set('view engine', 'jade');
// 把上面的代码改为下面的
// view engine setup
var app = express(),
swig = require('swig'),
app.engine('html', swig.renderFile); //使用swig渲染html文件
app.set('view engine', 'html'); //设置默认页面扩展名
app.set('view cache', false); //设置模板编译无缓存
app.set('views', path.join(__dirname, 'views')); //设置项目的页面文件，也就是html文件的位置
swig.setDefaults({cache: false}); //关闭swig模板缓存
swig.setDefaults({loader: swig.loaders.fs(__dirname + '/views')}); //从文件载入模板，请写绝对路径，不要使用相对路径

```
然后把原来的views文件夹下得文件后缀都改为html

模板文件layout.html

```
<!doctype html>
<html>
<head>
  <meta charset="utf-8">
  <title>{% block title %}{% endblock %}</title>
  {% block head %}
  {% endblock %}
</head>
<body>
  {% block content %}{% endblock %}
</body>
</html>
```
index.html

```
{% extends 'layout.html' %}

{% block title %}index {{title}} {%endblock%}

{% block head %}
{{title}}
{% endblock %}

{% block content %}
<p>This is just an awesome page.</p>
{% endblock %}
```
然后再路由中设置即可使用：

```
router.get('/', function(req, res) {
  res.render('index', { title: '标题' });
});
```


## 参考
- http://www.iqianduan.net/blog/how_to_use_swig
- http://node-swig.github.io/swig-templates/docs/
- http://www.joryhe.com/2016-05-21-hexo-swig-advance-grammar.html
- http://www.ithao123.cn/content-10830341.html
