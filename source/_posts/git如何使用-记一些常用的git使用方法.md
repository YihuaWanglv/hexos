---
title: git如何使用-记一些常用的git使用方法
date: 2017-03-11 15:47:31
tags: [git, github, 使用方法]
---

1. git如何提交修改到远程仓库？
    - $ git commit –m "desc.."  //对你更新或修改了哪些内容做一个描述
    - $ git remote add origin https://github.com/YihuaWanglv/post.git
    //如果你是第一次提交项目，这一句非常重要，这是你本地的当前的项目与远程的哪个仓库建立连接。
    - $ git push -u origin master  //将本地的项目提交到远程仓库中

2. 如果你是第一次想把github上面的项目克隆到本地或者要克隆别人的项目到地。
    - $ git clone git@github.com:defnngj/hibernate-demo.git  //在git下面切换到想存放此项目的文件目录下，运行这条命令就可以将项目克隆下来。

3. 假如本地已经存在了这个项目，而仓库中又有一新的更新，如何把更的合并到本地的项目中？
    - $ git fetch origin    //取得远程更新，这里可以看做是准备要取了
    - $ git merge origin/master  //把更新的内容合并到本地分支/master

4. 添加和提交
    - git add *
    - git commit -m "代码提交信息"
    - git push origin master
    - 输入用户名密码
    - 如果你还没有克隆现有仓库，并欲将你的仓库连接到某个远程服务器，你可以使用如下命令添加：
    - git remote add origin <server>
5. 重装系统后，git项目如何恢复？
    - 本地先进入原项目路径，git clone [path of project], 连接上
    - 然后告诉git全局配置你是谁，输入你的邮箱和名称
    - git config --global user.email "you@example.com"
    - git config --global user.name "Your Name"
    - 然后该干嘛干嘛


