---
layout:     post
title:      安装devstack之配置proxy
date:       2014-08-27
tags: [cnblogs]
---
Ubuntu 版本：Ubuntu 12.04.5 LTS

配置环境变量

$ cat /etc/environment PATH="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games"export http_proxy=http://10.0.0.2:3128export https_proxy=https://10.0.0.2:3128

配置apt.conf

$cat /etc/apt/apt.confAcquire::http::proxy "http://10.0.0.2:3128/";Acquire::https::proxy "https://10.0.0.2:3128/";Acquire::ftp::proxy "ftp://10.0.0.2:3128/";

配置git

git config --global http.proxy http://10.0.0.2:3128/git config --global core.gitproxy gitproxy

sudo vim /usr/bin/gitproxy====================================#!/bin/sh# Use socat to proxy git through an HTTP CONNECT firewall.# Useful if you are trying to clone git:// from inside a company.# Requires that the proxy allows CONNECT to port 9418.## Save this file as gitproxy somewhere in your path (e.g., ~/bin) and then run#   chmod +x gitproxy#   git config --global core.gitproxy gitproxy## More details at http://tinyurl.com/8xvpny # Configuration. Common proxy ports are 3128, 8123, 8000._proxy=10.0.0.2_proxyport=3128 exec socat STDIO PROXY:$_proxy:$1:$2,proxyport=$_proxyport====================================sudo chmod +x /usr/bin/gitproxy

代理服务器squid.conf新增（注意位置）

acl git port 9418       http_access allow git
