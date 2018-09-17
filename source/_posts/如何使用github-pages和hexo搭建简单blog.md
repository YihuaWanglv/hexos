---
title: 如何使用github-pages和hexo搭建简单blog
date: 2015-12-31 15:41:30
tags: [git,github pages,hexo,blog]
categories: deploy
---

前言
- github，写代码必备
- hexo，一个很方便的静态blog生成系统
- 还不太熟悉，暂时不放图片

步骤
### 1.首先得有一个github账号
没有的得先上github创建一个
### 2.创建一个repositories
- Repository name的填写格式是xxx.github.io
- xxx要与你github的用户名一致（这样创建好后，可以直接通过xxx.github.io访问）
- 创建Repository后，接着是对它进行设置，点击Repository的“setting”进行设置
- 使用默认设置即可，最后点击“Launch automatic page generator”启动自动生成blog
- 接下来是编辑用户界面，点击绿色的“continue to layouts”按钮配置布局和样式
- 最后，点击“Publish page”，你的页面就公布出来了。

### 3.安装hexo
- 首先安装git
- 安装node
- 安装npm
- 安装hexo
npm install hexo-cli -g
npm install hexo --save

- hexo初始化
**根据自己需要创建一个hexo文件夹
$ hexo init <folder>
$ cd <folder>
$ npm install
新建完成后，指定文件夹的目录如下
.
├── _config.yml
├── package.json
├── scaffolds
├── scripts
├── source
|      ├── _drafts
|      └── _posts
└── themes

- 安装Hexo插件
```
npm install hexo-generator-index --save
npm install hexo-generator-archive --save
npm install hexo-generator-category --save
npm install hexo-generator-tag --save
npm install hexo-server --save
npm install hexo-deployer-git --save
npm install hexo-deployer-heroku --save
npm install hexo-deployer-rsync --save
npm install hexo-deployer-openshift --save
npm install hexo-renderer-marked@0.2 --save
npm install hexo-renderer-stylus@0.2 --save
npm install hexo-generator-feed@1 --save
npm install hexo-generator-sitemap@1 --save
```

- 本地查看效果
执行hexo server命令启动，然后访问localhost:4000查看效果


### 4.创建一篇文章
- git shell进入之前创建的hexo文件夹，hexo new "hello world"创建一篇新的文章

### 5.同步并发布
- hexo g 生成静态文件
- git clone https://github.com/YihuaWanglv/yihuawanglv.github.io.git（替换成你之前在github上创建的Repository的地址）把项目clone到本地
- 复制生成的public文件夹内所有文件到clone到本地的yihuawanglv.github.io文件夹内

### 6.提交更改到github
- git add *
- git commit -m "代码提交信息"
- git push origin master
- 输入用户名密码

这样再次访问你github上的blog地址，即可发现内容已更新

### 7.使用hexo别的主题美化blog样式
- 我这里选用的是简介美观的next主题
- 在终端窗口下，定位到 Hexo 站点目录下
$ cd your-hexo-site
$ git clone https://github.com/iissnan/hexo-theme-next themes/next
- 启用 NexT 主题
克隆/下载 完成后，打开 站点配置文件，找到 theme 字段，并将其值更改为 next。
- 验证主题是否启用
运行 hexo s --debug，并访问 http://localhost:4000，确保站点正确运行

### 8.同步新的更改
现在要把新的更改同步上github，重新执行步骤5和6即可。
当然把更新从hexo public文件夹同步到本地的Repository xxx.github.io文件夹内可以使用beyongcompare此类同步软件执行