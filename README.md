# myhexo

##my hexo local project for github  blog
在这个项目编辑文章，可以直接发布到Repository https://github.com/YihuaWanglv/yihuawanglv.github.io.git

### 1.npm安装需要的东西
$ npm install hexo-deployer-git --save
$ npm install hexo-deployer-heroku --save
$ npm install hexo-deployer-rsync --save
$ npm install hexo-deployer-openshift --save
$ npm install hexo-deployer-ftpsync --save

### 2.修改配置文件_config.yml
deploy:
  type: git
  repository: https://github.com/YihuaWanglv/yihuawanglv.github.io.git
  branch: master

### 3.执行命令提交发布
$ hexo clean
$ hexo generate
$ hexo deploy

