---
layout:     post
title:      安装puppetopensource
date:       2012-08-11
tags: [cnblogs]
---
OS : REDHAT 5.3 X64

[root@testdb1 el-5-x86_64]# ruby -vruby 1.8.5 (2006-08-25) [x86_64-linux]

下载所有puppet软件

-rw-r--r--  1 root   root       160028 06-12 16:01 facter-2.0.0rc4.tar.gz-rw-r--r--  1 root   root        10629 06-12 16:01 mcollective-2.0.0-1.el5.noarch.rpm-rw-r--r--  1 root   root      1898410 06-12 16:01 puppet-2.7.14.tar.gz-rw-r--r--  1 root   root      4921341 06-12 16:01 puppet-dashboard-1.2.8.tar.gz

tar -zxvf  facter-2.0.0rc4.tar.gz

cd facter-2.0.0rc4

ruby install.rb

========

tar -zxvf  puppet-2.7.14.tar.gz

cd puppet-2.7.14

ruby install.rb

启动服务端 

如果有防火墙请打开端口

启动客户端

在服务端

[root@testdb1 ~]# puppet cert -la  testdb2.test.com (50:D4:15:5C:26:13:2E:61:00:07:D9:88:9D:75:D1:AF)+ testdb1.test.com (64:D1:AF:50:CF:1D:8C:FA:F0:14:48:35:84:0B:60:D2) (alt names: DNS:puppet, DNS:puppet.test.com, DNS:testdb1.test.com)

[root@testdb1 ~]# puppet cert sign testdb2.test.comnotice: Signed certificate request for testdb2.test.comnotice: Removing file Puppet::SSL::CertificateRequest testdb2.test.com at '/etc/puppet/ssl/ca/requests/testdb2.test.com.pem'

[root@testdb2 ~]# puppet agent --server testdb1.test.com --testinfo: Caching certificate_revocation_list for cainfo: Caching catalog for testdb2.test.cominfo: Applying configuration version '1339569016'notice: /Stage[main]//Node[default]/File[/tmp/temp1.txt]/ensure: defined content as '{md5}19ee62e0c6b5f00aaf9b02280c0dad66'info: Creating state file /var/lib/puppet/state/state.yamlnotice: Finished catalog run in 0.05 seconds

[http://puppet.wikidot.com/](http://puppet.wikidot.com/) 

[http://projects.puppetlabs.com/projects/puppet/wiki/Simplest_Puppet_Install_Pattern](http://projects.puppetlabs.com/projects/puppet/wiki/Simplest_Puppet_Install_Pattern)
