---
title: 在linux下部署spring boot项目过程记录
date: 2016-03-21 17:52:34
tags: [linux,centos,java,spring boot,部署]
---



# 在linux下部署spring boot项目过程记录



## 说明
本文章是在centos7下部署spring boot架构的java web项目的过程中的一些记录，记录了linux服务器中需要用到的软件服务的安装和使用

## 环境说明
linux版本：CentOs7
配备的开发环境软件服务：
    - mysql（centos下默认使用mariadb，也可以，两者兼容）
    - jdk
    - ftp
    - redis

### 1. linux下开放端口
先把一些已知要用到的端口放开了，省的后面要用到的时候连不上
由于centos7默认没有iptables服务，所以需要先安装
首先暂停防火墙
```
systemctl stop firewalld
systemctl mask firewalld
```
Then, install the iptables-services package:
```
yum install iptables-services
```
Enable the service at boot-time:
```
systemctl enable iptables
```

配置要放开的端口
```
iptables -I INPUT -p tcp --dport 80 -j ACCEPT
iptables -I INPUT -p tcp --dport 22 -j ACCEPT
iptables -I INPUT -p tcp --dport 3306 -j ACCEPT
iptables -I INPUT -p tcp --dport 8080 -j ACCEPT
iptables -I INPUT -p tcp --dport 21 -j ACCEPT
iptables -I INPUT -p tcp --dport 8081 -j ACCEPT
```
保存配置
```
service iptables save
or
/usr/libexec/iptables/iptables.init save
```

Managing the service开启停止防火墙服务
```
systemctl [stop|start|restart] iptables
```


### 2. linux下安装使用mysql
一般的linux安装mysql，会使用yum install mysql mysql-server mysql-devel
然而，在centos7下这样安装的时候，却发现mysql-server安装不上。
原来，由于Oracle收购mysql后，开源社区担心mysql有闭源的风险，于是使用mysql的一个分支mariadb替代mysql。mariadb完全兼容mysql。 而centos7就是默认推荐使用mariadb代替mysql。
那么，centos7下安装mysql就有两种方式
1）使用mariadb；
2）卸载mariadb，安装mysql

#### 方法1:使用mariadb
```
yum install mariadb-server mariadb
systemctl start mariadb
mysql -u root -p
```

#### 方法2:安装mysql：
```
# wget http://dev.mysql.com/get/mysql-community-release-el7-5.noarch.rpm
# rpm -ivh mysql-community-release-el7-5.noarch.rpm
# yum install mysql-community-server
```
安装成功后重启mysql服务。
```
# service mysqld restart
```
初次安装mysql，root账户没有密码。
```
[root@yl-web yl]# mysql -u root
```
设置密码
```
mysql> set password for 'root'@'localhost' =password('password');
```

#### 实践结论：如果不介意使用mariadb的话，就不用折腾安装mysql了，因为我在实际使用方法2安装MySQL的过程中，遇到不少需要折腾的问题。
记录如下：
1）在进入mysql工具时，总是有错误提示:
```
# mysql -u root -p
Enter password:
ERROR 1045 (28000): Access denied for user 'root'@'localhost' (using password: NO)
```
解决：方法操作很简单，如下：
```
# /etc/init.d/mysql stop
# mysqld_safe --user=mysql --skip-grant-tables --skip-networking &
# mysql -u root mysql
mysql> UPDATE user SET Password=PASSWORD('newpassword') where USER='root' and host='root' or host='localhost';//把空的用户密码都修改成非空的密码就行了。
mysql> FLUSH PRIVILEGES;
mysql> quit
# /etc/init.d/mysqld restart
# mysql -uroot -p
Enter password: <输入新设的密码newpassword>
```

2）MySQL服务在启动的时候，不能创建pid文件。

在终端看一下该目录是否存在，果然，不存在。
于是，创建了/var/run/mysqld/目录，重启MySQL服务
```
[root@spark01 ~]# mkdir -p /var/run/mysqld/
[root@spark01 ~]# /etc/init.d/mysqld start
```
如果依旧不能成功启动，那有可能是/var/run/mysqld/的属主和属组还是root，mysql并不能在其中创建文件，那么修改该目录的属主和属组，应该就行了。
```
[root@spark01 ~]# ls -ld /var/run/mysqld/
drwxr-xr-x 2 root root 40 Jan 20 18:28 /var/run/mysqld/
[root@spark01 ~]# chown mysql.mysql /var/run/mysqld/
[root@spark01 ~]# /etc/init.d/mysqld start
```


### 3. linux下安装jdk
一、卸载系统自带的openjdk
1、查询系统内置的jdk，使用命令如下：
```
rpm -qa | grep java
```
此时会列出系统中存在的jdk，如果存在就进行卸载，不存在就直接进行安装。
如下：
python-javapackages-3.4.1-11.el7.noarch
java-1.8.0-openjdk-1.8.0.65-3.b17.el7.x86_64
java-1.7.0-openjdk-headless-1.7.0.91-2.6.2.3.el7.x86_64
java-1.8.0-openjdk-headless-1.8.0.65-3.b17.el7.x86_64
tzdata-java-2015g-1.el7.noarch
javapackages-tools-3.4.1-11.el7.noarch
java-1.7.0-openjdk-1.7.0.91-2.6.2.3.el7.x86_64

2、进行卸载，使用命令如下：
```
rpm -e --nodeps jdk相关名称
```
依次卸载如下：
```
rpm -e --nodeps python-javapackages-3.4.1-11.el7.noarch
rpm -e --nodeps java-1.8.0-openjdk-1.8.0.65-3.b17.el7.x86_64
rpm -e --nodeps java-1.7.0-openjdk-headless-1.7.0.91-2.6.2.3.el7.x86_64
rpm -e --nodeps java-1.8.0-openjdk-headless-1.8.0.65-3.b17.el7.x86_64
rpm -e --nodeps tzdata-java-2015g-1.el7.noarch
rpm -e --nodeps javapackages-tools-3.4.1-11.el7.noarch
rpm -e --nodeps java-1.7.0-openjdk-1.7.0.91-2.6.2.3.el7.x86_64
```

二、jdk安装
1、下载jdk并上传到/usr/java目录
jdk7下载地址为：http://www.oracle.com/technetwork/java/javase/downloads/jdk7-downloads-1880260.html 选择对应的linux版本，下载rpm文件。这里选择的是jdk-7u79-linux-x64.rpm。并上传到Linux的/urs/java目录下（java目录不存在则进行创建）。

2、解压安装
进入/usr/java目录，运行如下命令进行解压(rpm -ivh rpm文件名称)
```
rpm -ivh jdk-7u79-linux-x64.rpm
```

3、配置profile文件
运行如下命令
```
vi /etc/profile
```
将如下内容添加到profile文件末尾并保持
```
export JAVA_HOME=/usr/java/jdk1.7.0_79
export CLASSPATH=.:$JAVA_HOME/jre/lib/rt.jar:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar export PATH=$PATH:$JAVA_HOME/bin 
```
/usr/java/jdk1.7.0_79 指的是jdk的路径

保存之后，运行如下命令使配置生效
```
source /etc/profile
```
检查jdk是否安装成功，运行如下命令
```
java -version
```


### 4. linux下安装redis
1）方法1：使用命令安装
安装redis:
```
yum -y install redis
```
启动/停止/重启 Redis
启动服务：
```
systemctl start redis.service
```
停止服务：
```
systemctl stop redis.service
```
重启服务：
```
systemctl restart redis.service
```
检查状态：
```
systemctl status redis.service
```
随系统启动服务：
```
systemctl enable redis.service
```

2）方法二：编译安装
下载安装编译:
```
wget http://download.redis.io/releases/redis-2.8.17.tar.gz
tar xzf redis-2.8.17.tar.gz
cd redis-2.8.17
make
make install
```
设置配置文件路径:
```
mkdir -p /etc/redis && cp redis.conf /etc/redis
```
修改配置文件：
```
vim /etc/redis/redis.conf
```
修改为： daemonize yes
启动Redis:
```
/usr/local/bin/redis-server /etc/redis/redis.conf
```



### 5. linux配置ftp服务
在安装前查看是否已安装vsftpd
```
[root@localhost ~]# rpm -q vsftpd
vsftpd-3.0.2-9.el7.x86_64
```
如果有显示类似以上的信息，说明已经安装vsftpd，如果没有，用yum安装：
```
[root@localhost ~]# yum -y install vsftpd
```
查看一下vsftpd安装在哪：
```
[root@localhost ~]# whereis vsftpd
vsftpd: /usr/sbin/vsftpd /etc/vsftpd /usr/share/man/man8/vsftpd.8.gz
```
启动vsftpd服务：
```
[root@localhost ~]# systemctl start vsftpd.service
```
修改配置
```
vi /etc/vsftpd/vsftpd.conf
```
修改如下配置：
anonymous_enable=NO
chroot_local_user=YES
allow_writeable_chroot=YES #加上这行解决了无法登陆的问题

启动／重新启动ftp
```
service vsftpd start
service vsftpd restart
```
设置开机启动ftp
```
chkconfig vsftpd on
```
配置用户
```
[root@localhost ~]# useradd -g root -M -d /var/www/html -s /sbin/nologin ftpuser
[root@localhost ~]# passwd ftpuser
[root@localhost ~]# 输入密码
```
把 /var/www/html 的所有权给ftpuser.root
```
[root@localhost ~]# chown -R ftpuser.root /var/www/html
```


### 6. 附：使用maven把java project打成包含依赖包的jar包，上传到服务器，并启动
#### 编译打包：
方法1：使用maven-assembly-plugin插件
```
<plugin>
    <artifactId>maven-assembly-plugin</artifactId> 
    <configuration> 
        <archive>
            <manifest>
                <mainClass>com.iyihua.itimes.App</mainClass></manifest> 
            </archive>
        <descriptorRefs>
            <descriptorRef> jar-with-dependencies </descriptorRef> 
        </descriptorRefs>
    </configuration>
    <executions>
        <execution>
            <id>make-assembly</id> 
            <phase>package</phase>
            <goals>
                <goal>single</goal>
            </goals>
        </execution> 
    </executions>
</plugin>
```
运行命令：mvn assembly:assembly
或者命令：mvn package

由于我使用这个方式，打出来的jar包里，依赖的jar和模块的class都正常，但是自己项目内的所有class都是空的class，用反编译看class文件，发现都是
```
class {}
```
解压出来取单个class查看又是正常的，这个问题一直百撕不得骑姐，所以我最后使用了方法2的方式解决了问题。

方法2：使用spring-boot-maven-plugin插件
由于我的project是spring boot项目，所以可以使用此方法
```
<plugin>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-maven-plugin</artifactId>
    <executions>
        <execution>
            <goals>
                <goal>repackage</goal>
            </goals>
        </execution>
    </executions>
</plugin>
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-jar-plugin</artifactId>
    <configuration>
        <archive>
            <manifest>
                <addClasspath>true</addClasspath>
                <mainClass>com.iyihua.itimes.App</mainClass>
            </manifest>
        </archive>
    </configuration>
</plugin>
```
运行mvn package打jar包，done！

#### 上传并启动
连接sftp
lcd 打开本地路径
cd 进入服务器目标路径
put xxx.jar 把目标jar包上传到服务器对应路径
java -jar xxx.jar 启动java程序
