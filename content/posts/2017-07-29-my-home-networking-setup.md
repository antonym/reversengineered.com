+++
title = "My Home Networking Setup"
author = "antonym"
date = "2017-07-29"
description = "My Home Networking Setup"
tags = [
    "networking"
]
+++

I've had a few asks from various people about my networking setup at home so I thought I'd throw together a blog post detailing it.

I wanted something that was going to be stable, performant, and reliable so I bit the bullet and decided to go with equipment that typically isn't used in most households. No Linksys or Netgear consumer grade equipment here!

My current setup consists:

1 x [Ubiquiti EdgeRouter 4][1]  
1 x [Ubiquiti Edge Switch ES-24-250W][2]  
4 x [Ubiquiti Unifi 802.11ac Dual-Radio PRO Access Point (UAP-AC-PRO-US)][3]

My Edge Router 4 ([datasheet][4]) is connected to AT&T Fiber for a lightning fast 1GB up and down. Eventually, I may add a secondary failover provider to ensure the internet is always up and keep the family happy at all times. The router also runs Debian which lets you ssh into the router and use the CLI to configure or hack around on the router.

The 24 Port PoE Switch is a relatively cheap switch that provides PoE (Power over Ethernet). All of the AC-Pro Wifi access points are powered with PoE from the switch and distributed around the house for maximum signal. This is great because installation of devices just requires an ethernet line instead of the additional power. An added benefit is that all of the equipment can be battery backed up from one central location. My home is hard wired for ethernet so all wiring aggregates in one location to the switches. Wifi is usually nice to have for roaming devices, but for dedicated devices that usually require higher bandwidth, like gaming consoles, Apple TVs, and the TVs themselves, it's nice to have those hardwired into the network.

Overall I've been really happy with the setup and haven't really had to mess with it much other than the occasional firmware updates to keep the devices up to date.

 [1]: https://amzn.to/40ZANM1
 [2]: https://amzn.to/40yyzSg
 [3]: https://amzn.to/3Q6Oe6h
 [4]: https://dl.ubnt.com/datasheets/edgemax/EdgeRouter_ER-4_DS.pdf