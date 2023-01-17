---
title: Git13 撤销提交
top: false
date: 2017-12-04 18:11:00
updated: 2020-03-04 09:29:54
tags:
- checkout
- reset
- revert
categories: Git
---

Git中撤销提交的各种方法。

<!-- more -->

# Git中进行撤销操作

Git强大的撤销、版本回退功能，让我们在开发的过程中能够随意的回到任何一个时间点的状态

![](http://image.oldzhou.cn/Fphg-ofFVaA98cWq5GLUpO_wFr5I)

## 撤销工作区的代码

在未提交之前，想将代码恢复到一开始的状态，可以使用

```BASH
git checkout -- <file>
```

命令来撤销工作区的代码修改

![](http://image.oldzhou.cn/FsM2ie1CGrwkz5nmyDQEpkPrfZ3r)

在Webstorm中，对应的命令就是`revert`命令（新版本中改为了`rollback`命令）

## 撤销`add`到暂存区的代码

add到暂存区的代码想要撤销分为两个步骤：

1. 将暂存区的代码撤销到工作区
2. 将工作区的代码撤销（具体操作和撤销工作区的代码相同）

撤销暂存区的代码到工作区，使用的命令是：

```BASH
git reset HEAD
```

`reset`会将当前`head`的内容重置，不会留任何痕迹。

![](http://image.oldzhou.cn/Fq9cWUn9Z1RtFsFXij5QUo1f8Cpf)

## 利用`Reset`撤销本地代码

提交到本地仓库的代码一样也可以撤销，我们可以利用`git reset --mixed <版本号>`命令来实现版本回退，该命令中的版本号有几种不同的写法：

1. 可以使用`HEAD^`来描述版本，一个`^`表示前一个版本，两个`^^`表示前两个版本，以此类推。
2. 也可以使用数字来代替`^`，比如说前`100`个版本可以写作`HEAD~100`。
3. 也可以直接写版本号，表示跳转到某一个版本处。我们每次提交成功后，都会生成一个哈希码作为版本号，所以这里我们也可以直接填版本号，哈希码很长，但是我们不用全部输入，只需要输入前面几个字符即可，就能识别出来。

在Wetstorm中对应的命令是`Git - Repository RESET Head`，输入对应的版本号就可以退回到当时的版本状态

`git reset`有三种模式：

1. `git reset - mixed`：此为默认方式，将本地版本库的头指针全部重置到指定版本，且会重置暂存区，即这次提交之后的所有变更都移动到未暂存阶段
2. `git reset - soft`： 回退到某个版本，本地版本库的头指针全部重置到指定版本，且将这次提交之后的所有变更都移动到暂存区
3. `git reset - hard`：不仅仅是将本地版本库的头指针全部重置到指定版本，也会重置暂存区，并且会将工作区代码也回退到这个版本

## 利用`revert`撤销本地代码

```BASH
git revert <commidId>
```

利用`reset`是直接改写之前的历史，传入的`commitId`意味着，这之后的`commit`都直接被消除。

`revert`则不同，传入的`commitId`意味着，重新生成一次提交，将`commitId`对应的`coomit`修改会`commit`之前的状态。即利用了一次新的`commit`来回滚之前的操作。

上面的这种写法每次只能处理一个`commit`，如果想要处理多个`commit`，可以按照下面的写法：

```BASH
git revert <olderCommitId>^..<newerCommitId>
```

举个例子，`commit`的历史下如下：

```BASH
A -> B -> C -> D
```

如果想要`revert`的`commit`是B、C、D的话，除了一个一个来，可以这样：

```
git revert B^..D
```

这样`commit`的历史就变成了

```BASH
A -> B -> C -> D  -> D' -> C' -> B'
```

如果想把这三次`coommit`用一次提交来`revert`，可以这样：

```
git revert -n B^..D
git commit -m 'revert commit B to commit D'
```

## `reset`和`revert`的应用场景

- 如果回退的代码以后还需要，那么最好使用`revert`
- 如果分支错误，以后在不需要了，也不想保留记录，使用`reset`
- 如果合并了`a`/`b`/`c`/`d`四个分支，只希望撤销`b`分支，保留`c`和`d`分支，那么不能使用`reset`，需要`revert b`这样可以实现

## 撤销远程仓库代码

如果代码已经提交到了远程仓库，那么在使用`reset`本地撤销提交之后，在向远程仓库`push`的时候，本地的分支是落后于远程分支的，那么`push`会被拒绝，需要使用`--force`命令：

```BASH
git push origin dev --force
```

这种情况对于一些多人合作开发的上游分支来说，是危险且很可能被禁止的，所以需要如果代码已经提交到了远程仓库的情况下，更推荐使用`revert`命令来撤销本地提交，继而变相的撤销远程仓库的代码。

因为`revert`是利用新的『相反』的提交，撤销了之前的代码，是一次新的`commit`，所以可以直接推送到远程仓库，从而撤销远程仓库的代码。

## 参考

- [git reset 和 git revert@掘金](https://juejin.im/post/5b0e5adc6fb9a009d82e4f20)
- [git revert@git](https://git-scm.com/docs/git-revert)
- [git revert多个提交@CSDN](https://blog.csdn.net/zhangxiaoyang0/article/details/90213597)C]
