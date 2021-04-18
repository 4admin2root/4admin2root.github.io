---
layout:     post
title:      转：如何修复MySQL数据库(MyISAMInnoDB)
date:       2014-07-11
---
http://blog.csdn.net/mypcstyle/article/details/37501123

[如何修复MySQL数据库(MyISAM / InnoDB)](http://blog.csdn.net/mypcstyle/article/details/37501123)

本文总结了7种修复MySQL数据库的方法，当我们重启数据库无法解决问题或者有数据表损坏时，可以尝试从下面列出的方法中找出相应的解决方案。

重启MySQL：

```
/usr/local/mysql/bin/mysqladmin -uUSERNAME -pPASSWORD shutdown
/usr/local/mysql/bin/mysqld_safe &
```

**1.MyISAM表损坏**MySQL数据库可以为不同的数据表指定不同的存储引擎，其中最流行的两种存储引擎是MyISAM和InnoDB。MyISAM类型的表容易损坏，这是个不争的事实。幸运的是，在大多数情况下，MyISAM表损坏是很容易修复。要修复单个损坏的数据表，可以连接数据库执行 ：

```
repair TABLENAME
```

要修复所有的表，执行如下命令：

```
/usr/local/mysql/bin/mysqlcheck --all-databases -uUSERNAME -pPASSWORD -r
```

很多时候，我们不会知道MyISAM表已经损坏了，除非查看了日志文件。我强烈建议您将此行添加到你的/etc/my.cnf配置文件中， 它会自动修复损坏的MyISAM表。

[mysqld]myisam-recover=backup,force

**2.多个MySQL实例**这种情况很常见，当你启动MySQL时，MySQL进程马上死掉。这时若查看MySQL日志，会发现这是因为有另一个MySQL进程正在运行。停止所有的MySQL实例

/usr/local/mysql/bin/mysqladmin -uUSERNAME -pPASSWORD shutdown

killall mysqlkillall mysqld

再启动MySQL后就只有一个MySQL实例运行了。

**3.修改MySQL日志设置**一旦我们在运行一个InnoDB的数据库，我们就不能修改/etc/my.cnf文件中的以下几行：

```
datadir = /usr/local/mysql/data
innodb_data_home_dir = /usr/local/mysql/data
innodb_data_file_path = ibdata1:10M:autoextend
innodb_log_group_home_dir = /usr/local/mysql/data
innodb_log_files_in_group = 2
innodb_log_file_size = 5242880
```

InnoDB的日志文件的大小一但确定就不能再修改了，否则会造成数据库无法启动。

**4.丢失MySQL host表**这种情况我碰到过几次，这可能是一个MyISAM的bug，修复方法如下：

/usr/local/bin/mysql_install_db

**5.损坏的MyISAM auto_increment**假如一个MyISAM表中的auto_increment变得紊乱，你将不能向其表中插入新记录。通过将最后一条记录的auto_increment设为-1，可以告诉auto_increment，他现在的工作不正常。找到最后一条记录的auto_increment的有效值

然后再更新表的auto_increment

ALTER TABLE tablename AUTO_INCREMENT = id+1

**6.太多的数据库连接**当我们的数据库连接数过多时，会造成无法连接到数据库的问题。这时，我们要先停止数据库

/usr/local/mysql/bin/mysqladmin -uUSERNAME -pPASSWORD shutdown

当数据库停止后，我们就可以根据服务器的配置情况适当的增加连接数在一台专用的数据库服务器上，我们可以设置如下

```
max_connections = 200
wait_timeout = 100
```

最后再启动数据库。

**7.InnoDB表损坏**InnoDB拥有内部恢复机制，假如数据库崩溃了，InnoDB通过从最后一个时间戳开始运行日志文件，来尝试修复数据库。大多数情况下会修复成功，而且整个过程是透明的。假如InnoDB自行修复失败，那么数据库将不能启动。无论你怎样一次又一次的尝试启动MySQL，它都只是发出一条错误信息。这也是我们在使用 InnoDB时，需要运行master/master的原因，只有这样才能在一个master宕掉时，还有一台冗余master做后备。在继续操作前，先浏览下MySQL的日志文件，确定数据库是因为InnoDB表的损坏而崩溃。有一种方法是更新InnoDB的日志文件计数器以跳过引起崩溃的查询，但是经验告诉我们这不是个好方法。这种情况下，将造成数据的不一致性而且会经常使主从复制中断。一旦确定MySQL因为InnoDB表损坏无法启动时，我们就可以按照以下5步进行修复：1.添加如下配置到/etc/my.cnf文件中

innodb_force_recovery = 4

2.这时就可以重新启动数据库了，在innodb_force_recovery配置的作用，所有的插入与更新操作将被忽略;3.导出所有的数据表;4.关闭数据库并删除所有数据表文件及目录，再运行 mysql_install_db来创建MySQL默认数据表;5.在/etc/my.cnf中删除innodb_force_recovery这一行，再启动MySQL（这时MySQL正常启动）;6.从第3步备份的文件中恢复所有的数据。

文章翻译自：http://www.softwareprojects.com/resources/programming/t-how-to-fix-mysql-database-myisam-innodb-1634.html
