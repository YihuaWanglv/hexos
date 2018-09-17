---
title: Windows下开启React-Native征途
date: 2016-06-04 09:55:48
tags: [react, react-native, nodejs, android, windows]
categories: javascript
---

我们的目标是星辰大海（说白了就是同一个东西咱不想Android做一份，iOS又做一份），现在我们要开启React-Native的征途（在Windows下）。
不过说出来你可能不太相信，这故事的开头，来的并没有想象中的困难。

以下是过程：
## 1. 安装Nodejs

React Native要求是4.0以上。

## 2. 安装react-native-cli

```
npm install -g react-native-cli
```

## 3. 初始化第一个项目
```
react-native init TestProject
```
初始化第一个测试Demo，init的时候可能需要一点时间. 伟大的目标值的那么些耐心的等待。

## 4. 设置环境变量

设置环境变量ANDROID_HOME指向Android SDK的目录
eg:
![](/images/react-001.png)


## 5. 安装Genymotion模拟器

Genymotion官网下载安装Genymotion模拟器，并创建Android emulator。
创建实例后，将得到如下画面：
![](/images/react-002.png)
Genymotion运行Android模拟实例需要VirtualBox和AndroidSdk的支持。
安装Genymotion后默认的VirtualBox和AndroidSdk不好使，我通过Setting设置配置好自己本地的VirtualBox和AndroidSdk。
设置VirtualBox
![](/images/react-003.png)
设置sdk
![](/images/react-004.png)

配置好后，就把模拟的Android实例启动起来放着吧。


## 6. 启动

### 1-启动React Native Server
```
react-native start
```
![](/images/react-005.png)

### 2-开一个新的命令行终端(cmd), 切换到项目目录安装APP
```
cd android
gradlew.bat installDebug
```
执行gradlew.bat installDebug时，需要先正常打开Genymotion模拟器，否则build 和 install会报错。
![](/images/react-006.png)
安装成功后，在Genymotion里手动打开APP ，正常的话，你就可以看到React-Native的欢迎画面。
![](/images/react-007.png)

Windows下的React Native开发环境搭建 -- done.