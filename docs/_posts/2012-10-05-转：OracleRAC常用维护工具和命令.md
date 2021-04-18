---
layout:     post
title:      转：OracleRAC常用维护工具和命令
date:       2012-10-05
---
记忆力不好，一直记不住，转个帖子在此

================================================

原文链接http://www.2cto.com/database/201202/120555.html

Oracle 的管理可以通过OEM或者命令行接口。Oracle Clusterware的命令集可以分为以下4种：
节点层：osnodes
网络层：oifcfg
集群层：crsctl, ocrcheck,ocrdump,ocrconfig
应用层：srvctl,onsctl,crs_stat
下面分别来介绍这些命令。
一． 节点层
只有一个命令：　osnodes， 这个命令用来显示集群点列表，可用的参数如下，这些参数可以混合使用。
[root@raw1 bin]# ./olsnodes --help
Usage: olsnodes [-n] [-p] [-i] [<node> | -l] [-g] [-v]
        where
                -n print node number with the node name
                -p print private interconnect name with the node name
                -i print virtual IP name with the node name
                <node> print information for the specified node
                -l print information for the local node
                -g turn on logging
                -v run in verbose mode
[root@raw1 bin]# ./olsnodes -n -p -i
raw1    1       raw1-priv       raw1-vip
raw2    2       raw2-priv       raw2-vip
二． 网络层
 网络层由各个节点的网络组件组成，包括2个物理网卡和3个IP 地址。 也只有一个命令：oifcfg.
 Oifctg 命令用来定义和修改Oracle 集群需要的网卡属性，这些属性包括网卡的网段地址，子网掩码，接口类型等。 要想正确的使用这个命令， 必须先知道Oracle 是如何定义网络接口的，Oracle的每个网络接口包括名称，网段地址，接口类型3个属性。
Oifcfg 命令的格式如下：interface_name/subnet:interface_type
这些属性中没有IP地址，但接口类型有两种，public和private，前者说明接口用于外部通信，用于Oracle Net和VIP 地址，而后者说明接口用于Interconnect。
接口的配置方式分为两类：global 和node-specific。 前者说明集群所有节点的配置信息相同，也就是说所有节点的配置是对称的；而后者意味着这个节点的配置和其他节点配置不同，是非对称的。
Iflist：显示网口列表
Getif: 获得单个网口信息
Setif：配置单个网口
Delif：删除网口
[root@raw1 bin]# ./oifcfg --help
PRIF-9: incorrect usage
Name:
        oifcfg - Oracle Interface Configuration Tool.
Usage:  oifcfg iflist [-p [-n]]
        oifcfg setif {-node <nodename> | -global} {<if_name>/<subnet>:<if_type>}...
        oifcfg getif [-node <nodename> | -global] [ -if <if_name>[/<subnet>] [-type <if_type>] ]
        oifcfg delif [-node <nodename> | -global] [<if_name>[/<subnet>]]
        oifcfg [-help]
        <nodename> - name of the host, as known to a communications network
        <if_name>  - name by which the interface is configured in the system
        <subnet>   - subnet address of the interface
        <if_type>  - type of the interface { cluster_interconnect | public | storage }
[root@raw1 bin]# ./oifcfg iflist
eth0  10.85.10.0
eth1  192.168.1.0
[root@raw1 bin]# ./oifcfg getif
eth0  10.85.10.119  global  public
eth0  10.85.10.121  global  public
eth0  10.85.10.0  global  public
eth1  192.168.1.119  global  cluster_interconnect
eth1  192.168.1.121  global  cluster_interconnect
eth1  192.168.1.0  global  cluster_interconnect
-- 查看public 类型的网卡
[root@raw1 bin]# ./oifcfg getif -type public
eth0  10.85.10.119  global  public
eth0  10.85.10.121  global  public
eth0  10.85.10.0  global  public
-- 删除接口配置
[root@raw1 bin]# ./oifcfg delif -global
-- 添加接口配置
[root@raw1 bin]# ./oifcfg setif -global eth0/192.168.1.119:public
[root@raw1 bin]# ./oifcfg setif -global eth1/10.85.10.119:cluster_interconnect
三． 集群层
集群层是指由Clusterware组成的核心集群， 
这一层负责维护集群内的共享设备，并为应用集群提供完整的集群状态视图，应用集群依据这个视图进行调整。 这一层共有4个命令：crsctl, 
ocrcheck,ocrdump,ocrconfig. 后三个是针对OCR 磁盘的。
3.1 CRSCTL
Crsctl 命令可以用来检查CRS 进程栈，每个crs 进程状态，管理Votedisk，跟踪CRS进程功能。
[root@raw1 bin]# ./crsctl
Usage: crsctl check  crs          - checks the viability of the CRS stack
       crsctl check  cssd         - checks the viability of CSS
       crsctl check  crsd         - checks the viability of CRS
       crsctl check  evmd         - checks the viability of EVM
       crsctl set    css <parameter> <value> - sets a parameter override
       crsctl get    css <parameter> - gets the value of a CSS parameter
       crsctl unset  css <parameter> - sets CSS parameter to its default
       crsctl query  css votedisk    - lists the voting disks used by CSS
       crsctl add    css votedisk <path> - adds a new voting disk
       crsctl delete css votedisk <path> - removes a voting disk
       crsctl enable  crs    - enables startup for all CRS daemons
       crsctl disable crs    - disables startup for all CRS daemons
       crsctl start crs  - starts all CRS daemons.
       crsctl stop  crs  - stops all CRS daemons. Stops CRS resources in case of cluster.
       crsctl start resources  - starts CRS resources.
       crsctl stop resources  - stops  CRS resources.
       crsctl debug statedump evm  - dumps state info for evm objects
       crsctl debug statedump crs  - dumps state info for crs objects
       crsctl debug statedump css  - dumps state info for css objects
       crsctl debug log css [module:level]{,module:level} ...
                             - Turns on debugging for CSS
       crsctl debug trace css - dumps CSS in-memory tracing cache
       crsctl debug log crs [module:level]{,module:level} ...
                             - Turns on debugging for CRS
       crsctl debug trace crs - dumps CRS in-memory tracing cache
       crsctl debug log evm [module:level]{,module:level} ...
                             - Turns on debugging for EVM
       crsctl debug trace evm - dumps EVM in-memory tracing cache
       crsctl debug log res <resname:level> turns on debugging for resources
       crsctl query crs softwareversion [<nodename>] - lists the version of CRS software installed
       crsctl query crs activeversion - lists the CRS software operating version
       crsctl lsmodules css - lists the CSS modules that can be used for debugging
       crsctl lsmodules crs - lists the CRS modules that can be used for debugging
       crsctl lsmodules evm - lists the EVM modules that can be used for debugging
 If necesary any of these commands can be run with additional tracing by
 adding a "trace" argument at the very front.
 Example: crsctl trace check css
3.1.1 检查CRS 状态
[root@raw1 bin]# ./crsctl check crs
CSS appears healthy
CRS appears healthy
EVM appears healthy
-- 检查单个状态
[root@raw1 bin]# ./crsctl check cssd
CSS appears healthy
[root@raw1 bin]# ./crsctl check crsd
CRS appears healthy
[root@raw1 bin]# ./crsctl check evmd
EVM appears healthy
3.1.2 配置CRS 栈是否自启动
   CRS 进程栈默认随着操作系统的启动而自启动，有时出于维护目的需要关闭这个特性，可以用root 用户执行下面命令。
[root@raw1 bin]# ./crsctl disable crs
[root@raw1 bin]# ./crsctl enable crs
这个命令实际是修改了/etc/oracle/scls_scr/raw/root/crsstart 这个文件里的内容。
3.1.3 启动，停止CRS 栈。
Oracle 在10.1时，必须通过重新启动系统重启Clusterware，但是从Oracle 10.2 开始，可以通过命令来启动和停止CRS.
-- 启动CRS：
[root@raw1 bin]# ./crsctl start crs
Attempting to start CRS stack
The CRS stack will be started shortly
-- 关闭CRS：
[root@raw1 bin]# ./crsctl stop crs
Stopping resources.
Successfully stopped CRS resources
Stopping CSSD.
Shutting down CSS daemon.
Shutdown request successfully issued.
3.1.4 查看Votedisk 磁盘位置
[root@raw1 bin]# ./crsctl query css votedisk
 0.     0    /dev/raw/raw2
located 1 votedisk(s).
3.1.5 查看和修改CRS 参数
-- 查看参数：用get
[root@raw1 bin]# ./crsctl get css misscount
60
-- 修改参数： 用set， 但是这个功能要慎用
[root@raw1 bin]# ./crsctl set css miscount 60
3.1.6 跟踪CRS 模块，提供辅助功能
CRS由CRS，CSS，EVM 三个服务组成，每个服务又是由一系列module组成，crsctl 允许对每个module进行跟踪，并把跟踪内容记录到日志中。
[root@raw1 bin]# ./crsctl lsmodules css
The following are the CSS modules ::
    CSSD
    COMMCRS
COMMNS
[root@raw1 bin]# ./crsctl lsmodules crs
The following are the CRS modules ::
    CRSUI
    CRSCOMM
    CRSRTI
    CRSMAIN
    CRSPLACE
    CRSAPP
    CRSRES
    CRSCOMM
    CRSOCR
    CRSTIMER
    CRSEVT
    CRSD
    CLUCLS
    CSSCLNT
    COMMCRS
    COMMNS
[root@raw1 bin]# ./crsctl lsmodules evm
The following are the EVM modules ::
   EVMD
   EVMDMAIN
   EVMCOMM
   EVMEVT
   EVMAPP
   EVMAGENT
   CRSOCR
   CLUCLS
   CSSCLNT
   COMMCRS
   COMMNS
--跟踪CSSD模块，需要root 用户执行：
[root@raw1 bin]# ./crsctl debug log css "CSSD:1"
Configuration parameter trace is now set to 1.
Set CRSD Debug Module: CSSD  Level: 1
-- 查看跟踪日志
[root@raw1 cssd]# pwd
/u01/app/oracle/product/crs/log/raw1/cssd
[root@raw1 cssd]# more ocssd.log
...
[    CSSD]2010-03-08 00:19:27.160 [36244384] >TRACE:   
clssscSetDebugLevel: The logging level is set to 1 ,the cache level is 
set to 2
[    CSSD]2010-03-08 00:19:52.139 [119085984] >TRACE:   
clssgmClientConnectMsg: Connect from con(0x834fd18) proc(0x8354c70) 
pid() proto(10:2:1:1)
...
3.1.7 维护Votedisk
以图新方式安装Clusterware的过程中，在配置Votedisk时，如果选择External 
Redundancy策略。则只能填写一个Votedisk。但是即使使用External 
Redundancy作为冗余策略，也可以添加多个Vodedisk，只是必须通过crsctl 
命令来添加，添加多个Votedisk后，这些Votedisk 互为镜像，可以防止Votedisk的单点故障。
需要注意的是，Votedisk使用的是一种多数可用算法，如果有多个Votedisk，，则必须一半以上的Votedisk同时使
用，Clusterware才能正常使用。 
比如配置了4个Votedisk，坏一个Votedisk，集群可以正常工作，如果坏了2个，则不能满足半数以上，集群会立即宕掉，所有节点立即重启，所
以如果添加Votedisk，尽量不要只添加一个，而应该添加2个。这点和OCR 不一样。OCR 只需配置一个。
添加和删除Votedisk的操作比较危险，必须停止数据库，停止ASM,停止CRS Stack后操作，并且操作时必须使用-force参数。
1） 查看当前配置
[root@raw1 bin]# ./crsctl query css votedisk
2)  停止所有节点的CRS：
[root@raw1 bin]# ./crsctl stop crs
3） 添加Votedisk
     [root@raw1 bin]# ./crsctl add css votedisk /dev/raw/raw1 -force
注意：即使在CRS 关闭后，也必须通过-force 参数来添加和删除Votedisk，并且-force 参数只有在CRS关闭的场合下使用才安全。
 否则会报：Cluter is not a ready state for online disk addition.
4)  确认添加后的情况：
[root@raw1 bin]# ./crsctl query css votedisk
5） 启动CRS
[root@raw1 bin]# ./crsctl start crs
3.2  OCR命令系列
Oracle Clusterware把整个集群的配置信息放在共享存储上，这个存储就是OCR Disk. 在整个集群中，只有一个节点能对OCR 
Disk 进行读写操作，这个节点叫作Master Node，所有节点都会在内存中保留一份OCR的拷贝，同时哟一个OCR Process 
从这个内存中读取内容。OCR 内容发生改变时，由Master Node的OCR Process负责同步到其他节点的OCR Process。
因为OCR的内容如此重要，Oracle 每4个小时对其做一次备份，并且保留最后的3个备份，以及前一天，前一周的最后一个备份。 
这个备份由Master Node 
CRSD进程完成，备份的默认位置是$CRS_HOME/crs/cdata/<cluster_name>目录下。 
每次备份后，备份文件名自动更改，以反应备份时间顺序，最近一次的备份叫作backup00.ocr。这些备份文件除了保存在本地，DBA还应该在其他存
储设备上保留一份，以防止意外的存储故障。
3.2.1 ocrdump
该命令能以ASCII的方式打印出OCR的内容，但是这个命令不能用作OCR的备份恢复，也就是说产生的文件只能用作阅读，而不能用于恢复。
命令格式：ocrdump [-stdout] [filename] [-keyname name] [-xml]
参数说明：
        -stdout: 把内容打印输出到屏幕上
Filename：内容输出到文件中
-keyname：只打印某个键及其子健内容
-xml：以xml格式打印输出
   示例：把system.css键的内容以.xml格式打印输出到屏幕
[root@raw1 bin]# ./ocrdump -stdout -keyname system.css -xml|more
<OCRDUMP>
<TIMESTAMP>03/08/2010 04:28:41</TIMESTAMP>
<DEVICE>/dev/raw/raw1</DEVICE>
<COMMAND>./ocrdump.bin -stdout -keyname system.css -xml </COMMAND>
......
这个命令在执行过程中，会在$CRS_HOME/log/<node_name>/client 目录下产生日志文件，文件名ocrdump_<pid>.log,如果命令执行出现问题，可以从这个日志查看问题原因。
3.2.2 ocrcheck
Ocrcheck 命令用于检查OCR内容的一致性，命令执行过程会在$CRS_HOME/log/nodename/client 目录下产生ocrcheck_pid.log 日志文件。 这个命令不需要参数。
[root@raw1 bin]# ./ocrcheck
Status of Oracle Cluster Registry is as follows :
         Version                  :          2
         Total space (kbytes)     :     147352
         Used space (kbytes)      :       4360
         Available space (kbytes) :     142992
         ID                       : 1599790592
         Device/File Name         : /dev/raw/raw1
                                    Device/File integrity check succeeded
                                    Device/File not configured
         Cluster registry integrity check succeeded
3.2.3 ocrconfig
该命令用于维护OCR 磁盘，安装clusterware过程中，如果选择External 
Redundancy冗余方式，则只能输入一个OCR磁盘位置。 但是Oracle允许配置两个OCR 磁盘互为镜像，以防止OCR 
磁盘的单点故障。OCR 磁盘和Votedisk磁盘不一样，OCR磁盘最多只能有两个，一个Primary OCR 和一个Mirror OCR。
[root@raw1 bin]# ./ocrconfig --help
Name:
        ocrconfig - Configuration tool for Oracle Cluster Registry.
Synopsis:
        ocrconfig [option]
        option:
                -export <filename> [-s online]
                                                    - Export cluster register contents to a file
                -import <filename>                  - Import cluster registry contents from a file
                -upgrade [<user> [<group>]]
                                                    - Upgrade cluster registry from previous version
                -downgrade [-version <version string>]
                                                    - Downgrade cluster registry to the specified version
                -backuploc <dirname>                - Configure periodic backup location
                -showbackup                         - Show backup information
                -restore <filename>                 - Restore from physical backup
                -replace ocr|ocrmirror [<filename>] - Add/replace/remove a OCR device/file
                -overwrite                          - Overwrite OCR configuration on disk
                -repair ocr|ocrmirror <filename>    - Repair local OCR configuration
                -help                               - Print out this help information
Note:
        A log file will be created in
        $ORACLE_HOME/log/<hostname>/client/ocrconfig_<pid>.log. Please ensure
        you have file creation privileges in the above directory before
        running this tool.
-- 查看自助备份
[root@raw1 bin]# ./ocrconfig -showbackup
在缺省情况下，OCR自动备份在$CRS_HOME/CRS/CDATA/cluster_name 目录下，可以通过ocrconfig -backuploc <directory_name> 命令修改到新的目录
3.2.4 使用导出，导入进行备份和恢复
Oracle 推荐在对集群做调整时，比如增加，删除节点之前，应该对OCR做一个备份，可以使用export 
备份到指定文件，如果做了replace或者restore 等操作，Oracle 建议使用cluvfy comp ocr -n all 
命令来做一次全面的检查。该命令在clusterware 的安装软件里。
1） 首先关闭所有节点的CRS
[root@raw1 bin]# ./crsctl stop crs
Stopping resources.
Successfully stopped CRS resources
Stopping CSSD.
Shutting down CSS daemon.
Shutdown request successfully issued.
     2） 用root 用户导出OCR内容
[root@raw1 bin]# ./ocrconfig -export /u01/ocr.exp
     3） 重启CRS
[root@raw1 bin]# ./crsctl start crs
Attempting to start CRS stack
The CRS stack will be started shortly
     4） 检查CRS 状态
[root@raw1 bin]# ./crsctl check crs
CSS appears healthy
CRS appears healthy
EVM appears healthy
    5）破坏OCR内容
[root@raw1 bin]# dd if=/dev/zero of=/dev/raw/raw1 bs=1024 count=102400
102400+0 records in
102400+0 records out
6） 检查OCR一致性
[root@raw1 bin]# ./ocrcheck
PROT-601: Failed to initialize ocrcheck
    7）使用cluvfy 工具检查一致性
[root@raw1 cluvfy]# ./runcluvfy.sh comp ocr -n all
Verifying OCR integrity
Unable to retrieve nodelist from Oracle clusterware.
Verification cannot proceed.
8） 使用Import 恢复OCR 内容
[root@raw1 bin]# ./ocrconfig -import /u01/ocr.exp
9）再次检查OCR
[root@raw1 bin]# ./ocrcheck
Status of Oracle Cluster Registry is as follows :
         Version                  :          2
         Total space (kbytes)     :     147352
         Used space (kbytes)      :       4364
         Available space (kbytes) :     142988
         ID                       :  610419116
         Device/File Name         : /dev/raw/raw1
                                    Device/File integrity check succeeded
                                    Device/File not configured
         Cluster registry integrity check succeeded
10） 使用cluvfy工具检查
[root@raw1 cluvfy]# ./runcluvfy.sh comp ocr -n all
Verifying OCR integrity
WARNING:
These nodes cannot be reached:
        raw2 
Verification will proceed with nodes:
        raw1
ERROR:
User equivalence unavailable on all the nodes.
Verification cannot proceed.
Verification of OCR integrity was unsuccessful on all the nodes.
注：此处不成功是因为我的机器卡，故raw2节点没有启动
3.2.5 移动OCR 文件位置
实例演示将OCR从/dev/raw/raw1 移动到/dev/raw/raw3上。
1） 查看是否有OCR备份
[root@raw1 bin]# ./ocrconfig -showbackup
如果没有备份，可以立即执行一次导出作为备份：
[root@raw1 bin]# ./ocrconfig -export /u01/ocrbackup -s online
2） 查看当前OCR配置
[root@raw1 bin]# ./ocrcheck
Status of Oracle Cluster Registry is as follows :
         Version                  :          2
         Total space (kbytes)     :     147352
         Used space (kbytes)      :       4364
         Available space (kbytes) :     142988
         ID                       :  610419116
         Device/File Name         : /dev/raw/raw1
                                    Device/File integrity check succeeded
                                    Device/File not configured
         Cluster registry integrity check succeeded
输出显示当前只有一个Primary OCR，在/dev/raw/raw1。 没有Mirror OCR。 
因为现在只有一个OCR文件，所以不能直接改变这个OCR的位置，必须先添加镜像后在修改，否则会报：Failed to initialize 
ocrconfig.
3) 添加一个Mirror OCR
[root@raw1 bin]# ./ocrconfig -replace ocrmirror /dev/raw/raw4
4) 确认添加成功
[root@raw1 bin]# ./ocrcheck
5）改变primary OCR 位置
[root@raw1 bin]# ./ocrconfig -replace ocr /dev/raw/raw3
确认修改成功：
[root@raw1 bin]# ./ocrcheck
    6）使用ocrconfig命令修改后，所有RAC节点上的/etc/oracle/ocr.loc 文件内容也会自动同步了，如果没有自动同步，可以手工的改成以下内容。
[root@raw1 bin]# more /etc/oracle/ocr.loc
ocrconfig_loc=/dev/raw/raw1
Ocrmirrorconfig_loc=/dev/raw/raw3
local_only=FALSE
四． 应用层
应用层就是指RAC数据库了，这一层有若干资源组成，每个资源都是一个进程或者一组进程组成的完整服务，这一层的管理和维护都是围绕这些资源进行的。 有如下命令: srvctl, onsctl, crs_stat 三个命令。
4.1 crs_stat
Crs_stat 这个命令用于查看CRS维护的所有资源的运行状态，如果不带任何参数时，显示所有资源的概要信息。每个资源显示是各个属性：资源名称，类型，目录，资源运行状态等。
[root@raw1 bin]# ./crs_stat
NAME=ora.raw.db
TYPE=application
TARGET=ONLINE
STATE=OFFLINE
......
也可以指定资源名，查看指定资源的状态，并可以使用-V 和-P 选项，以查看详细信息，其中-p 参数显示的内容比-V 更详细。
1） 查看制定资源状态
[root@raw1 bin]# ./crs_stat ora.raw2.vip
NAME=ora.raw2.vip
TYPE=application
TARGET=ONLINE
STATE=OFFLINE
2） 使用-v 选项，查看详细内容，这时输出多出4项内容，分别是允许重启次数，已执行重启次数，失败阀值，失败次数。
[root@raw1 bin]# ./crs_stat -v ora.raw2.vip
NAME=ora.raw2.vip
TYPE=application
RESTART_ATTEMPTS=0
RESTART_COUNT=0
FAILURE_THRESHOLD=0
FAILURE_COUNT=0
TARGET=ONLINE
STATE=OFFLINE
3） 使用-p 选项查看更详细内容
[root@raw1 bin]# ./crs_stat -p ora.raw2.vip
NAME=ora.raw2.vip
TYPE=application
ACTION_SCRIPT=/u01/app/oracle/product/crs/bin/racgwrap
ACTIVE_PLACEMENT=1
AUTO_START=1
CHECK_INTERVAL=60
DESCRIPTION=CRS application for VIP on a node
FAILOVER_DELAY=0
FAILURE_INTERVAL=0
FAILURE_THRESHOLD=0
HOSTING_MEMBERS=raw2
OPTIONAL_RESOURCES=
PLACEMENT=favored
REQUIRED_RESOURCES=
RESTART_ATTEMPTS=0
SCRIPT_TIMEOUT=60
START_TIMEOUT=0
STOP_TIMEOUT=0
UPTIME_THRESHOLD=7d
USR_ORA_ALERT_NAME=
USR_ORA_CHECK_TIMEOUT=0
USR_ORA_CONNECT_STR=/ as sysdba
USR_ORA_DEBUG=0
USR_ORA_DISCONNECT=false
USR_ORA_FLAGS=
USR_ORA_IF=eth0
USR_ORA_INST_NOT_SHUTDOWN=
USR_ORA_LANG=
USR_ORA_NETMASK=255.255.255.0
USR_ORA_OPEN_MODE=
USR_ORA_OPI=false
USR_ORA_PFILE=
USR_ORA_PRECONNECT=none
USR_ORA_SRV=
USR_ORA_START_TIMEOUT=0
USR_ORA_STOP_MODE=immediate
USR_ORA_STOP_TIMEOUT=0
USR_ORA_VIP=10.85.10.123
这些字段是所有资源共有的，但是根据资源类型不同，某些字段可以空值。
4） 使用-ls 选项，可以查看每个资源的权限定义，权限定义格式和[Linux](http://www.2cto.com/os/linux/) 一样。
[root@raw1 bin]# ./crs_stat -ls
Name           Owner          Primary PrivGrp          Permission
-----------------------------------------------------------------
ora.raw.db     oracle         oinstall                 rwxrwxr--
ora.raw.dmm.cs oracle         oinstall                 rwxrwxr--
ora....aw2.srv   oracle         oinstall                 rwxrwxr--
ora....w1.inst   oracle         oinstall                 rwxrwxr--
ora....w2.inst    oracle         oinstall                 rwxrwxr--
ora....SM1.asm  oracle         oinstall                 rwxrwxr--
ora....W1.lsnr   oracle         oinstall                 rwxrwxr--
ora.raw1.gsd   oracle         oinstall                 rwxr-xr--
ora.raw1.ons   oracle         oinstall                 rwxr-xr--
ora.raw1.vip   root           oinstall                 rwxr-xr--
ora....SM2.asm  oracle         oinstall                 rwxrwxr--
ora....W2.lsnr   oracle         oinstall                 rwxrwxr--
ora.raw2.gsd   oracle         oinstall                 rwxr-xr--
ora.raw2.ons   oracle         oinstall                 rwxr-xr--
ora.raw2.vip   root           oinstall                 rwxr-xr--
4.2 onsctl
这个命令用于管理配置ONS(Oracle Notification Service). ONS 是Oracle Clusterware 实现FAN Event Push模型的基础。
在传统模型中，客户端需要定期检查服务器来判断服务端状态，本质上是一个pull模型，Oracle 10g 引入了一个全新的PUSH 
机制--FAN(Fast Application 
Notification),当服务端发生某些事件时，服务器会主动的通知客户端这种变化，这样客户端就能尽早得知服务端的变化。 
而引入这种机制就是依赖ONS实现， 在使用onsctl命令之前，需要先配置ONS服务。
4.2.1 ONS 配置内容
在RAC 环境中，需要使用$CRS_HOME下的ONS,而不是$ORACLE_HOME下面的ONS， 这点需要注意。 配置文件在$CRS_HOME/opmn/conf/ons.config.
[root@raw1 conf]# pwd
/u01/app/oracle/product/crs/opmn/conf
[root@raw1 conf]# more ons.config
localport=6100
remoteport=6200
loglevel=3
useocr=on
参数说明：
Localport: 这个参数代表本地监听端口，这里本地特指：127.0.0.1 这个回环地址，用来和运行在本地的客户端进行通信
Remoteport：这个参数代表的是远程监听端口，也就是除了127.0.0.1 以外的所有本地IP地址，用来和远程的客户端进行通信。
Loglevel: Oracle 允许跟踪ONS进程的运行，并把日志记录到本地文件中，这个参数用来定义ONS进程要记录的日志级别，从1-9，缺省值是3.
Logfile: 这个参数和loglevel参数一起使用，用于定义ONS进程日志文件的位置，缺省值是$CRS_HOME/opmn/logs/opmn.log
nodes和useocr: 这两个参数共同决定饿了本地的ONS daemon要和哪些远程节点上的ONS daemon进行通信。
Nodes 参数值格式如下：Hostname/IP:port[hostname/ip:port]
如：useoce=off
Nodes=rac1:6200,rac2:6200
而useocr 参数值为on/off, 如果useocr 是ON， 说明信息保存在OCR中，如果是OFF，说明信息取nodes中的配置。对于单实例而言，要把useocr设置为off。
4.2.2 配置ONS
可以直接编译ONS的配置文件来修改配置，如果使用了OCR，则可以通过racgons命令进行配置，但必须以root用户来执行，如果用oracle 用户来执行，不会提示任何错误，但也不会更改任何配置。
若要添加配置，可以使用下面命令：
Racgons add_config rac1:6200 rac2:6200
若要删除配置，可以用下面命令：
Racgons remove_config rac1:6200 rac2:6200
4.2.3 onsctl 命令
使用onsctl命令可以启动，停止，调试ONS，并重新载入配置文件，其命令格式如下：
[root@raw1 bin]# ./onsctl
usage: ./onsctl start|stop|ping|reconfig|debug
start                            - Start opmn only.
stop                             - Stop ons daemon
ping                             - Test to see if ons daemon is running
debug                            - Display debug information for the ons daemon
reconfig                         - Reload the ons configuration
help                             - Print a short syntax description (this).
detailed                         - Print a verbose syntax description.
  ONS 进程运行，并不一定代表ONS 正常工作，需要使用ping命令来确认。
1） 在OS级别查看进程状态。
[root@raw1 bin]# ps -aef|grep ons
root      1924  6953  0 03:17 pts/1    00:00:00 grep ons
oracle   30723     1  0 Mar08 ?        00:00:00 /u01/app/oracle/product/crs/opmn/bin/ons -d
oracle   30724 30723  0 Mar08 ?        00:00:04 /u01/app/oracle/product/crs/opmn/bin/ons -d
2） 确认ONS服务的状态
[root@raw1 bin]# ./onsctl ping
Number of onsconfiguration retrieved, numcfg = 2
onscfg[0]
   {node = raw1, port = 6200}
Adding remote host raw1:6200
onscfg[1]
   {node = raw2, port = 6200}
Adding remote host raw2:6200
ons is running ...
3） 启动ONS服务
[root@raw1 bin]# ./onsctl start
4） 使用debug 选项，可以查看详细信息，其中最有意义的就是能显示所有连接。
[root@raw1 bin]# ./onsctl debug
Number of onsconfiguration retrieved, numcfg = 2
onscfg[0]
   {node = raw1, port = 6200}
Adding remote host raw1:6200
onscfg[1]
   {node = raw2, port = 6200}
Adding remote host raw2:6200
HTTP/1.1 200 OK
Content-Length: 1357
Content-Type: text/html
Response:
======== ONS ========
Listeners:
 NAME    BIND ADDRESS   PORT   FLAGS   SOCKET
------- --------------- ----- -------- ------
Local   127.000.000.001  6100 00000142      7
Remote  010.085.010.119  6200 00000101      8
Request     No listener
Server connections:
    ID           IP        PORT    FLAGS    SENDQ     WORKER   BUSY  SUBS
---------- --------------- ----- -------- ---------- -------- ------ -----
         1 010.085.010.121  6200 00104205          0               1     0
Client connections:
    ID           IP        PORT    FLAGS    SENDQ     WORKER   BUSY  SUBS
---------- --------------- ----- -------- ---------- -------- ------ -----
         3 127.000.000.001  6100 0001001a          0               1     0
         4 127.000.000.001  6100 0001001a          0               1     1
Pending connections:
    ID           IP        PORT    FLAGS    SENDQ     WORKER   BUSY  SUBS
---------- --------------- ----- -------- ---------- -------- ------ -----
         0 127.000.000.001  6100 00020812          0               1     0
Worker Ticket: 3/3, Idle: 180
   THREAD   FLAGS
  -------- --------
   17faba0 00000012
   67f6ba0 00000012
   32d6ba0 00000012
Resources:
  Notifications:
    Received: 1, in Receive Q: 0, Processed: 1, in Process Q: 0
  Pools:
    Message: 24/25 (1), Link: 25/25 (1), Subscription: 24/25 (1)
[root@raw1 bin]#
4.3 srvctl
           该命令是RAC维护中最常用的命令，也是最复杂的命令。 
这个工具可以操作下面的几种资源：Database，Instance，ASM，Service，Listener 和Node 
Application，其中Node application又包括GSD，ONS，VIP。 
这些资源除了使用srvctl工具统一管理外，某些资源还有自己独立的管理工具，比如ONS可以使用onsctl命令进行管理；Listener 
可以通过lsnrctl 管理。
[root@raw1 bin]# ./srvctl --help
Usage: srvctl <command> <object> [<options>]
    command: enable|disable|start|stop|relocate|status|add|remove|modify|getenv|setenv|unsetenv|config
    objects: database|instance|service|nodeapps|asm|listener
For detailed help on each command and object and its options use:
    srvctl <command> <object> -h
4.3.1 使用config查看配置
1）查看[数据库](http://www.2cto.com/database/)配置
-- 不带任何参数时，显示OCR中注册的所有数据库
[root@raw1 bin]# ./srvctl config database
raw
-- 使用-d 选项，查看某个数据库配置
[root@raw1 bin]# ./srvctl config database -d raw
raw1 raw1 /u01/app/oracle/product/10.2.0/db_1
raw2 raw2 /u01/app/oracle/product/10.2.0/db_1
注： 该输出结果显示数据库raw由2个节点组成，各自实例名交raw1和raw2. 两个实例的$ORACLE_HOME是/u01/app/oracle/product/10.2.0/db_1
-- 使用-a 选项查看配置的详细信息
[root@raw1 bin]# ./srvctl config database -d raw
raw1 raw1 /u01/app/oracle/product/10.2.0/db_1
raw2 raw2 /u01/app/oracle/product/10.2.0/db_1
[root@raw1 bin]# ./srvctl config database -d raw -a
raw1 raw1 /u01/app/oracle/product/10.2.0/db_1
raw2 raw2 /u01/app/oracle/product/10.2.0/db_1
DB_NAME: raw
ORACLE_HOME: /u01/app/oracle/product/10.2.0/db_1
SPFILE: +DATA/raw/spfileraw.ora
DOMAIN: null
DB_ROLE: null
START_OPTIONS: null
POLICY:  AUTOMATIC
ENABLE FLAG: DB ENABLED
2）查看Node Application的配置
-- 不带任何参数，返回节点名，实例名和$ORACLE_HOME
[root@raw1 bin]# ./srvctl config nodeapps -n raw1
raw1 raw1 /u01/app/oracle/product/10.2.0/db_1
               -- 使用-a 选项，查看VIP 配置
[root@raw1 bin]# ./srvctl config nodeapps -n raw1 -a
VIP exists.: /raw1-vip/10.85.10.122/255.255.255.0/eth0
               -- 使用-g 选项， 查看GSD：
[root@raw1 bin]# ./srvctl config nodeapps -n raw1 -g
GSD exists.
-- 使用-s 选项，查看ONS:
[root@raw1 bin]# ./srvctl config nodeapps -n raw1 -s
ONS daemon exists.
-- 使用-l 选项，查看Listener:
[root@raw1 bin]# ./srvctl config nodeapps -n raw1 -l
Listener exists.
3) 查看Listener.
[root@raw1 bin]# ./srvctl config listener -n raw1
raw1 LISTENER_RAW1
[root@raw1 bin]# ./srvctl config listener -n raw2
raw2 LISTENER_RAW2
4) 查看ASM
[root@raw1 bin]# ./srvctl config asm -n raw1
+ASM1 /u01/app/oracle/product/10.2.0/db_1
[root@raw1 bin]# ./srvctl config asm -n raw2
+ASM2 /u01/app/oracle/product/10.2.0/db_1
5） 查看Service
-- 查看数据库所有service配置
[root@raw1 bin]# ./srvctl config service -d raw -a
dmm PREF: raw2 AVAIL: raw1 TAF: basic
-- 查看某个Service 配置
[root@raw1 bin]# ./srvctl config service -d raw -s dmm
dmm PREF: raw2 AVAIL: raw1
-- 使用-a 选项，查看TAF 策略
[root@raw1 bin]# ./srvctl config service -d raw -s dmm -a
dmm PREF: raw2 AVAIL: raw1 TAF: basic
4.3.2 使用add 添加对象
一般情况下，应用层资源都是在图形界面的帮助下注册到OCR中的，比如VIP，ONS实在安装最后阶段创建的，而数据库，ASM是执行DBCA的过程中自
动注册到OCR中的，Listener是通过netca工具。 但是有些时候需要手工把资源注册到OCR中。 这时候就需要add 命令了。
1） 添加数据库
[root@raw1 bin]# ./srvctl add database -d dmm -o $ORACLE_HOME
2)  添加实例
[root@raw1 bin]# ./srvctl add instance -d dmm -n rac1 -i dmm1
[root@raw1 bin]# ./srvctl add instance -d dmm -n rac2 -i dmm2
3) 添加服务，需要使用4个参数
-s : 服务名
-r：首选实例名
-a：备选实例名
-P：TAF策略，可选值为None（缺省值），Basic，preconnect。
[root@raw1 bin]# ./srvctl add service -d dmm -s dmmservice -r rac1 -a rac2 -P BASIC
4) 确认添加成功
[root@raw1 bin]# ./srvctl config service -d dmm -s dmmservice -a
4.3.3 使用enable/disable 启动，禁用对象
缺省情况下数据库，实例，服务，ASM都是随着CRS的启动而自启动的，有时候由于维护的需要，可以先关闭这个特性。
1） 配置数据库随CRS的启动而自动启动
-- 启用数据库的自启动：
[root@raw1 bin]# ./srvctl enable database -d raw
--查看配置
[root@raw1 bin]# ./srvctl config database -d raw -a
raw1 raw1 /u01/app/oracle/product/10.2.0/db_1
raw2 raw2 /u01/app/oracle/product/10.2.0/db_1
DB_NAME: raw
ORACLE_HOME: /u01/app/oracle/product/10.2.0/db_1
SPFILE: +DATA/raw/spfileraw.ora
DOMAIN: null
DB_ROLE: null
START_OPTIONS: null
POLICY:  AUTOMATIC
ENABLE FLAG: DB ENABLED
-- 禁止数据库在CRS启动后自启动，这时需要手动启动
[root@raw1 bin]# ./srvctl disable database -d raw
2） 关闭某个实例的自动启动
[root@raw1 bin]# ./srvctl disable instance -d raw -i raw1
[root@raw1 bin]# ./srvctl enable instance -d raw -i raw1
-- 查看信息
[root@raw1 bin]# ./srvctl config database -d raw -a
raw1 raw1 /u01/app/oracle/product/10.2.0/db_1
raw2 raw2 /u01/app/oracle/product/10.2.0/db_1
DB_NAME: raw
ORACLE_HOME: /u01/app/oracle/product/10.2.0/db_1
SPFILE: +DATA/raw/spfileraw.ora
DOMAIN: null
DB_ROLE: null
START_OPTIONS: null
POLICY:  AUTOMATIC
ENABLE FLAG: DB ENABLED
3) 禁止某个服务在实例上运行
[root@raw1 bin]# ./srvctl enable service -d raw -s rawservice -i raw1
[root@raw1 bin]# ./srvctl disable service -d raw -s rawservice -i raw1
-- 查看
[root@raw1 bin]# ./srvctl config service -d raw -a
dmm PREF: raw2 AVAIL: raw1 TAF: basic
4.3.4 使用remove 删除对象
使用remove命令删除的是对象在OCR中的定义信息，对象本省比如数据库的数据文件等不会被删除，以后随时可以使用add命令重新添加到OCR中。
1) 删除Service，在删除之前，命令会给出确定提示
[root@raw1 bin]# ./srvctl remove service -d raw -s rawservice
2）删除实例，删除之前同样会给出提示
[root@raw1 bin]# ./srvctl remove instance -d raw -i raw1
3）删除数据库
[root@raw1 bin]# ./srvctl remove database -d raw
4.3.5 启动，停止对象与查看对象
在RAC 环境下启动，关闭数据库虽然仍然可以使用SQL/PLUS方法，但是更推荐使用srvctl命令来做这些工作，这可以保证即使更新CRS中运行信息，可以使用start/stop 命令启动，停止对象，然后使用status 命令查看对象状态。
1） 启动数据库，默认启动到open状态
[root@raw1 bin]# ./srvctl start database -d raw
2） 指定启动状态
[root@raw1 bin]# ./srvctl start database -d raw -i raw1 -o mount
[root@raw1 bin]# ./srvctl start database -d raw -i raw1 -o nomount
3） 关闭对象，并指定关闭方式
[root@raw1 bin]# ./srvctl stop instance -d raw -i raw1 -o immediate
[root@raw1 bin]# ./srvctl stop instance -d raw -i raw1 -o abort
4)  在指定实例上启动服务：
[root@raw1 bin]# ./srvctl start service -d raw -s rawservice -i raw1
-- 查看服务状态
[root@raw1 bin]# ./srvctl status service -d raw -v
5） 关闭指定实例上的服务
[root@raw1 bin]# ./srvctl stop service -d raw -s rawservice -i raw1
-- 查看服务状态
[root@raw1 bin]# ./srvctl status service -d raw -v
4.3.6 跟踪srvctl
在[Oracle](http://www.2cto.com/database/Oracle/) 10g中要跟踪srvctl 非常简单，只要设置srvm_trace=true 这个OS环境变量即可，这个命令的所有函数调用都会输出到屏幕上，可以帮助用户进行诊断。
[root@raw1 bin]# export SRVM_TRACE=TRUE
[root@raw1 bin]# ./srvctl config database -d raw
/u01/app/oracle/product/crs/jdk/jre/bin/java -classpath 
/u01/app/oracle/product/crs/jlib/netcfg.jar:/u01/app/oracle/product/crs/jdk/jre/lib/rt.jar:/u01/app/oracle/product/crs/jdk/jre/lib/i18n.jar:/u01/app/oracle/product/crs/jlib/srvm.jar:/u01/app/oracle/product/crs/jlib/srvmhas.jar:/u01/app/oracle/product/crs/jlib/srvmasm.jar:/u01/app/oracle/product/crs/srvm/jlib/srvctl.jar
 -DTRACING.ENABLED=true -DTRACING.LEVEL=2 oracle.ops.opsctl.OPSCTLDriver
 config database -d raw
[main] [6:58:44:858] [OPSCTLDriver.setInternalDebugLevel:165]  tracing is true at level 2 to file null
[main] [6:58:44:911] [OPSCTLDriver.<init>:95]  Security manager is set
[main] [6:58:44:955] [CommandLineParser.parse:173]  parsing cmdline args
[main] [6:58:44:959] [CommandLineParser.parse2WordCommandOptions:940]  parsing 2-word cmdline
[main] [6:58:44:961] [OPSCTLDriver.execute:174]  executing srvctl command
[main] [6:58:44:963] [OPSCTLDriver.execute:199]  executing 2-word command verb=10 noun=101
[main] [6:58:44:995] [Action.getOPSConfig:162]  get db config for: raw
[main] [6:58:45:2] [CommandLineParser.obtainOPSConfig:1410]  srvctl: get db config for: raw
[main] [6:58:45:47] [GetActiveNodes.create:213]  Going into GetActiveNodes constructor...
... ...
4.4 恢复
假设OCR磁盘和Votedisk磁盘全部破坏，并且都没有备份，该如何恢复， 这时最简单的方法就是重新初始话OCR和Votedisk， 具体操作如下：
4.4.1 停止所有节点的Clusterware Stack
Crsctl stop crs;
4.4.2 分别在每个节点用root用户执行$CRS_HOME/install/rootdelete.sh脚本
4.4.3 在任意一个节点上用root用户执行$CRS_HOME/install/rootinstall.sh 脚本
4.4.4 在和上一步同一个节点上用root执行$CRS_HOME/root.sh脚本
4.4.5 在其他节点用root执行行$CRS_HOME/root.sh脚本
4.4.6 用netca 命令重新配置监听，确认注册到Clusterware中
#crs_stat -t -v
到目前为止，只有Listener，ONS,GSD,VIP 注册到OCR中，还需要把ASM， 数据库都注册到OCR中。
4.4.7  向OCR中添加ASM
 #srvctl add asm -n rac1 -i +ASM1 -o /u01/app/product/database
 #srvctl add asm -n rac2 -i +ASM2 -o /u01/app/product/database
4.4.8 启动ASM
#srvctl start asm -n rac1
#srvctl start asm -n rac2
若在启动时报ORA-27550错误。是因为RAC无法确定使用哪个网卡作为Private Interconnect，解决方法：在两个ASM的pfile文件里添加如下参数：
+ASM1.cluster_interconnects='10.85.10.119'
+ASM2.cluster_interconnects='10.85.10.121'
4.4.9 手工向OCR中添加Database对象。
#srvctl add database -d raw -o /u01/app/product/database
4.4.10 添加2个实例对象
#srvctl add instance -d raw -i raw1 -n raw1
#srvctl add instance -d raw -i raw2 -n raw2
4.4.11 修改实例和ASM实例的依赖关系
#srvctl modify instance -d raw -i raw1 -s +ASM1
#srvctl modify instance -d raw -i raw2 -s +ASM2
4.4.12 启动数据库
#srvctl start database-d raw
若也出现ORA-27550错误。也是因为RAC无法确定使用哪个网卡作为Private Interconnect，修改pfile参数在重启动即可解决。
SQL>alter system set cluster_interconnects='10.85.10.119' scope=spfile sid='raw1';
SQL>alter system set cluster_interconnects='10.85.10.121' scope=spfile sid='raw2';
Srvctl 命令的用法还有很多，下面是在线文档的一个目录，感兴趣的可以自己研究下。
 
 http://download-west.oracle.com/docs/cd/B19306_01/rac.102/b14197/toc.htm
 
 srvctl add
 srvctl add database
 srvctl add instance
 srvctl add service
 srvctl add nodeapps
 srvctl add asm
 
 srvctl config
 srvctl config database
 srvctl config service
 srvctl config nodeapps
 srvctl config asm
 srvctl config listener
 
 srvctl enable
 srvctl enable database
 srvctl enable instance
 srvctl enable service
 srvctl enable asm
 
 srvctl disable
 srvctl disable database
 srvctl disable instance
 srvctl disable service
 srvctl disable asm
 
 srvctl start
 srvctl start database
 srvctl start instance
 srvctl start service
 srvctl start nodeapps
 srvctl start asm
 srvctl start listener
 
 srvctl stop
 srvctl stop database
 srvctl stop instance
 srvctl stop service
 srvctl stop nodeapps
 srvctl stop asm
 srvctl stop listener
 
 srvctl modify
 srvctl modify database
 srvctl modify instance
 srvctl modify service
 srvctl modify nodeapps
 
 srvctl relocate
 srvctl relocate service
 
 srvctl status
 srvctl status database
 srvctl status instance
 srvctl status service
 srvctl status nodeapps
 srvctl status asm
 
 srvctl getenv
 srvctl getenv database
 srvctl getenv instance
 srvctl getenv service
 srvctl getenv nodeapps
 
 srvctl setenv and unsetenv
 srvctl setenv database
 srvctl setenv instance
 srvctl setenv service
 srvctl setenv nodeapps
 srvctl unsetenv database
 srvctl unsetenv instance
 srvctl unsetenv service
 srvctl unsetenv nodeapps
 
 srvctl remove
 srvctl remove database
 srvctl remove instance
 srvctl remove service
 srvctl remove nodeapps
 srvctl remove asm
 
注：本文整理自<大话Oracle RAC>
