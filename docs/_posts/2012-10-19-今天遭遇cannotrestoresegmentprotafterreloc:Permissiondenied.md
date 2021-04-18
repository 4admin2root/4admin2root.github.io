---
layout:     post
title:      今天遭遇cannotrestoresegmentprotafterreloc:Permissiondenied
date:       2012-10-19
tags: [cnblogs]
---
服务器重新安装了操作系统

安装好oracle软件后

修改/etc/ld.so.conf

增加oracle的lib目录

可是在执行业务应用时

报错cannot restore segment prot after reloc:Permission denied

后来发现 要关闭selinux

重启后，应用正常

==========================

附上

chcon命令：修改对象（文件）的安全上下文。比如：用户：角色：类型：安全级别。 命令格式：

说明：
