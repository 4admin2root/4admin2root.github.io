---
layout:     post
title:      windows764位下安装python-2.7.5.amd64.msi时installer报错2203
date:       2013-10-08
---
The installer has encountered an unexpected error installing this package.

this may indicate a problem with package.the error code is 2203.

实际上 该package 并没有问题，而是权限问题。

参考解决方法就是在用户目录 如C:\Users\admin\Local Settings

取得访问权限

参考

[http://blog.csdn.net/greystar/article/details/4718235](http://blog.csdn.net/greystar/article/details/4718235)

[http://blogs.msdn.com/b/astebner/archive/2006/06/09/624546.aspx](http://blogs.msdn.com/b/astebner/archive/2006/06/09/624546.aspx)
