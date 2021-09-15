+++
title = "Configuring bonding with systemd"
author = "antonym"
date = "2014-08-21"
description = "Configuring bonding with systemd"
tags = [
    "linux",
    "networking",
    "systemd"
]
+++

With release of [systemd][1] version [216][2], a number of bugs were corrected and configurations options added related to bonding support in systemd-networkd.

Here's a basic example on setting up bonding in systemd:

**/etc/systemd/network/bond0.network**:

    [Match]
    Name=enp1*
    
    [Network]
    Bond=bond0
    

_You can also specify the interfaces like enp1s0f[01] if you need to be more specific._

**/etc/systemd/network/bond0.netdev:**

    [NetDev]
    Name=bond0
    Kind=bond
    
    [Bond]
    Mode=802.3ad
    LACPTransmitRate=fast
    MIIMonitorSec=1s
    UpDelaySec=2s
    DownDelaySec=8s
    

**/etc/systemd/network/Management.network:**

    [Match]
    Name=bond0
    
    [Network]
    Address=192.168.20.20/24
    Gateway=192.168.20.1
    DNS=8.8.8.8
    

**Enable and start systemd-networkd:**

    systemctl enable systemd-networkd
    systemctl start systemd-networkd
    

For more options, make sure to check out the systemd.network and systemd.netdev man pages.

 [1]: http://www.freedesktop.org/wiki/Software/systemd/
 [2]: http://lists.freedesktop.org/archives/systemd-devel/2014-August/022295.html