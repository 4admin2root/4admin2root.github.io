---
layout:     post
title:      linux磁盘报错信息（盘阵、rdac等相关）
date:       2012-09-21
---
linux message 里面报信息如下（时间、主机名略）：kernel: sdd: Current: sense key: Recovered Errorkernel:     <<vendor>> ASC=0xe0 ASCQ=0x6ASC=0xe0 ASCQ=0x6kernel:kernel: sde: Current: sense key: Recovered Errorkernel:     <<vendor>> ASC=0xe0 ASCQ=0x6ASC=0xe0 ASCQ=0x6kernel:kernel: sde: Current: sense key: Recovered Errorkernel:     <<vendor>> ASC=0xe0 ASCQ=0x6ASC=0xe0 ASCQ=0x6kernel:kernel: 506 [RAIDarray.mpp]Device T0L0Ctrl1Path0 is not in running state. sdev_state 7.kernel: 506 [RAIDarray.mpp]Device T0L0Ctrl1Path1 is not in running state. sdev_state 7.kernel: sdc: Current: sense key: Recovered Errorkernel:     <<vendor>> ASC=0xe0 ASCQ=0x6ASC=0xe0 ASCQ=0x6kernel:kernel: sdd: Current: sense key: Recovered Errorkernel:     <<vendor>> ASC=0xe0 ASCQ=0x6ASC=0xe0 ASCQ=0x6kernel:kernel: sdd: Current: sense key: Recovered Errorkernel:     <<vendor>> ASC=0xe0 ASCQ=0x6ASC=0xe0 ASCQ=0x6kernel:kernel: sdd: Current: sense key: Recovered Errorkernel:     <<vendor>> ASC=0xe0 ASCQ=0x6ASC=0xe0 ASCQ=0x6kernel:snmpd[6581]: Connection from UDP: [10.128.254.61]:3078kernel: sdc: Current: sense key: Recovered Errorkernel:     <<vendor>> ASC=0xe0 ASCQ=0x6ASC=0xe0 ASCQ=0x6kernel:

出现报错信息后，磁盘使用正常，

oracle 数据库rac环境 使用asm 管理这些盘的使用

现在几天过去了，系统使用一直正常，报错原因未知
