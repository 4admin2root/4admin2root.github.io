---
layout:     post
title:      mysql忘记root密码进行重置
date:       2012-08-11
tags: [cnblogs]
---
1、登陆到服务器端2、service mysql stop3、mysqld --skip-grant-tables4、重置密码[root@testdb1 ~]# mysql -u root -pEnter password: Welcome to the MySQL monitor.  Commands end with ; or \g.Your MySQL connection id is 1Server version: 5.5.25 MySQL Community Server (GPL)

Copyright (c) 2000, 2011, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or itsaffiliates. Other names may be trademarks of their respectiveowners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> use mysqlReading table information for completion of table and column namesYou can turn off this feature to get a quicker startup with -A

Database changedmysql> update user set password=password('xxxxxxxx') where user='root';Query OK, 4 rows affected (0.09 sec)Rows matched: 4  Changed: 4  Warnings: 0

mysql> exit5、kill -TERM pid (pid 为mysqld的进程id)6、service mysql start
