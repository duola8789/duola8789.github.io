---
title: React提高08 Create React App
top: false
date: 2019-03-01 09:38:04
updated: 2019-05-09 15:23:41
tags:
- 脚手架
- Create React App
categories: React
---

一个由Facebook官方出品的React脚手架工具，无需额外配置，迅速搭建React应用脚手架。

这里只对它进行简单的尝试和入门，如果需要进一步的学习，[官网在这里](https://facebook.github.io/create-react-app/)，[文档在这里](https://facebook.github.io/create-react-app/docs/getting-started)，也可以参考[这篇文章](https://juejin.im/post/59dcd87451882578c2084515)进行更高阶更深入的配置和学习。

<!-- more -->

使用Create React App开发React应用不必再安装Webpack或者Babel，它们已经被内置在脚手架中了

它提供的功能：

1. 开箱即用的React支持
2. 开发模式和生产模式的编译
3. 开发模式的热更新
4. 提供了单元测试测试的接口支持
5. 其他配置工具的默认配置（亦可以个性化配置）

## 快速开始

```BASH
npx create-react-app my-app
cd my-app
npm start
```

React应用会运行在`http://localhost:3000/`，开发完成后使用`npm run build`打包

## 安装

安装需要Node的版本在8.10.0以上

```BASH
# 使用npx
npx create-react-app my-app

# npm在5.1版本下不能使用npx
# 需要使用npm，首先进行全局安装
npm install -g create-react-app
# 创建应用
create-react-app my-app
```

## 运行

```BASH
npm start
```

在`http://localhost:3000`以开发模式运行React应用，页面提供了热跟新功能，在控制台会显示错误和警告。

## 单元测试

### 基本使用

```BASH
npm test
```

使用Jest运行单元测试，默认情况下会运行从上次提交commit后有改动的文件的单元测试。

需要`react-scripts@0.3.0`及更高版本，老项目开启单元测试看[这里](https://github.com/facebook/create-react-app/blob/master/CHANGELOG-0.x.md#migrating-from-023-to-030)。

Jest是基于Node的运行期，速度很快，并且动过jsdom提供了浏览器的全局变量，比如`window`，但Jest对于DOM的测试是不准确的，它的目的是对逻辑和组件进行单元测试，而非测试DOM

在运行时Jest会自动寻找以`test.js`/`spec.js`或者`__tests__`文件夹下以`.js`结尾的文件，==这些文件可以位于`src`目录下任意深度的文件夹内==。

建议将测试文件（或`__test__`文件夹）和北侧文件放在一起，有两个好处：

1. 便于管理，一眼就能看到文件的单元测试文件
2. 引入组件的时候更简洁, 例如`import App from './App'`

### 命令行接口

使用`npm test`时，Jest会以`watch`模式运行，每次更改文件都会重新运行测试文件

这个模式下的命令行接口提供了各种能力，可以一直开着这个窗口进行快速的重复测试

### 版本管理接口

使用`npm test`默认情况下会运行从上次提交commit后有改动的文件的单元测试。可以在`watch`模式下按`a`来要求Jest执行全部的测试

如果当前的工程没有使用版本管理，那么Jest会默认运行全部的测试

### 测试组件

关于Jest的使用，以前学习[在Vue中使用Jest时总结过](https://blog.csdn.net/duola8789/article/details/80434962/)，Jest的基本用法是相同的，在测试组件时有所区别，Vue测试组件使用的将组件和Jest进行连接的工具是Vue-test-utils，而React则是[Enzyme](https://airbnb.io/enzyme/)（更准确些，[jest-enzyme](https://github.com/FormidableLabs/enzyme-matchers)更接近于Vue-test-utils，封装了很多方便的API）

内容比较多，这里不展开，[文档在这里](https://facebook.github.io/create-react-app/docs/running-tests)，慢慢单独学习。

## 构建生产文件

```BASH
npm run build
```

打包出的文件是经过压缩的，文件名带有Hash值

## 个性化配置

因为Create-React-App将Webpack、Babel、ESLit的配置隐藏起来，简化了用户的配置操作，可以快速开始开发。

但是这只适用于一些小型的、没有特殊需求的应用的开发，如果构建大型应用还需要对上面这些工具进行个性化的配置：

```BASH
npm run eject
```

运行后，Create-React-App会将上面工具的配置文件复制到项目中，以后对配置文件进行修改后，项目式中会采用项目中复制修改后的配置文件

要注意，这个操作是不可逆的。

## CSS-Loader

在新版本的Create React App中增加了对CSS Modules的支持，要求`react-scripts`版本高级2.0.0。

CSS文件的命名形式为`[name].module.css`，对应的类名会通过添加后缀的形式来实现局部作用域，类名的格式是`[filename]\_[classname]\_\_[hash]`

详情参考[文档](https://facebook.github.io/create-react-app/docs/adding-a-css-modules-stylesheet)。

如果需要在老版本的Create React App中增加了对CSS Modules的支持，则首先需要先通过`eject`命令暴露配置文件，参考[这篇文章](https://medium.com/nulogy/how-to-use-css-modules-with-create-react-app-9e44bec2b5c2)。

## ESLint

由于Create React App将默认的构建配置封装了起来，而ESLint仅仅开启了最基本的规则，更重要的是默认情况下，ESLint仅仅会在IDE中对违反规则的情况进行提示，并不会在构建时在终端的输出进行终端和提示。

如果这种情况可以满足需要，而只需要开启更多的规则，那么就可以在根目录下新建一个文件`.eslintrc.json`，然后添加：

```JS
{
  "extends": "react-app"
}
```

但是如果要起到更强制性的提示作用（中断构建、终端提示），Create React App建议使用[Prettier](https://github.com/prettier/prettier)代替ESLint。如果要使用ESLint，那么就需要使用`npm run eject`，将配置文件吐出，按照AlloyTeam的提示进行配置即可，参考[这篇笔记](https://duola8789.github.io/2017/12/05/01%20%E5%89%8D%E7%AB%AF%E7%AC%94%E8%AE%B0/07%20%E9%9B%B6%E6%95%A3%E4%B8%93%E9%A2%98/%E9%9B%B6%E6%95%A3%E4%B8%93%E9%A2%9816%20EditorConfig%E5%92%8CESLint/)。

## Ant Design按需引入

安装antd：

```BASH
npm install antd -S
```

然后进行按需引入分为两种情况：

（1）未eject出所有配置：

参考[antd的文档](https://ant.design/docs/react/use-with-create-react-app-cn)，

安装[react-app-rewired](https://github.com/timarney/react-app-rewired)和[customize-cra](https://github.com/arackaf/customize-cra)（CRA）。

```BASH
npm install react-app-rewired customize-cra -D
```
然后修改`package.json`文件的启动命令：

```JS
"scripts": {
  "start": "react-app-rewired start",
  "build": "react-app-rewired build",
  "test": "react-app-rewired test",
}
```
然后安装[babel-plugin-import](https://github.com/ant-design/babel-plugin-import)

```BASH
npm install babel-plugin-import -D
```

然后在根目录下创建`config-overrides.js`，用来修改默认配置：

```JS
const { override, fixBabelImports } = require('customize-cra');

module.exports = override(
  fixBabelImports('import', {
    libraryName: 'antd',
    libraryDirectory: 'es',
    style: 'css', // true
  }),
);
```

然后按按照下面的格式按需引入模块：

```JS
import { Button } from 'antd';
```

（2）已经eject出所有配置文件：

这个时候直接按照[babel-plugin-import文档](https://github.com/ant-design/babel-plugin-import)的说明配置即可。

安装`babel-plugin-import`：

```BASH
npm install babel-plugin-import -D
```

然后在`package.json`中找到`babel`选项，修改为：

```JS
"babel": {
  "presets": [
    "react-app"
  ],
  "plugins": [
    [
      "import",
      {
        "libraryName": "antd",
        "style": "css"
      }
    ]
  ]
}
```

引入方式与上面相同：

```JS
import { Button } from 'antd';
```

## 配置Less

首先安装`less`和`less-loader`：

```
npm install less less-loader -D
```

然后同样分为是否eject配置两种情况：

（1）未eject出所有配置，仍遵循上面的步骤，安装`react-app-rewired`和`customize-cra`,修改`package.json`中的启动脚本。

然后修改`config-overrides.js`文件：

```JS
const { override, fixBabelImports, addLessLoader } = require('customize-cra');

module.exports = override(
  fixBabelImports('import', {
    libraryName: 'antd',
    libraryDirectory: 'es',
    style: true, 
  }),
  addLessLoader({
    javascriptEnabled: true,
    modifyVars: { '@primary-color': '#1DA57A' },
  }),
);
```
这里利用了`less-loader`的`modifyVars`来进行主题配置，变量和其他配置方式可以参考[配置主题](https://ant.design/docs/react/customize-theme-cn)文档。

（2）已经eject出所有配置的情况，参考[这篇文章](https://juejin.im/post/5c3d67066fb9a049f06a8323)：

在`config`目录下的`webpack.config.js`文件，找到`// style files regexes`注释位置，添加：

```JS
// 添加 less 解析规则
const lessRegex = /\.less$/;
const lessModuleRegex = /\.module\.less$/;
```

然后找到`rules`属性，在其中添加less解析配置：

```JS
// Less 解析配置
{
  test: lessRegex,
  // exclude: lessModuleRegex,
  use: getStyleLoaders(
    {
      importLoaders: 2,
      sourceMap: isEnvProduction && shouldUseSourceMap,
    },
    'less-loader'
  ),
  sideEffects: true,
},
{
  test: lessModuleRegex,
  use: getStyleLoaders(
    {
      importLoaders: 2,
      sourceMap: isEnvProduction && shouldUseSourceMap,
      modules: true,
      getLocalIdent: getCSSModuleLocalIdent
    },
    'less-loader'
  ),
},
```

要注意的是，新添加的`less-loader`必须在`file-loader`的前面才能生效，因为Webpack在解析Loader是从右至左进行的（从下到上），只有先经过`file-loader`对文件路径的处理，`less`文件才能够被正确引入。

## 配置Alias

在Vue中习惯了使用`@`来代替一连串的`..`表示的相对地址，这个功能是webpack提供的，需要在webpack中进行配置

现在使用了React，也同样希望能够配置Alias来实现路径的更优雅的表示。如果已经eject处所有配置的情况下，直接在webpackd的配置文件下添加相关的代码即可：

```JS
module.exports = {
  //...
  resolve: {
    alias: {
      '@': path.resolve(__dirname, 'src/'),
    }
  }
};
```
但是如果在未eject的情况下，同样需要借助上面使用的`react-app-rewired`实现，首先在根目录建立一个`alias.js`文件，在这个文件中编写Alias的配置代码：`react-app-rewired`

```JS
const path = require('path');

module.exports = {
  resolve: {
    alias: {
      '@': path.join(__dirname, 'src'),
    },
  },
};
```
然后在`config-overrides.js`文件中引入并进行配置：

```JS
const { override, addWebpackAlias } = require('customize-cra');
const alias = require('./alias');

module.exports = override(
  addWebpackAlias(alias.resolve.alias),
);
```
重新编译后`@`就生效了。

这时有两个问题要解决

（1）IDE的点击跳转失效了

Webstorm是可以识别webpack的配置文件，对Alias进行相应的处理，但是这里并没有eject出Webpack的配置文件，但是我们的`alias.js`文件就是按照Webpack的配置模块的格式来编写的，所以可以将`alias.js`作为配置文件，传递给Webstorm，这样IDE的文件跳转就正常了

（2）ESLint的导入导出规则报错

这是因为ESLint不能识别我们的Alias，这需要安装`eslint-import-resolver-webpack`这个插件，让ESLint使用Webpack的解析规则。

首先安装

```
yarn add eslint-import-resolver-webpack -D
```

然后在`.eslintrc.js`中添加如下的配置：


```JS
module.exports = {
  "settings": {
    "import/resolver": {
      "webpack": {
        "config": "alias.js"
      }
    }
  },
}
```
同样使用`alias.js`来代替Webpack的配置文件，配置完之后ESLint也就能正常工作了。

## 配置webpack-bundle-analyzer

首先需要安装：

```BASH
# NPM 
npm install --save-dev webpack-bundle-analyzer
# Yarn 
yarn add -D webpack-bundle-analyzer
```

在未eject的情况下，同样可以使用`customize-cra`来添加webpack-bundle-analyzer的配置，在`cvonfig-overrides.js`中，引入`addBundleVisualizer`，进行配置：

```JS
const { override, addBundleVisualizer } = require('customize-cra');

module.exports = override(
  // 添加 webpack-bundle-analyzer
  addBundleVisualizer({
    analyzerMode: 'static',
    reportFilename: 'report.html',
  }, true),
);
```
addBundleVisualizer接受两个参数，第一个对象是`webpack-bundle-analyzer`的配置项，可以[参考文档](https://www.npmjs.com/package/webpack-bundle-analyzer)。第二个选项用来配置自动开启，设置为`ture`就不需要在每次`build`时传入`--analyze`来开启分析了

在已经eject了的情况下，在`webpack.config.js`做进行配置，将`webpack-bundle-analyzer`作为插件进行引入

```JS
const BundleAnalyzerPlugin = require('webpack-bundle-analyzer').BundleAnalyzerPlugin;
 
module.exports = {
  plugins: [
    new BundleAnalyzerPlugin()
  ]
}
```



## 环境变量

环境变量在构建期间嵌入。

可以使用`process.env.NODE_ENV`来读取内置的环境变量，当运行`npm start`时，它等于`development`，当运行`npm test`时，它等于`test`，当运行`npm run build`时它等于`production`。不能手动覆盖`NODE_ENV`，这可以防止开发人员意外地将开发环境部署到生产环境中

除了`process.env.NODE_ENV`之外也可以使用自定义的环境变量，自定义的环境变量必须以`REACT_APP_`开头。

添加环境变量有两种方式：

（1）在Shell中添加临时环境变量。

操作系统不同，在Shell中定义环境变量的方法也不相同:

```BASH
# Windows (cmd.exe)
set "REACT_APP_SECRET_CODE=abcdef" && npm start

# Linux, macOS (Bash)
REACT_APP_SECRET_CODE=abcdef npm start
```

为了统一在不同的操作系统中的设置方法，可以使用`cross-env`这个库

安装：

```BASH
npm install cross-env --save-dev
```

使用时只需要在原来的脚本前面加上`cross-env`就可以了

```BASH
cross-env NODE_ENV=development nodemon ./index.js
```

（2）在`.env`中添加开发环境变量

在项目根目录中创建名为`.env`的文件，在文件创建以`REACT_APP_`开头的自定义环境变量。除了`NODE_ENV`之外的任何其他变量都将被会略

==实际上，`NODE_ENV`是不能被覆盖的，也就意味着在`.env`中定义`NODE_ENV`也是同样被忽略的，在默认配置条件下，脚手架中的`NODE_ENV`是无法更改的。==


`.env`文件应该提交到git进行管理（除了`.env&.local`之外）

除了`.env`之外，还可以使用特殊的`.env`文件

- `.env`：默认。
- `.env.local`：本地覆盖。除`test`之外的所有环境都加载此文件。
- `.env.development`, `.env.test`, `.env.production`：设置特定环境。
- `.env.development.local`, `.env.test.local`,`.env.production.local`：设置特定环境的本地覆盖。

如果使用一个新的自定义的`.env`文件，比如使用`.env.stage`的环境变量文件，需要在运行`npm`命令时使用`env-cmd`

安装：

```BASH
npm install env-cmd --save-dev
```
使用时直接将`.env.stage`的路径引入即可，在`Package.json`文件中：

```JS
{
  "scripts": {
    "test": "env-cmd ./.env.stage npm run build"
  }
}
```
在命令行中：

```BASH
./node_modules/.bin/env-cmd ./.env.stage npm run build
```

这种情况下，`process.env.NODE_ENV`仍然是`production`，但加载的`.env`文件已经不再是`.env.build`，而是变为了`env.stage`

## 参考

- [Docs@Create React App](https://facebook.github.io/create-react-app/docs/documentation-intro)
- [从React脚手架工具学习React项目的最佳实践（上）：前端基础配置@掘金](https://juejin.im/post/59dcd87451882578c2084515)
- [在create-react-app中使用@Ant Design](https://ant.design/docs/react/use-with-create-react-app-cn)
- [ant-design/babel-plugin-import@github](https://github.com/ant-design/babel-plugin-import)
- [在 Create React App 中启用 Sass 和 Less@掘进](https://juejin.im/post/5c3d67066fb9a049f06a8323)
- [eslint-import-resolver-webpack@npm](https://www.npmjs.com/package/eslint-import-resolver-webpack)
- [关于eslint-plugin-import无法识别webpack alias问题@JERMY'S BLOG](http://jeremyfan.name/blog/2018/08/22/2018-08-eslint-webpack-alias/)
- [aze3ma/react-app-rewire-aliases@github](https://github.com/aze3ma/react-app-rewire-aliases)
- [create-react-app 通过 react-app-rewired 添加 webpack 的 alias@OnlyLing](https://www.onlyling.com/archives/321)
- [Resolve@Webpack](https://webpack.js.org/configuration/resolve/)
- [env-cmd@npm](https://www.npmjs.com/package/env-cmd)
- [webpack-contrib/webpack-bundle-analyzer@github](https://www.npmjs.com/package/webpack-bundle-analyzer)
