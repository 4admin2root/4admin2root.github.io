---
layout:     post
title:      转：sshProxyCommand
date:       2015-10-21
tags: [cnblogs]
---
http://benno.id.au/blog/2006/06/08/ssh_proxy_command

# ssh ProxyCommand

The ssh ProxyCommand option is just really insanely useful. The reason I want to use it is that it makes it easy to tunnel ssh through a firewall. So for example you have a machine on your corporate network that is is sitting behind a firewall, but you have a login to the firewall. Now of course you could just do:

```
laptop$ ssh gateway
gateway$ ssh internal
internal$

```

or something like:

```
laptop$ ssh -t gateway ssh internal
internal$

```

```
Host internal
        ProxyCommand ssh gw nc -w 1 internal 22

```

For this to work you will need netcat (nc) installed on the gw, but that generally isn't too much of a problem. Now you can transparently ssh, scp or whatever to internal, and ssh deals with it.

```
ssh -f -nNT -R 1100:localhost:22 somehost

```

Will setup a remote tunnel, which basically means if you connect to port 1100, it will be forwarded to port 22 on the machine on which you ran `ssh -R`. Unfortunately unless you have full control over the host you are creating the reverse tunnel too, you will find that port 1100, will only be bound to localhost, which means you will probably still need to use the trick mentioned above to get seemless access to it.

It is also helpful to know that you can avoid netcat all together by using the -W flag of ssh. ProxyCommand ssh gw -W internal:22

<li id="post-830258547" class="post">
 

 
[<img src="http://a.disquscdn.com/uploads/users/444/2113/avatar92.jpg?1372633369" alt="Avatar" data-role="user-avatar" data-user="4442113" />](https://disqus.com/by/benno37/)
[benno37](https://disqus.com/by/benno37/) Mod [ Yarg](http://benno.id.au/blog/2006/06/08/ssh_proxy_command#comment-830228431)  [3 years ago](http://benno.id.au/blog/2006/06/08/ssh_proxy_command#comment-830258547)




That is a neat option. As far as I can tell it is relatively recent. (It doesn't seem to be available in OpenSSH 5.2). Do you know when it was added? I couldn't find anything useful in the OpenSSH changelogs, but I'm probably not patient enough to look through carefully.





 

<ul class="children" data-role="children">
<li id="post-834933928" class="post">
 

 
[<img src="http://a.disquscdn.com/uploads/users/4479/7785/avatar92.jpg?1422573346" alt="Avatar" data-role="user-avatar" data-user="44797785" />](https://disqus.com/by/disqus_ScgeKaTDpN/)
[Yarg](https://disqus.com/by/disqus_ScgeKaTDpN/) [ benno37](http://benno.id.au/blog/2006/06/08/ssh_proxy_command#comment-830258547)  [3 years ago](http://benno.id.au/blog/2006/06/08/ssh_proxy_command#comment-834933928)




It looks like 5.4:
* Added a 'netcat mode' to ssh(1): "ssh -W host:port ..." This connectsstdio on the client to a single port forward on the server. Thisallows, for example, using ssh as a ProxyCommand to route connectionsvia intermediate servers. bz#1618






</li>
