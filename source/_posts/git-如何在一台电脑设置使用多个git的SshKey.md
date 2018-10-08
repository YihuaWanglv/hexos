---
title: '[git]如何在一台电脑设置使用多个git的SshKey'
date: 2018-10-08 10:12:00
tags: []
categories: git
---


有时候，我们在同一台机器，会需要检出连接多个git仓库的代码，比如个人的代码库github和公司的代码库gitlab，如果都想生成ssh key来免除输入用户名密码的繁琐，那么就需要针对github和gitlab都生成ssh key。

但是，这两者如果邮箱不同的话，在生成第二个key的时候会覆盖第一个的key，会导致一个用不了。如何解决这个问题呢？

1. 首先，在使用命令生成ssh key的时候，使用命令
```
$ ssh-keygen -t rsa -C "your_email@example.com"
```
的时候，在第一步要求输入key存放地址的时候，不能默认回车，您需要输入一个不同的存放地址，例如
```
/c/Users/xxx/.ssh/id_rsa_github
```
后面的一直回车即可，就可以看到在.ssh目录生成了2个新文件：
```
id_rsa_github.pub
id_rsa_github
```

另外一个同理，只要名称路径不一样即可。

2. 在.ssh目录添加一个文件名称为'config'的文件

编辑该文件，把github和gitlab的对应配置分别写进去，例如：
```
Host github.com
    HostName github.com
    User youusername
    IdentityFile ~/.ssh/id_rsa_github

Host code.aliyun.com
    HostName code.aliyun.com
    User youusername
    IdentityFile ~/.ssh/id_rsa
```

