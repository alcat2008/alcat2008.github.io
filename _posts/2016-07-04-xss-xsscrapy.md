---
layout: post
title: XSS 漏洞检查 — xsscrapy
date: '2016-07-04 10:00:00 +0800'
tags: [PT]
categories: test
---

# xsscrapy

xsscrapy是一个快速、直接的XSS漏洞检测爬虫，你只需要一个URL，它便可以帮助你发现XSS跨站脚本漏洞。

xsscrapy的XSS漏洞攻击测试将会覆盖:

- Cookies
- User-Agent
- Referer
- URL variables
- End of URL
- URL path
- Forms both hidden and explicit ‍‍‍‍‍‍

因为xsscrapy并不是一个浏览器，所以对AJAX无能为力.

## 安装

xsscrapy使用python写的，安装比较简单。

```shell
git clone https://github.com/DanMcInerney/xsscrapy.git
cd xsscrapy
pip install -r requirements.txt
```

## 使用

Fast, thorough, XSS/SQLi spider. Give it a URL and it'll test every link it finds for cross-site scripting and some SQL injection vulnerabilities. See FAQ for more details about SQLi detection.

From within the main folder run:

```shell
./xsscrapy.py -u http://example.com
```

If you wish to login then crawl:

```shell
./xsscrapy.py -u http://example.com/login_page -l loginname
```

If you wish to login with HTTP Basic Auth then crawl:

```shell
./xsscrapy.py -u http://example.com/login_page -l loginname --basic
```

If you wish to use cookies:

```shell
./xsscrapy.py -u http://example.com/login_page --cookie "SessionID=abcdef1234567890"
```

If you wish to limit simultaneous connections to 20:

```shell
./xsscrapy.py -u http://example.com -c 20
```

If you want to rate limit to 60 requests per minute:

```shell
./xsscrapy.py -u http://example.com/ -r 60
```

XSS vulnerabilities 检测结果将会存储在 `xsscrapy-vulns.txt` 文件中。
