---
title: Node02 NPM
top: false
date: 2017-05-07 09:36:17
updated: 2019-04-30 10:56:21
tags:
- NPM
categories: Node
---

重新学习Node，整理以前的日志。

NPM（node package manager）是NodeJS的包管理器，用于Node插件管理（包括安装、卸载、管理依赖等），npm已经在NodeJS安装的时候顺带装好了

<!-- more -->

## 安装插件

```BASH
npm install <name> [-g] [--save-dev]；

npm install gulp-less --save-dev
```

说明：

（1）通过`-g`来控制是否全局安装：

- 全局安装。将会安装在`C:\Users\Administrator\AppData\Roaming\npm`，并且写入系统环境变量；
- 非全局安装：将会安装在当前定位目录；全局安装可以通过命令行在任何地方调用它，本地安装将安装在定位目录的node_modules文件夹下，通过`require()`调用；


（2）通过`--save-dev`控制插件是记录到`package.json`，以及记录到什么位置

- `--save`（`-S`）指明将插件信息记录到`package.json`的`dependencies`字段
- `--save-dev`（`-D`）指明将插件信息记录到`package.json`的`devDependencies`字段，

`devDependencies`里面的插件只用于开发环境，不用于生产环境，而`dependencies`是需要发布到生产环境的。

## `package.json`

为什么要保存至`package.json`？因为Node插件包相对来说非常庞大，将配置信息写入`package.json`并更新`devDependencies`值，以表明项目需要依赖该插件

`dependencies`的值可以向其他参与项目的人指明项目在开发环境和生产环境中的Bode模块依懒关，其他开发者对应下载即可


## 卸载插件

```BASH
npm uninstall <name> [-g] [--save-dev]
```
*PS：不要直接删除本地插件包*

删除指定插件：

```BASH
npm uninstall gulp-less gulp-uglify gulp-concat
```

删除全部插件需要借助`rimraf`：

```BASH
npm install rimraf -g
rimraf node_modules
```

## 更新插件

```BASH
# 更新指定插件
npm update <name> [-g] [--save-dev]

# 更新全部插件
npm update [--save-dev]
```

## 当前目录已安装模块；

```BASH
# 查看所有弄快
npm list

# 查看一级模块
npm list –depth 1 
```
## cnpm

因为npm安装插件是从国外服务器下载，受网络影响大，可能出现异常，所以[淘宝团队](http://npm.taobao.org)提供了一个完整`npmjs.org`镜像，可以用此代替官方版本，同步频率目前为10分钟一次以保证尽量与官方服务同步。

安装：

```BASH
npm install cnpm -g --registry=https://registry.npm.taobao.org
```

注意：安装完后最好查看其版本号`cnpm-v`或关闭命令提示符重新打开，安装完直接使用有可能会出现错误；cnpm跟npm用法完全一致，只是在执行命令时将`npm`改为`cnpm`；

```
# 使用
cnpm install express
```

## 直接使用npm注册淘宝镜像

```BASH
npm config set registry https://registry.npm.taobao.org
npm config set disturl https://npm.taobao.org/mirrors/node

# 配置后可通过下面方式来验证是否成功
npm config get registry

# 或
npm info express
```

推荐使用npm安装插件，但是安装源改为淘宝

## 删除淘宝镜像

```BASH
npm config delete registry
npm config delete disturl

# 或者 
npm config edit 

# 在打开的文件中找到淘宝那两行，删除
```

## npm本身的升级

用管理员权限打开PowerShell，然后执行以下命令：

```
Set-ExecutionPolicy Unrestricted -Scope CurrentUser -Force
npm install -g npm-windows-upgrade
npm-windows-upgrade
```

## npx

npm从5.2版开始，增加了`npx`命令

npx有三个作用：

（1）Npx会到`node_modules/.bin`路径和环境变量`$PATH`里调用项目内部安装的模块，比如项目内安装了Mocha，如果想要在命令行中调用

```BASH
# 在项目根目录下调用
node-modules/.bin/mocha --version

# 使用npx
npx mocha --version
```

（2）避免安装全局模块，npx会将本地不存在的模块下载到临时目录，使用后再删除

```BASH
npx create-react-app my-react-app

# 可以指定版本
npx uglify-js@3.1.0 main.js -o ./dist/main.js

# 使用--no-install强制使用本地模块
npx --no-install http-server

# 使用--ignore-existing强制使用远程模块
npx --ignore-existing create-react-app my-react-app
```

（3）利用下载模块的特点还可以指定某个版本的Node运行脚本

```BASH
npx node@0.12.8 -v
```

更详细可以参考阮一峰的网络日志。


## 参考

- [国内优秀npm镜像推荐及使用](http://www.ruanyifeng.com/blog/2019/02/npx.html)
