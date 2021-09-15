---
title: XenServer Auto Patcher
author: antonym
type: post
date: 2013-01-20T16:28:25+00:00
url: /2013/01/20/xenserver-auto-patcher/
dsq_thread_id:
  - 3762205331
categories:
  - code
  - linux
  - xenserver

---
I put together a little script that might come in handy to get Citrix XenServer fully up to date after doing a factory install. You can find it here:

[https://github.com/amesserl/xs_patcher][1]

It will detect the version of XenServer you are running and install all of the latest Citrix XenServer hotfixes that are available in sequential order. It will also detect any previous patches and install anything that might not be present. If you don not have the hotfixes on the machine, it will retrieve them for you. After running the script, all you will need to do is reboot so it will pick up the latest kernel.

To install it automatically during an install, you will need to put the patcher script on the disk with the cache prepropulated with all of the patches to avoid the script retrieving them each time. It's usually best to put this in place during the post install. You won't want to run it during the post install because XAPI isn't up and running at that point which the hotfixes require. You'll want to install a script into /etc/firstboot.d with a starting number higher than all the other processes that run during firstboot. Once the initial firstboot has run which sets up XenServer and all of it's storage repositories, you can then kick off the xs_patcher.sh script which will install all of the needed hotfixes. I usually then have one more call to reboot occur after that.

I'll try and maintain the script going forward as new hotfixes are released by Citrix. Currently it supports Boston, Sanibel, and Tampa. I'll probably go back and grab earlier versions as well in the future as I have time.

 [1]: https://github.com/amesserl/xs_patcher "xs_patcher"