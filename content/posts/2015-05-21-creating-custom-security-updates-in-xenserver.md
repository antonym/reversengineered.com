+++
title = "Creating Custom Security Updates In XenServer"
author = "antonym"
date = "2015-05-21"
description = "Creating Custom Security Updates In XenServer"
tags = [
    "code",
    "linux",
    "xenserver"
]
+++

Some of you may have heard about the latest vulnerability affecting QEMU codenamed [VENOM][1].

Sometimes security vulnerabilities are released faster than the vendor can qualify a valid hot fix. In this post, I'll walk you through how to generate your own XenServer hotfix in order to rapidly patch the issue.

## How XenServer Patching Works

The sources for XenServer are provided each release, usually in a binpkg.iso. Here's are some links for the latest version of XenServer 6.5:

[XenServer Primary Download Page][2]

[XenServer 6.5 Hypervisor][3]

[XenServer 6.5 Sources][4]

[XenServer 6.5 DDK][5]

### Creating your Own Custom Patch

The first thing you'll need to do is to download the DDK for the affected version. The DDKs are released for each version of XenServer and also released anytime the kernel revs within the major release. The DDK provides the same environment that the SRPMs were created under, so it makes it really easy to rebuild the RPMs. It comes packaged as an appliance, so you'll want to import that appliance into a build of XenServer and boot it up

### Determine What Needs to be Patched

If QEMU needs patching, more than likely it's the qemu-dm binary (/usr/lib64/xen/bin/qemu-dm). To determine which packages sources you need to retrieve, run a rpm query on that binary:

    [root@hostname /]# rpm -qf /usr/lib64/xen/bin/qemu-dm
    xen-device-model-1.9.0-199.7656
    

Now we know that we need to make the changes to the xen-device-model.

If we needed to patch Xen:

    [root@hostname boot]# rpm -qf xen-4.4.1-xs100346.gz
    xen-hypervisor-4.4.1-1.9.0.462.28802
    

And so on. Once you know what the package is, then we can go about finding the source rpm.

### Obtaining the Source RPM

Assuming the version of XenServer you're using is up to date on patches, you'll want to grab either the latest deployed patch to your environment, or grab the latest patch that contained the version you want to update. Each xsupdate contains updated RPMs, so you might need to run through all of the latest patches to find the right one.

Anytime a hotfix is released, the hotfix will include the sources that were changed as part of the update release. For example, within the zip of a hotfixed release, 6.5SP1 in this [case][6], you'll have two files:

  * the xsupdate that is used to apply to the server, XS65ESP1.xsupdate
  * the sources package, XS65ESP1-src-pkgs.tar.bz2

The sources package includes all of the SRPMs that were used to create the latest xsupdate.

### Extracting the Sources

We'll want to take the latest available sources, grab the Source RPM, and install it to the DDK server. We'll use the one out of this [hotfix][7] to simulate updating QEMU for VENOM:

    wget http://downloadns.citrix.com.edgesuite.net/10325/XS62ESP1021.zip
    unzip XS62ESP1021.zip
    bunzip2 XS62ESP1021-src-pkgs.tar.bz2
    tar xvf XS62ESP1021-src-pkgs.tar 
    

Create .rpmmacros so that the sources extract to a known location:

    # ~/.rpmmacros
    %packager %(echo "$USER")
    %_topdir %(echo "$HOME")/rpmbuild
    

Make directories:

    mkdir ~/rpmbuild ~/rpmbuild/SOURCES ~/rpmbuild/RPMS ~/rpmbuild/BUILD ~/rpmbuild/SRPMS ~/rpmbuild/SPECS 
    

Install the sources:

    rpm -i xen-device-model-1.8.0-105.7582.i686.src.rpm
    

Copy the patch file to ~/rpmbuild/SOURCES/:

    cp xsa133-qemut.patch ~/rpmbuild/SOURCES/
    

Update the SPEC file to include the new patch and bump the release from 105.7582 to 105.7582.1custom. We do this so we can prevent conflicts from future versions but still differentiate which version we're on:

    [root@localhost]# diff -u xen-device-model.spec xen-device-model.spec.mod
    --- xen-device-model.spec   2015-03-17 12:02:05.000000000 -0400
    +++ xen-device-model.spec.mod   2015-05-12 19:35:53.000000000 -0400
    @@ -1,11 +1,12 @@ 
     Summary: qemu-dm device model
     Name: xen-device-model
     Version: 1.8.0
    -Release: 105.7582
    +Release: 105.7582.1custom
     License: GPL
     Group: System/Hypervisor 
     Source0: xen-device-model-%{version}.tar.bz2
     Patch0: xen-device-model-development.patch
    +Patch1: xsa133-qemut.patch
     BuildRoot: %{_tmppath}/%{name}-%{version}-%{release}-buildroot
     BuildRequires: SDL-devel, zlib-devel, xen-devel, ncurses-devel, pciutils-devel
    @@ -14,6 +15,7 @@
     %prep
     %setup -q
     %patch0 -p1
    +%patch1 -p1
     %build
     ./xen-setup --disable-opengl --disable-vnc-tls --disable-blobs
    @@ -37,6 +39,9 @@
     %dir /var/xen/qemu
     %changelog
    +* Tue Mar 17 2015 MyPatch <www.mypatch.com> [1.8.0 105.7582.1custom]
    +- xsa133-qemu
    +
     * Tue Mar 17 2015 Citrix Systems, Inc. <www.citrix.com> [1.8.0 105.7582]
     - Build ioemu.
    

Regenerate the RPM from the sources, and watch for errors.

    rpmbuild -ba xen-device-model.spec
    

Make sure your patches apply cleanly and if they do, after the compile has completed, the fresh RPMs will be present in ~/RPMS:

    ls ~/rpmbuild/RPMS/i386/xen-device-model* 
    xen-device-model-1.8.0-105.7582.1custom.i386
    xen-device-model-debuginfo-1.8.0-105.7582.1custom.i386.rpm 
    

### Deploying the RPMs to XenServer

You'll want to take the new RPM and deploy it using:

    rpm -Uvh xen-device-model-1.8.0-105.7582.1custom.i386 
    

If you need to revert to the original version, you can run

    rpm --force -Uvh xen-device-model-1.8.0-105.7582.i386
    

Depending on the type of patching you're doing, you'll need to determine your reload strategy. If it's Xen or a kernel for instance, you'll know you'll have to reboot. If it QEMU, you know that you'll have to detach the disks and reload them so that they get the newly patched process.

 [1]: https://access.redhat.com/articles/1444903
 [2]: http://xenserver.org/overview-xenserver-open-source-virtualization/download.htm
 [3]: http://downloadns.citrix.com.edgesuite.net/akdlm/10175/XenServer-6.5.0-xenserver.org-install-cd.iso
 [4]: http://downloadns.citrix.com.edgesuite.net/akdlm/10182/XenServer-6.5.0-binpkg.iso
 [5]: http://downloadns.citrix.com.edgesuite.net/akdlm/10106/XenServer-6.5.0-DDK.iso
 [6]: http://support.citrix.com/article/CTX142355
 [7]: http://support.citrix.com/article/CTX142272