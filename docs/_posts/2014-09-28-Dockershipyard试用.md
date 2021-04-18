---
layout:     post
title:      Dockershipyard试用
date:       2014-09-28
tags: [cnblogs]
---
Shipyard 提供一个docker web访问方式，配置安装简单

docker主机操作系统ubuntu 14.04

1、首先配置docker，将访问接口打开

修改/etc/default/docker.io，在DOCKER_OPTS=""

加入-H tcp://0.0.0.0:4243

2、重启docker host

3、下载shipyard并启动

docker run -d -P --name rethinkdb shipyard/rethinkdb

docker run -d -p 8080:8080 --link rethinkdb:rethinkdb shipyard/shipyard

（注意端口别冲突）

 

4、打开浏览器访问ip:8080

用户名admin 密码shipyard

 

5、增加engines

点击engines--add，输入cpu、内存、地址和端口4243

6、点击containers

已经可以看到你启动的容器了
