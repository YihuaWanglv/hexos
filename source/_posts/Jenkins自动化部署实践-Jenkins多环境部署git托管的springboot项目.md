---
title: '[Jenkins自动化部署实践]Jenkins多环境部署git托管的springboot项目'
date: 2016-06-01 18:56:42
tags:
---


[Jenkins自动化部署实践]Jenkins多环境部署git托管的springboot项目

### 目标

#### 场景
Jenkins部署在服务器192.168.1.119:8080.
有一个在git托管的springboot构建的web项目，项目有n个依赖的父模块，现在想要把这个项目部署到服务器192.168.1.134上.
现在，我们需要使用Jenkins自动化部署实现这个目标.

#### 为什么是springboot？
Sprinboot允许项目使用内嵌的tomcat像启动普通java程序一样启动一个web项目.由于有了这个特性，项目不再需要打war包部署到tomcat，而是打成jar包，直接使用java -jar命令启动.
#### 如何实现？
1-在jenkins上建立

### 实现步骤

脚本：
shell to start project itimes
```
#!/bin/bash
# script to update and restart project itimes
remote_source_path=/root/.jenkins/workspace/dev-timeitem-itimes/target
target_jar=itimes-1.0.0-SNAPSHOT.jar
target_path=/usr/local/itimes
target_bak_path=/usr/local/itimes_bak

now=$(date '+%Y%m%d%H%M%S')
echo "update and restart project itimes start..."
echo "now time: ${now}"

mkdir -p ${target_bak_path}/${now}
/usr/bin/sshpass -p "root" scp root@192.168.1.119:${remote_source_path}/itimes-1.0.0-SNAPSHOT.jar ${target_bak_path
}/${now}

cd $target_bak_path/$now
if [ -f ${target_jar} ]; then
  pkill -f 'java.*itimes'
  sleep 3
  temppid=$(ps aux | grep java| grep "${target_jar}" | grep -v grep | awk '{ print $2}')
  if [[ "$temppid" != "" ]]; then
    echo "temppid is ${temppid}"
    ps -ef | grep ${target_jar} | grep -v grep | awk '{print $2}' | xargs kill
  fi
  echo "kill ${target_jar} done.."
  cd $target_path
  if [ -f ${target_jar} ]; then
    rm -f ${target_jar}
    echo "target_jar deleted.."
  else
    echo "${target_jar} not exist, go on to copy.."
  fi
  cp ${target_bak_path}/${now}/${target_jar} ${target_path}
  echo "cp done!"
  chmod 755 ${target_jar}
  nohup java -jar ${target_path}/${target_jar} < /dev/null > ${target_bak_path}/${now}/itime-${now}.log 2>&1 &
  echo "app startting.."
  for i in 2 4 6 10; do
    sleep $i
    echo "wait and check app starting up..."
    APP_PID=`ps aux | grep java| grep "${target_jar}" | grep -v grep | awk '{ print $2}'`
    if [ $APP_PID > 0 ]; then
        break;
    else
        echo "app is still starting up..."
    fi
  done
  pid=$(ps aux | grep java| grep "${target_jar}" | grep -v grep | awk '{ print $2}')
  echo "app id is: ${pid}"
  if [ $pid ]; then
    echo "${target_jar} is running now!"
  else
    echo "${target_jar} is not running! ${target_jar} startup failed!"
  fi
  success_flag="false"
  for i in 2 4 6 8 10 12; do
    echo "checking app startup status..."
    success_target_tag=`cat ${target_bak_path}/${now}/itime-${now}.log | grep "Started App in"`
    if [[ "" != "$success_target_tag" ]]; then
      echo "Found success target tag, app startup successfully.."
      success_flag="true"
      break;
    else
        echo "app is still starting up..."
        sleep $i
        echo "sleep $i ..."
    fi
  done
  if [[ "true" != "$success_flag"  ]]; then
    echo "Not found success target tag, app startup failed..."
  fi
else
  echo "Target jar not exist! Update not complete!"
fi

echo "update done!"
```