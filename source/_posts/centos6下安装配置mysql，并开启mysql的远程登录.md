---
title: centos6下安装配置mysql，并开启mysql的远程登录
date: 2016-04-25 21:00:38
tags: [mysql, install, database]
---


## 安装：

### 查看是否已安装mysql：
yum list installed | grep mysql
### 如果已安装，如下一次卸载：
yum -y remove mysql-libs.x86_64
### 查看可用mysql：
yum list | grep mysql 或 yum -y list mysql*
### 安装mysql：
yum -y install mysql-server mysql mysql-devel
### 查看已安装的mysql状态：
rpm -qi mysql-server

### 安装后一开始登陆不上的问题：在进入mysql工具时，总是有错误提示:
mysql -u root -p
Enter password:
ERROR 1045 (28000): Access denied for user 'root'@'localhost' (using password: NO)

### 解决：方法操作很简单，如下：
/etc/init.d/mysql stop
mysqld_safe --user=mysql --skip-grant-tables --skip-networking &
mysql -u root mysql
mysql> UPDATE user SET Password=PASSWORD('root') where USER='root' and host='root' or host='localhost';//把空的用户密码都修改成非空的密码就行了。
mysql> FLUSH PRIVILEGES;
mysql> quit
/etc/init.d/mysqld restart
mysql -uroot -p
Enter password: <输入新设的密码newpassword>

## 开启远程连接
### 设置任意ip可以使用root账户和root密码远程登录
grant all privileges  on *.* to root@'%' identified by "root";
### 刷新
FLUSH PRIVILEGES;
### 重启：
service mysqld restart

done！