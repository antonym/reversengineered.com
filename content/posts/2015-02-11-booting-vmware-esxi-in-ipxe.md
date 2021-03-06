+++
title = "Booting VMware ESXi in iPXE"
author = "antonym"
date = "2015-02-11"
description = "Booting VMware ESXi in iPXE"
tags = [
    "esx",
    "ipxe",
    "openstack",
    "vmware"
]
+++

This is a quick method of setting up VMware ESXi installers to run in iPXE. This particular version works on the 5.5 Update 2 ISO. You'll need to drop the ISO contents in a directory and then these other files in either the same directory or another to make it cleaner:

iPXE Code:

    :esx55u2
    kernel http://httpserver/configs/vmware/esx55u2/mboot.c32 -c http://httpserver/configs/vmware/esx55u2/boot.cfg
    boot
    

boot.cfg contents:

    bootstate=0
    title=Loading ESXi installer
    prefix=http://httpserver/configs/vmware/esx55u2/bits
    kernel=tboot.b00
    kernelopt=runweasel ks=http://httpserver/configs/vmware/esx55u2/ks.cfg
    modules=b.b00 --- jumpstrt.gz --- useropts.gz --- k.b00 --- chardevs.b00 --- a.b00 --- user.b00 --- sb.v00 --- s.v00 --- ata_pata.v00 --- ata_pata.v01 --- ata_pata.v02 --- ata_pata.v03 --- ata_pata.v04 --- ata_pata.v05 --- ata_pata.v06 --- ata_pata.v07 --- block_cc.v00 --- ehci_ehc.v00 --- elxnet.v00 --- weaselin.t00 --- esx_dvfi.v00 --- xlibs.v00 --- ima_qla4.v00 --- ipmi_ipm.v00 --- ipmi_ipm.v01 --- ipmi_ipm.v02 --- lpfc.v00 --- lsi_mr3.v00 --- lsi_msgp.v00 --- misc_cni.v00 --- misc_dri.v00 --- mtip32xx.v00 --- net_be2n.v00 --- net_bnx2.v00 --- net_bnx2.v01 --- net_cnic.v00 --- net_e100.v00 --- net_e100.v01 --- net_enic.v00 --- net_forc.v00 --- net_igb.v00 --- net_ixgb.v00 --- net_mlx4.v00 --- net_mlx4.v01 --- net_nx_n.v00 --- net_tg3.v00 --- net_vmxn.v00 --- ohci_usb.v00 --- qlnative.v00 --- rste.v00 --- sata_ahc.v00 --- sata_ata.v00 --- sata_sat.v00 --- sata_sat.v01 --- sata_sat.v02 --- sata_sat.v03 --- sata_sat.v04 --- scsi_aac.v00 --- scsi_adp.v00 --- scsi_aic.v00 --- scsi_bnx.v00 --- scsi_bnx.v01 --- scsi_fni.v00 --- scsi_hps.v00 --- scsi_ips.v00 --- scsi_lpf.v00 --- scsi_meg.v00 --- scsi_meg.v01 --- scsi_meg.v02 --- scsi_mpt.v00 --- scsi_mpt.v01 --- scsi_mpt.v02 --- scsi_qla.v00 --- scsi_qla.v01 --- uhci_usb.v00 --- tools.t00 --- xorg.v00 --- imgdb.tgz --- imgpayld.tgz
    build=
    updated=0
    

On future versions, you have to make sure the boot.cfg in the ISO modules line up to your custom boot.cfg.

And a quick example ks.cfg for some automation:

    # Sample scripted installation file
    # Accept the VMware End User License Agreement
    vmaccepteula
    # Set the root password for the DCUI and ESXi Shell
    rootpw mypassword
    # Install on the first local disk available on machine
    install --firstdisk --overwritevmfs
    # Set the network to DHCP on the first network adapater, use the specified hostname and do not create a portgroup for the VMs
    network --bootproto=dhcp --device=vmnic0 --addvmportgroup=0
    # reboots the host after the scripted installation is completed
    reboot
    

**Update 11/11/2015**: You'll need to enable COMBOOT support within iPXE in order to properly boot ESXi. You can do this by creating a source/config/local/general.h in the iPXE source with the following contents:

    #define IMAGE_COMBOOT
    

Then you can recompile iPXE and use the new binaries to boot.