---
title: The Dreaded Flipping of NICs
author: antonym
type: post
date: 2010-12-26T07:26:43+00:00
url: /2010/12/26/the-dreaded-flipping-of-nics/
categories:
  - linux
  - networking
  - xenserver

---
I recently had a problem with NICs flipping around after removing all traces of MAC address rules from the server. I did this because I wanted the flexibility to be able to swap machines around at any point in time and not have to worry about tracking the MAC addresses on all of the devices. The gear was identical in specifications and after doing some research, I ran across a solution that has worked really well so far. It involves creating udev rules that don't contain any MAC addresses but that instead check the vendor id and bus location of the device. By knowing these items, you can guarantee you'll always have the correct ethernet device assigned to the correct physical network and you can make the rules a lot more generic in nature. As an example, first you'll want to identify the devices (example is from an HP ProLiant DL385):

<pre lang="text" line="1">lspci | grep -i eth
04:00.0 Ethernet controller: Broadcom Corporation NetXtreme II BCM5708 Gigabit Ethernet (rev 12)
42:00.0 Ethernet controller: Broadcom Corporation
NetXtreme II BCM5708 Gigabit Ethernet (rev 12)
</pre>

We'll take the first line for this example and break it down. The first group of numbers is the bus number (04), device number (00), and function  
number (0). From here we should be able to generate our udev rules file. Create /etc/udev/rules.d/70-persistent-net.rules and enter in the following or whatever your setup looks like:

<pre lang="text" line="1">SUBSYSTEM=="net",ACTION=="add",BUS=="pci",KERNEL=="eth*",ID=="0000:04:00.0",NAME="eth0"
SUBSYSTEM=="net",ACTION=="add",BUS=="pci",KERNEL=="eth*",ID=="0000:42:00.0",NAME="eth1"
</pre>

Once that's in place, you should be able to reboot and not have to worry about the NICs flipping around. If you're curious, you can also view more device information by looking at /sys:

<pre lang="text" line="1">ls -la /sys/bus/pci/devices/0000:04:00.0
</pre>

I've had success with this in Citrix XenServer (dom0 is based on CentOS) and Debian.