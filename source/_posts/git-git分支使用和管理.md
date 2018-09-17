---
title: '[git]git分支使用和管理'
date: 2016-06-06 17:37:36
tags: [git, branch, 版本控制, 分支管理]
categories: git
---

## git分支常用命令
```
从master创建dev分支，并checkout dev分支：
git checkout -b dev
此命令相当于：
git branch dev
git checkout dev

回到master，合并dev的更改到master：
git checkout master
git merge dev
合并后删除dev分支：
git branch -d dev

如果直接用git merge dev合并分支，git使用的是fast forward的模式，并不会留下commit历史，如果想要留下commit历史，就要加上--no-ff参数，并使用-m添加commit描述
git merge --no-ff -m "merge with no-ff" dev
```

## 保存临时状态
当你正在dev分支进行开发，还没有开发完成能提交，此时突然有一个线上版本的bug要紧急修复，这个时候你就需要用到git stash功能，保存正在开发的dev分支的状态
```
保存当前开发状态：
git stash

查看当前已保存的stash列表：
git stash list

取出之前保存的stash状态：
git stash pop


```

## 多人协作的工作模式
```
多人协作的工作模式通常是这样：

首先，可以试图用git push origin branch-name推送自己的修改；

如果推送失败，则因为远程分支比你的本地更新，需要先用git pull试图合并；

如果合并有冲突，则解决冲突，并在本地提交；

没有冲突或者解决掉冲突后，再用git push origin branch-name推送就能成功！

如果git pull提示“no tracking information”，则说明本地分支和远程分支的链接关系没有创建，用命令git branch --set-upstream branch-name origin/branch-name。

这就是多人协作的工作模式，一旦熟悉了，就非常简单。

小结

查看远程库信息，使用git remote -v；

本地新建的分支如果不推送到远程，对其他人就是不可见的；

从本地推送分支，使用git push origin branch-name，如果推送失败，先用git pull抓取远程的新提交；

在本地创建和远程分支对应的分支，使用git checkout -b branch-name origin/branch-name，本地和远程分支的名称最好一致；

建立本地分支和远程分支的关联，使用git branch --set-upstream branch-name origin/branch-name；

从远程抓取分支，使用git pull，如果有冲突，要先处理冲突。
```