---
layout:     post
title:      tab被字面的意思迷惑了,DBA_TAB_PRIVS
date:       2012-08-11
tags: [cnblogs]
---
今天创建一个过程报编译错误

于是show error 找到了 错误的地方

是对一个包没有执行权限

于是想多个数据库对比下用户权限

结果看到DBA_TAB_PRIVS以为是表的权限，直接忽略了

悲催啊，

DBA_TAB_PRIVS<a id="sthref1904" name="sthref1904"></a>

`DBA_TAB_PRIVS` describes all object grants in the database.

<a id="sthref1905" name="sthref1905"></a>Related View

`USER_TAB_PRIVS` describes the object grants for which the current user is the object owner, grantor, or grantee.

<tr align="left" valign="top"><th id="r1c1-t131" style="text-align: left; vertical-align: bottom; font-weight: bold;" align="left" valign="bottom">Column</th><th id="r1c2-t131" style="text-align: left; vertical-align: bottom; font-weight: bold;" align="left" valign="bottom">Datatype</th><th id="r1c3-t131" style="text-align: left; vertical-align: bottom; font-weight: bold;" align="left" valign="bottom">NULL</th><th id="r1c4-t131" style="text-align: left; vertical-align: bottom; font-weight: bold;" align="left" valign="bottom">Description</th></tr>|------
