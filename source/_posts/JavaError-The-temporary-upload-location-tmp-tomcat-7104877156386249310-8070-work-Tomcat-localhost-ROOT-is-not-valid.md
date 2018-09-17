---
title: >-
  JavaError-The-temporary-upload-location-[/tmp/tomcat.7104877156386249310.8070/work/Tomcat/localhost/ROOT]-is-not-valid
date: 2018-02-25 14:13:44
tags: [java,error,tomcat,spring boot]
categories: java
---

# SpringBoot内置Tomcat缓存文件目录被意外删除导致异常

在项目中，一般会将文件临时保存到缓存目录

当时使用
```
File.createTempFile("tmp", ext,
                        (File) request.getServletContext().getAttribute(ServletContext.TEMPDIR))
```

创建临时文件时，项目一直运行正常，然而有一次报异常：

```
org.springframework.web.multipart.MultipartException: Could not parse multipart servlet request; nested exception is java.io.IOException: 
    The temporary upload location [/tmp/tomcat.7104877156386249310.8070/work/Tomcat/localhost/ROOT] is not valid
```

检查文件目录，文件确实不在，检查代码，也未发现问题。实在不知道原因，只有重启了服务器，问题也就不再出现。

 

今天偶然查看官方文档，发现问题所在，也提供了解决方法
```
    If you choose to use Tomcat on CentOS be aware that, by default, a temporary directory is
used to store compiled JSPs, file uploads etc. This directory may be deleted by tmpwatch
while your application is running leading to failures. To avoid this, you may want to customize 
your tmpwatch configuration so that tomcat.* directories are not deleted, or configure
server.tomcat.basedir so that embedded Tomcat uses a different location 
```
 

前往目录 /etc/cron.daily/ 中，修改 tmpwatch 文件：

```
#! /bin/sh
flags=-umc
/usr/sbin/tmpwatch "$flags" -x /tmp/.X11-unix -x /tmp/.XIM-unix \
        -x /tmp/.font-unix -x /tmp/.ICE-unix -x /tmp/.Test-unix \
        -X '/tmp/hsperfdata_*' 10d /tmp \
        -X '/tmp/tomcat.*' 10d /tmp
/usr/sbin/tmpwatch "$flags" 30d /var/tmp
for d in /var/{cache/man,catman}/{cat?,X11R6/cat?,local/cat?}; do
    if [ -d "$d" ]; then
        /usr/sbin/tmpwatch "$flags" -f 30d "$d"
    fi
done
```
 

可以看到添加了一行
```
-X '/tmp/tomcat.*' 10d /tmp
```