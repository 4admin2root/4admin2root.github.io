---
layout:     post
title:      ruby用watir登录CU的代码
date:       2012-08-11
tags: [cnblogs]
---
require 'rubygems'require 'watir'url_logout="[http://bbs.chinaunix.net/member.php?mod=logging&action=logout](http://bbs.chinaunix.net/member.php?mod=logging&action=logout)"url_login="[http://bbs.chinaunix.net/member.php?mod=logging&action=login&logsubmit=yes](http://bbs.chinaunix.net/member.php?mod=logging&action=login&logsubmit=yes)"ie = Watir::IE.newie.goto(url_logout)ie.goto(url_login)

ie.text_field(:name,"username").set("test")ie.text_field(:name,"password").set("pass")ie.button(:name,"loginsubmit").click

url="[http://bbs.chinaunix.net/forum-141-1.html](http://bbs.chinaunix.net/forum-141-1.html)"ie.goto(url)

感觉不如python 的 cookielib 、urllib 和 urllib2 的方式好
