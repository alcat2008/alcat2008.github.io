---
layout: post
title: Nessus 填坑之旅
date: '2016-06-29 15:37:01 +0800'
categories: test
---

# Nessus

Nessus号称是世界上最流行的漏洞扫描程序，全世界有超过75000个组织在使用它。该工具提供完整的电脑漏洞扫描服务，并随时更新其漏洞数据库。Nessus不同于传统的漏洞扫描软件，Nessus可同时在本机或远端上遥控，进行系统的漏洞分析扫描。

此文记录安装及使用Nessus过程中已经或者即将发生的问题。

## Pre-install

官方手册列出了几点，罗列如下：

- The Nessus UI uses port **_`8834`_**.
- If your virtual machine is using Network Address Translation (`NAT`) to reach the network, many of Nessus vulnerability checks, host enumeration, and operating system identification will be negatively affected. (如果虚拟机网络设置为 `NAT`，会对Nessus的漏洞检测、主机列举分析、操作系统探测等功能产生影响。)
- By default, Nessus is installed and managed using HTTPS and SSL, uses port 8834, and the default installation of Nessus uses a self-signed SSL certificate. （Nessus使用自签的SSL证书，浏览器会有警告。）

## 下载

当然是选择 `Nessus Home` 版本了，其他的忽略。

下载地址： <http://www.tenable.com/products/nessus/select-your-operating-system>

选择操作系统并下载，没什么好所的。

## 安装

安装也很简单，双击，一步一步就可以了。

## 卸载 （`Mac`）

删除下面的文件或者文件夹

```
/Library/Nessus
/Library/LaunchDaemons/com.tenablesecurity.nessusd.plist
/Library/PreferencePanes/Nessus Preferences.prefPane
/Applications/Nessus
```

保险起见，禁用 Nessus 服务

```
sudo launchctl remove com.tenablesecurity.nessusd
```

## 启动 & 关闭 （命令行）

Mac OS X

```
// start
launchctl load –w /Library/LaunchDaemons/com.tenablesecurity.nessusd.plist

// stop
launchctl unload –w /Library/LaunchDaemons/com.tenablesecurity.nessusd.plist
```

Windows

```
// start
C:\Windows\system32>net start “Tenable Nessus”

// stop
C:\Windows\system32>net stop “Tenable Nessus”
```

## 获取激活码

在Nessus官网注册完成，就会生成一个激活码发送到邮箱。

注册地址： <http://www.tenable.com/products/nessus-home>

需要注意的是，生成的激活码是一次性的，如果重装的时候需要重新注册获取新的激活码。

## 配置 & 插件下载

浏览器中打开Nessus主页，地址为： <https://localhost:8834/>

欢迎信息 => 创建Nessus服务管理账号 => 输入激活码=> 下载插件

最后的下载插件页面在国内基本上不能安装成功，基本上有两种办法解决：

- 在程序安装目录下，反复执行 `nessuscli update`，直至成功。多试几次，我尝试的结果是直接访问，把翻墙工具都关掉。
- 参考文章 <http://www.tenable.com/products/nessus/documentation/activation-code-installation> 中 `Nessus without Internet Access` 一节的内容。
