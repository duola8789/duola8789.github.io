---
title: Egg03 断点调试
top: false
date: 2019-05-16 19:16:43
updated: 2019-05-05 19:44:45
tags:
- Chrome
- Webstorm
categories: Egg
---

断点调试Egg应用的方法。

<!-- more -->

## 使用Chrome调试

需要Node 8.x+版本以上。

首先执行`npm run debug`（与`node app.js --inspect`原理相同）

然后有两种方法启动控制台（不是访问接口的控制台，而是直接启动一个新的针对Node的控制台）

（1）执行`npm run debug`之后，在控制台最后会输出`DevTools`对应的地址，该地址是代理后的worker，无需担心重启问题。直接访问该地址即可进行断点调试。

```BASH
➜ showcase git:(master) ✗ npm run debug

> showcase@1.0.0 debug /Users/tz/Workspaces/eggjs/test/showcase
> egg-bin debug

Debugger listening on ws://127.0.0.1:9229/f8258ca6-d5ac-467d-bbb1-03f59bcce85b
For help see https://nodejs.org/en/docs/inspector
2017-09-14 16:01:35,990 INFO 39940 [master] egg version 1.8.0
Debugger listening on ws://127.0.0.1:5800/bfe1bf6a-2be5-4568-ac7d-69935e0867fa
For help see https://nodejs.org/en/docs/inspector
2017-09-14 16:01:36,432 INFO 39940 [master] agent_worker#1:39941 started (434ms)
Debugger listening on ws://127.0.0.1:9230/2fcf4208-4571-4968-9da0-0863ab9f98ae
For help see https://nodejs.org/en/docs/inspector
9230 opened
Debug Proxy online, now you could attach to 9999 without worry about reload.
DevTools → chrome-devtools://devtools/bundled/inspector.html?experiments=true&v8only=true&ws=127.0.0.1:9999/__ws_proxy__
```

（2）第二种方法对于普通的Node应用都适用（前提时使用了`--inspect`模式），访问`chrome://inspect`，配置相应的端口（Egg需要将`9229`和`9230`端口加入到配置中），然后点击`Open dedicated DevTools for Node`即可打开调试控制台。

![image](http://image.oldzhou.cn/30419047-a54ac592-9967-11e7-8a05-5dbb82088487.png)

## 使用Webstorm调试

`egg-bin`会自动读取Webstorm下设置的环境变量`$NODE_DEBUG_OPTION`。

按照下图进行配置：

![image](http://image.oldzhou.cn/30423086-5dd32ac6-9974-11e7-840f-904e49a97694.png)

使用Webstorm的NPM调试启动即可进行Debug。

果然Webstorm太牛逼。

## 使用VSCode调试

可以通过2个方式：

方式一：开启VSCode配置`Debug: Toggle Auto Attach`，然后在Terminal执行`npm run debug` 即可。

方式二：配置VSCode的`.vscode/launch.json`，然后F5一键启动即可。（注意，需要关闭方式一中的配置）

```JS
// .vscode/launch.json
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Launch Egg",
      "type": "node",
      "request": "launch",
      "cwd": "${workspaceRoot}",
      "runtimeExecutable": "npm",
      "windows": { "runtimeExecutable": "npm.cmd" },
      "runtimeArgs": [ "run", "debug" ],
      "console": "integratedTerminal",
      "protocol": "auto",
      "restart": true,
      "port": 9229,
      "autoAttachChildProcesses": true
    }
  ]
}
```
也提供了一个[vscode-eggjs](https://github.com/eggjs/vscode-eggjs)扩展来自动生成配置

![image](http://image.oldzhou.cn/35954428-7f8768ee-0cc4-11e8-90b2-67e623594fa1.png)

## 参考

- [Node 调试工具入门教程@阮一峰的网络日志](http://www.ruanyifeng.com/blog/2018/03/node-debugger.html)
- [使用 DevTools 进行调试@Egg](https://eggjs.org/zh-cn/core/development.html#%E4%BD%BF%E7%94%A8-devtools-%E8%BF%9B%E8%A1%8C%E8%B0%83%E8%AF%95)
