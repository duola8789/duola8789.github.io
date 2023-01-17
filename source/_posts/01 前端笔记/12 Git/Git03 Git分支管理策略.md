---
title: Git03 Git分支管理策略
top: false
date: 2019-03-21 17:23:00
updated: 2019-07-05 11:15:00
tags: 
- Git分支
categories: Git
---

Git分支管理策略学习笔记。

<!-- more -->

## 1 创建与合并分支

在版本回退里，每次提交，Git都把它们串成一条时间线，这条时间线就是一个分支。在Git里，这个分支叫主分支，即master分支。HEAD严格来说不是指向提交，而是指向master，master才是指向提交的，所以，HEAD指向的就是当前分支。

一开始的时候，master分支是一条线，Git用master指向最新的提交，再用HEAD指向master，就能确定当前分支，以及当前分支的提交点：

![](http://image.oldzhou.cn/FtDNEAit9UXkDRZxmvg2uj4Percs)

每次提交，master分支都会向前移动一步，这样，随着你不断提交，master分支的线也越来越长。

当我们创建新的分支，例如dev时，Git新建了一个指针叫dev，指向master相同的提交，再把HEAD指向dev，就表示当前分支在dev上：

```BASH
$ git checkout -b dev

# 创建dev分支，然后切换到dev分支
# git checkout命令加上-b参数表示创建并切换，相当于以下两条命令：
# $ git branch dev
# $ git checkout dev
```

![](http://image.oldzhou.cn/Fm-aOAweu1P2wQS41gHFGzrPxioP)

用`git branch`命令查看当前分支，`git branch`命令会列出所有分支，当前分支前面会标一个`*`号。

```BASH
$ git branch
* dev
  master
```

从现在开始，对工作区的修改和提交就是针对dev分支了，对`readme.txt`做个修改，新提交一次后，dev指针往前移动一步，而master指针不变：

```
$ git add readme.txt 

$ git commit -m "branch test"

[dev fec145a] branch test
 1 file changed, 1 insertion(+)
```

![](http://image.oldzhou.cn/FgeFdR8bsbpSA9t6JXUmLVHdzRUy)

现在，dev分支的工作完成，我们就可以切换回master分支：

```BASH
$ git checkout master
Switched to branch 'master'
```
切换回master分支后，再查看`readme.txt`文件，刚才添加的内容不见了！因为那个提交是在dev分支上，而master分支此刻的提交点并没有变：

![](http://image.oldzhou.cn/Fj9dvKw67u4j5UZYrAfF4DlHvTrG)

现在，我们把dev分支的工作成果合并到master分支上：

```BASH
$ git merge dev

Updating d17efd8..fec145a
Fast-forward
 readme.txt |    1 +
 1 file changed, 1 insertion(+)
```

`git merge`命令用于合并指定分支到当前分支。合并后，再查看`readme.txt`的内容，就可以看到，和dev分支的最新提交是完全一样的。

注意到上面的Fast-forward信息，Git告诉我们，这次合并是“快进模式”，也就是直接把master指向dev的当前提交，所以合并速度非常快。

当然，也不是每次合并都能Fast-forward。

![](http://image.oldzhou.cn/Fq8koWFV3zo_IBJtB2zzvquKIruf)

合并完分支后，甚至可以删除dev分支。删除dev分支就是把dev指针给删掉，删掉后，我们就剩下了一条master分支

```BASH
$ git branch -d dev
Deleted branch dev (was fec145a).
```

删除后，查看branch，就只剩下master分支了：

```BASH
$ git branch
* master
```
## 2 解决冲突

对master分支和feature1分支下面的`README.md`同时进行修改后`add`并且`commit`，返回master分支。

现在，master分支和feature1分支各自都分别有新的提交，变成了这样：

![](http://image.oldzhou.cn/FnsyeI6-7N6oGqNqT8O9S9xvtuJs)

这种情况下，Git无法执行“快速合并”，只能试图把各自的修改合并起来，但这种合并就可能会有冲突，我们试试看：

```BASH
$ git merge feature1

Auto-merging readme.txt
CONFLICT (content): Merge conflict in readme.txt
Automatic merge failed; fix conflicts and then commit the result.
```

对冲突的文件进行手动修改后，再次提交

```BASH
$ git add readme.txt 
$ git commit -m "conflict fixed"
[master 59bc1cb] conflict fixed
```
现在，master分支和feature1分支变成了下图所示：

![](http://image.oldzhou.cn/FjMXla8YRl8IxQ44CuFQnkEHFtx7)

用带参数的git log也可以看到分支的合并情况：

```BASH
$ git log --graph --pretty=oneline --abbrev-commit
*   59bc1cb conflict fixed
|\
| * 75a857c AND simple
* | 400b400 & simple
|/
* fec145a branch test
```

最后删除feature1分支

```BASH
$ git branch -d feature1
Deleted branch feature1 (was 75a857c).
```

## 3 `git merge`的`--no-ff`方式

通常，合并分支时，如果可能，Git会用Fast forward模式，但这种模式下，删除分支后，会丢掉分支信息。

如果要强制禁用Fast forward模式，Git就会在merge时生成一个新的commit，这样，从分支历史上就可以看出分支信息。

下面我们实战一下--no-ff方式的`git merge`：

创建分支dev并进行修改`add`、`commit`后，准备合并dev分支，请注意`--no-ff`参数，表示禁用`Fast forward`：

```BASH
$ git merge --no-ff -m "merge with no-ff" dev

Merge made by the 'recursive' strategy.
 readme.txt |    1 +
 1 file changed, 1 insertion(+)
```
因为本次合并要创建一个新的commit，所以加上`-m`参数，把commit描述写进去。

合并后，我们用git log看看分支历史：

```BASH
$ git log --graph --pretty=oneline --abbrev-commit

*   7825a50 merge with no-ff
|\
| * 6224937 add merge
|/
*   59bc1cb conflict fixed

```

![](http://image.oldzhou.cn/FuXD-HIQ4NrjEefzXCdvppXCRrGW)

在实际开发中，应当注意，master分支应该是非常稳定的，也就是仅用来发布新版本，平时不能在上面干活

那在哪干活呢？干活都在dev分支上，也就是说，dev分支是不稳定的，到某个时候，比如1.0版本发布时，再把dev分支合并到master上，在master分支发布1.0版本；

你和你的小伙伴们每个人都在dev分支上干活，每个人都有自己的分支，时不时地往dev分支上合并就可以了。

所以，团队合作的分支看起来就像这样：

![](http://image.oldzhou.cn/Fs-YGVKi7Ro_tL5ISWK0bH-uFQMT)

## 4 `git stash`

Git还提供了一个stash功能，可以把当前工作现场“储藏”起来，等以后恢复现场后继续工作：

```BASH
$ git stash
Saved working directory and index state WIP on dev: 6224937 add merge
HEAD is now at 6224937 add merge
```

现在，用`git status`查看工作区，就是干净的（除非有没有被Git管理的文件），因此可以放心地创建分支来修复bug。

首先确定要在哪个分支上修复bug，假定需要在master分支上修复，就从master创建临时分支：


```BASH
$ git checkout master
Switched to branch 'master'
Your branch is ahead of 'origin/master' by 6 commits.

$ git checkout -b issue-101
Switched to a new branch 'issue-101'
```

现在修复bug，然后提交：

```BASH
$ git add readme.txt 
$ git commit -m "fix bug 101"

[issue-101 cc17032] fix bug 101
 1 file changed, 1 insertion(+), 1 deletion(-)
```
修复完成后，切换到master分支，并完成合并，最后删除issue-101分支：

```BASH
$ git checkout master
Switched to branch 'master'
Your branch is ahead of 'origin/master' by 2 commits.

$ git merge --no-ff -m "merged bug fix 101" issue-101
Merge made by the 'recursive' strategy.
 readme.txt |    2 +-
 1 file changed, 1 insertion(+), 1 deletion(-)
$ git branch -d issue-101
Deleted branch issue-101 (was cc17032).
```
现在，是时候接着回到dev分支干活了！

```BASH
$ git checkout dev
Switched to branch 'dev'

$ git status
# On branch dev
nothing to commit (working directory clean)
```
工作区是干净的，刚才的工作现场存到哪去了？用`git stash list`命令看看：

```BASH
$ git stash list
stash@{0}: WIP on dev: 6224937 add merge
```
工作现场还在，Git把stash内容存在某个地方了，但是需要恢复一下，有两个办法：

1. 用`git stash apply`恢复，但是恢复后，stash内容并不删除，你需要用`git stash drop`来删除；
2. 用`git stash pop`，恢复的同时把stash内容也删了：

```bash
$ git stash pop

On branch dev
Changes to be committed:
   (use "git reset HEAD <file>..." to unstage)

       new file:   hello.py

Changes not staged for commit:
   (use "git add <file>..." to update what will be committed)
   (use "git checkout -- <file>..." to discard changes in working directory)

       modified:   readme.txt

Dropped refs/stash@{0} (f624f8e5f082f2df2bed8a4e09c12fd2943bdd40)
```

再用·git stash list·查看，就看不到任何stash内容了：

```BASH
$ git stash list
```

你可以多次stash，恢复的时候，先用`git stash list`查看，然后恢复指定的stash，用命令：

```BASH
$ git stash apply stash@{0}
```

## 5 推送分支

推送分支，就是把该分支上的所有本地提交推送到远程库。推送时，要指定本地分支，这样，Git就会把该分支推送到远程库对应的远程分支上：

```BASH
$ git push origin master
```

如果要推送其他分支，比如dev，就改成：

```BASH
$ git push origin dev
```

但是，并不是一定要把本地分支往远程推送，那么，哪些分支需要推送，哪些不需要呢？

- master分支是主分支，因此要时刻与远程同步；
- dev分支是开发分支，团队所有成员都需要在上面工作，所以也需要与远程同步；
- bug分支只用于在本地修复bug，就没必要推到远程了，除非老板要看看你每周到底修复了几个bug；
- feature分支是否推到远程，取决于你是否和你的小伙伴合作在上面开发。

总之，就是在Git中，分支完全可以在本地自己藏着玩，是否推送，视你的心情而定！

## 6 抓取分支

多人协作时，大家都会往master和dev分支上推送各自的修改。

当从远程库clone时，默认情况下，只能看到本地的master分支。不信可以用git branch命令看看：、

```BASH
$ git branch
master
```

现在要在dev分支上开发，就必须创建远程origin的dev分支到本地，用这个命令创建本地dev分支，本地和远程分支的名称最好一致：

```BASH
git checkout -b dev origin/dev
```

现在他就可以在dev上继续修改，然后，时不时地把dev分支push到远程

```BASH
git push origin branch-name
```

如果推送失败，则因为远程分支比你的本地更新，需要先用`git pull`试图合并

```BASH
$ git pull
```

如果`git pull`提示“no tracking information”，原因是没有指定本地dev分支与远程origin/dev分支的链接，根据提示，设置dev和origin/dev的链接：

```BASH
$ git branch --set-upstream branch-name origin/branch-name
```

如果合并有冲突，则解决冲突，并在本地提交；

没有冲突或者解决掉冲突后，再用`git push origin branch-name`推送就能成功！


## 7 分支管理

### 7.1 主分支Master

首先，代码库应该有一个、且仅有一个主分支。所有提供给用户使用的正式版本，都在这个主分支上发布。

![](http://image.oldzhou.cn/FtK1mff22zp-WzhfzWoejeKB5UeU)

Git主分支的名字，默认叫做Master。它是自动建立的，版本库初始化以后，默认就是在主分支在进行开发。

### 7.2 开发分支Develop

主分支只用来分布重大版本，日常开发应该在另一条分支上完成。我们把开发用的分支，叫做Develop。

![](http://image.oldzhou.cn/FjtgjGeo7hhX5KDuMk8KfaKmeGzp)

这个分支可以用来生成代码的最新隔夜版本（nightly）。如果想正式对外发布，就在Master分支上，对Develop分支进行"合并"（merge）。

Git创建Develop分支的命令

```BASH
git checkout -b develop master
```

将Develop分支发布到Master分支的命令

```BASH
git checkout master
git merge --no-ff develop
```

上一条命令的`--no-ff`参数是什么意思。默认情况下，Git执行"快进式合并"（fast-farward merge），会直接将Master分支指向Develop分支。

![](http://image.oldzhou.cn/Fkd-PjXcm7s04D4OHmMzGLxOxKs-)

使用`--no-ff`参数后，会执行正常合并，在Master分支上生成一个新节点。为了保证版本演进的清晰，我们希望采用这种做法。

![](http://image.oldzhou.cn/Fh2rIG6IkHJTJbfPrJSmTpWNcqf-)

### 7.3 临时性分支

前面讲到版本库的两条主要分支：Master和Develop。前者用于正式发布，后者用于日常开发。其实，常设分支只需要这两条就够了，不需要其他了。

但是，除了常设分支以外，还有一些临时性分支，用于应对一些特定目的的版本开发。临时性分支主要有三种：

- 功能（feature）分支
- 预发布（release）分支
- 修补bug（fixbug）分支

这三种分支都属于临时性需要，使用完以后，应该删除，使得代码库的常设分支始终只有Master和Develop。

### 7.4 功能分支

接下来，一个个来看这三种"临时性分支"。

第一种是功能分支，它是为了开发某种特定功能，从Develop分支上面分出来的。开发完成后，要再并入Develop。

![](http://image.oldzhou.cn/FpOxTxsWSN2YLmooi_s8yYHr1_6I)

功能分支的名字，可以采用`feature-*`的形式命名。

创建一个功能分支：

```BASH
git checkout -b feature-x develop
```

开发完成后，将功能分支合并到develop分支：

```BASH
git checkout develop
git merge --no-ff feature-x
```

删除feature分支：

```BASH
git branch -d feature-x
```

### 7.5 预发布分支

预发布分支是指发布正式版本之前（即合并到Master分支之前），我们可能需要有一个预发布的版本进行测试。

预发布分支是从Develop分支上面分出来的，预发布结束以后，必须合并进Develop和Master分支。它的命名，可以采用`release-*`的形式。

创建一个预发布分支：

```BASH
git checkout -b release-1.2 develop
```

确认没有问题后，合并到`master`分支：

```BASH
git checkout master
git merge --no-ff release-1.2

# 对合并生成的新节点，做一个标签
git tag -a 1.2
```

开发完成后，合并到develop分支：

```BASH
git checkout develop
git merge --no-ff release-1.2
```
删除预发布分支：

```BASH
git branch -d release-1.2
```

### 7.6  修补bug分支

最后一种是修补bug分支。软件正式发布以后，难免会出现bug。这时就需要创建一个分支，进行bug修补。

修补bug分支是从Master分支上面分出来的。修补结束以后，再合并进Master和Develop分支。它的命名，可以采用`fixbug-*`的形式。

![](http://image.oldzhou.cn/Fr-cWgluU7JPlIu4ReEWL8X1TdF9)

创建一个修补bug分支：

```BASH
git checkout -b fixbug-0.1 master
```

修补结束后，合并到master分支：

```BASH
git checkout master
git merge --no-ff fixbug-0.1
git tag -a 0.1.1
```

再合并到develop分支：

```BASH
git checkout develop
git merge --no-ff fixbug-0.1
```

最后，删除"修补bug分支"：

```BASH
git branch -d fixbug-0.1
```

## 参考

- [Git分支管理策略@阮一峰的网络日志](http://www.ruanyifeng.com/blog/2012/07/git.html)
- [创建与合并分支@廖雪峰的官方网站](https://www.liaoxuefeng.com/wiki/896043488029600/900003767775424)
- [解决冲突@廖雪峰的官方网站](https://www.liaoxuefeng.com/wiki/896043488029600/900004111093344)
- [分支管理策略@廖雪峰的官方网站](https://www.liaoxuefeng.com/wiki/896043488029600/900005860592480)
