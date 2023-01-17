---
title: Git02 常用命令
top: false
date: 2017-03-21 11:54:00
updated: 2019-07-05 11:14:00
tags: 
- Git
categories: Git
---

![](http://image.oldzhou.cn/FlWMWzIX9WE7PW-7eyeq8uaEJ_3p)

```TEXT
- Workspace：工作区
- Index / Stage：暂存区
- Repository：仓库区（或本地仓库）
- Remote：远程仓库
```

<!-- more -->

## 1 新建代码库

```BASH
# 在当前目录新建一个Git代码库
$ git init

# 新建一个目录，将其初始化为Git代码库
$ git init [project-name]

# 下载一个项目和它的整个代码历史
$ git clone [版本库的网址] [本地目录名]
```
### 1.1  `git init`

用init命令进行GIT项目的初始化：

```BASH
$ git init
```

这个命令会为你创建`.git`的目录，并生成一些基本的信息。仅进行初始化没有生成master branch，也没有任何commit的信息。需要自己做第一个commit(root commit)，这时GIT会为你自己生成master branch。

```BASH
C:\workspace\git\startuproject>git add README.txt
C:\workspace\git\startuproject>git commit -m "initialization"
```

### 1.2 `git clone`

每个项目都有一个 Git 目录，它是Git用来保存元数据和对象数据库的地方。该目录非常重要，每次克隆镜像仓库的时候，实际拷贝的就是这个目录里面的数据。

执行如下命令以创建一个本地仓库的克隆版本：

```BASH
git clone /path/to/repository
```

如果是远端服务器上的仓库，该命令会在本地主机生成一个目录，与远程主机的版本库同名。如果要指定不同的目录名，可以将目录名作为git clone命令的第二个参数。

```BASH
$ git clone https://github.com/notechsolution/gitDemo.git mylibgit
```

以上命令告诉GIT：用https协议从`github.com/notechsolution/gitDemo.git`把代码克隆下来，并且放在本地的`mylibgit`目录下面。

关于checkout分支，clone的时候如果不指明，则默认checkout`master`分支，如果需要，则可以在clone命令的最后用`-b`参数 指明：

```BASH
$ git clone https://github.com/notechsolution/gitDemo.git mygitdemo -b test
```

克隆版本库的时候，所使用的远程主机自动被Git命名为`origin`。如果想用其他的主机名，需要用g`it clone`命令的`-o`选项指定。

```BASH
$ git clone -o jQuery https://github.com/jquery/jquery.git
$ git remote
jQuery
```

## 2 配置

Git的设置文件为`.gitconfig`，它可以在用户主目录下（全局配置），也可以在项目目录下（项目配置）。

```BASH
# 显示当前的Git配置
$ git config --list

# 编辑Git配置文件
$ git config -e [--global]

# 设置提交代码时的用户信息
$ git config [--global] user.name "[name]"
$ git config [--global] user.email "[email address]"

# 增加提交代码时的用户信息
$ git config -add [--global] user.name "[name]"
$ git config -add [--global] user.email "[email address]"
```

读取配置文件时GIT会查看你项目里面`.git\config`这个文件，如果没有配置，则再读取用户目录（`%HOMEPATH%`）下的`.gitconfig`文件。

我们可以直接把作者的用户名和email配置写到这些文件里面，也可以通过命令行写进去。通过`git config –add`添加的配置参数时,该参数加到项目里面的`.git\config`，而如果是通过`git config –add –global`的话，那么将添加到全局环境里面，也就是所有的项目都共享这个config。

## 3 增加/删除文件

```BASH
# 添加指定文件到暂存区
$ git add [file1] [file2] ...

# 添加指定目录到暂存区，包括子目录
$ git add [dir]

# 添加当前目录的所有文件到暂存区
$ git add .

# 添加每个变化前，都会要求确认
# 对于同一个文件的多处变化，可以实现分次提交
$ git add -p

# 删除工作区文件，并且将这次删除放入暂存区
$ git rm [file1] [file2] ...

# 停止追踪指定文件，但该文件会保留在工作区
$ git rm --cached [file]

# 改名文件，并且将这个改名放入暂存区
$ git mv [file-original] [file-renamed]
```

## 4 代码提交

```BASH
# 提交暂存区到仓库区
$ git commit -m [message]

# 提交暂存区的指定文件到仓库区
$ git commit [file1] [file2] ... -m [message]

# 提交工作区自上次commit之后的变化，直接到仓库区
$ git commit -a

# 提交时显示所有diff信息
$ git commit -v

# 使用一次新的commit，替代上一次提交
# 如果代码没有任何新变化，则用来改写上一次commit的提交信息
$ git commit --amend -m [message]

# 重做上一次commit，并包括指定文件的新变化
$ git commit --amend [file1] [file2] ...
```

> 在所有的版本控制系统中，每一个commit都有4W 要素(Who-When-Why-What)
> - Who: 谁做了这些代码改动，并提交了commit，
> - When: 什么时候做的这个commit （请注意：这个时间是做`git commit`命令的时候，而不是你push的时间）
> - Why: 这个commit是为了什么而做的。也就是commit message
> - What: 列出这个commit究竟做了哪些改动，增删改了哪些文件的内容
>
> 以上4W要素中，when是系统生成的，why是用户自己写的（在提交commit的时候，`commit –m “message”`，其中的message就是why)，what是GIT从暂存区detect出来的，而who是从配置文件读取的。

commit时如果没有加上`-m`参数，系统也会需要输入提交信息，进入编辑器：按`i`进入插入状态，编写提交信息。按`Esc`键退出插入状态，然后`:wq`退出保存即可。

## 5 分支

```BASH
# 列出所有本地分支
$ git branch

# 列出所有远程分支
$ git branch -r

# 列出所有本地分支和远程分支
$ git branch -a

# 新建一个分支，但依然停留在当前分支
$ git branch [branch-name]

# 新建一个分支，并切换到该分支
$ git checkout -b [branch]

# 新建一个分支，指向指定commit
$ git branch [branch] [commit]

# 新建一个分支，与指定的远程分支建立追踪关系
$ git branch --track [branch] [remote-branch]

# 切换到指定分支，并更新工作区
$ git checkout [branch-name]

# 切换到上一个分支
$ git checkout -

# 建立追踪关系，在现有分支与指定的远程分支之间
$ git branch --set-upstream-to [remote-branch]

# 合并指定分支到当前分支
$ git merge [branch]

# 选择一个commit，合并进当前分支
$ git cherry-pick [commit]

# 删除分支
$ git branch -d [branch-name]

# 删除远程分支
$ git push origin --delete [branch-name]
$ git branch -dr [remote/branch]
```

除非将分支推送到远端仓库，不然该分支就是不为他人所见的

## 6 标签

```BASH
# 列出所有tag
$ git tag

# 新建一个tag在当前commit
$ git tag [tag]

# 新建一个tag在指定commit
$ git tag [tag] [commit]

# 删除本地tag
$ git tag -d [tag]

# 删除远程tag
$ git push origin :refs/tags/[tagName]

# 查看tag信息
$ git show [tag]

# 提交指定tag
$ git push [remote] [tag]

# 提交所有tag
$ git push [remote] --tags

# 新建一个分支，指向某个tag
$ git checkout -b [branch] [tag]
```

## 7 查看信息

```BASH
# 显示有变更的文件
$ git status

# 显示当前分支的版本历史
$ git log

# 显示commit历史，以及每次commit发生变更的文件
$ git log --stat

# 搜索提交历史，根据关键词
$ git log -S [keyword]

# 显示某个commit之后的所有变动，每个commit占据一行
$ git log [tag] HEAD --pretty=format:%s

# 显示某个commit之后的所有变动，其"提交说明"必须符合搜索条件
$ git log [tag] HEAD --grep feature

# 显示某个文件的版本历史，包括文件改名
$ git log --follow [file]
$ git whatchanged [file]

# 显示指定文件相关的每一次diff
$ git log -p [file]

# 显示过去5次提交
$ git log -5 --pretty --oneline

# 显示所有提交过的用户，按提交次数排序
$ git shortlog -sn

# 显示指定文件是什么人在什么时间修改过
$ git blame [file]

# 显示暂存区和工作区的差异
$ git diff

# 显示暂存区和上一个commit的差异
$ git diff --cached [file]

# 显示工作区与当前分支最新commit之间的差异
$ git diff HEAD

# 显示两次提交之间的差异
$ git diff [first-branch]...[second-branch]

# 显示今天你写了多少行代码
$ git diff --shortstat "@{0 day ago}"

# 显示某次提交的元数据和内容变化
$ git show [commit]

# 显示某次提交发生变化的文件
$ git show --name-only [commit]

# 显示某次提交时，某个文件的内容
$ git show [commit]:[filename]

# 显示当前分支的最近几次提交
$ git reflog

# 显示远程分支提交历史
$ git log origin/master -n 3
```

## 8 远程同步

```BASH
# 下载远程仓库的所有变动
$ git fetch [remote]

# 显示所有远程仓库
$ git remote -v

# 显示某个远程仓库的信息
$ git remote show [remote]

# 增加一个新的远程仓库，并命名
$ git remote add [shortname] [url]

# 取回远程仓库的变化，并与本地分支合并
$ git pull [remote] [branch]

# 上传本地指定分支到远程仓库
$ git push [remote] [branch]

# 删除远程主机。
$ git remote rm <主机名>

# 远程主机的改名
$ git remote rename <原主机名> <新主机名>

# 强行推送当前分支到远程仓库，即使有冲突
$ git push [remote] --force

# 推送所有分支到远程仓库
$ git push [remote] --all
```

需要记住，`fetch`命令只是将远端的数据拉到本地仓库，并不自动合并到当前工作分支，只有当你确实准备好了，才能手工合并。

可以使用`git pull`命令自动抓取数据下来，然后将远端分支自动合并到本地仓库中当前分支。在日常工作中我们经常这么用，既快且好。实际上，默认情况下`git clone`命令本质上就是自动创建了本地的master分支用于跟踪远程仓库中的master分支（假设远程仓库确实有master分支）。所以一般我们运行`git pull`，目的都是要从原始克隆的远端仓库中抓取数据后，合并到工作目录中的当前分支。

### 8.1 `git fetch`

一旦远程主机的版本库有了更新（Git术语叫做commit），需要将这些更新取回本地，这时就要用到`git fetch`命令。


```BASH
$ git fetch <远程主机名>
```

上面命令将某个远程主机的更新，全部取回本地。

`git fetch`命令通常用来查看其他人的进程，因为它取回的代码对你本地的开发代码没有影响。

默认情况下，`git fetch`取回所有分支（branch）的更新。如果只想取回特定分支的更新，可以指定分支名。

```BASH
$ git fetch <远程主机名> <分支名>
```

比如，取回`origin`主机的`master`分支。

```BASH
$ git fetch origin master
```

所取回的更新，在本地主机上要用"远程主机名/分支名"的形式读取。比如origin主机的master，就要用origin/master读取。

取回远程主机的更新以后，可以在它的基础上，使用`git checkout`命令创建一个新的分支。

```BASH
$ git checkout -b newBrach origin/master
```

上面命令表示，在origin/master的基础上，创建一个新分支。

此外，也可以使用`git merge`命令或者`git rebase`命令，在本地分支上合并远程分支。merge之前需要先却换到需要被merge的分支上。

```BASH
$ git checkout master
$ git merge origin/master

```
上面命令表示在当前分支上，合并origin/master。

### 8.2 `git pull`

`git pull`命令的作用是，取回远程主机某个分支的更新，再与本地的指定分支合并。它的完整格式稍稍有点复杂。

```BASH
$ git pull <远程主机名> <远程分支名>:<本地分支名>
```

比如，取回origin主机的next分支，与本地的master分支合并，需要写成下面这样

```BASH
$ git pull origin next:master
```

如果远程分支是与当前分支合并，则冒号后面的部分可以省略。

```BASH
$ git pull origin next
```

上面命令表示，取回origin/next分支，再与当前分支合并。实质上，这等同于先做`git fetch`，再做`git merge`。

```BASH
$ git fetch origin
$ git merge origin/next
```

如果远程主机删除了某个分支，默认情况下，`git pull`不会在拉取远程分支的时候，删除对应的本地分支。这是为了防止，由于其他人操作了远程主机，导致`git pull`不知不觉删除了本地分支。

但是，你可以改变这个行为，加上参数`-p`就会在本地删除远程已经删除的分支。

```BASH
$ git pull -p
# 等同于下面的命令
$ git fetch --prune origin 
$ git fetch -p
```

### 8.3 `git push`

`git push`命令用于将本地分支的更新，推送到远程主机。

```BASH
$ git push <远程主机名> <本地分支名>:<远程分支名>
```

如果省略远程分支名，则表示将本地分支推送与之存在"追踪关系"的远程分支（通常两者同名），如果该远程分支不存在，则会被新建。

```BASH
$ git push origin master
```

上面命令表示，将本地的master分支推送到origin主机的master分支。如果后者不存在，则会被新建。

如果省略本地分支名，则表示删除指定的远程分支，因为这等同于推送一个空的本地分支到远程分支。

```BASH
$ git push origin :master
# 等同于
$ git push origin --delete master
```

上面命令表示删除origin主机的master分支。

可以使用下面的命令建立本地分支和远程分支之间的追踪关系，这样在`push`的时候就不必每次都输入远程分支的名字，更不会因为忘记输入远程分支名，而master分支未开启保护的情况下，导致误推到master分支上

```BASH
git branch --set-upstream-to=origin/remote_branch  your_branch
```

其中，`origin/remote_branch`是你本地分支对应的远程分支，`your_branch`是你当前的本地分支。

## 9 撤销


```BASH
# 恢复指定文件，工作区没有add的（没有提交到暂存区）的修改
$ git checkout [file]
 
# 恢复所有文件，工作区没有add的（没有提交到暂存区）的修改
$ git checkout .

# 撤销commit，保留工作空间改动代码，保留add
$ git reset --soft [file] [commit] 

# 撤销commit，保留工作空间改动代码，撤销add（不加参数时的默认值)
$ git reset --mixed [file] [commit] 

# 撤销commit，删除工作空间改动代码，撤销add
$ git reset --hard [file] [commit] 

# 新建一个commit，用来撤销指定commit
# 后者的所有变化都将被前者抵消，并且应用到当前分支
$ git revert [commit]

# 暂时将未add的变化移除，存储起来
$ git stash

# 恢复stash的变化
$ git stash pop

# 修改commit注释
$ git commit -amend
```


#### 参考
- [Git远程操作详解@阮一峰的网络日志](http://www.ruanyifeng.com/blog/2014/06/git_remote.html)
- [GIT - 一些基本概念@CSDN](https://blog.csdn.net/notechsolution/article/details/50493702)
- [git使用情景2：commit之后，想撤销commit@CSDN](https://blog.csdn.net/w958796636/article/details/53611133)
