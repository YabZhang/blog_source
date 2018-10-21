---
title: revert一个merge提交
date: 2018-08-20 00:27:48
tags:
 -- git
---

`git`是最常用的代码版本管理工具，撤销已经提交的代码的操作包括：`git checkout`、`git reset`、`git revert`。我再使用`revert`撤销一个合并后，遇到了一些特别的情况。

<!--more-->

### 区别

1. `git checkout` 主要用于本地代码库的分支切换，和调到指定的提交快照。但并不改变分支的记录和`HEAD`的位置。

2. `git reset` 主要用于本地代码的撤销操作，是可以改变提交历史的和`HEAD`指针位置的。例如：`git reset <commit-hash>`，将会把当前分支到指定的提交前全部撤销，并改变分支`HEAD`指针的位置。这个命令可能具有破坏性，请谨慎使用。对于丢失的提交记录可以尝试使用`git reflog`来找回。

3. `git revert`的效果是增加一个提交(`HEAD` 自然前移一个位置)，`抵消`掉要回退的代码提交，但不改变已有的提交历史，注意这个命令与`git reset`的区别。


### `revert`一个合并提交

`git checkout`和`git reset`的使用请参考手册和各种资料。这里记录下我再使用`revert`时遇到的一个特别情况。

`git revert <commit-hash>`的效果是新增一个提交，用于抵消掉某个历史提交。具体的逻辑就是在这个提交里添加历史提交相反的代码改动来回退代码，但不会改变已有的历史。

一个特别的情况: 有一个功能分支`feature-a`在合并到`master`后，发现新功能有隐藏的bug。
所以在`master`上使用`git revert`来回退这个合并，并在`feature-a`上尽心修复。
待修复胡再尝试将`feature-a`合并到`master`会因为`revert`而无法改变当前代码。

原因就是`revert`并没有改变历史，所以在`master`历史上是有新功能的提交的（但已经被回退了），所以简单的再次合并不能生效。

在网络上找到了`linus`回答的解决方案：

1. 在`master`上再进行一次`revert`操作，用于回退第一次的`revert`那个提交。然后再合并修复后的`feature-a`到`master`；

2. 因为使用`revert`是保留历史的，所以需要重新构造已有的历史提交，尝试再次合并。简单来说就是把被撤销的代码放在新的提交中再合并;具体的做法很多，比如可以在修复前，从原有的`feature-a`重新生成新分支（`git rebase --no-ff`，会创建新的历史，提交的hash会改变），做修复操作后再合并会`master`就不会有问题。

在文章中，linus给出的建议是优先方法2。因为方法1的做法不是正规的工作流方法(理解可能有偏差，文后有链接，请自行查看)。

### 参考资料

1. https://git-scm.com/docs/git-cherry-pick Git资源
2. https://www.atlassian.com/git/tutorials/learn-git-with-bitbucket-cloud Git资源
3. https://opensource.apple.com/source/Git/Git-26/src/git-htmldocs/howto/revert-a-faulty-merge.txt linus的回答
4. https://mijingo.com/blog/reverting-a-git-merge revert一个合并提交

