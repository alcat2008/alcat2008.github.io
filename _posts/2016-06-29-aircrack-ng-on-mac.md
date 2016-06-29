---
layout: post
title: Aircrack-ng on mac
date: '2016-06-29 23:37:01 +0800'
categories: test
---

# aircrack-ng

## 安装 MacPorts

## 安装 aircrack-ng

```
sudo port install aircrack-ng
```

## 监听无线网络

aircrack-ng在OSX中使用时会出现一些问题，主要是aircrack-ng的一个附属工具airmon-ng不能在OSX环境上正确工作，原因是airmon-ng依赖于wireless-tools，而linux wireless-tool在OSX上编译有问题。可以使用OSX本身的命令行工具：`airport`，利用这个工具可以方便的实现airmon-ng中的功能，在很方便的扫面和监听。

先为airport建立软连接，方便调用该命令。

```
sudo ln -s /System/Library/PrivateFrameworks/Apple80211.framework/Versions/Current/Resources/airport /usr/local/bin/airport
```

### 扫描

```
airport en0 scan
```

### 监听

```
sudo airport en0 sniff 6
```

en0是所使用网卡的名称，sniff表示模式，6表示信道。

执行这个命令后就开启了监听模式，可以看到无线网络的链接状态已经改变了。

等待一段时间，后就可以按control+C结束监听，这时看到提示，一个airportSniffXXXXXX.cap的文件被保存在/tmp文件夹中。其中XXXXXX是唯一的，不用担心多次监听会覆盖文件。

## 破解

```
aircrack-ng -w ~/wordlist.txt /tmp/airportSniffXXXXXX.cap
```

-w表示使用字典，~/wordlist.txt是字典所在的路径，/tmp/airportSniffXXXXXX.cap是刚才监听生成的文件。执行后会分析这个文件，最后会询问你得目标网络是哪个。输入序号并回车，就可以看到破解的密码了。
