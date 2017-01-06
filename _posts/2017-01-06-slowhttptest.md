---
layout: post
title: 缓慢的 http 拒绝服务攻击测试 - slowhttptest
keywords: slowhttptest, ddos
description: 缓慢的 http 拒绝服务攻击测试 - slowhttptest
date: '2017-01-06 10:00:00 +0800'
categories: test
---

# slowhttptest

slowhttptest 是一个依赖于实际 HTTP 协议的 Slow HTTP DoS 攻击工具，它的设计原理是要求服务器所有请求被完全接收后再进行处理。

slowhttptest 是一个可配置的应用层拒绝服务攻击测试攻击，它可以工作在 Linux，OSX 和 Cygwin 环境以及 Windows 命令行接口，可以帮助安全测试人员检验服务器对慢速攻击的处理能力。

这个工具可以模拟低带宽耗费下的 DoS 攻击，比如慢速攻击，慢速 HTTP POST，通过并发连接池进行的慢速读攻击（基于TCP持久时间）等。慢速攻击基于HTTP协议，通过精心的设计和构造，这种特殊的请求包会造成服务器延时，而当服务器负载能力消耗过大即会导致拒绝服务。

## 下载

slowhttptest 的源码目前托管在 Github 上： <https://github.com/shekyan/slowhttptest>

## 安装

```bash
./configure
make
sudo make install
```

## 使用

```bash
slowhttptest -c 1000 -B -g -o output-file-name -i 100 -r 300 -s 10240 -u http://www.example.com/url/page.html -x 20
```

参数说明

```bash
—a 开始开始值范围说明符用于范围头测试
-b 将字节限制的范围说明符用于范围头测试
-c 连接数限制为65539
-d proxy host:port 用于指导所有流量通过web代理
-e proxy host:port 端口用于指导只有探针交通通过web代理
-h,B,R或x 指定减缓在头部分或在消息体,-R 允许范围检验,使慢读测试-x
-g 生成统计数据在CSV和HTML格式,模式是缓慢的xxx。csv / html,其中xxx是时间和日期
-i seconds 秒间隔跟踪数据在几秒钟内,每个连接
-k 管道因子次数重复请求在同一连接慢读测试如果服务器支持HTTP管道内衬。
-l 在几秒钟内，持续测试时间
-n 秒间隔从接收缓冲区读取操作
-o 文件定义输出文件路径和/或名称,如果指定有效-g
-p 秒超时等待HTTP响应在探头连接后,服务器被认为是不可访问的
-r seconds 连接速度
-s 字节值的内容长度标题详细说明,如果指定-b
-t verb 自定义
-u URL 目标URL,相同的格式键入浏览器，e.g: https://host[:port]/
-v level 冗长等级0 -4的日志
-w 字节范围广告的窗口大小会选择从
-x 字节最大长度的跟踪数据结束
-y 字节范围广告的窗口大小会选择从
-z 字节从接收缓冲区读取字节与单一的 read() 操作
```

更多参数说明请输入

```bash
slowhttptest -h
```
