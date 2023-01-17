---
title: 持续集成02 Travis CI
top: false
date: 2019-06-27 10:52:03
updated: 2019-10-20 21:55:30
tags:
- 持续集成
- 持续部署
- Travis ci
categories: 前端工程
---

[Travis CI](https://travis-ci.org/)是市场份额最大的持续集成工具，它对于开源项目是免费的，但是对私人项目和公司项目则是收费的。

Travis CI提供的是持续集成服务，它绑定Github上的项目，只要有新的diamante，就会自动抓取，然后提供一个运行环境，执行测试，完成构建，然后部署到服务器。

<!-- more -->

## 使用准备

Travis CI只支持Github，不支持其他代码托管项目，所以使用Travis CI的前提是:

1. 代码位于Github的仓库
2. 项目中有可运行的代码
3. 项目中包含构建或者测试脚本

满足上述条件之后，需要访问其[官网](https://travis-ci.org/)，使用Github账户登陆Travis CI。登陆后Travis CI会列出Github上的所有代码仓库

![](http://image.oldzhou.cn/FtpyeOe4CS49Zkzgb08QyCEQdsUt)

选择按钮打开，就激活了这个仓库，Travis会监听这个仓库的所有变化。

## `.traivs.yml`

Travis要求项目下面，必须有一个`.travis.yml`文件，且必须提交到代码仓库。这是一个配置文件，指定了Travis的行为，是必不可少的。

文件是[YAML格式](http://www.ruanyifeng.com/blog/2016/07/yaml.html)：

```YAML
# 指定默认运行环境
language: node_js 

## 指定 node 版本
node_js:
  - "node"

# 指定执行的脚本
script: test.js

# 需要 sudo 权限
sudo: required

# 安装依赖之前执行
before_instasll: sudo pip install foo
```

## 运行流程

Travis的运行流程包括两个阶段： `install`阶段（安装依赖）和`script`阶段（运行脚本）

### `install`

`install`字段用来指定安装脚本：

```YAML
install: npm install
```

如果有多个脚本，可以写成下面的形式：

```YAML
install:
  - command1
  - command2
```

如果`command1`失败了，整个构建就会停下来，不再向下进行。

如果不需要安装，则跳过安装阶段，直接设为`true`：

```YAML
install: true
```

### `script`

`script`字段用来指定构建或者测试脚本

```YAML
script: npm run test
```

如果有多个脚本，可以写成下面的形式：

```YAML
script:
  - command1
  - command2
```

`script`与`install`不一样，如果`command1`失败，`command2`会继续执行，但整个构建阶段的状态是失败的。

如果`command2`只有在`command1`成功后才能执行，就需要写成

```YAML
script: command1 && command2
```

### Node项目

Node项目的`install`和`script`都有默认脚本，可以省略：

- `install`默认值`npm install`
- `script`默认值`npm test`

更多配置见[官方文档](https://docs.travis-ci.com/user/languages/javascript-with-nodejs/)。

### 部署

`script`阶段结束后，可以设置[通知步骤](https://docs.travis-ci.com/user/notifications/)和[部署步骤](https://docs.travis-ci.com/user/notifications/)，它们不是必须的。

部署的脚本可以在script阶段执行，也可以使用Travis为几十种常见服务提供的快捷部署功能。

### 通知

在`.travis.yml`进行如下配置

```YAML
notifications:
  email:
    recipients:
      - duola8789@126.com
  slack:
    on_success: never
    on_failure: always
```

当失败时会发邮件进行通知

### 钩子方法

Travis为上面这些阶段提供了七个钩子：

- `before_install`，`install`阶段之前执行
- `before_script`，`script`阶段之前执行
- `after_failure`，`script`阶段失败后执行
- `after_success`，`script`阶段成功后执行
- `before_deploy`，`depoly`步骤之前执行
- `after_depoly`，`depoly`步骤之后执行
- `after_script`，`script`阶段之后执行

### 生命周期

完整的生命周期：

1. `before_install`
2. `install`
3. `before_script`
4. `script`
5. `aftersuccess`/`afterfailure`
6. `before_deploy`（可选的）
7. `deploy`（可选的）
8. `after_deploy`（可选的）
9. `after_script`

### 运行状态

Travis每次运行后，可能会返回四种状态

- `passed`，运行成功，所有步骤的退出码都是`0`
- `canceled`，用户取消执行
- `errored`，`before_install`、`install`、`before_script`有非零退出码，运行会立即停止
- `failed`，`script`有非零状态码，会继续执行

## 使用技巧

### 环境变量

`.travis.yml`的`env`字段可以定义环境变量。


```YAML
env:
  - DB=postgres
  - SH=bash
  - PACKAGE_VERSION="1.0.*"
```
脚本内部就可以使用这些变量了，使用时前面加`$`符号
有一些环境变量（比如密码）不能同开，可以通过Travis网站，写在每个项目的设置里面，Travis会将其自动加入环境变量，这样脚本内部依然可以使用环境变量，但是只有管理员才能看到变量的值

![](http://image.oldzhou.cn/FjcfvJPpJLiravK1zBc5JAqPZCKE)

### 加密信息

如果不放心保密信息明文存在Travis的网站，可以使用Travis提供的加密功能。可以参考[阮老师的介绍](http://www.ruanyifeng.com/blog/2017/12/travis_ci_tutorial.html)和[官方文档](https://docs.travis-ci.com/user/encryption-keys/)。

## 部署Github Pages

以我的Github Pages博客的部署做例子，根据[官网文档](https://hexo.io/zh-cn/docs/github-pages.html)，`.travis.yml`配置如下：

```YAML
sudo: false

language: node_js

node_js: stable

cache: npm

branches:
  only:
    - master # build master branch only

script:
  - hexo generate # generate static files

deploy:
  provider: pages
  skip_cleanup: true
  github_token: $GH_TOKEN
  keep_history: true
  local_dir: public
  on:
    branch: master

notifications:
  email:
    recipients:
      - duola8789@126.com
  slack:
    on_success: never
    on_failure: always
```

要说明的是：

（1）通过`.travis.yml`配置后，就不再需要`_config.yml`中配置`deploy`相关东西了，但是要注意，使用`.travis.yml`配置，Travis-CI网站上存放`github_token`的变量名必须是`$GH_TOKEN`

（2）`github_token`需要提供Github的个人凭证，生成方式参考[Github的文档](https://help.github.com/en/articles/creating-a-personal-access-token-for-the-command-line)

![](http://image.oldzhou.cn/Fm5YpSC1WIhonBFudILNit9Zs9uz)

![](http://image.oldzhou.cn/Fp0lrCW8WwkTEJucgIpVNR9ASPDO)

![](http://image.oldzhou.cn/Fv1KDfUVNNhsgjqOGX4nT5xp_s54)

![](http://image.oldzhou.cn/Fv1KDfUVNNhsgjqOGX4nT5xp_s54)

这里个人凭证需要勾选`public_repo`或`repo`。

![](http://image.oldzhou.cn/FhEw_g9Kt7yUkDdFJLmOjINT0huu)

生成凭证后，为了防止个人凭证的泄露，不能直接在`.travis.yml`直接填写个人凭证，需要将凭证一边量的方式添加到Travis CI的对应的项目中的设置里面

![](http://image.oldzhou.cn/FikGhMaTB1jE_KQw0xc37gVw2EWr)

（3）需要将`skip_cleanup`设为`true`，否则Travis CI会将在构建过程中生成的文件全部删除，可能会导致本来想上传的文件删除

更多的配置项参考[文档](https://docs.travis-ci.com/user/deployment/pages/)。

配置完成后来实验一下，新添加一篇文章《持续集成01 入门》。

（2019.10.20更新）根据Hexo的[新的文档](https://hexo.io/zh-cn/docs/github-pages.html)配置了`.travis.yml`，很详细却准确，不需要下面的步骤了。

文章结束。
---

本以为会很顺利，结果尝试了多次才按照上面Travis CI的配置推送成功，因为我的博客的源码和页面是放在了两个不同的仓库，源码的仓库是`https://github.com/duola8789/blog.git`，而页面的仓库是`https://github.com/duola8789/duola8789.github.io.git`，所以直接推送的时候总是推送失败，是因为它默认将`duola8789/blog`仓库的本地全部目录推送到`duola8789/blog`的`gh-pages`分支，而我现在想要的是将`duola8789/blog`仓库的`public`目录推送到`duola8789.github.io.git`的`master`分支，所以根据[文档](https://docs.travis-ci.com/user/deployment/pages/)的提示，在`.travis.yml`中增加下面的配置

```YAML
deploy:
  # 新增
  repo: duola8789/duola8789.github.io
  local_dir: public/
  target_branch: master
```

新增的第一项指明要推送的页面所在的仓库，第二项后者指明要推送的本地目录，第三项修改要推送的分支。同时将`_config.yml`的Hexo的推送配置注释掉：

```YAML
deploy:
  type: git
  # repo: https://$github_token@github.com/duola8789/duola8789.github.io.git
  branch: master
```
这样修改完成后就推送成功了：

![](http://image.oldzhou.cn/FnBzQXwAUgehD99enHE0Fagm2pdD)

在网上查找的资料，都没有用Travis CI的辅助配置，而是使用了自定义的配置，也都可以成功，下面这种是[比较简单的一种](https://juejin.im/post/5c887419e51d45334d2f9a00)，也推荐使用（将`_config.yml`的Hexo的推送配置注释掉）：

```YAML
language: node_js

node_js: stable

cache:
  directories:
    - node_modules

before_install:
  - git config --global user.name "zh"
  - git config --global user.email "duola8789@126.com"
  - npm install -g hexo-cli
  - export HEXO_DEPLOYER_REPO=https://$github_token@github.com/duola8789/duola8789.github.io.git

script:
  - npm run deploy
```

果然不亲自尝试，都是没有说服力的。

如果遇到403无权限推送的错误，一般是Github Token的问题，是否没有在环境变量中正确的设置，挥着没有在`.travis.yml`正确引用。

原来管理博客，在文章写完之后需要分两步，首先`npm run deploy`，本地重新生成、打包文章，将文章发布到`https://github.com/duola8789/duola8789.github.io.git`，再`commit`、`push`当前的源码，提交到`https://github.com/duola8789/blog.git`。现在配置过后，直接提交源码到仓库即可，其他的Travis CI都会帮助我们完成。

## 参考

- [持续集成服务 Travis CI 教程@阮一峰的网络日志](http://www.ruanyifeng.com/blog/2017/12/travis_ci_tutorial.html)
- [GitHub Pages Deployment@Travis CI](https://docs.travis-ci.com/user/deployment/pages/)
- [Creating a personal access token for the command line@Github](https://help.github.com/en/articles/creating-a-personal-access-token-for-the-command-line)
- [通过travis自动部署hexo博客到github pages@掘金](https://juejin.im/post/5c887419e51d45334d2f9a00)
- [Configuring Build Notifications@Travis CI](https://docs.travis-ci.com/user/notifications/)
