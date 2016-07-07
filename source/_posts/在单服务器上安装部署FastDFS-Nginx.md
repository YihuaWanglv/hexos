---
title: 在单服务器上安装部署FastDFS+Nginx
date: 2016-07-07 10:54:21
tags: [linux, centos, FastDFS, Nginx, 图片服务器]
---

# 在单服务器上安装部署FastDFS+Nginx

那么多服务器和软件安装配置中，FastDFS算是比较复杂的一个了。
这个例子storage和tracker均部署在同一台服务器（ip：192.168.1.134）
我部署的服务系版本是centos6.7

## FastDFS安装配置

### Tracker的安装及配置
#### 1.安装编译器
```
yum install -y gcc gcc-c++
```

#### 2.下载安装libevent
```
wget https://sourceforge.net/projects/levent/files/libevent/libevent-2.0/libevent-2.0.22-stable.tar.gz
tar xvzf libevent-2.0.22-stable.tar.gz
cd libevent-2.0.22-stable
./configure
make && make install
```

#### 3.下载安装fastDFS
```
wget http://code.google.com/p/fastdfs/downloads/detail?name=FastDFS_v4.06.tar.gz
（code.google.com已无法访问，可以自己手动下载再上传到服务器）
tar -xvzf FastDFS_v4.06.tar.gz
cd FastDFS
./make.sh
./make.sh install
```
安装成功后/usr/local/bin下会出现一系列fastDFS命令

#### 4.配置tracker
```
vim /etc/fdfs/tracker.conf
```
修改base_path以存储tracker信息（这里不做修改，使用默认路径/home/yuqing/fastdfs，需要先行创建目录）

#### 5.启动tracker
```
/usr/local/bin/fdfs_trackerd /etc/fdfs/tracker.conf
```

### Storage配置 -> Storage1配置
正常情况下storage1部署一个服务器，storage2部署一个服务器，tracker部署一个服务器。
我这里只部署一个storage一个tracker，并部署在同一个服务器

#### 1~3：参考Tracker安装步骤

#### 4.配置storage
```
vim /etc/fdfs/storage.conf
```
修改tracker_server=192.168.1.134:22122
修改base_path以存储storage信息（这里不做修改，使用默认路径/home/yuqing/fastdfs，需要先行创建目录）
例子：
```
group_name=group1
base_path=/home/yuqing/fastdfs
store_path0=/home/yuqing/fastdfs
tracker_server=192.168.1.134:22122
http.server_port=8080      #web server的端口改成8080（与nginx 端口一致）
```

#### 5.启动storage
```
/usr/local/bin/fdfs_storaged /etc/fdfs/storage.conf
```
启动过程中，fastDFS会在base_path下的data目录中创建一系列文件夹，以存储数据

### 安装nginx以及fastdfs-nginx-module模块
nginx的rewrite模块和cache模块需要先安装pcre和openssl
#### 安装pcre
下载pcre
```
tar zxvf pcre-8.12.tar.gz
cd pcre-8.12
./configure
make
make install
```

#### 安装openssl
centos下解决办法:
```
yum -y install openssl openssl-devel
```

#### 安装nginx
```
cd /home/yihua
wget http://nginx.org/download/nginx-0.8.55.tar.gz
tar zxvf nginx-0.8.55.tar.gz
cd nginx-0.8.55
./configure --prefix=/opt/nginx --with-http_stub_status_module
make && make install
```

#### 安装fastdfs-nginx-module
下载并上传fastdfs-nginx-module_v1.15.tar.gz
```
tar xzf fastdfs-nginx-module_v1.15.tar.gz
cd /home/yihua/nginx-0.8.55 
./configure --add-module=/home/yihua/fastdfs-nginx-module/src
make; make install
```

#### 配置nginx和fastdfs-nginx-module
```
cp /home/yihua/fastdfs-nginx-module/src/mod_fastdfs.conf /etc/fdfs
vim /etc/fdfs/mod_fastdfs.conf
```
修改 fastdfs的nginx模块的配置文件 mod_fastdfs.conf
一般只需改动以下几个参数即可：
```
base_path=/home/yuqing/fastdfs      #保存日志目录
tracker_server=192.168.1.134:22122    #tracker 服务器的 IP 地址以及端口号
storage_server_port=23000            #storage 服务器的端口号
group_name=group1                    #当前服务器的 group 名
url_have_group_name = true           #文件 url 中是否有 group 名
store_path_count=1                   #存储路径个数，需要和 store_path 个数匹配
store_path0=/home/yuqing/fastdfs    #存储路径
http.need_find_content_type=true     # 从文件 扩展 名查 找 文件 类型 （ nginx 时 为true）
group_count = 1                      #设置组的个数
```
然后在末尾添加分组信息，目前只有一个分组，就只写一个
```
[group1]
group_name=group1
storage_server_port=23000
store_path_count=1
store_path0=/home/yuqing/fastdfs
```

建立 M00 至存储目录的符号连接
```
ln -s /home/yuqing/fastdfs/data /home/yuqing/fastdfs/data/M00
```
将 server 段中的 listen 端口号改为 8080：
```
vim /usr/local/nginx/conf/nginx.conf
listen       8080;
```
在 server 段中添加fastdfs的配置
```
        location /group1/M00 {
               root   /home/yuqing/fastdfs/data;
               ngx_fastdfs_module;
        }
```

#### 准备nginx启动脚本
编辑 /etc/init.d/nginx，如下内容:
```
#!/bin/bash
# nginx Startup script for the Nginx HTTP Server
# it is v.1.4.7 version.
# chkconfig: - 85 15
# description: Nginx is a high-performance web and proxy server.
# It has a lot of features, but it's not for everyone.
# processname: nginx
 
# pidfile: /usr/local/nginx/logs/nginx.pid
# config: /usr/local/nginx/conf/nginx.conf
 
nginxd=/usr/local/nginx/sbin/nginx
nginx_config=/usr/local/nginx/conf/nginx.conf
nginx_pid=/usr/local/nginx/logs/nginx.pid
nginx_lock=/var/lock/subsys/nginx
RETVAL=0
prog="nginx"
 
# Source function library.
. /etc/rc.d/init.d/functions
 
# Source networking configuration.
. /etc/sysconfig/network
 
# Check that networking is up.
[ ${NETWORKING} = "no" ] && exit 0
[ -x $nginxd ] || exit 0
 
# Start nginx daemons functions.
start() {
    nginx_is_run=`ps -ef | egrep 'nginx:\s*(worker|master)\s*process' | wc -l`
    if [ ${nginx_is_run} -gt 0 ];then
        echo "nginx already running...."
        exit 1
    fi
    echo -n $"Starting $prog: "
    daemon $nginxd -c ${nginx_config}
    RETVAL=$?
    echo
    [ $RETVAL = 0 ] && touch ${nginx_lock}
    return $RETVAL
}
 
# Stop nginx daemons functions.
stop() {
    echo -n $"Stopping $prog: "
    killproc $nginxd
    RETVAL=$?
    echo
    [ $RETVAL = 0 ] && rm -f ${nginx_lock} ${nginx_pid}
}
 
# Reload nginx config file
reload() {
    echo -n $"Reloading $prog: "
    #kill -HUP `cat ${nginx_pid}`
    killproc $nginxd -HUP
    RETVAL=$?
    echo
}
 
# See how we were called.
case "$1" in
    start)
        start
        ;;
    stop)
        stop
        ;;
    reload)
        reload
        ;;
    restart)
          stop
        start
        ;;
    status)
        status $prog
        RETVAL=$?
        ;;
    *)
        echo $"Usage: $prog {start|stop|restart|reload|status|help}"
        exit 1
esac
```

Nginx启动提示找不到libpcre.so.1解决方法
如果是32位系统
```
ln -s /usr/local/lib/libpcre.so.1 /lib
```
如果是64位系统
```
ln -s /usr/local/lib/libpcre.so.1 /lib64
```

### 启动nginx

```
# chmod u+x /etc/init.d/nginx
# chkconfig --add nginx
# chkconfig nginx on
# service nginx start
正在启动 nginx：                                           [确定]
# service nginx status
nginx (pid 26500) 正在运行...
```
查看nginx的日志 错误日志logs/error.log 看是否有问题

### 其他
#### 启动nginx，tracker和storage
重启tracker：
/usr/local/bin/restart.sh /usr/local/bin/fdfs_trackerd /etc/fdfs/tracker.conf
重启storage：
/usr/local/bin/restart.sh /usr/local/bin/fdfs_storaged /etc/fdfs/storage.conf
启动nginx：
/usr/local/nginx/sbin/nginx
检查nginx状态：
/usr/local/nginx/sbin/nginx -t
重启nginx：
/usr/local/nginx/sbin/nginx -s reload

#### tracker运行
直接使用 fdfs_trackerd 来启动tracker进程，然后使用netstat 查看端口是否起来。
```
[root@tracker fdfs]# fdfs_trackerd /etc/fdfs/tracker.conf restart
[root@tracker fdfs]# netstat -antp | grep trackerd
tcp        0      0 0.0.0.0:22122               0.0.0.0:*                   LISTEN      14520/fdfs_trackerd 
[root@tracker fdfs]# 
```

#### storage运行
```
# fdfs_storaged /etc/fdfs/storage.conf restart
查看端口是否起来
# netstat -antp | grep storage
tcp        0      0 0.0.0.0:23000               0.0.0.0:*                   LISTEN      10316/fdfs_storaged 
```
也可以以下命令来监控服务器的状态
```
# fdfs_monitor /etc/fdfs/client.conf
```
看到ACTIVE,就说明已经成功注册到了tracker。

#### 开机启动
设置tracker开机自动启动
```
[root@tracker tracker]# echo "/usr/bin/fdfs_trackerd /etc/fdfs/tracker.conf restart" >> /etc/rc.local
[root@tracker tracker]# cat /etc/rc.local
#!/bin/sh
#
# This script will be executed *after* all the other init scripts.
# You can put your own initialization stuff in here if you don't
# want to do the full Sys V style init stuff.
touch /var/lock/subsys/local
/usr/bin/fdfs_trackerd /etc/fdfs/tracker.conf restart
[root@tracker tracker]# 
```

设置storage开机启动
```
[root@server1 fdfs]# echo "/usr/bin/fdfs_storaged /etc/fdfs/storage.conf restart" >> /etc/rc.local
[root@server1 fdfs]# cat /etc/rc.local
#!/bin/sh
#
# This script will be executed *after* all the other init scripts.
# You can put your own initialization stuff in here if you don't
# want to do the full Sys V style init stuff.
touch /var/lock/subsys/local
/usr/bin/fdfs_storaged /etc/fdfs/storage.conf restart
```


### 使用client测试文件上传
先配置一下client
vi /etc/fdfs/client.conf
保证一下配置：
```
base_path=/home/yuqing/fastdfs
tracker_server=192.168.1.134:22122
http.tracker_server_port=8080
```
vi test.txt

/usr/local/bin/fdfs_test /etc/fdfs/client.conf  upload  test.txt
得到：
```
This is FastDFS client test program v4.06

Copyright (C) 2008, Happy Fish / YuQing

FastDFS may be copied only under the terms of the GNU General
Public License V3, which may be found in the FastDFS source kit.
Please visit the FastDFS Home Page http://www.csource.org/
for more detail.

[2016-07-06 07:31:57] DEBUG - base_path=/home/yuqing/fastdfs, connect_timeout=30, network_timeout=60, tracker_server_count=1, anti_steal_token=0, anti_steal_secret_key length=0, use_connection_pool=0, g_connection_pool_max_idle_time=3600s, use_storage_id=0, storage server id count: 0

tracker_query_storage_store_list_without_group:
        server 1. group_name=, ip_addr=192.168.1.134, port=23000

group_name=group1, ip_addr=192.168.1.134, port=23000
storage_upload_by_filename
group_name=group1, remote_filename=M00/00/00/wKgBhld9Fl2ATSOYAAAAB9mqc8s085.txt
source ip address: 192.168.1.134
file timestamp=2016-07-06 07:31:57
file size=7
file crc32=3651826635
file url: http://192.168.1.134:8080/group1/M00/00/00/wKgBhld9Fl2ATSOYAAAAB9mqc8s085.txt
storage_upload_slave_by_filename
group_name=group1, remote_filename=M00/00/00/wKgBhld9Fl2ATSOYAAAAB9mqc8s085_big.txt
source ip address: 192.168.1.134
file timestamp=2016-07-06 07:31:57
file size=7
file crc32=3651826635
file url: http://192.168.1.134:8080/group1/M00/00/00/wKgBhld9Fl2ATSOYAAAAB9mqc8s085_big.txt
[root@localhost yihua]# This is FastDFS client test program v4.06
-bash: This: command not found
Copyright (C) 2008, Happy Fish / YuQing

```
在浏览器打开http://192.168.1.134:8080/group1/M00/00/00/wKgBhld9Fl2ATSOYAAAAB9mqc8s085.txt
可以看见你的文件。那么就成功了。

done.