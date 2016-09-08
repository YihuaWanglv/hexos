---
title: ELK+Kafka企业日志收集平台搭建总结
date: 2016-09-09 01:31:02
tags: [ELK, elasticsearch, es, kafka, zookeeper, kibana, logstash, nginx, log]
---

# 3.ELK+Kafka 企业日志收集平台搭建总结



### 平台架构：
- 1台生产应用服务器：192.168.1.119
- 3台zookeeper+kafka集群服务器：192.168.1.245, 192.168.1.246, 192.168.1.247
- 2台es+kibana集群服务器：192.168.1.162, 192.168.1.163
- 1台nginx服务器反向代理到kibana集群：192.168.1.244

### 软件选用
```
elasticsearch-1.7.3.tar.gz 
Logstash-2.3.*
kibana-4.1.2-linux-x64.tar.gz
以上软件都可以从官网下载:https://www.elastic.co/downloads
java-1.8.65
nginx采用yum安装
```

### 平台安装配置

####部署步骤：
1.ES集群安装配置;
2.Logstash客户端配置(直接写入数据到ES集群，写入系统messages日志);
3.Kafka(zookeeper)集群配置;(Logstash写入数据到Kafka消息系统);
4.Kibana部署;
5.Nginx负载均衡Kibana请求;
6.案例：nginx日志收集以及MySQL慢日志收集;
7.Kibana报表基本使用;


### 1.ES集群安装配置;
- 需要先安装java1.8
- 2.获取es软件包
```
wget https://download.elastic.co/elasticsearch/elasticsearch/elasticsearch-1.7.3.tar.gz
tar -xf elasticsearch-1.7.3.tar.gz -C /usr/local
ln -sv /usr/local/elasticsearch-1.7.3 /usr/local/elasticsearch
```

- 3.修改配置文件

```
[root@es1 ~]# vim /usr/local/elasticsearch/config/elasticsearch.yml 
32 cluster.name: es-cluster                         #组播的名称地址
40 node.name: "es-node1 "                           #节点名称，不能和其他节点重复
47 node.master: true                                #节点能否被选举为master
51 node.data: true                                  #节点是否存储数据
107 index.number_of_shards: 5                       #索引分片的个数
111 index.number_of_replicas: 1                     #分片的副本个数
145 path.conf: /usr/local/elasticsearch/config/     #配置文件的路径
149 path.data: /data/es/data                        #数据目录路径
159 path.work: /data/es/worker                      #工作目录路径
163 path.logs:  /usr/local/elasticsearch/logs/      #日志文件路径
167 path.plugins:  /data/es/plugins                 #插件路径
184 bootstrap.mlockall: true                        #内存不向swap交换
232 http.enabled: true                              #启用http
```

- 4.创建相关目录
```
mkdir /data/es/{data,worker,plugins} -p
```

- 5.获取es服务管理脚本
```
​[root@es1 ~]# git clone https://github.com/elastic/elasticsearch-servicewrapper.git
[root@es1 ~]# mv elasticsearch-servicewrapper/service /usr/local/elasticsearch/bin/
[root@es1 ~]# /usr/local/elasticsearch/bin/service/elasticsearch install 
Detected RHEL or Fedora:
Installing the Elasticsearch daemon..
[root@es1 ~]# 
```
这时就会在/etc/init.d/目录下安装上es的管理脚本啦

- 修改其配置:
```
[root@es1 ~]# 
set.default.ES_HOME=/usr/local/elasticsearch   #安装路径
set.default.ES_HEAP_SIZE=1024
```

- 6.启动es ，并检查其服务是否正常
es启动方式是进入bin目录
```
/usr/local/elasticsearch/bin/
```
运行
```
./elasticsearch
```
启动

```
[root@es1 ~]# netstat -nlpt | grep -E "9200|"9300
tcp        0      0 0.0.0.0:9200                0.0.0.0:*                   LISTEN      1684/java           
tcp        0      0 0.0.0.0:9300                0.0.0.0:*                   LISTEN      1684/java
```

访问http://192.168.2.18:9200/ 返回es节点信息，说明安装配置完成.

- 7.复制同样的步骤和配置到es2，只需要修改node.name即可，其他都与es1相同配置
- 8.安装es的管理插件
es官方提供一个用于管理es的插件，可清晰直观看到es集群的状态，以及对集群的操作管理，安装方法如下：
```
[root@es1 local]# /usr/local/elasticsearch/bin/plugin -i mobz/elasticsearch-head
```

安装好之后，访问方式为： http://192.168.2.18:9200/_plugin/head

### 2.Logstash客户端安装配置
- 安装前，需要先安装java
- YUM方式安装Logstash

Download and install the public signing key:
```
rpm --import https://packages.elastic.co/GPG-KEY-elasticsearch
```

Add the following in your /etc/yum.repos.d/ directory in a file with a .repo suffix, for example logstash.repo
```
[logstash-2.3]
name=Logstash repository for 2.3.x packages
baseurl=https://packages.elastic.co/logstash/2.3/centos
gpgcheck=1
gpgkey=https://packages.elastic.co/GPG-KEY-elasticsearch
enabled=1
```

And your repository is ready for use. You can install it with:
```
yum install logstash
```

此时logstash会安装在目录：/opt/logstash/
进入/opt/logstash/bin/目录
vim logstash.conf
即可对这个logstash客户端进行设置，可以设置输入和输出
输入定义信息来源于何处，输出定义消息发送到哪里


- logstash一些操作：
检查配置是否正确
/opt/logstash/bin/logstash -f logstash.conf --configtest --verbose
启动logstash
/opt/logstash/bin/logstash -f logstash.conf

- 3.Logstash 向es集群写数据
编写一个logstash配置文件
```
[root@webserver1 etc]# vi logstash.conf 
input {              #数据的输入从标准输入
  stdin {}   
}

output {             #数据的输出我们指向了es集群
  elasticsearch {
    hosts => ["192.168.1.162:9200","192.168.1.163:9200"]　　　＃es主机的ip及端口
  }
}
```

- 命令/opt/logstash/bin/logstash -f logstash.conf 启动后，如果有信息产生，就会向es集群写入数据，可以在es页面中看到

- 例子：将web app的log日志写入es中，使用如下脚本
```
input {
  file {
    path => "/usr/local/log.log"
    start_position => "beginning"
  }
}

output {
  elasticsearch {
    hosts => ["192.168.1.162:9200","192.168.1.163:9200"]
    index => "web-app-%{+YYYY-MM}"
  }
}
```


### 3.Kafka(zookeeper)集群配置;(Logstash写入数据到Kafka消息系统);

kafka的软件包中自带zookeeper，并且是解压即可使用。可以官网下载上传服务器，也可直接服务器下载

- 1.获取软件包.官网：http://kafka.apache.org
```
[root@kafka1 ~]# wget http://mirror.rise.ph/apache/kafka/0.8.2.1/kafka_2.11-0.8.2.1.tgz
[root@kafka1 ~]# tar -xf kafka_2.11-0.8.2.1.tgz -C /usr/local/
[root@kafka1 ~]# cd /usr/local/
[root@kafka1 local]# ln -sv kafka_2.11-0.8.2.1 kafka
```

- 2.配置zookeeper集群，修改配置文件
```
[root@kafka1 ~]# vim /usr/local/kafka/config/zookeeper.propertie
dataDir=/data/zookeeper
clienrtPort=2181
tickTime=2000
initLimit=20
syncLimit=10
server.2=192.168.1.245:2888:3888
server.3=192.168.1.246:2888:3888
server.4=192.168.1.247:2888:3888
```

- 3.创建zookeeper所需要的目录
```
[root@kafka1 ~]# mkdir /data/zookeeper
```

- 4.在/data/zookeeper目录下创建myid文件，里面的内容为数字，用于标识主机，如果这个文件没有的话，zookeeper是没法启动的哦
```
[root@kafka1 ~]# echo 2 > /data/zookeeper/myid
```
以上就是zookeeper集群的配置，下面等我配置好kafka之后直接复制到其他两个节点即可

- 5.kafka配置
```
[root@kafka1 ~]# vim /usr/local/kafka/config/server.properties 
broker.id=2    　　　　    ＃　唯一，填数字，本文中分别为2/3/4
prot=9092　　　　　　　     ＃　这个broker监听的端口　
host.name=192.168.2.22　  ＃　唯一，填服务器IP
log.dir=/data/kafka-logs  #  该目录可以不用提前创建，在启动时自己会创建
zookeeper.connect=192.168.2.22:2181,192.168.2.23:2181,192.168.2.24:2181　　＃这个就是zookeeper的ip及端口
num.partitions=16         # 需要配置较大 分片影响读写速度
log.dirs=/data/kafka-logs # 数据目录也要单独配置磁盘较大的地方
log.retention.hours=168   # 时间按需求保留过期时间 避免磁盘满
```


- 6.将kafka(zookeeper)的程序目录全部拷贝至其他两个节点

- 7.修改两个借点的配置，注意这里除了以下两点不同外，都是相同的配置
```
（1）zookeeper的配置
mkdir /data/zookeeper
echo "x" > /data/zookeeper/myid
（2）kafka的配置
broker.id=2
host.name=192.168.2.22
```

- 8.修改完毕配置之后我们就可以启动了，这里先要启动zookeeper集群，才能启动kafka
```
[root@kafka1 ~]# /usr/local/kafka/bin/zookeeper-server-start.sh /usr/local/kafka/config/zookeeper.properties &   #zookeeper启动命令
[root@kafka1 ~]# /usr/local/kafka/bin/zookeeper-server-stop.sh                                                   #zookeeper停止的命令
```

- 9.zookeeper服务检查
```
[root@kafka1~]#  netstat -nlpt | grep -E "2181|2888|3888"
```

ok.  这时候zookeeper集群已经启动起来了，下面启动kafka，也是依次按照顺序启动
```
[root@kafka1 ~]# nohup /usr/local/kafka/bin/kafka-server-start.sh /usr/local/kafka/config/server.properties &   #kafka启动的命令
[root@kafka1 ~]#  /usr/local/kafka/bin/kafka-server-stop.sh                                                         #kafka停止的命令
```
注意，跟zookeeper服务一样，如果kafka有问题 nohup的日志文件会非常大,把磁盘占满，这个kafka服务同样可以通过自己些服务脚本来管理服务的启动与关闭。

- 10，下面我们将webs app上面的logstash的输出改到kafka上面，将数据写入到kafka中
```
input {             #这里的输入还是定义的是从日志文件输入
  file {
    type => "web-app" 
    path => "/usr/local/log.log"
    start_position => "beginning"
  }
}

output {
    #stdout { codec => rubydebug }
    kafka {
      bootstrap_servers => "192.168.1.245:9092,192.168.1.246:9092,192.168.1.247:9092"
      topic_id => "web-app-message"
      compression_type => "snappy"
    }
}
```

- 然后再每台kafka上安装logstash，把kafka的消息发送到es
使用logstash脚本：
```
input {
    kafka {
        zk_connect => "192.168.1.245:2181,192.168.1.246:2181,192.168.1.247:2181"
        topic_id => "web-app-message"
        codec => plain
        reset_beginning => false
        consumer_threads => 5
        decorate_events => true
    }
}

output {
    elasticsearch {
      hosts => ["192.168.1.162:9200","192.168.1.163:9200"]
      index => "test-web-app-message-%{+YYYY-MM}"
  }
}
```


### 4.Kibana部署;

我们在两台es上面搭建两套kibana

- 1.获取kibana软件包
```
[root@es1 ~]# wget https://download.elastic.co/kibana/kibana/kibana-4.1.2-linux-x64.tar.gz
[root@es1 ~]# tar -xf kibana-4.2.0-linux-x64.tar.gz -C /usr/local/
```

- 2.修改配置文件
```
[root@es1 ~]# cd /usr/local/
[root@es1 local]# ln -sv kibana-4.1.2-linux-x64 kibana
`kibana' -> `kibana-4.2.0-linux-x64'
[root@es1 local]# cd kibana

[root@es1 kibana]# vim config/kibana.yml
server.port: 5601      #默认端口可以修改的
server.host: "0.0.0.0" #kibana监听的ip
elasticsearch.url: "http://localhost:9200" #由于es在本地主机上面，所以这个选项打开注释即可
```

- kibana安装后，直接进入kibana的安装目录
/user/local/kibana/bin
运行
./kibana
启动

- 5.服务检查
```
[root@es1 config]# ss -tunl | grep "5601"
tcp    LISTEN     0      511                    *:5601                  *:*
```
此时访问es1主机的5601端口，即可看见kibana的页面：
http://192.168.1.163:5601

- 6.es2上的kibana与es1一样




### 5.Nginx负载均衡Kibana请求;

- 1.在nginx-proxy上面yum安装nginx
```
yum install -y nignx
```

- 2.编写配置文件es.conf
```
[root@saltstack-node1 conf.d]# pwd 
/etc/nginx/conf.d
[root@saltstack-node1 conf.d]# cat es.conf 
upstream es {
    server 192.168.2.18:5601 max_fails=3 fail_timeout=30s;
    server 192.168.2.19:5601 max_fails=3 fail_timeout=30s;
}
 
server {
    listen       80;
    server_name  localhost;
 
    location / {
        proxy_pass http://es/;
        index index.html index.htm;
        #auth
        auth_basic "ELK Private";
        auth_basic_user_file /etc/nginx/.htpasswd;
    }
 
 }
```

3.创建认证
```
[root@saltstack-node1 conf.d]# htpasswd -cm /etc/nginx/.htpasswd elk
New password: 
Re-type new password: 
Adding password for user elk-user
[root@saltstack-node1 conf.d]# /etc/init.d/nginx restart
Stopping nginx:                                            [  OK  ]
Starting nginx:                                            [  OK  ]
[root@saltstack-node1 conf.d]# 
```

- 4.直接输入认证用户及密码就可访问啦http://192.168.1.244/




### 6.案例：nginx日志收集以及MySQL慢日志收集;

- 先在web app服务器上安装nginx和mysql

- 1.为了方便nginx日志的统计搜索，这里设置nginx访问日志格式为json

(1)修改nginx主配置文件

说明：如果想实现日志的报表展示，最好将业务日志直接以json格式输出，这样可以极大减轻cpu负载，也省得运维需要写负载的filter过滤正则。

```
[root@webserver1 nginx]# vim nginx.conf
log_format json '{"@timestamp":"$time_iso8601",'
                '"@version":"1",'
                '"client":"$remote_addr",'
                '"url":"$uri",'
                '"status":"$status",'
                '"domain":"$host",'
                '"host":"$server_addr",'
                '"size":$body_bytes_sent,'
                '"responsetime":$request_time,'
                '"referer": "$http_referer",'
                '"ua": "$http_user_agent"'
                '}';
  access_log  /var/log/access_json.log  json;
```

(2)收集nginx日志和MySQL日志到消息队列中；这个文件我们是定义在客户端，即生产服务器上面的Logstash文件
```
input {
  file {
    type => "web-app"
    path => "/usr/local/log.log"
    start_position => "beginning"
  }
  file {
    type => "nginx-access"
    path => "/var/log/nginx/access.log"
    start_position => "beginning"
    codec => "json"
  }
  file {
   type => "slow-mysql"
   path => "/var/run/mysqld/mysqld-slow.log"
   start_position => "beginning"
   codec => multiline {
     pattern => "^# User@Host"
     negate => true
     what => "previous"
   }
  }
}

output {
  if [type] == "nginx-access" {
    kafka {
      bootstrap_servers => "192.168.1.245:9092,192.168.1.246:9092,192.168.1.247:9092"
      topic_id => "nginx-access"
      compression_type => "snappy"
    }
  }
  if [type] == "slow-mysql" {
    kafka {
      bootstrap_servers => "192.168.1.245:9092,192.168.1.246:9092,192.168.1.247:9092"
      topic_id => "slow-mysql"
      compression_type => "snappy"
    }
  }
  if [type] == "web-app" {
    kafka {
      bootstrap_servers => "192.168.1.245:9092,192.168.1.246:9092,192.168.1.247:9092"
      topic_id => "web-app-message"
      compression_type => "snappy"
    }
  }
}
```

/opt/logstash/bin/logstash -f logstash.conf --configtest --verbose

/opt/logstash/bin/logstash -f logstash.conf

(3)Logstash 从kafka集群中读取日志存储到es中，这里的定义logstash文件是在三台kafka服务器上面
```
input {
    kafka {
        zk_connect => "192.168.1.245:2181,192.168.1.246:2181,192.168.1.247:2181"
        topic_id => "web-app-message"
        codec => plain
        reset_beginning => false
        consumer_threads => 5
        decorate_events => true
    }
    kafka {
        zk_connect => "192.168.1.245:2181,192.168.1.246:2181,192.168.1.247:2181"
        type => "nginx-access"
        topic_id => "nginx-access"
        codec => plain
        reset_beginning => false
        consumer_threads => 5
        decorate_events => true
    }
    kafka {
        zk_connect => "192.168.1.245:2181,192.168.1.246:2181,192.168.1.247:2181"
        type => "slow-mysql"
        topic_id => "slow-mysql"
        codec => plain
        reset_beginning => false
        consumer_threads => 5
        decorate_events => true
    }
}

output {
    if [type] == "nginx-access" {
         elasticsearch {
                hosts => ["192.168.1.162:9200","192.168.1.163:9200"]
                 index => "nginx-access-%{+YYYY-MM}"
         }
    }
    if [type] == "slow-mysql" {
         elasticsearch {
                hosts => ["192.168.1.162:9200","192.168.1.163:9200"]
                index => "slow-mysql-%{+YYYY-MM}"
        }
    }
    if [type] == "web-app-message" {
         elasticsearch {
                hosts => ["192.168.1.162:9200","192.168.1.163:9200"]
                index => "test-web-app-message-%{+YYYY-MM}"
        }
    }
}
```

(4)创建nginx-access 日志索引
(5)创建MySQL慢日志索引

这两个都是在kibana的页面上创建配置


### 7.Kibana报表基本使用;