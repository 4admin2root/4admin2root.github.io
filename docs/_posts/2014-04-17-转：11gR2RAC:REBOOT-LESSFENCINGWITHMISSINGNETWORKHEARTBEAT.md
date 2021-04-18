---
layout:     post
title:      转：11gR2RAC:REBOOT-LESSFENCINGWITHMISSINGNETWORKHEARTBEAT
date:       2014-04-17
tags: [cnblogs]
---
原文：http://oracleinaction.com/reboot-less-fencing-missing-nhb/

In my[ earlier post](http://oracleinaction.com/11g-r2-rac-reboot-less-node-fencing/), I had discussed about reboot-less node fencing , a new feature introduced since 11.2.0.2. In this post, I will demonstrate reboot-less node fencing when network heartbeat is lost.

** Check that clusterware version is 11.2.0.3**

```
[root@host02 ~]# crsctl query crs activeversion
Oracle Clusterware active version on the cluster is [**11.2.0.3.0**]
```

** check that both the nodes in the cluster are active**

```
[root@host02 ~]# olsnodes -s
host01  Active
host02  Active
```

** Find out pvt interconnect**

```
[root@host02 ~]# oifcfg getif
eth0  192.9.201.0  global  public
**eth1  10.0.0.0  global  cluster_interconnect**
```

** Stop pvt interconnect on node2**

```
[root@host02 ~]# ifdown eth1
```

** Alert log of node2 **

** Note that instead of rebooting the node, CRSD resources are cleaned up**

```
[cssd(802)]CRS-1612:Network communication with node host01 (1) missing for 50% of timeout interval.
..
Removal of this node from cluster in 6.640 seconds
2013-10-09 10:45:53.924
..
[cssd(802)]CRS-1609:This node is unable to communicate with other nodes in the cluster and is going  down to preserve cluster integrity;

[cssd(802)]CRS-1656:The CSS daemon is terminating due to a fatal error; ..
[cssd(802)]CRS-1652:**Starting clean up of CRSD resources.**
2013-10-09 10:46:01.918
...

[cssd(802)]CRS-1654:**Clean up of CRSD resources finished successfully.**
2013-10-09 10:46:03.794
```

** Check that OHAS service  is still up on host02**

```
[root@host02 ~]# crsctl check crs

CRS-4638:** Oracle High Availability Services is online**
CRS-4535: Cannot communicate with Cluster Ready Services
CRS-4530: Communications failure contacting Cluster Synchronization Services daemon
CRS-4534: Cannot communicate with Event Manager
```

** Check that resources cssd , crsd and HAIP are down on host02**[

```
[root@host02 ~]# crsctl stat res -t -init

NAME           TARGET  STATE        SERVER                   STATE_DETAILS

Cluster Resources

ora.asm
1        ONLINE  OFFLINE
**ora.cluster_interconnect.haip**
**      1        ONLINE  OFFLINE              **
ora.crf
1        ONLINE  ONLINE       host02
**ora.crsd**
**      1        ONLINE  OFFLINE      **
**ora.cssd**
**      1        ONLINE  OFFLINE **                              STARTING
ora.cssdmonitor
1        ONLINE  ONLINE       host02
ora.ctssd
1        ONLINE  OFFLINE
ora.diskmon
1        OFFLINE OFFLINE
ora.drivers.acfs
1        ONLINE  ONLINE       host02
ora.evmd
1        ONLINE  INTERMEDIATE host02
ora.gipcd
1        ONLINE  ONLINE       host02
ora.gpnpd
1        ONLINE  ONLINE       host02
ora.mdnsd
1        ONLINE  ONLINE       host02
```

**-- Restart private interconnect on host02**

```
[root@host02 ~]# ifup eth1
```

**- Alert log of host02**

**-- Note that as soon as private network is started, CSSD, CRSD and EVMD services start immediately and host02 joins the cluster**

```
[cssd(2876)]CRS-1713:**CSSD daemon is started in clustered mode**
2013-10-09 10:47:04.944
...
[cssd(2876)]CRS-1601:**CSSD Reconfiguration complete. Active nodes are host01 host02 .**
2013-10-09 10:55:22.403
..
[crsd(3973)]CRS-1012:The OCR service started on node host02.
2013-10-09 10:56:25.304
...
[evmd(2753)]CRS-1401:**EVMD started on node host02**.
2013-10-09 10:56:41.996
...
[crsd(3973)]CRS-1201:**CRSD started on node host02.**
2013-10-09 10:56:45.274
```

**-- check that resources haip, cssd and crsd have started on host02**

```
[root@host02 ~]# crsctl stat res -t -init
--------------------------------------------------------------------------------
NAME           TARGET  STATE        SERVER                   STATE_DETAILS
--------------------------------------------------------------------------------
Cluster Resources
--------------------------------------------------------------------------------
ora.asm
1        ONLINE  ONLINE       host02                   Started
**ora.cluster_interconnect.haip**
**      1        ONLINE  ONLINE       host02  **
ora.crf
1        ONLINE  ONLINE       host02
**ora.crsd**
**      1        ONLINE  ONLINE       host02                                       **
**ora.cssd**
**      1        ONLINE  ONLINE       host02        **
ora.cssdmonitor
1        ONLINE  ONLINE       host02
ora.ctssd
1        ONLINE  ONLINE       host02                   OBSERVER
ora.diskmon
1        OFFLINE OFFLINE
ora.drivers.acfs
1        ONLINE  ONLINE       host02
ora.evmd
1        ONLINE  ONLINE       host02
ora.gipcd
1        ONLINE  ONLINE       host02
ora.gpnpd
1        ONLINE  ONLINE       host02
ora.mdnsd
1        ONLINE  ONLINE       host02
```

**-- Check that host02 has joined the cluster**

```
[root@host02 ~]# olsnodes -s
host01  Active
host02  Active
```

**References:**

[http://ora-ssn.blogspot.in/2011/09/reboot-less-node-fencing-in-oracle.html](http://ora-ssn.blogspot.in/2011/09/reboot-less-node-fencing-in-oracle.html)[http://www.trivadis.com/uploads/tx_cabagdownloadarea/Trivadis_oracle_clusterware_node_fencing_v.pdf](http://www.trivadis.com/uploads/tx_cabagdownloadarea/Trivadis_oracle_clusterware_node_fencing_v.pdf)[http://www.vmcd.org/2012/03/11gr2-rac-rebootless-node-fencing/](http://www.vmcd.org/2012/03/11gr2-rac-rebootless-node-fencing/)

---------------------------------------------------------------------------------------------

**Related Links:**

**[Home](http://oracleinaction.com/)**

**[11g R2 RAC Index](http://oracleinaction.com/11gr2rac/)**

**[11g R2 RAC: Node Eviction Due To Missing Network Heartbeat ](http://oracleinaction.com/eviction-netwk-heartbeat/)**[**11g R2 RAC :Reboot-less Node Fencing**](http://oracleinaction.com/11g-r2-rac-reboot-less-node-fencing/)[**11g R2 RAC :Reboot-less  Fencing With Missing Disk Heartbeat**](http://oracleinaction.com/reboot-less-fencing-missing-dhb/)
