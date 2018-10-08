---
title: git-git常用操作命令记录
date: 2018-10-08 11:07:07
tags: [git github]
categories: git
---


一些日常使用的git命令记录


- 强制更新远程仓库的更新到本地
```
git fetch --all && git reset --hard origin/master && git pull
```

- 创建、检出和删除分支

创建并切换分支
```
git branch -b dev
```

检出分支
```
git checkout dev
```

删除分支
```
git branch -d dev
```

- 合并dev分支到master
```
git checkout master
git merge dev
```


