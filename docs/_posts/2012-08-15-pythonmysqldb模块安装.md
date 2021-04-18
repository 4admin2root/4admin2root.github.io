---
layout:     post
title:      pythonmysqldb模块安装
date:       2012-08-15
---
总结下这个安装过程网上搜索了半天处处是坑啊既爱又恨linux下实在是让人不爽啊

操作系统：redhat Red Hat Enterprise Linux Server release 5.3 (Tikanga)

系统默认python：Python 2.4.3

现在需要安装mysqldb模块到python

1、下载setuptools-0.6c8.tar.gzMySQL-devel-5.5.27-1.rhel5.x86_64.rpmMySQL-shared-5.5.27-1.rhel5.x86_64.rpmMySQL-python-1.2.3.tar.gz

2、安装setuptoolstar -zxvf setuptools-0.6c8.tar.gzcd setuptools-0.6c8python setup.py buildpython setup.py install

3、安装MySQL-develrpm -ivh MySQL-devel-5.5.27-1.rhel5.x86_64.rpmrpm -ivh MySQL-shared-5.5.27-1.rhel5.x86_64.rpm

4、安装MySQL-pythontar -zxvf MySQL-python-1.2.3.tar.gzcd MySQL-python-1.2.3

以上为本人亲自测试，不报错版

 
