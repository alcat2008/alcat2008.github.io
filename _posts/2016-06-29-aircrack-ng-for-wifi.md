---
layout: post
title: aircrack-ng 对 wifi 渗透测试
date: '2016-06-29 23:37:01 +0800'
categories: Test
---

# aircrack-ng

> 注： 本文的测试环境为 Mac OS X EI Capitan

# 安装 MacPorts

# 安装 aircrack-ng

```
sudo port install aircrack-ng
```

# 监听无线网络

aircrack-ng在OSX中使用时会出现一些问题，主要是aircrack-ng的一个附属工具airmon-ng不能在OSX环境上正确工作，原因是airmon-ng依赖于wireless-tools，而linux wireless-tool在OSX上编译有问题。可以使用OSX本身的命令行工具：`airport`，利用这个工具可以方便的实现airmon-ng中的功能，在很方便的扫面和监听。

先为airport建立软连接，方便调用该命令。

```
sudo ln -s /System/Library/PrivateFrameworks/Apple80211.framework/Versions/Current/Resources/airport /usr/local/bin/airport
```

## 扫描

```
airport en0 scan
```

## 监听

```
sudo airport en0 sniff 6
```

en0是所使用网卡的名称，sniff表示模式，6表示信道。

执行这个命令后就开启了监听模式，可以看到无线网络的链接状态已经改变了。

等待一段时间，后就可以按control+C结束监听，这时看到提示，一个airportSniffXXXXXX.cap的文件被保存在/tmp文件夹中。其中XXXXXX是唯一的，不用担心多次监听会覆盖文件。

# 破解

```
aircrack-ng -w generatedCrunch.txt /tmp/airportSniffXXXXXX.cap
```

-w表示使用字典，generatedCrunch.txt是字典所在的路径，/tmp/airportSniffXXXXXX.cap是刚才监听生成的文件。执行后会分析这个文件，最后会询问你得目标网络是哪个。输入序号并回车，就可以看到破解的密码了。

# 创建密码字典

所谓的密码字典主要是配合密码破解软件所使用，密码字典里包括许多人们习惯性设置的密码。这样可以提高密码破解软件的密码破解成功率和命中率，缩短密码破解的时间。当然，如果一个人密码设置没有规律或很复杂，未包含在密码字典里，这个字典就没有用了，甚至会延长密码破解所需要的时间。

## Crunch

Crunch是一种创建密码字典工具，该字典通常用于暴力破解。使用Crunch工具生成的密码可以发送到终端、文件或另一个程序。

使用crunch命令生成密码的语法格式如下所示：

```
crunch [minimum length] [maximum length] [character set] [options]
```

crunch命令常用的选项如下所示。

- -o：用于指定输出字典文件的位置。
- -b：指定写入文件最大的字节数。该大小可以指定KB、MB或GB，但是必须与-o START选项一起使用。
- -t：设置使用的特殊格式。
- -l：该选项用于当-t选项指定@、%或^时，用来识别占位符的一些字符。

创建一个密码列表文件并保存。其中，生成密码列表的最小长度为8，最大长度为10，并使用ABCDEFGabcdefg0123456789为字符集。执行命令如下所示：

```
crunch 8 10 ABCDEFGabcdefg0123456789 –o generatedCrunch.txt
```
