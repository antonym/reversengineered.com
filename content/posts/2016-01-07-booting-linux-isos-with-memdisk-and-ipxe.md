+++
title = "Booting Linux ISOs with Memdisk and iPXE"
author = "antonym"
date = "2016-01-07"
description = "Booting Linux ISOs with Memdisk and iPXE"
tags = [
    "code",
    "debian",
    "esx",
    "linux",
    "ipxe",
    "live",
    "netboot",
    "netboot.xyz",
    "vmware",
    "xenserver"
]
+++

There are a number of distributions out there that provide proper support for booting the distribution over the network. A lot of the more popular distributions usually provide a installer kernels that can be easily downloaded for use. You point at the vmlinuz and the initrd and can them immediately proceed with the install streaming down packages as needed. These distributions make it great for tools like [netboot.xyz][1] to install using iPXE.

There are some distributions out there that don't have this functionality and typically only produce the ISO without any repositories that provide installer kernels or the rootfs.

In those cases, occasionally you can use memdisk and iPXE to boot those ISOs but they don't always work. In doing some research, I ran across one of the major issues as to why.

### Syslinux - Memdisk

_The following was taken from [syslinux - memdisk][2]._

The majority of Linux based CD images will also fail to work with MEMDISK ISO emulation. Linux distributions require kernel and initrd files to be specified, as soon as these files are loaded the protected mode kernel driver(s) take control and the virtual CD will no longer be accessible. If any other files are required from the CD/DVD they will be missing, resulting in boot error(s). Linux distributions that only require kernel and initrd files function fully via ISO emulation, as no other data needs accessing from the virtual CD/DVD drive once they have been loaded. The boot loader has read all necessary files to memory by using INT 13h, before booting the kernel.

There is also another solution, which requires the phram and mtdblock kernel module and memdiskfind utility of the Syslinux package (utils/memdiskfind). memdiskfind will detect the MEMDISK mapped image and will print the start and length of the found MEMDISK mapped image in a format phram understands:

    modprobe phram phram=memdisk,$(memdiskfind)
    modprobe mtdblock
    

This will create a /dev/mtdblock0 device, which should be the .ISO image, and should be mountable.

If your image is bigger than 128MiB and you have a 32-bit OS, then you have to increase the maximum memory usage of vmalloc, by adding:

    vmalloc=<at_least_size_of_your_image_in_mib>Mi</at_least_size_of_your_image_in_mib>
    

_Example: vmalloc=256Mi to your kernel parameters._

memdiskfind can be compiled with the klibc instead of with the glibc C library to get a much smaller binary for use in the initramfs:

    cd ./syslinux-4.04/utils/
    make spotless
    make CC=klcc memdiskfind
    

### Implementations of phram and mtdblock

ArchLinux has implemented the above concept [here][3] and [here][4].

Debian Live used it [here][5].

It's also been implemented in Clonezilla and [GParted][6].

[Antergos Linux][7] based on Arch Linux works great with memdisk using the phram module.

### Conclusion

I think it would be great for more distributions to attempt to implement something like this so that iPXE tools can be used to load the ISOs instead of actually having to burn or look for the location of the latest ISO every time.

Some of the distributions I'd love to see network support or better memdisk support are:

[Linux Mint][8]  
[Manjaro][9]  
[Elementary][10]  
[Solus Project][11]

There are also many other new distributions being released all the time. I typically use [DistroWatch][12] to determine the most popular distributions to attempt to add to [netboot.xyz][1]. I'd love to get a lot of these added to make it really easy to install anything on the fly.

I'd also love to see some of the hypervisors out there crack open the ISOs, pull them outside of their paywalls, and host the bits on their servers so that it's much easier to immediately boot an install to test something out without having to jump through many hoops. I have working installs for [VMware ESX][13] and [Citrix XenServer][14] but I'd need to have them host the bits or allow permission to do so for a public facing installer menu.

 [1]: http://netboot.xyz
 [2]: http://www.syslinux.org/wiki/index.php/MEMDISK
 [3]: https://projects.archlinux.org/mkinitcpio.git/tree/install/memdisk
 [4]: https://projects.archlinux.org/mkinitcpio.git/tree/hooks/memdisk
 [5]: https://anonscm.debian.org/cgit/debian-live/live-boot.git/commit/?id=e08c082e758afa3341a9ebb6e00927d9873c7230
 [6]: http://gparted-forum.surf4.info/viewtopic.php?id=17263
 [7]: https://antergos.com/
 [8]: http://www.linuxmint.com/
 [9]: http://manjaro.org/
 [10]: http://elementary.io/
 [11]: https://solus-project.com/
 [12]: http://distrowatch.com/
 [13]: https://www.vmware.com/products/esxi-and-esx/overview
 [14]: http://xenserver.org/overview-xenserver-open-source-virtualization/download.html