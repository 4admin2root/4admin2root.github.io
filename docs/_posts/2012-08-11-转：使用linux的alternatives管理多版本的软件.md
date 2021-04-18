---
layout:     post
title:      转：使用linux的alternatives管理多版本的软件
date:       2012-08-11
tags: [cnblogs]
---
# [原]使用linux的alternatives管理多版本的软件

Admin2010年8月23日

# [原]使用linux的alternatives管理多版本的软件

今天捣鼓Oracle的OS Watcher(简称osw) 的时候发现机器上的jdk不太好使，不能运行osw的oswg.jar。比较郁闷，看了一下 /usr/bin/java 是连接到 /etc/alternatives/java ，而 /etc/alternatives/java 是连接到 /usr/lib/jvm/jre-1.4.2-gcj/bin/java ，

```
[root@test03 bin]# ll /usr/bin/java

lrwxrwxrwx  1 root root 22 Dec 17  2009 /usr/bin/java -> /etc/alternatives/java

[root@test03 bin]# ll /etc/alternatives/java

lrwxrwxrwx  1 root root 35 Aug 23 14:52 /etc/alternatives/java -> /usr/lib/jvm/jre-1.4.2-gcj/bin/java
```

发现这个 alternatives 很眼生，于是 google 了一下，发现它是一个管理多版本软件的软件，于是借着升级jdk、jre的契机顺便捣鼓一下这个alternatives 。

首先到Oracle的网站下载最新的 jdk 和 jre ，然后安装。这个安装比较恶心，装在什么地方也不说一声，害我找了半天才发现装在 /usr/java/jdk1.6.0_21/ 和 /usr/java/jre1.6.0_21/ 这两个目录中。

将jdk和jre的java注册到alternatives中，顺便也将jdk的javac注册到alternatives中。

```
[root@test03 ~]# alternatives --install /usr/bin/java  java  /usr/java/jre1.6.0_21/bin/java  400

[root@test03 ~]# alternatives --install /usr/bin/java  java  /usr/java/jdk1.6.0_21/bin/java  400

[root@test03 ~]# alternatives --install /usr/bin/javac javac /usr/java/jdk1.6.0_21/bin/javac 400
```

现在可以看看注册的成果了：

```
[root@test03 ~]# alternatives --config java



There are 3 programs which provide "java".



  Selection    Command

-----------------------------------------------

*+ 1           /usr/lib/jvm/jre-1.4.2-gcj/bin/java

   2           /usr/java/jre1.6.0_21/bin/java

   3           /usr/java/jdk1.6.0_21/bin/java



Enter to keep the current selection[+], or type selection number: 
```

这里输入想要用的 java 就可以了，例如我选在了第2个。我们看看 /usr/bin/java的变化：

```
[root@test03 ~]# ll /usr/bin/java

lrwxrwxrwx  1 root root 22 Dec 17  2009 /usr/bin/java -> /etc/alternatives/java

[root@test03 ~]# ll /etc/alternatives/java

lrwxrwxrwx  1 root root 30 Aug 23 15:03 /etc/alternatives/java -> /usr/java/jre1.6.0_21/bin/java
```

可以看到 /usr/bin/java 的连接的地方没有变，改变了的/etc/alternatives/java 的连接，这其实是一个策略模式的实现：

<img src="http://pic.cnthub.com/NewsPic//M0/S437/437324CB-0.jpg" alt="" /> 

/usr/bin/java 的调用没有变，还是连接到 /etc/alternatives/java，/etc/alternatives/java的连接却被修改了，这个由 alternatives 管理。

通过linux的连接也可以简单地实现这种接口和具体实现的分离，但是 alternatives 提供一个配置清单，简单选一下就OK了，这为我们提供了很大的便利。

扩充一下，alternatives也可以管理我们自己的软件。例如，我自己写了个软件叫myjava，我也想实现这种基于策略模式的版本管理，我可以这样做：

```
alternatives --install /usr/bin/myjava  myjava  /usr/java/jdk1.6.0_21/bin/java  300

alternatives --install /usr/bin/myjava  myjava  /usr/java/jre1.6.0_21/bin/java  300
```

使用 alternatives  更换一下我的版本：

```
[root@test03 ~]# ll /usr/bin/myjava

lrwxrwxrwx  1 root root 24 Aug 23 14:43 /usr/bin/myjava -> /etc/alternatives/myjava

[root@test03 ~]# ll /etc/alternatives/myjava 

lrwxrwxrwx  1 root root 30 Aug 23 14:46 /etc/alternatives/myjava -> /usr/java/jre1.6.0_21/bin/java

[root@test03 ~]# 

[root@test03 ~]# 

[root@test03 ~]# alternatives  --config myjava



There are 2 programs which provide "myjava".



  Selection    Command

-----------------------------------------------

*  1           /usr/java/jdk1.6.0_21/bin/java

 + 2           /usr/java/jre1.6.0_21/bin/java



Enter to keep the current selection[+], or type selection number: 1

[root@test03 ~]# 

[root@test03 ~]# 

[root@test03 ~]# ll /usr/bin/myjava

lrwxrwxrwx  1 root root 24 Aug 23 14:43 /usr/bin/myjava -> /etc/alternatives/myjava

[root@test03 ~]# ll /etc/alternatives/myjava 

lrwxrwxrwx  1 root root 30 Aug 23 15:15 /etc/alternatives/myjava -> /usr/java/jdk1.6.0_21/bin/java
```
