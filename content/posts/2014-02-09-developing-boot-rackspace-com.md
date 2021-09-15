+++
title = "Developing boot.rackspace.com"
author = "antonym"
date = "2014-02-09"
description = "Developing boot.rackspace.com"
tags = [
    "code",
    "ipxe",
    "linux",
    "nova",
    "openstack",
    "xenserver"
]
+++

When I started down the path of building <a href="http://osimag.es/" target="_blank">osimag.es</a>, I started realizing that it could be really useful for others, especially in a cloud environment. Since my main focus has been working on Rackspace Cloud Servers for a number of years, I decided to see how feasible it would be to put together a menu driven installer for any Operating System working in a Infrastructure as a Service type of environment. I figured there's probably a number of power users who might not want to start out with the default images provided, but possibly would want the opportunity to create their own custom image from scratch.

**Will it even work?**

I started testing out the <a href="http://docs.openstack.org/grizzly/openstack-compute/install/apt/content/introduction-to-xen.html" target="_blank">XenServer boot from ISO</a> code in <a href="http://www.openstack.org/" target="_blank">Openstack</a> to see if someone might have already gotten that working for another use case. To my delight, the boot from ISO code worked out pretty well. I was able to upload the <a href="http://ipxe.org/" target="_blank">iPXE</a> 1MB iso into Glance and boot from that image type.

The next problem to solve was the fact that Rackspace Cloud Servers assigns static IP addresses and does not currently run a DHCP service to assign out the networking. iPXE usually works best when DHCP is used as the network stack gets set up automatically. Because of this, a customer launching a cloud server could boot the iPXE image but would have to specify the networking manually of the instance in order to chain load <a href="http://boot.rackspace.com" target="_blank">boot.rackspace.com</a>.

We started thinking about how to automate this, and with the help of a few developers came up with a <a href="https://review.openstack.org/#/c/38650/" target="_blank">solution</a>. The solution retrieves an iPXE image on boot, brings it down to the hypervisor, extracts the iPXE kernel, and regenerates the ISO with a new iPXE startup script that contains the networking information of the instance. Then when the instance is started, iPXE is able to get on the network and load up <a href="http://boot.rackspace.com" target="_blank">boot.rackspace.com</a> automatically. Once iPXE has those values, they can then be passed to kernel command line for distributions that support network options. This allows for the user to not have to worry about any networking input during installation.

**Hosting the Menu**

Because boot.rackspace.com is just a bunch of iPXE scripts, they are hosted on Cloud Files in a container. The domain is a CNAME to the containers URL and then hosted on the Akamai CDN. The source is deployed from Github to the Cloud Files container when new commits are checked in via a Jenkins job. This makes it very lightweight and very scalable to run. The next thing I'm probably going to look at is seeing if I can remove the Jenkins server completely and just run the deploy out of Github. I was also able to enable CDN logs within the container and I'm using a service called <a href="https://qloudstat.com/welcome" target="_blank">Qloudstat</a> to parse those logs and provide metrics on the usage of the scripts.

**Delete those old ISOs**  
Having a small 1MB image is really nice for those times when you need to deploy an OS onto a remote server, or just need to install something into Virtual Box or VMware. There's really no point in storing tons of ISOs on your machine if you can just stream the packages you need.

**What's Next?**  
I have a few ideas about some new features that I'd like to add. I'd like to add a menu of experimental items and I'd also like to have the ability to generate a new version of the menu from a pull request so that new changes can be quickly validated before being merged into the main code base. If you haven't tried out <a href="http://boot.rackspace.com" target="_blank">boot.rackspace.com</a> yet, I encourage you to check it out. You can get a quick overview from my <a href="http://developer.rackspace.com/blog/introducing-boot-dot-rackspace-dot-com.html" target="_blank">Rackspace blog post.</a>