---
title: '[git]如何把本地仓库中使用https连接方式的库改为使用ssh'
date: 2018-10-08 10:13:04
tags: [git github gitlab ssh]
categories: git
---


有时候，我们在检出github或者gitlab的repository时，原先是使用https的方式检出的。现在想把原先这个使用https方式检出的库改为使用ssh连接，要如何做呢？

很简单，使用一条命令即可：
```

git remote set-url origin git@github.com:username/repo-name-here.git

```
