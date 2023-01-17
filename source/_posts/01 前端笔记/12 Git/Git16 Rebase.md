---
title: Git16 Rebase
top: false
date: 2019-08-08 17:31:44
updated: 2019-08-08 17:31:44
tags: 
- Rebase
categories: Git
---

Git rebase 命令学习笔记。

<!-- more -->

## 使用

Rebase用于把一个分支的修改合并到当前分支，这是它最主要的功能。看下面这个例子：

（1）我们从当前的`master`分支切除一个`feature1`分支，进行开发：

```BASH
$ (master) git checkout -b feature
```

![](http://image.oldzhou.cn/FsNZ3hkxEHAnXgf7XffSP0x_Yjkb)

对应Webstorm中的操作：

![](http://image.oldzhou.cn/FudDfZ7MNDC1lFsSltJo3mqiaEHf)

（2）这时候，同事完成了一次提交，并且合并入了`master`分支，这时候`master`已经领先于`feature1`分支了：

![](http://image.oldzhou.cn/FvPRsb8ATfrqZsblbInq2OQxp6cz)

对应Webstorm中的操作：

![](http://image.oldzhou.cn/Fu7SCUDpg2XEvtqEbMROUdrM1TsV)

（4）这个时候，我们要同步`master`的改动，如果使用`merge`操作：

```BASH
$ (feature1) git merger master
```

那么会产生一次新的用于`merge`的提交信息:

![](http://image.oldzhou.cn/Fv3BTdRnGa4pDcvi8yUQFDBUQzLN)

对应Webstorm中的操作：

![](http://image.oldzhou.cn/FheOimNG23WQ8pTvzijRTdTPvO8H)

这一条`commit`记录，我们可以认为它是无效的（也许并不是这样，看你如何看待），想要保持一份干净的`commit`记录，就需要用到`git rebase`命令了

（4）回退到`merge`的状态

![](http://image.oldzhou.cn/FvPRsb8ATfrqZsblbInq2OQxp6cz)

（5）使用`rebase`命令：

```BASH
$ (feature1) git reabse master
```

**对应Webstorm中的操作**：（总是搞不清楚这个操作的主客关系，`Rebase Current onto Selected`的意思就是将当前分支的基准（`feature1`）变为选择的分支（`master`），`master`是基准。

![](http://image.oldzhou.cn/FkeD_vtz0uo8LBChB8l1HwdmA_gM)

`rebase`做了什么操作呢？

1. Git会将`feature1`分支里面每个`commit`取消，临时保存为`patch`文件，保存在`.git/rebase`目录下
2. 把`feature1`分支更新到最新的`master`分支上（也就是「变基」了）
3. 把上面保存的`patch`文件应用到`feature1`分支上

![](http://image.oldzhou.cn/FjUhbg7Y6A7LznYf9dBlIcyY6qhd)

这样的操作后提交记录里面就不会有`merge`的那条记录，而且`feature1`是基于更新后的`master`，自然也就成为了最领先的分支了，可以直接`push`到`master`上。

（6）在`reabse`的过程中，也许会出现冲突`conflict`，这种情况Git会停止`rebase`，让你去解决冲突，解决完冲突用`git add`命令去更新这些内容。

注意，你无需执行`git commit`，只需要执行`git rebase --continue`，Git就会继续应用剩下的`patch`补丁文件


（7）任何情况下，都可以使用`--abort`来终止`rebase`过程，并且分支会回到`rebase`开始前的状态

```BASH
$ git reabse --abort
```

## 应用场景

`git rebase`存在的价值是，对一个分支做「变基」的操作，最常见的应用场景是：**当我们在一个过时的分支上面是，执行`rebase`以此同步`master`分支最新改动**


## 危险性

在提交到代码到远程仓库前，记录是这样的：

![](http://image.oldzhou.cn/FvPRsb8ATfrqZsblbInq2OQxp6cz)

提交之后分支变成了：

![](http://image.oldzhou.cn/FjUhbg7Y6A7LznYf9dBlIcyY6qhd)

但是如果有同事也在`feature1`开发，它的分支依然基于过时的`master`：

![](http://image.oldzhou.cn/Fof8qQxEZJ7C_fX5n6CPNCjtskCE)

那么当它`push`到远程`master`就会有记录丢失，因为`reabse`改变了历史，应该谨慎使用。

所以，**除非你可以肯定改`feature1`分支只有你自己操作，否则请谨慎使用。只要你的分支上需要`rebase`的所有`commits`历史还没有被`push`过，就可以安全的使用`git rebase`来操作**。

## 使用`rebase`和`merge`的基本原则

- 下游分支（`featrue1`）更新上游分支（`master`）内容的时候用`rebase`
- 上游分支合并下游分支内容的时候使用`merge`




## 参考

- [彻底搞懂 Git-Rebase@Jartto's blog](http://jartto.wang/2018/12/11/git-rebase/)
- [GIT使用rebase和merge的正确姿势@知乎](https://zhuanlan.zhihu.com/p/34197548)
