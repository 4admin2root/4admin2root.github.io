---
layout:     post
title:      转：Linuxsmaps接口文件结构
date:       2012-09-21
---
原文地址[http://qianjigui.iteye.com/blog/1479109](http://qianjigui.iteye.com/blog/1479109)

### [Linux smaps接口文件结构](http://qianjigui.iteye.com/blog/1479109)

- [Linux 零散小知识](http://qianjigui.iteye.com/category/34086)

在Linux内核 2.6.16中引入了一个系统内存接口特性，这个接口位于/proc/$pid/目录下的smaps文件中 ，一看内容发现是进程内存映像信息，比同一目录下的maps文件更详细些。

 

 

1. 400df000-4048c000 r--s 00000000 1f:05 286        /data/dalvik-cache/system@framework@core.jar@classes.dex  
1. Size:               3764 kB  
1. Rss:                1804 kB  
1. Pss:                1804 kB  
1. Shared_Clean:          0 kB  
1. Shared_Dirty:          0 kB  
1. Private_Clean:      1804 kB  
1. Private_Dirty:         0 kB  
1. Referenced:         1804 kB  
1. Anonymous:             0 kB  
1. Swap:                  0 kB  
1. KernelPageSize:        4 kB  
1. MMUPageSize:           4 kB  

 

 以上述输出结果为例：400df000-4048c000 r--s 00000000 1f:05 286 /data/dalvik-cache/system@framework@core.jar@classes.dex





<li>
400df000-4048c000 是该虚拟内存段的开始和结束位置



</li>
<li>
r--s内存段的权限，最后一位p代表私有，s代表共享



</li>
<li>
00000000 该虚拟内存段在对应的映射文件中的偏移量



</li>
<li>
1f:05 文件的主设备和次设备号



</li>
<li>
286 被映射到虚拟内存的文件的索引节点号



</li>
<li>
/data/dalvik-cache/system@framework@core.jar@classes.dex 被映射到虚拟内存的文件名称。后面带(deleted)的是内存数据，可以被销毁。



</li>
<li>
size 是进程使用内存空间，并不一定实际分配了内存(VSS)



</li>
<li>
Rss是实际分配的内存(不需要缺页中断就可以使用的)



</li>
<li>
Pss是平摊计算后的使用内存(有些内存会和其他进程共享，例如mmap进来的)



</li>
<li>
Shared_Clean 和其他进程共享的未改写页面



</li>
<li>
Shared_Dirty 和其他进程共享的已改写页面



</li>
<li>
Private_Clean 未改写的私有页面页面



</li>
<li>
Private_Dirty 已改写的私有页面页面



</li>
<li>
Referenced 标记为访问和使用的内存大小



</li>
<li>
Anonymous 不来自于文件的内存大小



</li>
<li>
Swap 存在于交换分区的数据大小(如果物理内存有限，可能存在一部分在主存一部分在交换分区)



</li>
<li>
KernelPageSize 内核页大小 



</li>
<li>
MMUPageSize    MMU页大小，基本和Kernel页大小相同



</li>



其中Dirty页面如果没有交换机制的情况下，应该是不能回收的。
精确分析内存占用可以用Private内存信息来衡量。





 

详细解释见 http://www.kernel.org/doc/Documentation/filesystems/proc.txt

```
The first of these lines shows the same information as is displayed for the
mapping in /proc/PID/maps.  The remaining lines show the size of the mapping
(size), the amount of the mapping that is currently resident in RAM (RSS), the
process' proportional share of this mapping (PSS), the number of clean and
dirty private pages in the mapping.  Note that even a page which is part of a
MAP_SHARED mapping, but has only a single pte mapped, i.e.  is currently used
by only one process, is accounted as private and not as shared.  "Referenced"
indicates the amount of memory currently marked as referenced or accessed.
"Anonymous" shows the amount of memory that does not belong to any file.  Even
a mapping associated with a file may contain anonymous pages: when MAP_PRIVATE
and a page is modified, the file page is replaced by a private anonymous copy.
"Swap" shows how much would-be-anonymous memory is also used, but out on
swap.
```
