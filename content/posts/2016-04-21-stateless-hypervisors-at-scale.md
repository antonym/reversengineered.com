+++
title = "Stateless Hypervisors at Scale"
author = "antonym"
date = "2016-04-21"
description = "Stateless Hypervisors at Scale"
tags = [
    "code",
    "debian",
    "ipxe",
    "linux",
    "live",
    "nova",
    "openstack",
    "systemd"
]
+++

Running a public cloud that provisions infrastructure has many challenges, especially when you start getting to very large scale. Today I'm going to touch on the hypervisor piece, the main part of a public cloud that contains the customers data running in their instances.

Hypervisors typically run on bare metal, have some sort of operating system, host configuration, the customer's instance settings and then if using local storage, the virtual disks.

Traditionally, an operating system is installed and configuration management like Puppet, Chef, or Ansible is ran to bring the host machine up to deployed specification and then added to automation that ultimately provisions instances for a public cloud.

Over time, new features get implemented, bugs are fixed, and operational knowledge is gained so your deployed infrastructure will evolve over time. As your infrastructure grows, your older legacy style infrastructure can start looking a lot different and start becoming out of sync and inconsistent.

To break it down into a few points:

  * Hypervisors become inconsistent over time by ongoing maintenance, code releases, and manual troubleshooting by operations.
  * Optimizations, patches, and security fixes are pushed with newer builds but older builds in production never get caught up.
  * Critical kernel or hypervisor updates that require reboots are hard to do because of the uptime requirements of a public cloud.

So what if we got rid of the traditional methods of OS installation and configuration management and instead created a snapshot of your server build once and then deployed that to thousands of servers?

### "We’ll Do It Live"

![][1] 

If you’ve ever installed Ubuntu, typically you’ll use what’s called a Live CD to install the OS. The CD loads an OS into RAM and brings up the GUI so that you can then run the install from there. Many distributions over the years have used Live CDs for installation, rescue, or to serve as a tool for recovering from data loss.

The same concept can be applied to a hypervisor or a server running a work load. If you think about it, the hypervisors typically have one purpose, to run instances virtually for a user. Why have thousands of independent installs?

### Creating a Live Image

The process I've been using to create live images is relatively simple. I've detailed some very high level basics and will deep dive into each one of these at a later date:

  * Create an initial minimal chroot of the filesystem
  * Using Ansible, run configuration management one time within the chroot. This includes all additional packages needed, any customizations, and other additional things you'd normally do in your configuration run.
  * Install tools to allow for Live Booting to work
  * CentOS/Debian/Fedora/OpenSUSE/Ubuntu - dracut
  * Regenerate the initrd to inject the live boot tools into the initrd
  * Copy the kernel and initrd out
  * Create an image file and sync the filesystem into the image file.

From there you now the entire build of your OS represented by three files that can be used to boot the operating system over the network, from Grub, or via kexec.

### To persist or not persist?

Now at this point you essentially will have an image that can boot into RAM using iPXE, Grub, or even kexec which is fully stateless. But what if you want to actually make the data persist? With a few scripts added to the boot time, you can very easily separate the actual operating system and applications which will need updating over time from the user's data which will need to persist and be constant.

The scripts create symlinks from the filesystem in RAM to local storage on the server so that when the application tries to write to a directory, it gets redirected to a persistent storage on the local disk. The scripts to build the symlinks are part of the image so they are recreated every time the server boots the image.

In the example of an Openstack Nova Compute running Libvirt+KVM booting as a LiveOS, I have just a few locations on the filesystem that symlink to /data which is mounted on local storage on /dev/sda2:

  * /etc/libvirt - libvirt configurations
  * /etc/nova - Openstack Nova configuration
  * /etc/openvswitch - openvswitch settings and config
  * /etc/systemd/network - systemd networking configs
  * /var/lib/libvirt/ - libvirt files
  * /var/lib/nova/ - instance location
  * /var/lib/openvswitch/ - openvswitch database

Those locations and files within them make up the unique part of each hypervisor and keep them separate from the rest of the overall OS which will need to go through constant upgrading or changes.

### Squashible

I've been working on making some of the bits we've been working on available to the public. It's a project called Squashible. The name came from mashing SquashFS with Ansible. We switched away from using SquashFS for the time being but the name stuck for now until I can come up with a better name.

You can play around with it [here][2]. It's a constant work in progress so please use at your own risk. It currently runs through various roles to create an image with the minimal set of packages you need to run a hypervisor of a certain type. Many thanks to [Major Hayden][3] for working with me side by side on a lot of this project over the past year.

### Openstack

A video to my presentation and slides are below for [Openstack Austin 2016 - Stateless Hypervisors at Scale][4].



  


### Feedback

Comments, concerns, ideas? Let me know!

 [1]: https://media.giphy.com/media/l3V0oV8yskCLoocms/giphy.gif
 [2]: https://github.com/squashible/squashible
 [3]: https://major.io/
 [4]: https://www.openstack.org/summit/austin-2016/summit-schedule/events/8500