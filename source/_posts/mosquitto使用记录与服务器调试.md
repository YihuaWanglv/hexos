---
title: mosquitto使用记录与服务器调试
date: 2018-02-25 11:14:27
tags: [mosquitto,mqtt,linux,服务器配置]
categories: mqtt
---

# mosquitto使用记录：

## mqtt：

### 启动：

```
mosquitto -c /etc/mosquitto/mosquitto.conf
```

### 加-d表示后台运行：

```
mosquitto -c /etc/mosquitto/mosquitto.conf -d 
```

### sub一个主题：

```
mosquitto_sub -h localhost -t test -d
```

### pub一个消息到主题：

```
mosquitto_pub -h localhost -m "中文 的mqtt" -t test -d
```

### 重启：找到线程，kill

```
ps -A | grep mosquitto
kill -9 xxx
```

## linux最大连接数设置

```
ulimit -n20000 -s512

ulimit -f unlimited
ulimit -t unlimited
ulimit -v unlimited
ulimit -n 1048576
ulimit -m unlimited
ulimit -u 1048576
```

Till now I have achieved 74K concurrent connections on a broker. I have configured the ulimit of broker server by editing sysctl.conf and limit.conf file.

- vi /etc/sysctl.conf

```
fs.file-max = 10000000 
fs.nr_open = 10000000
net.ipv4.tcp_mem = 786432 1697152 1945728
net.ipv4.tcp_rmem = 4096 4096 16777216
net.ipv4.tcp_wmem = 4096 4096 16777216
net.ipv4.ip_local_port_range = 1000 65535
```

- vi /etc/security/limits.conf

```
* soft nofile 10000000
* hard nofile 10000000
root soft nofile 10000000
root hard nofile 10000000
```

After this reboot your system.



## mqtt启动后，需要开放对应端口的，则处理如下

CentOS 7.0默认使用的是firewall作为防火墙，使用iptables必须重新设置一下

1、直接关闭防火墙

```
systemctl stop firewalld.service #停止firewall

systemctl disable firewalld.service #禁止firewall开机启动
```

2、设置 iptables service

```
yum -y install iptables-services
```

如果要修改防火墙配置，如增加防火墙端口3306
```
vi /etc/sysconfig/iptables 
```

增加规则
```
-A INPUT -m state --state NEW -m tcp -p tcp --dport 3306 -j ACCEPT
```

保存退出后
```
systemctl restart iptables.service #重启防火墙使配置生效

systemctl enable iptables.service #设置防火墙开机启动
```

最后重启系统使设置生效即可。