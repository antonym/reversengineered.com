+++
title = "Building and Booting Debian Live Over the Network"
author = "antonym"
date = "2014-05-17"
description = "Building and Booting Debian Live Over the Network"
tags = [
    "debian",
    "ipxe",
    "linux",
    "live",
    "netboot"
]
+++

If you've ever downloaded a [LiveCD][1], you know it's a self contained distribution that can run in memory and usually serves some sort of purpose. It can act as the front end for an installer, run a series of tools like [Kali Linux][2], or be used to access the internet [anonymously][3]. But lets face it, who uses CDs today? Today, I'm going to run through what [Debian Live][4] is, how to build it, and how you can potentially use it an appliance you can boot over the network with having to worry about local storage.

You'll typically want to run the [live-build][5] utility on Debian. Live-build has active development currently so there are newer builds in testing (Jessie) but they can be unstable. The version in stable (wheezy) is currently 3.0.5-1 whereas the one in testing is 4.0~alpha36-1. You might want to start out with the stable version first as the testing one is in active development.

**What is Debian Live**

Debian Live is a Debian operating system that does not require a classical installer to use it. It comes on various media, including CD-ROM, USB sticks, or via netboot. Because it's ephemeral, it configures itself every time on boot. There are several phases it goes through on boot before init is started:

  * live-boot handles the initial start up and retrieval of the squashfs image. This runs within the initramfs.
  * live-config which handles all the configuration of how the image will be booted, including the initial setup of networking. This is ran from within the squashfs image.
  * Custom hooks are ran near the end of live-config which allow you to manipulate the filesystem and take care of any custom setup.  You can also key off custom kernel command line key values pairs that you've put in place to do different actions.

Once those phases are completed init is started and the image will boot as normal into the appropriate runlevel.

**Building the Debian Live Image**

To install live-build, run:

    apt-get install live-build
    

This will install the lb binary used to generate the live image. Create and change into a directory that will hold your image.

    mkdir debian-live
    cd debian-live
    lb init
    lb config \
    --distribution wheezy \
    --architectures amd64 \
    --binary-images netboot \
    --debconf-frontend dialog \
    --chroot-filesystem squashfs \
    --parent-mirror-bootstrap http://mirrors.kernel.org/debian/ \
    --parent-mirror-chroot-security http://mirrors.kernel.org/debian-security/ \
    --parent-mirror-binary http://mirrors.kernel.org/debian/ \
    --parent-mirror-binary-security http://mirrors.kernel.org/debian-security/ \
    --mirror-bootstrap http://mirrors.kernel.org/debian/ \
    --mirror-chroot-security http://mirrors.kernel.org/debian-security/ \
    --mirror-binary http://mirrors.kernel.org/debian/ \
    --mirror-binary-security http://mirrors.kernel.org/debian-security/ \
    --archive-areas "main non-free contrib" \
    --apt-options "--force-yes --yes" \
    --bootappend-live "keyboard-layouts=en"
    

This will create a default directory structure to generate your live build. Most of those options are optional, but they will give you a good head start. With that base configuration, you should be able to run the following to start generating the build:

    lb build
    

This will take a while, but once completed you'll have generated some files that you can use for your netbooted image. The files you'll need are here:

    debian-live/tftpboot/live/vmlinuz
    debian-live/tftpboot/live/initrd.img
    debian-live/binary/live/filesystem.squashfs
    

Those three files are all you need to netboot a Debian Live image. The vmlinuz and initrd.img are called first and then the filesystem.squashfs is retrieved during initrd.img bootup.

**Regenerating a New Image**

If you need to regenerate the image again, you'll need to clean the previous build up first:

To reset the directory but leave the package cache:

    lb clean
    

To reset the directory and clear the cache:

    lb clean --cache
    

To clean everything and just leave the config directory:

    lb clean --all
    

**Setting up iPXE**

Using [iPXE][6] is probably the easiest way to load up the new image. With the files hosted via HTTP, you can set up your iPXE config like this, add it to your iPXE menu and then use it to load your new image.

    #!ipxe
    kernel http://mywebserver.com/live/vmlinuz
    module http://mywebserver.com/live/initrd.img
    imgargs vmlinuz boot=live config hooks=filesystem ${conkern} username=live noeject fetch=http://mywebserver.com/live/filesystem.squashfs
    boot
    

It will create a default user of live with the password of live.

In a future article, I'll talk about how you can further customize the live distribution. For now, make sure to check out the Debian Live <a href="http://live.debian.net/manual/" target="_blank">manual</a> that gives a great run down on customization.

 [1]: http://en.wikipedia.org/wiki/Live_CD
 [2]: http://www.kali.org
 [3]: https://tails.boum.org/index.en.html
 [4]: http://live.debian.net
 [5]: http://live.debian.net/devel/live-build/
 [6]: http://ipxe.org