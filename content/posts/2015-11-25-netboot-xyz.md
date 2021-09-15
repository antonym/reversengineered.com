+++
title = "netboot.xyz"
author = "antonym"
date = "2015-11-25"
description = "netboot.xyz"
tags = [
    "code",
    "linux",
    "ipxe",
    "live",
    "netboot",
    "netboot.xyz",
    "windows"
]
+++

My newest project on the side is [netboot.xyz][1]. If you've seen [boot.rackspace.com][2], this should look pretty familiar to you. I ran across cheap .xyz domains from [Namecheap][3] (one dollar at the time!), and figured the netboot.xyz name space was much easier to remember and was more neutral to the goal I was trying to accomplish. I forked boot.rackspace.com (still doing basic maintenance) and am now focusing my efforts on netboot.xyz.

My goal with the project is to make it easy as possible to boot many of the popular operating systems from bare metal, virtual machines, etc without the hassle of hunting down the latest ISO for the OS you want to download. Ideally it's usable with any service provider or just someone who maintains their own servers.

I usually try and use operating systems that make their boot loaders available via mirrors, although there are occasionally some exceptions. I'm also experimenting with various new builds like WinPE, Live Booted OS, and I'd like to even pursue getting some hypervisors on there as well to make it as easy as possible to install everything.

It's also a great place to just let people play around with new operating systems with just a menu and learn about the many many distributions out there.

Check it out when you get a chance and drop me some feedback or make a pull request if you see something I'm missing. I've added a really easy way to test your pull request from the utility menu, all you need to do is enter in your github username and branch or hash of the commit you want to test.

I'm still working on a bunch of [documents][4] for demonstrating how easy it is to plug the 1MB [iPXE][5] ISO into things like VMware Fusion, Virtual Box, Openstack, so bear with me while I try and get all of those available.

Enjoy!

 [1]: http://netboot.xyz
 [2]: http://boot.rackspace.com
 [3]: https://www.namecheap.com/?aff=92617
 [4]: http://netbootxyz.readthedocs.org/en/latest/
 [5]: http://ipxe.org