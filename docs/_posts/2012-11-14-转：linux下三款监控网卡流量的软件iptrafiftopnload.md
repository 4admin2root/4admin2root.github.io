---
layout:     post
title:      转：linux下三款监控网卡流量的软件iptrafiftopnload
date:       2012-11-14
---
下一步准备做运维项目监测系统的操作系统监测其中对外服务涉及到网络流量的监测自己写程序有点麻烦，于是网上找了这篇文章只是没想好怎么整合到系统中原文地址：

[http://zhumeng8337797.blog.163.com/blog/static/10076891420091123104326703/](http://zhumeng8337797.blog.163.com/blog/static/10076891420091123104326703/)

==========================================================================

本文出自 [高兴F](http://gaoxingf.blog.51cto.com/) 博客，请务必保留此出处[http://gaoxingf.blog.51cto.com/612518/188966](http://gaoxingf.blog.51cto.com/612518/188966)

一.添加yum源方便安装bmon# rpm -Uhv [http://apt.sw.be/redhat/el5/en/x ... 1.el5.rf.x86_64.rpm](http://apt.sw.be/redhat/el5/en/x86_64/rpmforge/RPMS//rpmforge-release-0.3.6-1.el5.rf.x86_64.rpm)# yum install bmon- bandwidth monitor可以在 shell 下监控网络流量的状况( 具有动态效果 )

- RX: 流进   
- TX: 流出


安装成功后输入bmon命令后，选择eth0按g,按d。查看效果如如下:#bmon <img src="http://bbs.linuxtone.org/attachments/day_090319/09031910259814a4469ce6ab4b.jpg" alt="bmon.jpg" width="586" /> 

以基本的方式查看:#bmon -o ascii -p eth0

很好很强大!支持下！ <img src="http://bbs.linuxtone.org/images/smilies/default/hug.gif" alt="" border="0" /> linuxtone补充：再介绍两个查看网络状况的软件:# yum install nload <img src="http://bbs.linuxtone.org/attachments/day_090324/0903242124b94bf75a8831ac5e.png" alt="nload.png" width="600" /> 

nload默认分为上下两块：上半部分是：Incoming也就是进入网卡的流量，下半部分是：Outgoing，也就是从这块网卡出去的流量，每部分都有当前流量（Curr），平均流量（Avg），最小流量（Min），最大流量（Max），总和流量（Ttl）这几个部分，看起来还是蛮直观的。#nload --help 查看具体用法2.iftop# yum install iftop# iftop -i eth0 <img src="http://bbs.linuxtone.org/attachments/day_090324/0903242124e0db2cfbac7d9593.png" alt="iftop.png" width="600" /> 

TX：发送流量RX：接收流量TOTAL：总流量Cumm：运行iftop到目前时间的总流量peak：流量峰值rates：分别表示过去 2s 10s 40s 的平均流量#iftop -i eth0 -n       就可以看到eth0网卡的流量状况：iftop 相关命令 ： 监控eth1的网卡的流量 # iftop -i eth1 以位元组(bytes)为单位显示流量(预设是位元bits): $ iftop -B 直接显示IP, 不进行DNS反解: $ iftop -n 直接显示连接埠编号, 不显示服务名称: $ iftop -N 显示某个网段进出封包流量 $ iftop -F 192.168.1.0/24 or 192.168.1.0/255.255.255.0 其他参数可下 iftop -h 看说明. 进入iftop画面时, 可按 p 切换是否显示连接埠, n 切换显示IP或主机的domain name, N切换显 示连接埠代号或名称, p暂停显示, b切换是否显示长条, B切换计算几秒内的平均流量, 其他按键 可以按h观看说明.

很
久以前在sbilly的blog看到他介绍nload这个东西，后来忘记名字了，到是记下了ifstat这个看流量的东西，今天看了下nload，很强
大，还有很多选项以前没设置过的。nload eth0 -m，可以不显示那个柱图，只显示数据。man nload还有其他的参数。上图。

[<img src="http://baoz.net/wp-content/2009/06/nload.png" alt="nload" width="807" height="581" />](http://baoz.net/wp-content/2009/06/nload.png)

[<img src="http://baoz.net/wp-content/2009/06/nload2.png" alt="nload2" width="910" height="486" />](http://baoz.net/wp-content/2009/06/nload2.png)

或者直接设置下.nload文件也可以

[root@F4 ~]# cat .nloadVersion=1AverageWindow=300BarMaxIn=51200BarMaxOut=51200DataFormat=MBitDevices=eth0MultipleDevices=[ ]RefreshInterval=500TrafficFormat=MBitVersion=1

ifstat的输出也不赖

[root@F4 ~]# ifstat -b       eth0       Kbps in  Kbps out 4991.09   4938.29 5461.71   5207.63 4663.97   5433.32 5862.77   4878.62 6852.58   6936.53 6016.11   6218.63 7211.42   6264.69 4986.47   4965.24 4883.85   4828.53 5653.91   4948.54 6339.58   6968.81


      vnStat是一个Linux下的网络流量监控软件,它记录指定网卡每日的传输流量日志.它并非基于网络包的过滤,而是分析文件系统- /proc, 所以vnStat无需root的权限就可使用    在Ubuntu下安装很方便(已经集成到源里,安装需要使用root用户进行):    apt-get install vnstat    vnstat -u -i eth0    (eth0为你的网卡名称)    大约等待3-5分钟左右,就可以使用vnstat 命令查询流量状况了       -------    在Ubuntu下卸载:    dpkg --purge vnstat    使用也非常简单:$ vnstatDatabase updated: Mon Mar 5 09:15:00 2007        inet (eth0)           received:      1,002,061 MB (24.6%)        transmitted:      3,068,177 MB (75.4%)              total:      4,070,238 MB                        rx     |     tx     |  total        -----------------------+------------+-----------        yesterday       335 MB |   6,881 MB |   7,216 MB            today     1,493 MB |   9,808 MB |  11,301 MB        -----------------------+------------+-----------        estimated     1,610 MB |  10,579 MB |  12,189 MB还可以按每周,每日,每小时统计下面是按小时统计的图示:$ vnstat -heth0                                                                     15:45  ^                       r                                                       |                       rt                                                     |                       rt                                                     |                       rt    r                                                 |                       rt    r                                                 |                       rt    r                             r  r                |                       rt r  r  r  r     r  r  r  r  r  r  r  r  rt           |      t           t    rt r  r  r  r  r  r  r  r  r  r  r  r  rt rt           |     rt       rt rt    rt r  r  r  r  r  r  r  r  r  r  r  r  rt rt rt        |  rt rt rt rt rt rt rt rt rt rt rt r  r  r  r  r  r  r  r  rt rt rt rt rt   -+--------------------------------------------------------------------------->  |  16 17 18 19 20 21 22 23 00 01 02 03 04 05 06 07 08 09 10 11 12 13 14 15                                                                                   h   rx (kB)    tx (kB)      h   rx (kB)    tx (kB)      h   rx (kB)    tx (kB)16      2,684      3,135    00      8,960      3,033    08      8,071      1,07417      4,401      5,433    01     12,740      1,981    09      8,395      1,35418      2,443      3,056    02      8,955      2,658    10      8,702      1,67119      2,059      2,563    03      7,748      1,440    11      9,211      3,46320      3,920      5,227    04      6,386      1,615    12      9,530      5,98521      3,884      5,491    05      8,187      1,055    13      7,633      8,27622      2,139      2,289    06      8,100      1,130    14      3,771      4,91423     18,036     17,629    07      8,101      1,111    15      2,657      2,998------------------------   如果是FreeBSD下安装稍微多几个步骤,首先要下载vnStat :   [http://humdi.net/vnstat/](http://humdi.net/vnstat/)   使用命令解压缩:   #tar -xzvf vnstat-1.4_bsd.tar.gz   #cd vnstat   #make   #make install   #vnstat -u -i eth0   然后稍等3-5分钟,就可以运行vnStat查看流量了   (如果安装遇到问题,请查看INSTALL文档)
