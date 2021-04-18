---
layout:     post
title:      centos7systemddockerhttpproxy
date:       2015-09-07
tags: [cnblogs]
---
1、设置用户使用代理（非必须）

[root@localhost ~]# cat .bash_profile # .bash_profile

# Get the aliases and functionsif [ -f ~/.bashrc ]; then	. ~/.bashrcfi

# User specific environment and startup programs

export PATHhttp_proxy=http://10.3.1.40:3128https_proxy=http://10.3.1.40:3128

2、设置systemd docker.service

[root@localhost ~]# cat /usr/lib/systemd/system/docker.service[Unit]Description=Docker Application Container EngineDocumentation=https://docs.docker.comAfter=network.target docker.socketRequires=docker.socket

[Service]Environment=HTTP_PROXY=http://10.3.1.40:3128/Type=notifyExecStart=/usr/bin/docker daemon -H fd://MountFlags=slaveLimitNOFILE=1048576LimitNPROC=1048576LimitCORE=infinity

[Install]WantedBy=multi-user.target

3、配置重载

[root@localhost ~]# systemctl daemon-reload && systemctl start docker.service
