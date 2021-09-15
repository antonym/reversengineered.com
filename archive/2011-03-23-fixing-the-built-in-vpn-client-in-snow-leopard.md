---
title: Fixing the Built-In VPN Client in Snow Leopard
author: antonym
type: post
date: 2011-03-23T09:48:09+00:00
url: /2011/03/23/fixing-the-built-in-vpn-client-in-snow-leopard/
categories:
  - apple
  - mac

---
Occasionally I'll receive this error when trying to connect to VPN when using Mac's built-in Cisco VPN client.

VPN Connection  
A configuration error occurred. Verify your settings and try reconnecting.

To fix:

<pre lang="text" line="1">reaction:~ user$ ps -ef | grep racoon
    0   265     1   0   0:00.22 ??         0:00.34 /usr/sbin/racoon
  501   339   335   0   0:00.00 ttys001    0:00.00 grep racoon
reaction:~ user$ sudo kill -9 265
Password:
reaction:~ user$ ps -ef | grep racoon
  501   345   335   0   0:00.00 ttys001    0:00.00 grep racoon
reaction:~ user$ sudo /usr/sbin/racoon
reaction:~ user$ ps -ef | grep racoon
    0   347     1   0   0:00.00 ??         0:00.01 /usr/sbin/racoon -x
  501   349   335   0   0:00.00 ttys001    0:00.00 grep racoon
</pre>

Then try and reconnect to VPN. That'll save you from having to reboot to fix your VPN.