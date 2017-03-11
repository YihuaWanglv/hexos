---
title: shell-从svn更新代码maven打包后部署的一个简单脚本
date: 2017-03-11 15:48:45
tags: [shell, sh, spring boot, java, linux]
---

shell是个好东西，服务器上面的操作做多了，就需要想想办法把手工做的事情让脚本帮我们处理。以下是一个针对spring boot项目从svn更新出最新代码，然后maven打包，最后启动发布的一个脚本例子：

```
#!/bin/bash
#build icms app
path_src=/home/src/trunk/icms
path_target=${path_src}/target
path_app=/home/app
path_log=/data/logs/icms
file_app=icms-1.4.1.RELEASE.jar
url_app=localhost:8091

cd /home/src/trunk/icms
svn update
mvn clean package -P pro -Dmaven.test.skip=true

if [ -f ${path_target}/${file_app} ]; then
        echo "build success, now begin to deploy..."
        curl -X POST ${url_app}/shutdown
        yes | cp -rf ${path_target}/${file_app} ${path_app}/
        chmod +x ${path_app}/${file_app}
        nohup java -jar ${path_app}/${file_app} < /dev/null > ${path_log}/icms.log 2>&1 &
else
        echo "build failed!"
fi

echo "build process finish."
```

脚本中的“curl -X POST ${url_app}/shutdown”这里是因为使用了spring boot的优雅关闭应用，需要在应用添加maven依赖
```
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>
```

然后在配置文件中添加：
```
endpoints.shutdown.enabled=true
endpoints.shutdown.sensitive=false
```