---
layout: post
title: 源码安装 apache（httpd）
keywords: apache, httpd
description: 源码安装 apache（httpd）
date: '2017-01-10 10:00:00 +0800'
categories: ops
---

# 源码安装 apache（httpd）

记录手动安装 apache （httpd）的过程

## 准备

安装包如下：

```
apr-1.5.2.tar.gz
apr-util-1.5.4.tar.gz
httpd-2.4.25.tar.gz
pcre2-10.22.zip
```

## 安装

### apr

```
tar xzvf apr-1.5.2.tar.gz
cd apr-1.5.2/
./configure --prefix=/usr/local/apr
make && make install
```

### apr-util

```
tar xzvf apr-util-1.5.4.tar.gz
cd apr-util-1.5.4/
./configure --prefix=/usr/local/apr-util --with-apr=/usr/local/apr/bin/apr-1-config
make && make install
```

### pcre

```
unzip pcre2-10.22.zip
cd pcre2-10.22/
./configure --prefix=/usr/local/pcre
make && make install
```

### apache (httpd)

```
tar xzvf httpd-2.4.25.tar.gz
cd httpd-2.4.25/
./configure --with-pcre=/usr/local/pcre/bin/pcre2-config
make && make install
```

## 配置

最新版的 apache 安装后的服务名是 `apache2`，不再是 `httpd`，配置文件地址为 `/etc/apache2/apache2.conf`
