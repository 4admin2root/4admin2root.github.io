---
layout:     post
title:      转：ORACLE_SID正确，但还是Connectedtoanidleinstance
date:       2014-07-24
tags: [cnblogs]
---
今天也遭遇了这个问题，此前确实没注意过。！@#￥%&*（）

出处http://blog.itpub.net/22966231/viewspace-1029527/

很多人都可能在Unix-Like的操作系统上遇到过这样的问题：ORACLE_HOME和ORACLE_SID的设置是正确的，实例也启动了，但通过sqlplus / as sysdba连接时还是报告"Connected to an idle instance"，这是怎么回事？

这问题很有可能是因为启动实例的用户会话的ORACLE_HOME变量和想通过"sqlplus / as sysdba"连接实例的用户会话的ORACLE_HOME变量差一个后缀正斜杠"/"。

[@more@]

因为当你用sqlplus / as sysdba时，Oracle使用ORACLE_HOME和ORACLE_SID来判断你要连接哪个实例。Oracle允许在不同的ORACLE_HOME下创建相同实例名的实例，当你用sqlplus / as sysdba连接实例时，ORACLE_HOME同样是一个关键的区分因素。所以当ORACLE_HOME多了或少了一个后缀"/"之后，虽然ORACLE_SID相同，但加上ORACLE_HOME还是与运行中的实例不符，所以 不能连接。解决很简单，保持用户的ORACLE_HOME相同就可以了。可以参考一个标准，到/etc/oratab里去查看一下就知道了。sqlplus sys/oracle@TNSNAME as sysdba走的是TNS协议，没有以上的问题。
