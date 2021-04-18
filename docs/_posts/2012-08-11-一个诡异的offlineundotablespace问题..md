---
layout:     post
title:      一个诡异的offlineundotablespace问题.
date:       2012-08-11
---
数据库 oracle 10.2.0.4  RAC 2 nodes

操作系统 redhat 5.3  x64

周末升级系统，数据库一个过程出现编译问题，没太在意

周一发现错误还在，

错误信息：

Error 604 trapped in 2PC on transaction 67.38.6337026. Cleaning up.Error stack returned to user:ORA-00604: error occurred at recursive SQL level 1ORA-00376: file 40 cannot be read at this timeORA-01110: data file 40: '+DG1/xxx/datafile/undotbs3.422.720270075'

检查执行语句为调用一个包的过程

其过程很简单

类似如下

上面的undo是2个月前offline的 本来是打算删除的

两个实例的undo 参数均没有指定到该undo 表空间上

很奇怪啊后来发现执行SQL> select * from FLASHBACK_TRANSACTION_QUERY;select * from FLASHBACK_TRANSACTION_QUERY              *ERROR at line 1:ORA-01135: file 40 accessed for DML/query is offlineORA-01110: data file 40: '+DG1/xxx/datafile/undotbs3.422.720270075'

这个undo 不是offline 了吗？而且两个月了！

为了尽快恢复该问题

重新online 了该表空间

重新 编译 包：正常

查询FLASHBACK_TRANSACTION_QUERY ：正常

==

遗留问题
