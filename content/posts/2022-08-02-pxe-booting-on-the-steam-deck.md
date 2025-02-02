+++
title = "PXE Booting on the Steam Deck"
author = "antonym"
date = "2022-08-02"
description = "PXE Booting on the Steam Deck"
tags = [
    "pxe",
    "ipxe",
    "howto",
    "steamdeck",
    "netboot.xyz",
    "netbootxyz"
]
+++

The Valve [Steam Deck](https://amzn.to/4jzhCjn) is a really nice piece of hardware that supports Linux. When I first saw the specifications released for it, I hoped it would be a great device to tinker with so I put my pre-order in immediately. I received it in May and set it aside for a bit. One weekend I decided to start playing around to see what the hardware was capable of.

The first thing I wanted to try was to see if I could actually get the Steam Deck to PXE boot off the network. After a little trial and error, I found that I could. Here's how I did it.

## Requirements

To get the Steam Deck to PXE boot, you will need:

- [USB-C Hub](https://amzn.to/4hiGPNh) that supports Ethernet and USB
- USB Keyboard
- Hard Wired Ethernet

I had to pick up a newer USB-C Hub in order to get it functioning as some of my older hubs did not want to work properly. The one that worked for me was the [USB C Hub 4K 60Hz, CableCreation 7-in-1 USB-C Hub Multiport Adapter](https://amzn.to/4hiGPNh). The USB-C Hub has to be plugged in for the Ethernet to work properly. I'm hoping to switch to the Steam Deck dock once they are available.

Connect the hub, ethernet, and power up to the Steam Deck. The first thing you will want to do is set the BIOS to allow for PXE booting.

## BIOS Configuration

To bring up the Steam Deck Boot Loader menus, shutdown the Steam Deck and:

- Hold down `Volume +`, while pressing the power button `on` to access the Boot Manager, Setup Utility and Boot from File Menu. (`Volume -` will bring up just the Boot Manager)
- Select Setup Utility to enter into the Setup.
- Move down to the Boot Tab on the left and change these settings:
  - Quick Boot: Disabled
  - Quiet Boot: Disabled
  - PXE Boot Capability: UEFI: IPv4 (Can change to what is appropriate for your network)
  - Add Boot Options: First
- Select Exit and Exit Saving Changes.

## PXE Booting

The Steam Deck will now reboot and you will now see the Memory test as Quiet Boot has been disabled. If your Hub is connected to the network properly, and you have DHCP on the network, you should see:

```shell
>>Start PXE over IPv4...
```

At this point you should be able to PXE boot a UEFI image. The first thing I tested was to see if [netboot.xyz](https://netboot.xyz/) would boot from it.

I loaded up the [netboot.xyz UEFI kernel](https://boot.netboot.xyz/ipxe/netboot.xyz.efi), set my DHCP [next-server](https://netboot.xyz/docs/booting/tftp) to my TFTP server, and filename to the netboot.xyz UEFI image on my DHCP server.

I rebooted the Steam Deck and to my surprise, it loaded the netboot.xyz menu right up. From there I was able to install an OS to the SD Card, boot into Live OS images, and load up other various tools from the menu. 

![img][1] 

If you happen to break the Steam Deck when testing Operating Systems or tinkering with it, you can follow the Steam Deck Recovery Instructions [here](https://help.steampowered.com/en/faqs/view/1B71-EDF2-EB6D-2BB3). (**Disclaimer: I am not responsible for you breaking your Steam Deck, always have your data backed up!**)

If you want to set the BIOS back to the default settings, you can load the BIOS back up, select Restore Defaults, and Exit Saving Changes. That will return the Steam Deck back to its original behavior.

## What's Next

Because it will support [iPXE](https://ipxe.org/), I can see many different things that could be done on the hardware, including automating disk backups of the unit over network boot. There are lots of things to explore on the device over time. It's a great gaming system and I am very interested in seeing where Valve takes the platform especially since the platform is very open from the beginning.

Things to try next:

- Legacy support is mentioned in the BIOS, need to try that out to see if it works
- Test out USB Booting with netboot.xyz

Check back here later as I will update this post as I find out more about the Steam Deck's capabilities.

 [1]: /images/steamdeck-nbxyz.png