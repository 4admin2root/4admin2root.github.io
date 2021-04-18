---
layout:     post
title:      转：FixingApache(13)Permissiondenied:accessto403Forbidden
date:       2014-06-19
tags: [cnblogs]
---
http://www.petefreitag.com/item/793.cfm

Every so often I run into a **403 Forbidden** response when I'm setting up something in Apache, checking the log files will yield something like:

```
(13)Permission denied: access to /

```

There are a few things that could be the problem:

**Make sure it's not denied by Apache**

Most apache Configurations have something like this in there:

```
<Directory />
    Order deny,allow
    Deny from all
</Directory>

```

The above will block access to all files. You should also see something like this:

```
<Directory /path/to/webroot>
    Order allow,deny
    Allow from all
</Directory>

```

**Make sure Apache has Read, Execute Permissions**

The next thing to check is that Apache has read and execute permission (rx) on directories and read permission on files. You can run `chmod 750 /dir` (to give `-rwxr-x---` permission) or `chmod 755 /dir` (to give `-rwxr-xr-x` permission), etc.

**Make sure that the Directory Above has Execute Permission**

This is the one that tends to get me. Suppose you are creating an Alias like this:

```
Alias /foo /tmp/bar/foo

```

**If Running Security Enhanced Linux (SELinux)**

Another possibility for this error is that you are running SELinux (Security Enhanced Linux), inwhich case you need to use `chcon` to apply the proper security context to the directory. One easy way to do this is to copy from a directory that does work for example /var/www/

```
chcon -R --reference=/var/www /path/to/webroot
```

[Tweet](http://twitter.com/share)

### Related Entries

- [Apache Security Patches on CentOS / RHEL](http://www.petefreitag.com/item/826.cfm) - November 22, 2013
- [20 ways to Secure your Apache Configuration](http://www.petefreitag.com/item/505.cfm) - December 6, 2005
- [CheatSheet for Apache](http://www.petefreitag.com/item/480.cfm) - October 7, 2005



<ins><ins id="aswift_0_anchor"><iframe id="aswift_0" name="aswift_0" frameborder="0" marginwidth="0" marginheight="0" scrolling="no" width="728" height="90"></iframe></ins></ins>

### <a name="trackbacks"></a>Trackbacks

### <a name="comments"></a>Comments


<img class="gravitar" src="http://www.gravatar.com/avatar.php?gravatar_id=7ef7df6ca41eb93be291965329810bc3&rating=PG&size=30&default=http%3A%2F%2Fwww%2Epetefreitag%2Ecom%2Fimages%2Fdefault%5Fgravitar%2Ejpg" alt="" align="left" />

chcon -t httpd_sys_content_t

in all files

http://stackoverflow.com/questions/8816836/apache-403-error-13permission-denied-access-to-denied-fedora-16
